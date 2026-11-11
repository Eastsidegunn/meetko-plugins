# meetko-plugins

[meetko](https://github.com/Eastsidegunn/meetko) 예제 플러그인과, 앱의 **둘러보기**가 읽는 플러그인 목록(`index.json`)을 두는 저장소입니다. 플러그인 하나가 최상위 폴더 하나입니다.

| 폴더 | 하는 일 | 권한 |
|---|---|---|
| `action-items-py` | 규칙(정규식)으로 액션아이템과 결정 사항을 표시합니다. Python 표준 라이브러리만 씁니다. | 네트워크 없음, API 키 없음 |
| `summary-py` | 회의 내용을 AI로 요약하고 할 일·결정 사항을 뽑습니다. 근거로 든 단어 범위가 실제 전사본에 없으면 버립니다. | **네트워크 사용.** 대화 텍스트가 고른 AI 서비스로 전송됩니다. API 키: `ANTHROPIC_API_KEY` 또는 `OPENAI_API_KEY`(+ `OPENAI_BASE_URL`, OpenAI 호환 서버). 설정: `ANTHROPIC_MODEL`, `ANTHROPIC_BASE_URL`, `OPENAI_MODEL` |
| `talk-time-rs` | 화자별 발언 시간을 표로 보여 줍니다. Rust로 만든 실행 파일입니다. | 네트워크 없음, API 키 없음 |

## 설치

**로컬 폴더에서 (지금 되는 방법)**

```
git clone https://github.com/Eastsidegunn/meetko-plugins
meetko plugin add ./meetko-plugins/action-items-py          # 권한을 보여 주고 동의를 받은 뒤 설치
(cd meetko-plugins/talk-time-rs && cargo build --release --locked)
meetko plugin add ./meetko-plugins/talk-time-rs              # 먼저 빌드해야 합니다
meetko plugin add ./meetko-plugins/summary-py
pbpaste | meetko plugin secret set meetko/summary-py ANTHROPIC_API_KEY   # 키 값은 stdin으로
```

앱에서는 플러그인 화면에서 폴더를 골라 추가하고, `API 키 넣기`로 키를 넣습니다.

**GitHub에서 / 앱의 둘러보기에서.** meetko는 `meetko plugin install github:OWNER/REPO@REF`로 저장소 **루트**의 `meetko.toml`을 받아 설치하고, id의 게시자 부분이 저장소 소유자와 같아야 하며, 설치 중에 빌드하지 않습니다. 이 저장소처럼 폴더 여러 개를 담은 저장소의 하위 폴더는 그 방식으로 설치할 수 없어서, 지금 `index.json`의 목록은 비어 있습니다. 아래 "목록 만들기"를 보세요.

## 플러그인 만들기

매니페스트(`meetko.toml`) 형식은 meetko의 [docs/manifest-draft.md](https://github.com/Eastsidegunn/meetko/blob/main/docs/manifest-draft.md), 게시와 설치 규칙은 [docs/plugin-publishing.md](https://github.com/Eastsidegunn/meetko/blob/main/docs/plugin-publishing.md), 격리(샌드박스)는 [docs/plugin-security.md](https://github.com/Eastsidegunn/meetko/blob/main/docs/plugin-security.md)를 보세요. 입력과 출력의 데이터 형식은 meetko의 `contract/schemas/`가 정의합니다. 여기 예제를 복사해서 시작하면 됩니다.

## 테스트

```
python3 -I summary-py/test_summary.py                      # 네트워크 없음 (127.0.0.1의 가짜 서버)
(cd talk-time-rs && cargo test --locked && cargo build --release --locked)
```

`testdata/`는 meetko `contract/`의 회의 예시와 출력 스키마(계약 0.1)를 복사한 것입니다.

## 목록 만들기 (`index.json`)

`python3.11 -I tools/make-index`가 플러그인 폴더의 `meetko.toml`과 `sources.json`(플러그인 폴더 이름 → 게시된 저장소 `repo`, `ref`, 40자 커밋 `sha`)으로 `index.json`을 다시 씁니다. 출처가 없거나 설치 때 거부될 항목(게시자 ≠ 저장소 소유자, 빌드된 실행 파일이 커밋되지 않음)은 이유를 출력하고 뺍니다. `--check`는 파일이 최신인지만 봅니다. 여러 사람의 저장소를 모으는 목록은 meetko의 `tools/plugin-index/build_index.py`(GitHub topic `meetko-plugin` 검색)로 만듭니다.

## English

Example plugins for [meetko](https://github.com/Eastsidegunn/meetko) (one plugin per top-level folder) and the gallery index (`index.json`) the app's browser reads. `action-items-py`: rule-based action items and decisions, no network. `summary-py`: AI summary, decisions and action items; **uses the network** and sends the transcript text to the AI service you configure (`ANTHROPIC_API_KEY`, or `OPENAI_API_KEY` with `OPENAI_BASE_URL`). `talk-time-rs`: talk time per speaker, a Rust binary; build it with `cargo build --release --locked` before adding it. Install from a local folder with `meetko plugin add <folder>`. `meetko plugin install github:OWNER/REPO@REF` installs from the root of a repo whose owner matches the id's publisher, so these sub-folders are not installable that way yet and `index.json` lists nothing; `tools/make-index` regenerates it from `sources.json` once plugins are published. Manifest format: meetko's `docs/manifest-draft.md`; publishing: `docs/plugin-publishing.md`. License: AGPL-3.0-or-later. Copyright (C) 2026 Eastsidegunn.
