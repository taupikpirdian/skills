---
name: implement-message-mapper-dictionary
description: Guidance for implementing message_mapper in the activation service using dictionary-based error groups. Use when adding new error mappings, new journey mappers, bilingual ID/EN mappings, or integrating mapper output into handler responses.
---

# Implement Message Mapper With Dictionary

Use the existing `message_mapper` pattern: group error codes in `message_mapper/dictionary`, map them to UI/API responses in `message_mapper/journey`, then select the journey through the enum in `message_mapper/implement`.

## Workflow

1. Identify the error source and journey
   - Identify the error code from a service, use case, repository, or external API.
   - Decide which journey should use the mapping, such as activation, registration manual, submit NIK FR, or a new journey.
   - Ensure the thrown error uses `custom_error` so `pkgErrors.GetError(err)` produces `Code`, `CodeValue`, `Message`, and `IsBusinessError`.

2. Add the error-code dictionary
   - Create or update a file in `message_mapper/dictionary`.
   - Use the `InternalErrorData` type.
   - Name variables after the business condition, not the response text.

```go
package dictionary

var (
	ExampleInvalidCustomer = InternalErrorData{"EXM001", "EXM002"}
	ExampleLimitReached    = InternalErrorData{"EXM003"}
)
```

3. Add the journey mapping
   - Create or update a file in `message_mapper/journey`.
   - Store `language string` in the journey struct.
   - Implement `MessageID`, `MessageEN`, and `Message`.
   - Use `dictionary.CheckCodeExists(dictionary.X, errData.Code)` to select the response.
   - Always populate `HttpCode`, `Language`, and `PkgError`.
   - Set `JourneyCode` from the journey-specific code, or from `errData.CodeValue` when the response must preserve the original code.
   - For business-error fallback, use `errData.Message` with HTTP 422.
   - For non-business fallback, use HTTP 500.

```go
type exampleJourney struct {
	language string
}

func (e exampleJourney) MessageID(errData custom_error.ErrorCustomizeDTO) iFace.MapperErrorData {
	switch {
	case dictionary.CheckCodeExists(dictionary.ExampleInvalidCustomer, errData.Code):
		return iFace.MapperErrorData{
			Title:       "Data Pelanggan Tidak Valid",
			Description: "Mohon cek kembali data pelanggan yang dimasukkan.",
			HttpCode:    http.StatusUnprocessableEntity,
			Language:    constant.LanguageID,
			JourneyCode: errData.CodeValue,
			PkgError:    errData,
		}
	default:
		if errData.IsBusinessError {
			return iFace.MapperErrorData{
				Title:       "Gagal Memproses Permintaan",
				Description: errData.Message,
				HttpCode:    http.StatusUnprocessableEntity,
				Language:    constant.LanguageID,
				PkgError:    errData,
			}
		}
		return iFace.MapperErrorData{
			Title:       "Terjadi Kesalahan",
			Description: "Maaf terjadi kesalahan pada sistem, Silakan coba lagi",
			HttpCode:    http.StatusInternalServerError,
			Language:    constant.LanguageID,
			PkgError:    errData,
		}
	}
}

func (e exampleJourney) MessageEN(errData custom_error.ErrorCustomizeDTO) iFace.MapperErrorData {
	switch {
	case dictionary.CheckCodeExists(dictionary.ExampleInvalidCustomer, errData.Code):
		return iFace.MapperErrorData{
			Title:       "Invalid Customer Data",
			Description: "Please check the customer data you entered.",
			HttpCode:    http.StatusUnprocessableEntity,
			Language:    constant.LanguageEN,
			JourneyCode: errData.CodeValue,
			PkgError:    errData,
		}
	default:
		if errData.IsBusinessError {
			return iFace.MapperErrorData{
				Title:       "Failed to Process Request",
				Description: errData.Message,
				HttpCode:    http.StatusUnprocessableEntity,
				Language:    constant.LanguageEN,
				PkgError:    errData,
			}
		}
		return iFace.MapperErrorData{
			Title:       "Something Went Wrong",
			Description: "Sorry, something went wrong with the system. Please try again.",
			HttpCode:    http.StatusInternalServerError,
			Language:    constant.LanguageEN,
			PkgError:    errData,
		}
	}
}

func (e exampleJourney) Message(errData custom_error.ErrorCustomizeDTO) iFace.MapperErrorData {
	switch e.language {
	case constant.LanguageEN:
		return e.MessageEN(errData)
	default:
		return e.MessageID(errData)
	}
}

func ExampleJourney(language string) iFace.ErrorMapper {
	return exampleJourney{language: language}
}
```

4. Register the journey in the implementation mapper
   - Add a new enum value in `message_mapper/implement/message_mapper_impl.go`.
   - Add a case in `GetError`.
   - Do not change existing enum values unless it is required for internal backward compatibility.

```go
const (
	UndefineMapperError MessageMapper = iota
	// existing mapper...
	ERROREXAMPLEJOURNEY
)

func (m MessageMapper) GetError(ctx context.Context, err error) iFace.MapperErrorData {
	// existing language and custom error handling...
	switch m {
	case ERROREXAMPLEJOURNEY:
		return journey.ExampleJourney(langCode).Message(customizeError)
	}
	return iFace.MapperErrorData{}
}
```

5. Use the mapper in the handler
   - Import `message_mapper/implement`.
   - When sending an error response, pass the mapper enum that matches the journey.

```go
h.ResponseError(
	res,
	ctx,
	err,
	"EXM",
	nil,
	messageMapper.ERROREXAMPLEJOURNEY,
)
```

## Validation Checklist

- The dictionary contains every code that must be handled.
- `MessageID` and `MessageEN` have equivalent mappings.
- `Language` matches `constant.LanguageID` or `constant.LanguageEN`.
- `HttpCode` matches the API contract, usually 422 for business errors and 500 for system errors.
- `PkgError` is always populated with `errData`.
- The handler uses the correct mapper enum.
- Add at least focused unit tests for new journeys or risky mappings.

## Minimal Test

Create table tests to ensure each error code maps to the correct response and fallback behavior still works.

```go
func TestExampleJourney_MessageID(t *testing.T) {
	errData := custom_error.ErrorCustomizeDTO{
		Code:            "EXM001",
		CodeValue:       "EXM001",
		IsBusinessError: true,
	}

	result := journey.ExampleJourney(constant.LanguageID).Message(errData)

	assert.Equal(t, "Data Pelanggan Tidak Valid", result.Title)
	assert.Equal(t, http.StatusUnprocessableEntity, result.HttpCode)
	assert.Equal(t, constant.LanguageID, result.Language)
}
```

## Notes

- `dictionary.NewErrorCodeDigicore` is used to append some dictionary values from runtime config. Use it only for error codes that must be configurable.
- If a common error is used across journeys, consider mapping it in `message_mapper/journey/common.go`.
- Avoid duplicate error codes with different meanings in the same journey because the `switch` stops at the first match.
