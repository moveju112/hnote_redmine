# Redmine Skills — Claude Code Plugin

Redmine 프로젝트 관리를 위한 Claude Code 스킬 플러그인입니다.

## 설치

### Claude Code

1. `/plugin` 입력
2. **Add Marketplace** 선택
3. URL 입력: `https://github.com/moveju112/hnote_redmine`

### Codex CLI

1. **레포지토리 클론:**
   ```bash
   git clone https://github.com/moveju112/hnote_redmine.git ~/.codex/hnote_redmine
   ```

2. **스킬 심볼릭 링크 생성:**
   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/hnote_redmine/skills ~/.agents/skills/hnote_redmine
   ```

3. **Codex 재시작**

자세한 내용은 [`.codex/INSTALL.md`](.codex/INSTALL.md) 참고.

## 스킬 목록

### `protocol-spec`

서버 코드를 분석하여 클라이언트 전달용 프로토콜 명세서를 Redmine textile 서식으로 작성합니다.

**트리거 조건:**
- "프로토콜 명세서 작성", "명세 작성" 요청 시
- 메서드 번호를 언급하며 명세 요청 시
- 코드 분석 후 API 문서 생성 요청 시

**기능:**
- 컨텐츠 전체 또는 특정 메서드 단위로 작성 가능
- Request / Response 파라미터 테이블 자동 구성
- JSON 예시 자동 생성 (`{{random_id}}`, `{{sessoin}}` 포함)
- 코드 주석 없는 필드도 이름/타입으로 한국어 Description 추론
- 모든 출력은 한국어

**Redmine textile 출력 예시:**

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
```

## 스킬 추가 방법

1. `skills/` 디렉토리 아래에 새 폴더 생성 (예: `skills/my-skill/`)
2. 폴더 안에 `SKILL.md` 작성
3. `SKILL.md` 상단에 YAML frontmatter 추가:

```markdown
---
name: my-skill
description: Use when [트리거 조건]
---
```

## 구조

```
hnote_redmine/
├── .claude-plugin/
│   └── plugin.json          # 플러그인 메타데이터
├── skills/
│   ├── protocol-spec/
│   │   └── SKILL.md         # 프로토콜 명세서 작성 스킬
│   └── example-skill/
│       └── SKILL.md         # 새 스킬 작성용 템플릿
├── package.json
└── README.md
```

## 라이선스

MIT
