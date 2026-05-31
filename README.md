# ruster

ruster is a Windows local translation proxy that connects desktop apps to Gemini WebView, ChatGPT WebView, or Gemini CLI through OpenAI-compatible, Gemini-compatible, and simple Custom API endpoints.

## English Quick Start

### Features

- Desktop GUI with tray mode, logs, runtime status, and usage statistics
- Backend selection between Gemini WebView, ChatGPT WebView, and Gemini CLI
- OpenAI-compatible API: `/v1/chat/completions`, `/v1/responses`, `/v1/completions`, `/v1/models`
- Gemini-compatible API: `/models`, `/v1beta/models`, `/v1beta/models/{model}:generateContent`
- Simple Custom API endpoints: `/`, `/translate`, `/custom/{preset}`
- ivLyrics prompt handling for translation, pronunciation, study, and quiz requests
- Editable prompt configuration saved as `prompts.json`

### Requirements

- Windows 10/11
- Microsoft Edge WebView2 Runtime for WebView backends
- Node.js LTS and `@google/gemini-cli` when using Gemini CLI mode

### Run

1. Download or build `ruster.exe`.
2. Start the app and choose `Gemini WebView`, `Gemini CLI`, or `ChatGPT WebView`.
3. Check the local base URL. The default is `http://localhost:5000`.
4. Copy the local API key from the GUI, or disable local API key authentication for trusted local-only setups.
5. Enable tray mode if you want the backend to keep running without a console window. When started from the GUI, tray mode relaunches the backend as a detached tray process.

Headless examples:

```powershell
.\ruster.exe --headless --mode=webview
.\ruster.exe --headless --mode=chatgpt
.\ruster.exe --headless --mode=cli
```

### API Examples

OpenAI-compatible:

```http
POST http://127.0.0.1:5000/v1/chat/completions
Authorization: Bearer <local-api-key>
Content-Type: application/json
```

Gemini-compatible:

```http
POST http://127.0.0.1:5000/v1beta/models/gemini-2.5-flash:generateContent?key=<local-api-key>
Content-Type: application/json
```

Custom API:

```http
POST http://127.0.0.1:5000/translate
Authorization: Bearer <local-api-key>
Content-Type: application/json
```

```json
{
  "text": "Text to translate",
  "source": "auto",
  "target": "ko"
}
```

The default Custom API response shape is:

```json
{
  "result": "Translated text",
  "errorMessage": "",
  "errorCode": "0"
}
```

### ivLyrics Setup

ruster supports the ivLyrics `OpenAI ChatGPT` and `Google Gemini` addons.

For the `OpenAI ChatGPT` addon:

- API Key(s): the local API key shown in ruster, or any non-empty placeholder if local API key auth is disabled
- Base URL: `http://127.0.0.1:5000/v1`
- Model: select a model from the dropdown, or choose `Custom...` and set `Custom Model ID`

For the `Google Gemini` addon:

- API Key(s): the local API key shown in ruster, or any non-empty placeholder if local API key auth is disabled
- Base URL: `http://127.0.0.1:5000/v1beta`
- Model: select one of the models loaded from ruster

Do not paste a full `/chat/completions` or `/models/{model}:generateContent` URL into the ivLyrics Base URL field. The addons append those paths themselves.

Enable the ivLyrics features you want to route through ruster, then use `Test Connection`. In ruster, enable `ivLyrics study/quiz CLI fast lane` if you want study, quiz, expression, summary, and line-study prompts to go through the CLI-first path.

### Data Location

Settings, prompts, WebView profiles, and usage statistics are stored under `%LOCALAPPDATA%\ruster` by default. To use portable mode, place `ruster.portable` or `settings.json` next to `ruster.exe`; writable data will then be stored beside the executable.

### Libraries Used

- Runtime and HTTP: `tokio`, `axum`, `reqwest`
- GUI and window integration: `eframe`/`egui`, `raw-window-handle`
- Windows integration: `windows`, `webview2-com`
- Serialization and data: `serde`, `serde_json`, `chrono`, `dirs`, `uuid`
- Parsing and utilities: `regex`, `url`, `urlencoding`, `sha2`
- Errors and synchronization: `anyhow`, `thiserror`, `parking_lot`

### 📂 Project Structure

ruster/
├── src/
│   ├── main.rs              # Application entry point and initialization
│   ├── gui.rs               # GUI dashboard and user interface
│   ├── http_server.rs       # axum-based local API server and routing
│   ├── custom_api.rs        # Custom API preset handling
│   ├── gemini_gate.rs       # Gemini / ChatGPT WebView browser automation
│   ├── cli.rs               # Gemini CLI mode and process control
│   ├── ivlyrics.rs          # ivLyrics dedicated prompt adjustment and fast lane
│   ├── logging.rs           # Log recording and usage statistics management
│   └── diagnostics.rs       # System status diagnostics and runtime checks
├── assets/                  # Icons and UI resources
├── Cargo.toml               # Rust project blueprint and dependency configuration
├── build.rs                 # Build script (Windows resource compilation)
└── README.md                # Project documentation

---

### 🔄 System Workflow

1. **Request Reception:** External programs or the GUI send requests to http://localhost:5000
2. **Routing & Analysis:** ruster's internal HTTP server intercepts the request and parses the API format (OpenAI, Gemini, or Custom API)
3. **Backend Dispatch:** The prompt is forwarded to the currently active backend engine (Gemini WebView, Gemini CLI, or ChatGPT WebView) for processing
4. **Format Conversion:** The raw response from the backend is reformatted to match the requested API specification (compatible format)
5. **Result Return:** The final translation data is returned to the external application or user
6. **Post-Processing:** Usage statistics are updated and runtime logs are saved based on the request outcome

---

### 🎯 Development Objectives

ruster consolidates scattered AI service backends into a unified, local proxy environment, allowing desktop applications to effortlessly leverage AI features via standard OpenAI or Gemini API formats.

It focuses heavily on enhancing compatibility and scalability for tools that require specialized prompt manipulation, such as real-time screen translators (MORT), OCR tools, and lyrics translation/study software (ivLyrics), ensuring stable integration within a local ecosystem.

---

### 🚀 Future Roadmap

- **Additional AI Model Support:** Integration with a wider range of open-source and commercial LLM backends
- **Performance Optimization:** Reducing runtime overhead and implementing streaming responses
- **Plugin Extension System:** Introducing an architecture that allows users to inject custom processing logic
- **Expanded API Compatibility:** Continuous updates to align with the latest OpenAI and Gemini specification changes
- **Enhanced Metrics Visualization:** Adding comprehensive charts and analytics views directly into the GUI dashboard
- **Localization:** Implementing i18n support for multi-language user interfaces

  
## 한국어

ruster는 Gemini WebView, Gemini CLI, ChatGPT WebView를 로컬 번역 엔진으로 묶어 주는 Windows용 번역 프록시입니다. 외부 앱에서는 OpenAI 호환 API, Gemini 호환 API, 또는 단순 Custom API 형식으로 `http://localhost:5000`에 요청하면 됩니다.

## 주요 기능

- GUI 대시보드, 트레이 실행, 로그/통계 창
- Gemini WebView, ChatGPT WebView, Gemini CLI 백엔드 선택
- OpenAI 호환 API: `/v1/chat/completions`, `/v1/responses`, `/v1/completions`, `/v1/models`
- Gemini 호환 API: `/models`, `/v1beta/models`, `/v1beta/models/{model}:generateContent`
- Custom API 호환 엔드포인트: `/`, `/translate`, `/custom/{preset}`
- ivLyrics 번역/발음/학습/퀴즈 프롬프트 보정
- GUI에서 프롬프트 직접 수정 및 `prompts.json` 저장

## 설치

### 릴리스 실행 파일 사용

1. 배포된 `ruster.exe`를 원하는 폴더에 둡니다.
2. Windows WebView2 Runtime이 없다면 Microsoft Edge WebView2 Runtime을 설치합니다.
3. Gemini CLI 모드를 쓸 경우 Node.js LTS와 `@google/gemini-cli`가 필요합니다. GUI의 `CLI 초기설정` 버튼으로 설치/로그인 흐름을 실행할 수 있습니다.
4. `ruster.exe`를 실행하고 시작 화면에서 사용할 번역 모드를 선택합니다.

### 소스에서 빌드

```powershell
cargo build --release
.\target\release\ruster.exe
```

디버그 빌드는 아래처럼 실행합니다.

```powershell
cargo build
.\target\debug\ruster.exe
```

### 데이터 저장 위치

기본 설정/프롬프트/통계는 `%LOCALAPPDATA%\ruster`에 저장됩니다. 실행 파일이 있는 폴더에 `ruster.portable` 파일을 만들거나 `settings.json`을 함께 두면, 쓰기 권한이 있는 경우 포터블 모드로 같은 폴더에 데이터를 저장합니다.

## 초기 설정

1. 시작 화면에서 `Gemini WebView`, `Gemini CLI`, `ChatGPT WebView` 중 하나를 선택합니다.
2. `서버 / 프록시`에서 `Base URL`을 확인합니다. 기본값은 `http://localhost:5000`입니다.
3. 외부 앱에서 호출할 경우 `로컬 API 키`를 복사합니다. 인증을 끄려면 `로컬 API 키 인증 요구`를 비활성화합니다.
4. ivLyrics를 쓸 경우 `프롬프트 / ivLyrics`에서 학습/퀴즈 CLI fast lane과 프롬프트 내용을 조정합니다.
5. 트레이 모드가 필요하면 `트레이 모드로 실행`을 켭니다. GUI에서 시작할 때는 콘솔 없는 트레이 백엔드로 재실행되며, 로그/종료는 트레이 메뉴에서 처리합니다.

헤드리스 실행 예시는 아래와 같습니다.

```powershell
.\ruster.exe --headless --mode=webview
.\ruster.exe --headless --mode=chatgpt
.\ruster.exe --headless --mode=cli
```

## 로컬 API

인증이 켜져 있으면 다음 중 하나로 로컬 API 키를 전달합니다.

- `Authorization: Bearer <local-api-key>`
- `x-api-key: <local-api-key>`
- `api-key: <local-api-key>`
- `x-goog-api-key: <local-api-key>`
- 쿼리 문자열 `?key=<local-api-key>` 또는 `?api_key=<local-api-key>`

### OpenAI 호환 호출

```http
POST http://127.0.0.1:5000/v1/chat/completions
Authorization: Bearer <local-api-key>
Content-Type: application/json
```

```json
{
  "model": "gpt-4o-mini",
  "messages": [
    { "role": "user", "content": "Translate to Korean: hello" }
  ]
}
```

모델명은 외부 앱 호환용으로 받으며, 실제 처리 백엔드는 ruster에서 선택한 모드와 설정을 따릅니다.

### Gemini 호환 호출

```http
POST http://127.0.0.1:5000/v1beta/models/gemini-2.5-flash:generateContent?key=<local-api-key>
Content-Type: application/json
```

```json
{
  "contents": [
    {
      "role": "user",
      "parts": [{ "text": "Translate to Korean: hello" }]
    }
  ]
}
```

### Custom API 호환 호출

```http
POST http://127.0.0.1:5000/translate
Authorization: Bearer <local-api-key>
Content-Type: application/json
```

```json
{
  "text": "翻訳する文章",
  "source": "ja",
  "target": "ko"
}
```

응답은 기본적으로 아래 형식입니다.

```json
{
  "result": "번역 결과",
  "errorMessage": "",
  "errorCode": "0"
}
```

요청 본문은 `text`, `prompt`, `q`, `input` 필드를 우선 사용합니다. 일반 문자열 본문, OpenAI `messages`, Gemini `contents.parts.text` 형태도 추출합니다.

## ivLyrics 적용법

ivLyrics 코드 기준으로는 `OpenAI ChatGPT` 애드온과 `Google Gemini` 애드온 둘 다 ruster에 연결할 수 있습니다. 둘 다 `API Key(s)`는 필수 입력이라, ruster의 `로컬 API 키 인증 요구`를 꺼 둔 경우에도 `localhost` 같은 비어 있지 않은 값을 넣어야 합니다. 인증을 켜 둔 경우에는 ruster 화면의 `로컬 API 키`를 그대로 넣습니다.

### OpenAI ChatGPT 애드온

이 애드온은 `Base URL` 뒤에 `/chat/completions`를 붙여 요청하고, 모델 목록은 `/models`에서 가져옵니다. 따라서 `Base URL`에는 `/v1`까지만 넣습니다.

```text
Provider: OpenAI ChatGPT
API Key(s): <ruster 로컬 API 키 또는 localhost>
Base URL: http://localhost:5000/v1
Model: <목록에서 선택> 또는 Custom...
Custom Model ID: gpt-4o-mini
```

`Custom...`을 선택했다면 `Custom Model ID`를 비워 두면 안 됩니다. `gpt-4o-mini`, `gemini-3-flash-preview`, `gemini-2.5-flash`처럼 ruster가 받을 모델명을 넣습니다.

### Google Gemini 애드온

이 애드온은 `Base URL` 뒤에 `/models?key=...`로 모델 목록을 읽고, 실제 요청은 `/models/{model}:generateContent?key=...`로 보냅니다. 따라서 `Base URL`에는 `/v1beta`까지만 넣습니다.

```text
Provider: Google Gemini
API Key(s): <ruster 로컬 API 키 또는 localhost>
Base URL: http://localhost:5000/v1beta
Model: <목록에서 선택>
```

`Base URL`에 `.../models/gemini-...:generateContent` 전체 주소를 넣으면 안 됩니다. ivLyrics Gemini 애드온이 그 경로를 직접 붙입니다.

공통으로 필요한 기능 버튼을 켠 뒤 `Test Connection`을 눌러 확인합니다. ruster의 `프롬프트 / ivLyrics`에서 `ivLyrics 학습/퀴즈 CLI fast lane`을 켜면 학습, 퀴즈, 표현, 요약, 라인 학습 요청은 CLI 우선 경로로 처리됩니다.

## MORT Custom API 적용법

MORT의 기본 Custom API 모드는 아래 JSON을 `POST`로 보냅니다.

```json
{
  "name": "<source><target>",
  "text": "<OCR text>",
  "target": "<target language code>",
  "source": "<source language code>"
}
```

ruster의 `/translate`는 이 형식을 그대로 받을 수 있습니다. MORT Custom API URL에는 아래처럼 넣습니다.

```text
http://127.0.0.1:5000/translate?key=<ruster 로컬 API 키>
```

ruster에서 `로컬 API 키 인증 요구`를 꺼 둔 경우에는 `?key=...`를 빼도 됩니다. MORT의 기본 Custom API 모드는 별도 헤더 입력 없이 `Content-Type: application/json`을 붙여 요청합니다.

MORT 1.310 이상의 Custom API 프리셋을 쓰는 경우에는 아래처럼 넣습니다.

- Url: `http://127.0.0.1:5000/translate?key=<ruster 로컬 API 키>`
- Headers: 비워 둠
- Request:

```text
"text": "{OCR_TEXT}",
"source": "{SOURCE_CODE}",
"target": "{RESULT_CODE}"
```

- Response:

```text
"result": {RESULT_TEXT}
```

MORT 프리셋의 `Request`는 바깥 `{}`를 생략해도 되고, `{OCR_TEXT}`, `{SOURCE_CODE}`, `{RESULT_CODE}`는 MORT가 치환합니다. `Response`는 `{RESULT_TEXT}`가 붙은 키 이름을 실제 응답 JSON에서 찾아 번역 결과로 사용합니다.

### ruster Custom API 프리셋

`/custom/{preset}` 경로를 쓰면 데이터 폴더의 `CustomApi` 디렉터리에 있는 프리셋을 적용합니다.

기본 위치:

```text
%LOCALAPPDATA%\ruster\CustomApi
```

포터블 모드:

```text
<ruster.exe 폴더>\CustomApi
```

예시 파일: `CustomApi\game-ja-ko.json`

```json
{
  "Name": "game-ja-ko",
  "Mode": "translate",
  "RequestTemplate": "다음 일본어 게임 대사를 자연스러운 한국어로 번역해.\n\n{OCR_TEXT}",
  "ResponseTemplate": "\"result\":\"{RESULT_TEXT}\",\"errorMessage\":\"\",\"errorCode\":\"0\"",
  "TimeoutSeconds": 60
}
```

호출 URL:

```text
http://127.0.0.1:5000/custom/game-ja-ko
```

`Mode` 값은 `translate` 또는 `raw`를 사용할 수 있습니다. `translate`는 ruster의 번역 래핑을 적용하고, `raw`는 `RequestTemplate`으로 만든 프롬프트를 그대로 백엔드에 보냅니다. 프리셋 목록은 `GET /custom/presets`로 확인할 수 있습니다.

현재 로컬 수신 프리셋에서 실제로 영향을 주는 필드는 `Name`, `RequestTemplate`, `ResponseTemplate`, `Mode`, `TimeoutSeconds`입니다. `Url`, `Method`, `Headers`, `ResultPath` 필드는 기존 설정 파일 호환을 위해 읽을 수 있지만, 이 경로에서 외부 API로 재전송하는 용도로 사용하지 않습니다.

### 📂 프로젝트 구조

ruster/
├── src/
│   ├── main.rs              # 프로그램 진입점 및 초기화
│   ├── gui.rs               # GUI 대시보드 및 사용자 인터페이스
│   ├── http_server.rs       # axum 기반 로컬 API 서버 및 라우팅
│   ├── custom_api.rs        # 사용자 정의 프리셋(Custom API) 처리
│   ├── gemini_gate.rs       # Gemini / ChatGPT WebView 브라우저 자동화
│   ├── cli.rs               # Gemini CLI 모드 및 프로세스 제어
│   ├── ivlyrics.rs          # ivLyrics 전용 프롬프트 보정 및 가속 패스
│   ├── logging.rs           # 로그 기록 및 사용 통계 관리
│   └── diagnostics.rs       # 시스템 상태 진단 및 런타임 검사
├── assets/                  # 아이콘 및 UI 리소스
├── Cargo.toml               # Rust 프로젝트 매니페스트 및 의존성 설정
├── build.rs                 # 빌드 스크립트 (Windows 리소스 컴파일)
└── README.md                # 프로젝트 문서

---

### 🔄 시스템 동작 흐름

1. **요청 수신:** 외부 프로그램 또는 GUI에서 http://localhost:5000 으로 요청 전송
2. **라우팅 및 분석:** ruster 내부 HTTP 서버가 요청을 받아 API 형식(OpenAI, Gemini, Custom API)을 분석
3. **백엔드 전달:** 활성화된 백엔드 엔진(Gemini WebView, Gemini CLI, ChatGPT WebView)으로 프롬프트 전달 및 처리
4. **포맷 변환:** 백엔드로부터 받은 결과를 요청된 API 규격(호환 형식)에 맞게 변환
5. **결과 반환:** 최종 번역 데이터를 외부 프로그램 또는 사용자에게 응답
6. **사후 처리:** 요청 처리 결과를 기반으로 사용 통계 갱신 및 로그 저장

---

### 🎯 개발 목적

ruster는 파편화된 다양한 AI 서비스 백엔드를 하나의 로컬 프록시 환경으로 통합하여, 외부 데스크톱 애플리케이션들이 OpenAI API 또는 Gemini API 형식으로 AI 기능을 쉽게 활용할 수 있도록 돕습니다. 

특히 실시간 화면 번역기(MORT), OCR 프로그램, 가사 번역/학습 도구(ivLyrics) 등 고유의 프롬프트 제어가 필요한 서비스들과의 연동 편의성을 극대화하고, 로컬 환경에서의 호환성과 확장성을 확보하는 데 중점을 두었습니다.

---

### 🚀 향후 개선 사항

- **추가 AI 모델 지원:** 더 다양한 오픈소스 및 커머셜 LLM 백엔드 연동
- **응답 속도 최적화:** 런타임 오버헤드 단축 및 스트리밍 응답 개선
- **플러그인 확장 기능 제공:** 사용자 정의 가공 로직을 직접 주입할 수 있는 구조 도입
- **API 호환성 확대:** 최신 OpenAI/Gemini 규격 변경사항 지속 반영
- **사용 통계 시각화 강화:** GUI 대시보드 내 대용량 통계 그래프 및 분석 뷰 추가
- **다국어 UI 지원:** 글로벌 사용자를 위한 i18n 환경 구성
