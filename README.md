# Phone Number Checker API — Bulk Verification Examples

The **Phone Number Checker API** is a bulk, asynchronous, file-based API used to validate phone numbers and evaluate activity-related signals. Upload an input file, poll the returned task ID, then download the exported results. This repository contains runnable examples in Python, Node.js, Go, Java, C#, PHP, browser JavaScript, and Shell.

- **Base URL:** `https://api.checknumber.ai`
- **Auth:** `X-API-Key` request header
- **Full API docs:** https://docs.checknumber.ai/phone-number-validation
- **Get an API key:** https://platform.checknumber.ai

> [!NOTE]
> This is a bulk, file-based, asynchronous API. It does not perform a single real-time lookup in one request/response cycle.

## Table of Contents

- [How do I validate phone numbers via API?](#how-do-i-validate-phone-numbers-via-api)
- [How do I process Phone checks in bulk?](#how-do-i-process-phone-checks-in-bulk)
- [What request parameters does the API take?](#what-request-parameters-does-the-api-take)
- [What does the task response look like?](#what-does-the-task-response-look-like)
- [How do I check whether a phone number is active?](#how-do-i-check-whether-a-phone-number-is-active)
- [How do I detect high-value phone users via API?](#how-do-i-detect-high-value-phone-users-via-api)
- [What does the Phone number checker API cost?](#what-does-the-phone-number-checker-api-cost)
- [Phone number checker API vs alternatives](#phone-number-checker-api-vs-alternatives)
- [Code examples](#code-examples)
- [Status codes](#status-codes)
- [FAQ](#faq)
- [Support & legal](#support--legal)

## How do I validate phone numbers via API?

Send a `POST` request to `https://api.checknumber.ai/v1/tasks` with the API key, `numbers.txt`, and `task_type=phoneCheck`. The response contains a `task_id` used to retrieve task status and the final `result_url`.

```bash
curl --location 'https://api.checknumber.ai/v1/tasks' \
  --header 'X-API-Key: YOUR_API_KEY' \
  --form 'file=@"./numbers.txt"' \
  --form 'task_type="phoneCheck"'
```

Each line in `numbers.txt` is one phone number in E.164 format, such as `+14155552671`.

## How do I process Phone checks in bulk?

1. **Submit** the input file to `POST /v1/tasks`.
2. **Poll** `POST /v1/gettasks` with the returned `task_id` while status moves from `pending` to `processing`.
3. **Download** the file at `result_url` when status becomes `exported`.

```bash
curl --location 'https://api.checknumber.ai/v1/gettasks' \
  --header 'X-API-Key: YOUR_API_KEY' \
  --form 'task_id="YOUR_TASK_ID"'
```

See [`examples/`](./examples) for complete submit → poll → download implementations.

## What request parameters does the API take?

**Submit task — `POST /v1/tasks`**

| Parameter | Type | Description |
| --- | --- | --- |
| `file` | file | Text file containing one E.164 phone number per line for the main task type. Email variants use `emails.txt`. |
| `task_type` | string | One of: `phoneCheck`, `active_check`, `high_value_users`. |

**Check status — `POST /v1/gettasks`**

| Parameter | Type | Description |
| --- | --- | --- |
| `task_id` | string | Task ID returned by task creation. |

**Main task result fields — `task_type=phoneCheck`**

| Field         | Description |
| ------------- | ----------- |
| `Number`      | Phone number in E.164 format |
| `phoneCheck`  | Validation outcome / detail (per exported column headers) |

## What does the task response look like?

```json
{
  "created_at": "2026-07-13T08:24:56Z",
  "updated_at": "2026-07-13T08:25:43Z",
  "task_id": "example-task-id",
  "user_id": "example-user-id",
  "status": "exported",
  "total": 1000,
  "success": 998,
  "failure": 2,
  "result_url": "https://example.invalid/results.xlsx"
}
```

All variants use this task envelope. Only `task_type`, input content, and exported result columns vary.

## How do I check whether a phone number is active?

Classifies the documented activity status for each number. Use `task_type=active_check`; the submit, poll, and download workflow is otherwise unchanged.

```bash
curl --location 'https://api.checknumber.ai/v1/tasks' \
  --header 'X-API-Key: YOUR_API_KEY' \
  --form 'file=@"./numbers.txt"' \
  --form 'task_type="active_check"'
```

Input: one phone number per line in E.164 format (for example, `+14155552671`).

Result fields (copied from the [canonical documentation](https://docs.checknumber.ai/number-active-check)):

| Field          | Description |
| -------------- | ----------- |
| `Number`       | Phone number in E.164 format |
| `ActiveStatus` | Activity level of the number: `active`, `inactive`, or `unknown` |

## How do I detect high-value phone users via API?

Returns the documented high-value-user detection result. Use `task_type=high_value_users`; the submit, poll, and download workflow is otherwise unchanged.

```bash
curl --location 'https://api.checknumber.ai/v1/tasks' \
  --header 'X-API-Key: YOUR_API_KEY' \
  --form 'file=@"./numbers.txt"' \
  --form 'task_type="high_value_users"'
```

Input: one phone number per line in E.164 format (for example, `+14155552671`).

Result fields (copied from the [canonical documentation](https://docs.checknumber.ai/high-value-users-checker)):

| Field    | Description |
| -------- | ----------- |
| `Number` | Phone number |
| `Result` | High-value user detection result (`yes` / `no` style; labels per export) |

## What does the Phone number checker API cost?

Rates vary by task type and are billed against the account balance. See the current terms at **https://checknumber.ai/pricing** and query the remaining balance with `GET https://api.checknumber.ai/v1/balance`.

## Phone number checker API vs alternatives

| Capability | CheckNumber API | Generic carrier lookup | Manual checking |
| --- | --- | --- | --- |
| Checks the documented Phone signal | Yes | No | May be possible |
| Bulk file upload | Yes | Varies | No |
| Asynchronous processing | Yes | Varies | No |
| Downloadable structured results | Yes | Varies | No |

*This compares general approaches, not named vendors. Capabilities vary by provider and account access.*

## Code examples

| Language | Example |
| --- | --- |
| Python | [`examples/python`](./examples/python) |
| Node.js | [`examples/nodejs`](./examples/nodejs) |
| Go | [`examples/go`](./examples/go) |
| Java | [`examples/java`](./examples/java) |
| C# | [`examples/csharp`](./examples/csharp) |
| PHP | [`examples/php`](./examples/php) |
| JavaScript (browser) | [`examples/javascript`](./examples/javascript) |
| Shell (curl) | [`examples/shell`](./examples/shell) |

Set the API key before running a server-side example:

```bash
export CHECKNUMBER_API_KEY="YOUR_API_KEY"
```

## Status codes

| Status | Billing | Description |
| --- | --- | --- |
| `200` | `charge` | Request successful; task created or status retrieved. |
| `400` | `free` | Invalid parameters or file format. |
| `500` | `free` | Internal server error; retry later. |

## FAQ

**Is this a single-number real-time lookup?**  
No. Submit a file, poll for completion, and download the result file.

**Which task types are available in this repository?**

- `task_type=phoneCheck` — [Phone Number Validation](https://docs.checknumber.ai/phone-number-validation); Returns the documented phone validation outcome for each E.164 number. Input: numbers.txt.
- `task_type=active_check` — [Phone Number Activity Check](https://docs.checknumber.ai/number-active-check); Classifies the documented activity status for each number. Input: numbers.txt.
- `task_type=high_value_users` — [High-Value User Checker](https://docs.checknumber.ai/high-value-users-checker); Returns the documented high-value-user detection result. Input: numbers.txt.

**Which input format should I use?**  
Phone-based checks require one E.164 number per line. Email variants require one email address per line.

**How do I check my balance?**  
Call `GET https://api.checknumber.ai/v1/balance` with the `X-API-Key` header.

## Support & legal

- **Documentation:** https://docs.checknumber.ai/phone-number-validation
- **Dashboard and API keys:** https://platform.checknumber.ai
- **License:** [MIT](./LICENSE) for the sample code

Process only data you are authorized to use and comply with applicable privacy laws and platform terms. CheckNumber is not affiliated with or endorsed by Phone.

---

*Last updated: 2026-07-13 · Maintained by CheckNumber. Canonical docs: https://docs.checknumber.ai/phone-number-validation*
