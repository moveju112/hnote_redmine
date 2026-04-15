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

## 작업 범위 제한 (필수)

**1개 파일 초과 요청은 반드시 거부한다.**

- 분석 대상이 2개 이상의 파일에 걸치는 경우 즉시 중단
- "전체 프로토콜", "컨텐츠 전체", "모든 메서드" 등 광범위한 요청도 거부
- 거부 시 안내 문구: "한 번에 1개 파일만 처리할 수 있습니다. 파일을 하나 지정해 주세요."

## Method Number Lookup (필수)

**method 번호는 반드시 `wcnsvr/game/Enums/ActionMapper.php`에서 직접 읽어야 한다.**

- `ActionMapper.php`는 PHP `enum Action: int` 형태로 각 액션 이름과 정수 값을 정의한다.
  예: `case EQUIP_LOCK = 7006;`
- 핸들러 코드에서 `Action::EQUIP_LOCK` 같은 enum 참조를 발견하면, 반드시 `ActionMapper.php`를 열어 해당 case의 정수 값을 확인한 뒤 `* method : 7006` 형태로 기재한다.
- `ActionMetaData.php`(`wcnsvr/game/Enums/ActionMetaData.php`)에는 `[Action::XXX, 'Method.string', '레이블', isLogged]` 형태로 각 액션의 메서드 문자열과 한국어 레이블이 기재되어 있다. h2. 기능명 작성 시 이 레이블을 참고한다.
- 정수 값을 확인하지 않은 채 추정하거나 다른 파일에서 간접적으로 읽은 값을 사용하지 않는다.

## Code Analysis Procedure

### Full Content Request

1. `ActionMapper.php`에서 해당 컨텐츠의 메서드 번호 범위를 확인한다
2. 각 메서드 핸들러를 순서대로 분석한다
3. Request params: 파싱/유효성 검증 코드에서 추출
4. Response fields: 응답 생성/직렬화 코드에서 추출
5. 전체 컨텐츠를 하나의 문서로 작성 (h1. 하나, 기능별 h2. 하나씩)

### Single Method Request

1. 요청된 메서드 번호의 핸들러만 분석한다
2. `ActionMapper.php`에서 해당 정수 값을 확인한다
3. h2. 섹션만 작성 (문맥상 필요하지 않으면 h1./toc 생략)

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
