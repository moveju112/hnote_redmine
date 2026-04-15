# Installing hnote_redmine for Codex

Codex CLI에서 hnote_redmine 스킬을 사용하기 위한 설치 방법입니다.

## 설치

1. **레포지토리 클론:**
   ```bash
   git clone https://github.com/moveju112/hnote_redmine.git ~/.codex/hnote_redmine
   ```

2. **스킬 심볼릭 링크 생성:**
   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/hnote_redmine/skills ~/.agents/skills/hnote_redmine
   ```

   **Windows (PowerShell):**
   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
   cmd /c mklink /J "$env:USERPROFILE\.agents\skills\hnote_redmine" "$env:USERPROFILE\.codex\hnote_redmine\skills"
   ```

3. **Codex 재시작** — 스킬은 시작 시 자동으로 발견됩니다.

## 확인

```bash
ls -la ~/.agents/skills/hnote_redmine
```

심볼릭 링크가 `~/.codex/hnote_redmine/skills`를 가리키면 설치 완료입니다.

## 업데이트

```bash
cd ~/.codex/hnote_redmine && git pull
```

## 제거

```bash
rm ~/.agents/skills/hnote_redmine
```

클론도 삭제하려면: `rm -rf ~/.codex/hnote_redmine`
