---
name: golang-http-infrastructure-service
description: Use this skill when building or creating Go infrastructure/service layer for calling external HTTP APIs using standard net/http.
---

# Go HTTP Infrastructure Service Skill

Use this skill when building or creating infrastructure/service code in Go for calling external HTTP APIs.

This skill uses the standard Go `net/http` package, not Resty or other third-party HTTP clients.

## Goal

Build a clean, safe, and maintainable infrastructure service for external API integration using `net/http`.

The generated code must:

* Use `net/http` as the HTTP client.
* Use `context.Context` for request cancellation and timeout propagation.
* Build request payload clearly.
* Send JSON request body.
* Set required headers.
* Read and validate response body safely.
* Map HTTP error responses properly.
* Map upstream business response codes properly.
* Avoid logging sensitive data.
* Support mock mode if needed.
* Be easy to unit test.

## Generated files
* Application DTO for usecase input and output.
- path: application/dto/<context>.go
- example:
  - ReqPrepaidRegistrationFR
  - ResPrepaidRegistrationFR
- contains business-level fields only.
- does not contain vendor payload shape.
- does not use json tags.

* Outbound interface file.
- path: `application/repository/<feature>.go`
- defines interface consumed by application layer.
- method shape: `Method(ctx context.Context, req dto.<RequestDTO>) (*<ResponseDTO>, error)`.

* Infrastructure DTO.
- path: internal/infrastructure/service/<feature>/<feature>_dto.go
- contains vendor request/response structs with exact JSON tags.
- contains nested RequestPayload, bodyRequest, ListOfPrepaidRegUnReg, PrepaidRegUnReg, and vendor response struct.
- infrastructure response struct must be mapped to application response DTO before returning.

* HTTP implementation.
- path: `internal/infrastructure/service/<feature>/<feature>_http.go`
- implements the outbound interface method.
- responsibilities:
  - build signature if needed.
  - build vendor payload from application DTO.
  - marshal request using json.Marshal.
  - create request using http.NewRequestWithContext.
  - set headers: Authorization, api_key, Accept, Content-Type, x-signature, channel.
  - call endpoint using injected *http.Client.
  - read response body using io.LimitReader.
  - unmarshal response JSON.
  - map network/client errors to infrastructure error.
  - map HTTP 5xx to infrastructure error.
  - map HTTP 4xx based on API contract.
  - map non-success business response code to business error.
  - return application-level response DTO.

* Factory and concrete service.
- path: internal/infrastructure/service/<feature>/factory.go
- contains:
  - constants used by service.
  - Service struct with *http.Client, base URL, credentials, channel, callback, mock flag.
  - FactoryService struct if project convention requires it.
  - Create() constructor.
  - Validate() for required config/client.
  - default HTTP timeout.

* Mockery-generated mock.
- add outbound interface path to mockery.yml.
- generated mock is used for application/usecase tests, not for infrastructure HTTP behavior tests.

* Unit test.
- path: internal/infrastructure/service/<feature>/<feature>_test.go
- should cover:
  - factory validation.
  - mock mode behavior.
  - HTTP success using httptest.Server.
  - HTTP 4xx error status.
  - HTTP 5xx error status.
  - successful HTTP with non-success response code.
  - successful HTTP with invalid JSON response.
  - successful HTTP with empty response code.
  - connection/client error.
  - expected headers are sent.
  - expected payload is sent.

However, the implementation must use `net/http`, not Resty.

## Required Inputs Before Generating Code

Before generating the infrastructure code, ask only the required information below.

Do not ask questions for values that can be inferred from project convention or generated automatically.

### 1. What is the service or integration name?

Example:

```text
SubmitPrepaidRegistFR
```

This name will be used to generate:

* method name
* interface name
* folder name
* file name
* test name

### 2. What external API endpoint will be called?

Example:

```text
/siebel/v1.0/service/Prepaid/RegistrationV3
```

The base URL must come from service config.

### 3. What HTTP method is used?

Example:

```text
POST
```

### 4. What application request fields are needed?

Example input:

```text
Request fields:
- msisdn string required
- nik string required
- image string required
```

### 5. What application response fields should be returned?

Example input:

```text
Response fields:
- response_code string
- response_message string
- transaction_id string
- channel string
```

### 6. What vendor request and response payload shape is required?

This is the exact JSON contract from the external API.

### 7. What headers are required?

Example:

```text
Headers:
- Authorization
- api_key
- Accept
- Content-Type
- x-signature
- channel
```

Default headers if not specified:

```text
Accept: application/json
Content-Type: application/json
```

### 8. Does the API require signature generation?

If signature is required, ask for:

* signature function
* config values used by signature
* header name for signature

Example:

```text
x-signature
```

### 9. What response code means success?

Example:

```text
0000
```

### 10. What fields are sensitive and must be masked or omitted from logs?

Example:

```text
Sensitive fields:
- NIK
- MSISDN
- image
- document
```

Default sensitive fields:

```text
token
authorization
api_key
signature
secret
password
```

## Recommended Service Struct

Use dependency injection for the HTTP client.

Adjust config fields based on integration needs. Do not generate unused fields.

Examples:
- If the API does not require callback URL, do not generate callback field.
- If the API does not require signature, do not generate secret field.
- If the API does not require Basic Auth, do not generate authBasic field.
- If the API only needs static headers, generate headers map instead of explicit auth fields.

```go
type Service struct {
	httpClient *http.Client

	baseUrl    string
	headers    map[string]string
	apiKey     string
	secret     string
	channel    string
	callback   string
	enableMock bool
}
```

The `httpClient` should be injected from constructor.

```go
func NewService(
	httpClient *http.Client,
	baseUrl string,
	authBasic string,
	apiKey string,
	secret string,
	channel string,
	callback string,
	enableMock bool,
) (*Service, error) {
	if httpClient == nil {
		httpClient = &http.Client{
			Timeout: 30 * time.Second,
		}
	}

	if baseUrl == "" {
		return nil, errors.New("baseUrl is required")
	}

	if authBasic == "" {
		return nil, errors.New("authBasic is required")
	}

	if apiKey == "" {
		return nil, errors.New("apiKey is required")
	}

	if secret == "" {
		return nil, errors.New("secret is required")
	}

	if channel == "" {
		return nil, errors.New("channel is required")
	}

	return &Service{
		httpClient: httpClient,
		baseUrl:    strings.TrimRight(baseUrl, "/"),
		authBasic:  authBasic,
		apiKey:     apiKey,
		secret:     secret,
		channel:    channel,
		callback:   callback,
		enableMock: enableMock,
	}, nil
}
```

## Implementation Rules

### 1. Use `net/http`

Do not use Resty unless the project standard explicitly uses Resty.

Use:

```go
http.NewRequestWithContext(...)
httpClient.Do(...)
io.ReadAll(...)
json.Marshal(...)
json.Unmarshal(...)
```

Do not use:

```go
resty.New()
client.R()
SetResult()
SetBody()
```

### 2. Always use context

Every request must use:

```go
http.NewRequestWithContext(ctx, method, url, body)
```

Do not use:

```go
http.NewRequest(...)
```

unless there is a very specific reason.

### 3. Always set timeout

The HTTP client must have timeout.

```go
httpClient := &http.Client{
	Timeout: 30 * time.Second,
}
```

Avoid using `http.DefaultClient` directly for external integrations.

### 4. Do not log raw sensitive data

Never log raw payload if it contains sensitive data.

Avoid:

```go
logmanager.LogInfoWithContext(ctx, fmt.Sprintf("request payload: %+v", req))
```

Use masked logging:

```go
logmanager.LogInfoWithContext(ctx, fmt.Sprintf(
	"SubmitPrepaidRegistFR request trx_id=%s msisdn=%s image_size=%d",
	trxID,
	helper.MaskMsisdn(req.Msisdn),
	len(req.Image),
))
```

### 5. Generate transaction ID once

If the request needs transaction ID, generate it once and store it in a variable.

Recommended:

```go
trxID := helper.GenerateTrxId(req.Msisdn)

payload := s.buildPayload(req, trxID)
```

Avoid:

```go
TransactionID: helper.GenerateTrxId(req.Msisdn)
```

because it makes tracing harder.

### 6. Build endpoint safely

Avoid direct URL concatenation without trimming.

Recommended:

```go
endpoint := strings.TrimRight(s.baseUrl, "/") + "/siebel/v1.0/service/Prepaid/RegistrationV3"
```

Avoid:

```go
endpoint := s.baseUrl + "/siebel/v1.0/service/Prepaid/RegistrationV3"
```

This prevents double slash when baseUrl already ends with `/`.

### 7. Validate response body

If the API is expected to return JSON, always validate JSON response.

```go
var result ResponseSubmitPrepaidRegistFR

if len(respBody) > 0 {
	if err := json.Unmarshal(respBody, &result); err != nil {
		return nil, pkgErrors.NewInfrastructureError("SPRFR", "003", "Invalid response from upstream")
	}
}
```

### 8. Map HTTP error properly

Use different handling for 4xx and 5xx.

Recommended:

```go
if resp.StatusCode >= 500 {
	return nil, pkgErrors.NewInfrastructureError("SPRFR", "002", result.ResponseMessage)
}

if resp.StatusCode >= 400 {
	return nil, pkgErrors.NewBusinessError("SPRFR", result.ResponseCode, result.ResponseMessage)
}
```

General rule:

| Status Code | Error Type                                                                    |
| ----------- | ----------------------------------------------------------------------------- |
| 400         | Business error                                                                |
| 401         | Infrastructure/config/auth error                                              |
| 403         | Infrastructure/config/auth error or business error, depending on API contract |
| 404         | Business error or infrastructure error, depending on endpoint contract        |
| 422         | Business error                                                                |
| 429         | Infrastructure error or retryable error                                       |
| 500         | Infrastructure error                                                          |
| 502         | Infrastructure error                                                          |
| 503         | Infrastructure error                                                          |
| 504         | Infrastructure error                                                          |

### 9. Validate upstream business response code

If upstream returns business response code, validate it after HTTP status check.

Example success code:

```go
const successResponseCode = "0000"
```

Then:

```go
if result.ResponseCode == "" {
	return nil, pkgErrors.NewInfrastructureError("SPRFR", "003", "Empty response code from upstream")
}

if result.ResponseCode != successResponseCode {
	return nil, pkgErrors.NewBusinessError("SPRFR", result.ResponseCode, result.ResponseMessage)
}
```

### 10. Use constants for fixed integration values

Avoid scattered hardcoded values.

Recommended:

```go
const (
	moduleCode          = "SPRFR"
	successResponseCode = "0000"
	defaultUserID       = "User01"
	defaultIntRef       = "PrepaidRegUnRegV3"
	defaultADN          = "4444"
)
```