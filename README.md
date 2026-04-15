# Redmine Skills — Claude Code Plugin

Redmine 프로젝트 관리를 위한 Claude Code 스킬 플러그인입니다.

## 설치

Claude Code에서 다음 명령어로 설치:

```
/plugin install redmine@your-username
```

또는 GitHub URL로 직접 설치:

```
/plugin install https://github.com/your-username/redmine-skills
```

## 스킬 목록

| 스킬명 | 설명 |
|--------|------|
| `protocol-spec` | 서버 코드 분석 후 Redmine textile 서식 프로토콜 명세서 작성 |

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
redmine-skills/
├── .claude-plugin/
│   └── plugin.json       # 플러그인 메타데이터
├── skills/
│   └── skill-name/
│       └── SKILL.md      # 스킬 정의
├── package.json
└── README.md
```

## 라이선스

MIT
