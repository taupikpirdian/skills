---
name: root-cause-summary
description: Turn technical investigation logs, API responses, and findings into a concise Jira-ready summary that distinguishes symptoms, error propagation, and confirmed or suspected root causes. Use when asked to summarize an investigated issue or prepare a root-cause summary for Jira.
---

# Jira Technical Investigation Summary

## Purpose

Convert raw technical investigation data into a concise, structured, and Jira-friendly issue summary.

The input may contain:
- Application logs
- API request/response
- cURL commands
- Error messages
- Investigation notes
- Service-to-service flow
- Database findings
- Previous issue references
- Developer observations

The output should help developers, QA, SA, and other technical stakeholders quickly understand:
1. What happened
2. Where the failure happened
3. How the issue was traced
4. What the root cause is
5. What the final conclusion is

---

# Instructions

Analyze all provided information before generating the summary.

Do not simply summarize logs line by line.

Identify the relationship between logs, APIs, services, and errors to reconstruct the actual failure flow.

Prioritize information that helps explain the issue.

Remove unnecessary noise such as:
- Access tokens
- Authorization tokens
- API keys
- Full JWT values
- Cookies
- Unrelated headers
- Encrypted payloads
- Large response bodies that do not contribute to the investigation
- Duplicate logs

Never expose credentials or sensitive authentication values in the output.

Do not invent a root cause.

If the available evidence only indicates a suspected cause, explicitly use wording such as:

`Suspected Root Cause`

or

`Further investigation is required.`

---

# Output Format

### Summary

Briefly explain the issue.

Include:
- Feature/process affected
- Main error
- Relevant endpoint if available

Keep this section short and easy to understand.

Example:

`Login for Binding failed during OTP submission with error "Failed to bind in Orbit side".`

---

### Investigation

Explain the investigation chronologically.

Focus on the important chain of events.

For each important step, mention:
- Service
- Endpoint
- HTTP status or response code
- Relevant error

Example structure:

During:

`POST /v1/example`

Service A returned:

- HTTP Status: `422`
- Error: `Example error`

Further tracing showed that Service A called:

`POST /private/example`

which then called Service B.

Service B returned:

- Response Code: `0011`
- Response Message: `Example rejection`

Avoid dumping raw logs unless a specific field is important as evidence.

---

### Root Cause

State the deepest confirmed cause found during investigation.

Focus on the system/service that actually rejected or failed the process, rather than only the first service where the error appeared.

Example:

`The request was rejected by CRM because the MSISDN was not eligible for the binding process.`

If the root cause cannot be confirmed, write:

### Suspected Root Cause

and explain what the available evidence indicates.

---

### Flow

Create a compact technical flow showing how the error propagates between services.

Format:

`Client`
→ `Service A`
→ `Service B`
→ `Service C`
→ `External System`
→ `Error`

Use actual service or endpoint names when available.

Keep the flow short enough to scan quickly in Jira.

---

### Conclusion

Explain the final finding in simple technical language.

Clarify:
- What is NOT causing the issue, if confirmed
- Where the actual failure occurs
- Important response/error code
- Why the original API ultimately failed

Example:

`The issue is not caused by the initial API process directly. The failure occurs when Service B calls CRM, where CRM rejects the request with response code 0011.`

---

# Writing Style

Use concise technical English suitable for Jira.

Prefer:

`Further tracing showed that Activation called Personal Device.`

Instead of:

`After checking the logs again and tracing several other logs, we found that there was another API being called by Activation.`

Keep sentences short.

Use bullet points for:
- HTTP status
- Response code
- Error message
- Important request parameters

Use backticks for:
- Endpoint
- Service method
- Error code
- HTTP status
- Important technical values

Do not over-explain obvious technical details.

---

# Investigation Rules

Always distinguish between:

**Symptom**
The error visible from the initial API.

**Propagation**
How the error travels between services.

**Root Cause**
The deepest confirmed failure found from the evidence.

For example:

`Authorization returns 422`
↓
`CIAM returns binding error`
↓
`Activation callback fails`
↓
`Personal Device receives CRM rejection`
↓
`CRM rejects MSISDN`

In this situation, do NOT state that Authorization or CIAM is the root cause if CRM is the system actually rejecting the operation.

---

# Important Data

Preserve values when they are useful for investigation, such as:

- HTTP status
- Internal response code
- Error message
- Endpoint
- Service name
- TSEL ID / transaction ID when relevant
- Account type
- Feature/process name

Mask or remove:

- API key
- Authorization header
- Access token
- Refresh token
- JWT
- Session token
- Password
- OTP unless specifically necessary
- Personal information that is not required for debugging

---

# Final Quality Check

Before producing the output, verify:

- The summary describes the original issue.
- Investigation follows the actual request flow.
- The deepest confirmed failure is used as the root cause.
- Symptom and root cause are not confused.
- Important HTTP/error codes are preserved.
- Unnecessary logs are removed.
- Credentials and tokens are removed.
- No unsupported assumptions are presented as facts.
- The result can be pasted directly into Jira.
