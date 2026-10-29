# PetStore API: Contract Tests (POST /pet)

Contract test pack for the `POST /pet` endpoint of the public [Swagger PetStore API](https://petstore.swagger.io/). The goal was to check how the API validates incoming data and whether it follows its own contract (Swagger specification).

**Result:** 15 tests, **4 critical bugs** (the server returns `500 Server Error` instead of `400 Bad Request`) and **5 validation issues**.

## What is tested

| Group | Tests | What it checks |
|---|---|---|
| Positive | P1–P2 | Valid requests: minimal and full set of fields |
| Negative | N1–N8 | Missing required fields, wrong data types, invalid enum value, `null` |
| Edge cases | E1–E5 | Empty array, 1000-character name, special characters, emoji, Cyrillic |

Each request has automated checks (Postman test scripts):
- status code
- response structure (`id` is a number, `photoUrls` is an array)
- the name is returned unchanged
- the server does not crash with a 5xx error on invalid data
- response time is below 3000 ms

## Results

| ID | Test | Expected | Actual | Status |
|---|---|---|---|---|
| P1 | Baseline valid pet | 200 | 200 | ✅ Pass |
| P2 | Full valid pet | 200 | 200 | ✅ Pass |
| N1 | Missing `name` | 400 | 200 | ⚠️ Issue |
| N2 | `name` is a number | 400 | 200 (converted to string) | ⚠️ Issue |
| N3 | Missing `photoUrls` | 400 | 200 (empty array added) | ⚠️ Issue |
| N4 | `photoUrls` is a string | 400 | **500** | 🔴 Bug |
| N5 | Invalid `status` value | 400 | **500** | 🔴 Bug |
| N6 | `name` is `null` | 400 | 200 (treated as missing) | ⚠️ Issue |
| N7 | `name` is an array | 400 | **500** | 🔴 Bug |
| N8 | `name` is an object | 400 | **500** | 🔴 Bug |
| E1 | Empty `photoUrls` array | 200 | 200 | ✅ Pass |
| E2 | 1000-character name | 200 or 400 | 200 (no length limit) | ⚠️ Issue |
| E3 | Special characters | 200 | 200 | ✅ Pass |
| E4 | Emoji | 200 | 200 | ✅ Pass |
| E5 | Cyrillic | 200 | 200 | ✅ Pass |

## Bugs found

| Bug | Severity | Description |
|---|---|---|
| BUG-1 (N4) | Critical | `photoUrls` sent as a string instead of an array causes `500 Server Error`. Expected `400 Bad Request` with a clear validation message. |
| BUG-2 (N5) | Critical | An invalid `status` value (not `available`, `pending` or `sold`) causes `500 Server Error` instead of `400`. |
| BUG-3 (N7) | Critical | `name` sent as an array causes `500 Server Error` instead of `400`. |
| BUG-4 (N8) | Critical | `name` sent as an object causes `500 Server Error` instead of `400`. |

**Why it matters:** a 500 error means the server crashed on user input. The client gets no explanation of what is wrong, and this can hide deeper problems with input handling.

**Validation issues:** the API accepts requests without the required fields `name` and `photoUrls` (N1, N3, N6), silently converts wrong data types (N2) and has no maximum length for `name` (E2). This does not match the Swagger specification, where `name` and `photoUrls` are marked as required.

## How to run

### In Postman
1. Download `PetStore_Contract_Tests.postman_collection.json`.
2. In Postman: **Import** → choose the file.
3. Open the collection → **Run** → **Run PetStore API - Contract Tests**.

### From the command line (Newman)
```bash
npm install -g newman
newman run PetStore_Contract_Tests.postman_collection.json
```

Tests P1–P2 and E1–E5 should pass. Tests for N1–N8 are expected to fail: each failure shows a place where the API breaks its contract.

> Note: PetStore is a public demo API, so its behaviour can change over time. The results above were recorded in January 2026.

## Tools
Postman · Newman · REST API · JSON · contract testing · test design techniques (equivalence partitioning, boundary values)

## Author
**Nadiia Hodovanets**, QA Engineer (Manual, API and Accessibility Testing)
[LinkedIn](https://linkedin.com/in/nadiia-hodovanets-7a8098182)
