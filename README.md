# meetko-plugins

[meetko](https://github.com/Eastsidegunn/meetko) 플러그인 목록 저장소입니다. 앱의 **둘러보기**는 이 저장소의 `index.json`을 읽습니다. 공식 예제 플러그인도 여기 폴더로 들어 있습니다.

## 내 플러그인을 목록에 올리기

1. 플러그인을 **내 공개 GitHub 저장소**에 만듭니다. `meetko.toml`은 저장소 루트나 폴더 하나에 둡니다. 형식은 meetko의 [docs/manifest-draft.md](https://github.com/Eastsidegunn/meetko/blob/main/docs/manifest-draft.md)를 따르고, `package.id`의 앞부분은 내 GitHub 계정 이름(소문자)이어야 합니다. 예: `github.com/alice/meetko-summary` → `id = "alice/summary"`.
2. 설치 중에는 아무것도 빌드하지 않습니다. Python 같은 인터프리터 플러그인은 소스 그대로, 컴파일하는 플러그인은 빌드된 실행 파일을 커밋해야 합니다.
3. 버전을 낼 때 release 태그를 답니다.
4. 이 저장소에 **PR**을 보내 `sources.json`의 `plugins`에 한 줄을 더합니다.

   ```json
   {"repo": "alice/meetko-summary", "ref": "v1.0.0"}
   {"repo": "alice/my-plugins", "path": "summary", "ref": "v1.0.0"}
   ```

   `path`는 저장소 안 폴더(없으면 루트)입니다. **`ref`(release 태그 또는 40자 커밋 sha)는 꼭 적어야 합니다.** 목록이 PR에서 본 버전에 묶이도록, 기본 브랜치를 따라가는 줄은 받지 않습니다(이 저장소의 공식 예제 줄만 예외). 새 버전은 `ref`를 바꾸는 PR로 올립니다.
   설치 한도(내려받는 크기 25MB, 압축 푼 크기 100MB, 항목 5000개)는 폴더가 아니라 저장소 전체에 적용되므로, 아주 큰 저장소 안의 폴더는 설치할 수 없습니다.
5. PR에서는 `tools/make-index`가 형식만 확인합니다(이 저장소의 폴더는 앱의 설치 규칙도: 심볼릭 링크·서브모듈 금지, 이식 가능한 파일 이름, 깊이 32, 크기·항목 한도). 합쳐지면 `.github/workflows/index.yml`이 각 항목의 매니페스트를 GitHub에서 그 커밋 그대로 읽어 `index.json`을 다시 만들고 커밋 sha에 고정합니다.

목록에 오른다는 것은 **형식이 맞다**는 뜻일 뿐이고, 코드를 검토했다는 뜻이 아닙니다. 게시와 설치 규칙 전체는 meetko의 [docs/plugin-publishing.md](https://github.com/Eastsidegunn/meetko/blob/main/docs/plugin-publishing.md), 격리(샌드박스)는 [docs/plugin-security.md](https://github.com/Eastsidegunn/meetko/blob/main/docs/plugin-security.md)를 보세요. 문제가 있는 플러그인은 이 저장소의 issue로 알려 주세요.

## 공식 예제

| 폴더 | id | 하는 일 | 권한 |
|---|---|---|---|
| `action-items-py` | `eastsidegunn/action-items-py` | 규칙(정규식)으로 액션아이템과 결정 사항을 표시합니다. Python 표준 라이브러리만 씁니다. | 네트워크 없음, API 키 없음 |
| `summary-py` | `eastsidegunn/summary-py` | 회의 내용을 AI로 요약하고 할 일·결정 사항을 뽑습니다. 근거로 든 단어 범위가 실제 전사본에 없으면 버립니다. | **네트워크 사용.** 대화 텍스트가 고른 AI 서비스로 전송됩니다. API 키: `ANTHROPIC_API_KEY` 또는 `OPENAI_API_KEY`(+ `OPENAI_BASE_URL`, OpenAI 호환 서버). 설정: `ANTHROPIC_MODEL`, `ANTHROPIC_BASE_URL`, `OPENAI_MODEL` |
| `talk-time-rs` | `eastsidegunn/talk-time-rs` | 화자별 발언 시간을 표로 보여 줍니다. Rust로 만든 실행 파일입니다. **목록에는 없습니다**(빌드가 필요해서). 소스에서 빌드해 설치하는 예제입니다. | 네트워크 없음, API 키 없음 |

설치:

```
# 앱: 플러그인 화면 → 둘러보기 → 목록 불러오기 → 설치
meetko plugin install github:Eastsidegunn/meetko-plugins/action-items-py@main
meetko plugin install github:Eastsidegunn/meetko-plugins/summary-py@main
pbpaste | meetko plugin secret set eastsidegunn/summary-py ANTHROPIC_API_KEY   # 키 값은 stdin으로

# talk-time-rs: 소스에서 빌드한 뒤 로컬 폴더로
git clone https://github.com/Eastsidegunn/meetko-plugins
(cd meetko-plugins/talk-time-rs && cargo build --release --locked)
meetko plugin add ./meetko-plugins/talk-time-rs
```

`@main` 대신 태그나 커밋 sha를 쓰면 그 버전에 고정됩니다(어느 쪽이든 설치는 그 순간의 커밋에 고정됩니다).

## 테스트

```
python3.12 -I tools/make-index                       # sources.json과 플러그인 폴더 확인 (네트워크 없음)
python3 -I summary-py/test_summary.py                # 네트워크 없음 (127.0.0.1의 가짜 서버)
(cd talk-time-rs && cargo test --locked && cargo build --release --locked)
```

`testdata/`는 meetko `contract/`의 회의 예시와 출력 스키마(계약 0.1)를 복사한 것입니다. `tools/build_index.py`와 `index.schema.json`은 meetko `tools/plugin-index/`의 복사본입니다(함께 갱신합니다).

## index.json 만들기 (관리자용)

`index.json`은 실제 커밋 sha가 있어야 하므로 GitHub에 올린 뒤에 만듭니다. 그 전까지는 빈 목록(`"plugins": []`)입니다.

1. GitHub에 공개 저장소 `Eastsidegunn/meetko-plugins`를 만들고 `main`을 push합니다.
2. Actions가 켜져 있으면 `sources.json`이 든 push로 `plugin index` 워크플로가 돌아 `index.json`을 커밋합니다. 안 돌았으면 Actions 탭에서 `plugin index`를 수동 실행합니다(`commit` 작업이 `main`에 push할 수 있어야 합니다).
3. 네트워크 없이 직접 만들 수도 있습니다: `python3.12 -I tools/make-index --sha $(git rev-parse origin/main)`로 쓰고 커밋·push합니다. sha는 GitHub `main`에 이미 있는 커밋이어야 합니다(그렇지 않으면 앱이 설치를 거부합니다). 다른 사람 저장소의 줄은 워크플로만 만들 수 있고, 이 명령은 현재 `index.json`에 있는 그 줄을 그대로 둡니다.
4. 확인: `curl -s https://raw.githubusercontent.com/Eastsidegunn/meetko-plugins/main/index.json`에 두 항목(`action-items-py`, `summary-py`)과 `path`, 40자 `sha`가 보이고, 앱의 둘러보기에서 설치되면 됩니다.

## English

The catalog for [meetko](https://github.com/Eastsidegunn/meetko) plugins: the app's plugin browser reads this repo's `index.json`, and the official examples live here as folders.

**Get your plugin listed:** publish it in your own public GitHub repo (`meetko.toml` at the root or in a folder; the id's publisher part is your GitHub owner name in lowercase; nothing is built at install, so commit binaries for compiled plugins), tag a release, and open a pull request adding a row to `sources.json` (`{"repo": "OWNER/REPO", "ref": "v1.0.0"}`, plus `"path": "FOLDER"` for a subfolder). The `ref` (a release tag or a full commit sha) is required, so the listing stays on the version reviewed in the PR; only this repo's own example rows follow `main`. The app's download caps apply to the whole repository archive, so a folder in a very large repo cannot be installed. `tools/make-index` checks the row in the PR (and, for this repo's folders, the app's install rules); after the merge, `.github/workflows/index.yml` reads each manifest from GitHub and rebuilds `index.json`, pinned to commits. Being listed means the format is right, not that the code was reviewed. See meetko's `docs/plugin-publishing.md`.

**Official examples:** `eastsidegunn/action-items-py` (rule-based action items, no network), `eastsidegunn/summary-py` (AI summary; **uses the network** and sends the transcript text to the AI service you configure with `ANTHROPIC_API_KEY` or `OPENAI_API_KEY`/`OPENAI_BASE_URL`), and `eastsidegunn/talk-time-rs` (talk time per speaker, a Rust binary; not listed because it must be built, so build it with `cargo build --release --locked` and `meetko plugin add` the folder). Install the listed ones from the app or with `meetko plugin install github:Eastsidegunn/meetko-plugins/FOLDER@main`.

**index.json** needs real commit shas, so it is empty until this repo is on GitHub. Push `main`; the `plugin index` workflow then writes it (or run it by hand from the Actions tab), or run `python3.12 -I tools/make-index --sha <a commit already on GitHub main>` and commit the result.

License: AGPL-3.0-or-later. Copyright (C) 2026 Eastsidegunn.
