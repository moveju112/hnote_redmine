---
name: protocol-spec
description: Use when asked to write, generate, or update Redmine protocol specification documents from server code. Applies when user says "프로토콜 명세서 작성", "명세 작성", requests spec for a method number, or asks to analyze code and generate API documentation in Redmine textile format.
---

# Redmine Protocol Specification Writer

## Overview

Analyze server code and write client-facing protocol specification documents in Redmine textile format.
The request scope may be an entire content section or a single method.

## Redmine Textile Format Rules

### Document Structure

```textile
h1. 컨텐츠명

{{>toc}}

h2. 기능명

* method : XXXX

h3. Request

|_.Key |_.Subkey |_.Type |_.Description |
| 키 | | 타입 | 설명 |
| 키 | 서브키 | 타입 | 설명 |

<pre>
{
    "id": "{{random_id}}",
    "method": "XXXX",
    "params": { ... },
    "sess": "{{sessoin}}"
}
</pre>

h3. Response

|_.Key |_.Subkey |_.Type |_.Description |
| 키 | | 타입 | 설명 |

<pre>
{
    "id": "{{random_id}}",
    "result": { ... }
}
</pre>
```

### Table Rules

- Header row: `|_.Key |_.Subkey |_.Type |_.Description |`
- Top-level key: `| keyname | | type | description |` (Subkey cell empty)
- Subkey: `| | subkeyname | type | description |` (Key cell empty)
- Group key (array/object): leave Type cell empty, write description only
- Always maintain 4 columns

### JSON Example Rules

- Request: include `id`, `method`, `params`, `sess`
- Response: include `id`, `result` (use `true`/`false` for simple boolean results)
- id value: `"{{random_id}}"`
- sess value: **always `"{{sessoin}}"` — never modify**
- Use realistic values in examples (no dummy values like 0, 1, "string")

## Code Analysis Procedure

### Full Content Request

1. Find the method number range for the content in handler/router files
2. Analyze each method handler in order
3. Request params: extract from parsing/validation code
4. Response fields: extract from response generation/serialization code
5. Write entire content as one document (one `h1.`, one `h2.` per feature)

### Single Method Request

1. Analyze only the handler for the requested method number
2. Write one `h2.` section only (omit h1/toc unless context requires it)

### What to Extract from Code

| Code Pattern | Spec Field |
|---|---|
| Parameter parsing (`req.params.xxx`, struct fields) | Request Key/Type |
| Required/optional validation logic | Add "(선택)" in Description |
| DB query/update result fields | Response Key |
| Enum/constant definitions | Explain values in Description |
| Array iteration code | Array-type key |
| Conditionally included fields | Note condition in Description |

## Type Mapping

| Code Type | Spec Notation |
|---|---|
| string, varchar, text | string |
| int, int32, int64, number | int |
| float, double | float |
| bool, boolean | bool |
| array, slice, list | array |
| nested object | (leave Type empty, write "리스트" or "정보" in Description) |

## Checklist

- [ ] Every method has `* method : XXXX`
- [ ] Request table: 4 columns, subkey rows have Key cell empty
- [ ] Response table: 4 columns, subkey rows have Key cell empty
- [ ] JSON examples use `{{random_id}}` and `{{sessoin}}`
- [ ] JSON example values are realistic, not dummy
- [ ] Simple success response uses `"result": true`
- [ ] Full content request includes `h1.` and `{{>toc}}`
- [ ] Group keys have Type cell empty

## Example — Shop Purchase Method

```textile
h2. 상품 구매

* method : 3001

h3. Request

|_.Key |_.Subkey |_.Type |_.Description |
| gid | | int | 상점 그룹 ID |
| sid | | int | 상품 ID |
| amt | | int | 구매 수량 |

<pre>
{
    "id": "{{random_id}}",
    "method": "3001",
    "params": {
        "gid": 101,
        "sid": 2005,
        "amt": 3
    },
    "sess": "{{sessoin}}"
}
</pre>

h3. Response

|_.Key |_.Subkey |_.Type |_.Description |
| item | | | 변경된 아이템 목록 |
| | iid | int | 아이템 인덱스 |
| | iamt | int | 보유 수량 |
| rsoc | | | 변경된 재화 목록 |
| | rid | int | 재화 ID |
| | ramt | string | 보유량 |

<pre>
{
    "id": "{{random_id}}",
    "result": {
        "item": [
            {
                "iid": 2005,
                "iamt": 3
            }
        ],
        "rsoc": [
            {
                "rid": 1,
                "ramt": "9850000"
            }
        ]
    }
}
</pre>
```

## Notes

- `sess` key value: **always write `"{{sessoin}}"` — do not fix the typo**
- If no comments in code, infer meaning from field name and type, write Description in Korean
- Fields that cannot be confirmed: add `(확인 필요)` in Description and notify the user
- Never abbreviate repeated response structures — write them out fully every time

All output must be in Korean.
