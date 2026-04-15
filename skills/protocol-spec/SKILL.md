---
name: protocol-spec
description: Use when asked to write, generate, or update Redmine protocol specification documents from server code. Applies when user says "프로토콜 명세서 작성", "명세 작성", requests spec for a method number, or asks to analyze code and generate API documentation in Redmine textile format.
---

# Redmine Protocol Specification Writer

## Overview

Analyze server code and write client-facing protocol specification documents in Redmine textile format.
The request scope may be an entire content section or a single method.

## Output File Rule

- Also save the final Redmine textile output to `docs/specification/`.
- The saved file must contain **only** the Redmine body content that will actually be pasted into Redmine.
- Do not include analysis notes, assumptions, file lists, validation logs, or surrounding explanation in the saved file.
- Create the directory first if it does not exist.
- Use a predictable filename based on the request scope:
  - Single method: `method-XXXX.textile`
  - Single source file or feature scope: short kebab-case title + `.textile`
- When some fields are unconfirmed, keep only the Redmine markup in the file and express uncertainty inside the relevant Description cell as `(확인 필요)`.

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

## Scope Limit (Required)

**Reject any request spanning more than 1 file.**

- If the analysis target spans 2 or more files, stop immediately.
- Also reject broad requests like "전체 프로토콜", "컨텐츠 전체", "모든 메서드".
- Rejection message: "한 번에 1개 파일만 처리할 수 있습니다. 파일을 하나 지정해 주세요."

## Method Number Lookup (Required)

**Always read the method integer value directly from `wcnsvr/game/Enums/ActionMapper.php`.**

- `ActionMapper.php` defines `enum Action: int` mapping action names to integer values.
  e.g. `case EQUIP_LOCK = 7006;`
- When you encounter an enum reference like `Action::EQUIP_LOCK` in handler code, open `ActionMapper.php`, find the case, and use its integer value (e.g. `* method : 7006`).
- `ActionMetaData.php` (`wcnsvr/game/Enums/ActionMetaData.php`) maps each Action to `[Action::XXX, 'Method.string', '레이블', isLogged]`. Use the Korean label for the h2. section title.
- Never guess or infer method numbers; always verify from `ActionMapper.php`.

## Code Analysis Procedure

### Full Content Request

1. Identify the method number range for the content from `ActionMapper.php`
2. Analyze each method handler in order
3. Request params: extract from parsing/validation code
4. Response fields: extract from response generation/serialization code
5. Write entire content as one document (one `h1.`, one `h2.` per feature)

### Single Method Request

1. Analyze only the handler for the requested method number
2. Confirm the integer value from `ActionMapper.php`
3. Write one `h2.` section only (omit h1/toc unless context requires it)

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
