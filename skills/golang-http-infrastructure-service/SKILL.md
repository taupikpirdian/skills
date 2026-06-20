---

name: golang-http-infrastructure-service
description: Use this skill when building Go infrastructure/service layer for calling external HTTP APIs using standard net/http.
---

# Go HTTP Infrastructure Service Skill

Use this skill when building infrastructure/service code in Go for calling external HTTP APIs.

The implementation must use standard Go `net/http`, not Resty or other third-party HTTP clients.

If Speckit docs exist under `docs/speckit/<feature-name>/`, read `spec.md`, `plan.md`, and `tasks.md` first and use them as the source of truth for the generated infrastructure code. Ask only for missing values that are still marked `TBD` and cannot be safely inferred.

## Goal

Build a clean, safe, and testable infrastructure service for external API integration.

Generated code must:

* use `net/http`
* use `context.Context`
* inject `*http.Client`
* build vendor payload from application DTO
* send JSON request body
* set required headers
* read response body safely
* validate JSON response
* map HTTP and business errors properly
* avoid logging sensitive data
* support mock mode if needed
* include unit tests

## Generated Files

### 1. Application DTO

Path:

```text
application/dto/<context>.go
```

Contains application-level request and response DTOs.

Rules:

* contains business-level fields only
* does not contain vendor payload shape
* does not use `json` tags

Example:

```go
package dto

type ReqPrepaidRegistrationFR struct {
	Msisdn string
	Nik    string
	Image  string
}

type ResPrepaidRegistrationFR struct {
	ResponseCode    string
	ResponseMessage string
	TransactionID   string
	Channel         string
}
```

### 2. Outbound Interface

Recommended path:

```text
application/repository/<feature>.go
```

Defines interface consumed by application layer.

Example:

```go
type SubmitPrepaidRegistFRRepository interface {
	SubmitPrepaidRegistFR(ctx context.Context, req dto.ReqPrepaidRegistrationFR) (*dto.ResPrepaidRegistrationFR, error)
}
```

Rules:

* interface must return application response DTO
* do not return infrastructure/vendor response DTO
* if project uses port naming, path may be `application/port/outbound/<feature>.go`

### 3. Infrastructure DTO

Path:

```text
internal/infrastructure/service/<feature>/<feature>_dto.go
```

Contains vendor request and response structs.

Rules:

* uses exact vendor JSON shape
* uses `json` tags
* may contain nested structs
* vendor response must be mapped to application response DTO before returning

Example:

```go
type RequestPayload struct {
	Body bodyRequest `json:"body"`
}
```

### 4. HTTP Implementation

Path:

```text
internal/infrastructure/service/<feature>/<feature>_http.go
```

Responsibilities:

* build signature if needed
* build vendor payload from application DTO
* marshal request using `json.Marshal`
* create request using `http.NewRequestWithContext`
* set required headers
* call endpoint using injected `*http.Client`
* read response body using `io.LimitReader`
* unmarshal response JSON
* map network/client errors to infrastructure error
* map HTTP 5xx to infrastructure error
* map HTTP 4xx based on API contract
* map non-success business response code to business error
* return application response DTO

### 5. Factory and Concrete Service

Path:

```text
internal/infrastructure/service/<feature>/factory.go
```

Contains:

* constants
* `Config` struct
* `Service` struct
* constructor
* config validation
* default HTTP timeout
* mock flag if needed

### 6. Unit Test

Path:

```text
internal/infrastructure/service/<feature>/<feature>_test.go
```

Use `httptest.Server` for infrastructure HTTP tests.

Cover:

* factory validation
* mock mode behavior
* HTTP success
* HTTP 4xx
* HTTP 5xx
* non-success business response code
* invalid JSON response
* empty response code
* network/client error
* expected headers
* expected payload

Mockery-generated mock is used for application/usecase tests, not for infrastructure HTTP behavior tests.

## Required Inputs Before Generating Code

Ask only what cannot be safely inferred.

### 1. Service or integration name

Example:

```text
SubmitPrepaidRegistFR
```

Used for:

* method name
* interface name
* folder name
* file name
* test name

### 2. External API endpoint

Example:

```text
/siebel/v1.0/service/Prepaid/RegistrationV3
```

Base URL must come from service config.

### 3. HTTP method

Example:

```text
POST
```

Default:

```text
POST
```

### 4. Application request fields

Example:

```text
Request fields:
- msisdn string required
- nik string required
- image string required
```

Generate application request DTO automatically.

### 5. Application response fields

Example:

```text
Response fields:
- response_code string
- response_message string
- transaction_id string
- channel string
```

Generate application response DTO automatically.

### 6. Vendor request and response payload shape

Ask for the exact JSON contract from the external API.

This is used to generate infrastructure DTO with `json` tags.

### 7. Headers and signature requirement

Ask for required headers.

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

Default headers:

```text
Accept: application/json
Content-Type: application/json
```

If signature is required, ask for:

* signature function
* config values used by signature
* signature header name

### 8. Success response code and sensitive fields

Ask what upstream response code means success.

Default:

```text
0000
```

Ask for sensitive fields that must be masked or omitted from logs.

Default sensitive fields:

```text
nik
msisdn
image
document
file
base64
token
authorization
api_key
signature
secret
password
```

## Recommended Service Structure

Use dependency injection for HTTP client and config.

Generate only config fields needed by the integration. Do not generate unused fields.

Example:

```go
type Config struct {
	BaseURL    string
	AuthBasic  string
	APIKey     string
	Secret     string
	Channel    string
	Callback   string
	EnableMock bool
	Timeout    time.Duration
}

type Service struct {
	httpClient *http.Client
	config     Config
}
```

Constructor example:

```go
func NewService(httpClient *http.Client, config Config) (*Service, error) {
	if config.Timeout == 0 {
		config.Timeout = 30 * time.Second
	}

	if httpClient == nil {
		httpClient = &http.Client{
			Timeout: config.Timeout,
		}
	}

	if config.BaseURL == "" {
		return nil, errors.New("baseUrl is required")
	}

	if config.AuthBasic == "" {
		return nil, errors.New("authBasic is required")
	}

	if config.APIKey == "" {
		return nil, errors.New("apiKey is required")
	}

	if config.Secret == "" {
		return nil, errors.New("secret is required")
	}

	if config.Channel == "" {
		return nil, errors.New("channel is required")
	}

	config.BaseURL = strings.TrimRight(config.BaseURL, "/")

	return &Service{
		httpClient: httpClient,
		config:     config,
	}, nil
}
```

Config generation rules:

* if API does not require callback URL, do not generate `Callback`
* if API does not require signature, do not generate `Secret`
* if API does not require Basic Auth, do not generate `AuthBasic`
* if API does not require API key, do not generate `APIKey`
* if API only needs static headers, use `Headers map[string]string`

## Implementation Rules

### 1. Use `net/http`

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

### 3. Always set timeout

Default timeout:

```go
30 * time.Second
```

Avoid using `http.DefaultClient` directly for external integrations.

### 4. Do not log raw sensitive data

Do not log full request or response payload if it may contain sensitive data.

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

For file/base64 fields, log only size.

### 5. Generate transaction ID once

If transaction ID is needed, generate once and reuse it.

```go
trxID := helper.GenerateTrxId(req.Msisdn)

payload := s.buildPayload(req, trxID)
```

Do not generate transaction ID directly inside payload field assignment.

### 6. Build endpoint safely

Use trimmed base URL.

```go
endpoint := s.config.BaseURL + "/siebel/v1.0/service/Prepaid/RegistrationV3"
```

`BaseURL` must already be normalized in constructor:

```go
config.BaseURL = strings.TrimRight(config.BaseURL, "/")
```

### 7. Limit response body size

Always limit response body before reading.

```go
const maxResponseBodyBytes = 2 * 1024 * 1024

respBody, err := io.ReadAll(io.LimitReader(resp.Body, maxResponseBodyBytes))
```

### 8. Validate response body

If upstream is expected to return JSON, always unmarshal and validate it.

```go
var result responseSubmitPrepaidRegistFR

if len(respBody) > 0 {
	if err := json.Unmarshal(respBody, &result); err != nil {
		return nil, pkgErrors.NewInfrastructureError(moduleCode, "003", "Invalid response from upstream")
	}
}
```

### 9. Map HTTP errors

Default mapping:

| Scenario                           | Error Type                                                          |
| ---------------------------------- | ------------------------------------------------------------------- |
| network/client error               | infrastructure error                                                |
| timeout                            | infrastructure error                                                |
| invalid JSON response              | infrastructure error                                                |
| empty response code                | infrastructure error                                                |
| HTTP 5xx                           | infrastructure error                                                |
| HTTP 400/422                       | business error                                                      |
| HTTP 401/403                       | infrastructure/config/auth error unless API contract says otherwise |
| HTTP 404                           | depends on API contract                                             |
| HTTP 429                           | infrastructure or retryable error                                   |
| non-success business response code | business error                                                      |

Example:

```go
if resp.StatusCode >= 500 {
	return nil, pkgErrors.NewInfrastructureError(moduleCode, "002", result.ResponseMessage)
}

if resp.StatusCode >= 400 {
	return nil, pkgErrors.NewBusinessError(moduleCode, normalizeErrorCode(result.ResponseCode), normalizeErrorMessage(result.ResponseMessage, "Business error from upstream"))
}
```

### 10. Validate business response code

Default success code:

```go
const successResponseCode = "0000"
```

Validation:

```go
if result.ResponseCode == "" {
	return nil, pkgErrors.NewInfrastructureError(moduleCode, "003", "Empty response code from upstream")
}

if result.ResponseCode != successResponseCode {
	return nil, pkgErrors.NewBusinessError(moduleCode, result.ResponseCode, result.ResponseMessage)
}
```

### 11. Use constants for fixed values

Avoid scattered hardcoded values.

Example:

```go
const (
	moduleCode           = "SPRFR"
	successResponseCode  = "0000"
	maxResponseBodyBytes = 2 * 1024 * 1024
)
```

Add vendor-specific constants only when needed.

### 12. No automatic retry by default

Do not add automatic retry for submit, registration, payment, activation, or mutation APIs unless upstream supports idempotency.

Retry is allowed only if:

* same transaction ID is reused
* upstream guarantees idempotency
* retry only happens for network error or 5xx
* business errors are never retried

## Response Mapping Rule

Infrastructure response DTO must be mapped to application response DTO.

Example:

```go
func toApplicationResponse(resp responseSubmitPrepaidRegistFR) *dto.ResPrepaidRegistrationFR {
	return &dto.ResPrepaidRegistrationFR{
		ResponseCode:    resp.ResponseCode,
		ResponseMessage: resp.ResponseMessage,
		TransactionID:   resp.TransactionID,
		Channel:         resp.Channel,
	}
}
```

Do not return vendor response struct outside infrastructure package.

## Mock Mode Rule

If mock mode is needed, return application response DTO.

Example:

```go
if s.config.EnableMock {
	return &dto.ResPrepaidRegistrationFR{
		ResponseCode:    successResponseCode,
		ResponseMessage: "Success",
		TransactionID:   "mock-transaction-id",
		Channel:         s.config.Channel,
	}, nil
}
```

## Test Rules

Infrastructure HTTP tests must use `httptest.Server`.

Recommended test coverage:

* constructor validation
* mock mode
* HTTP success
* HTTP 4xx
* HTTP 5xx
* invalid JSON
* empty response code
* non-success business response code
* network/client error
* expected headers
* expected payload

Use mockery only for application/usecase tests that depend on the outbound interface.
