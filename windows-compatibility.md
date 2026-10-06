# Windows CLI 적용 검토

## 판단

Windows Terminal에서 실행하는 Codex CLI에 같은 코드비 효과를 적용할 수 있는 구조다.
이 판단은 `rust-v0.160.1`의 실제 렌더 호출부와 Windows 터미널 처리 코드에 근거한다.
Windows용 빌드와 실제 콘솔 실행은 아직 검증하지 않았다.
Codex CLI의 Windows 실행 지원은 [OpenAI 공식 문서](https://learn.chatgpt.com/docs/windows/windows-sandbox)에 명시돼 있다.

## 실제 호출 경로

1. `tui/src/lib.rs`가 `tui.fullscreen_transcript`와 alternate screen 설정으로 화면 모드를 결정한다.
2. `tui/src/transcript_mode.rs`의 `TranscriptMode::resolve`는 플랫폼 분기 없이 화면 모드를 선택한다.
3. `tui/src/app.rs`는 Codex가 화면 전체를 관리하는 모드에서 `render_owned_transcript`를 호출한다.
4. `tui/src/app/owned_transcript.rs`가 코드비 활성화 여부를 확인하고 `MatrixRain::render`를 호출한다.
5. `matrix_rain.rs`는 표준 Rust 시간·환경 변수 API와 Ratatui 버퍼에 녹색 문자를 그린다.

추가한 코드비 소스와 렌더 호출부에는 Mac 전용 조건부 컴파일 분기가 없다.
upstream의 `tui/src/tui/alternate_screen.rs`에는 Windows용 화면·입력 복원 처리와 기존 Windows 테스트가 있다.

## Windows 색상 처리 근거

`tui/src/terminal_palette.rs`는 `WT_SESSION`과 `TerminalName::WindowsTerminal`을 확인한다.
색상 강제 설정이 없으면 Windows Terminal을 TrueColor로 처리한다.

```rust
if has_wt_session && !has_force_color_override {
    return StdoutColorLevel::TrueColor;
}
```

코드비는 TrueColor 또는 ANSI 256색에서 활성화한다.
16색으로 강제하거나 색상 감지가 실패하면 코드비를 표시하지 않는다.
애니메이션 꺼짐, fullscreen transcript 꺼짐, `--no-alt-screen`도 효과 표시 조건에서 제외한다.

## 현재 Mac에 의존하는 부분

| 항목 | 현재 상태 | Windows에서 필요한 작업 |
| --- | --- | --- |
| 설치 실행 파일 | `aarch64-apple-darwin` 빌드 | Windows 호스트의 MSVC 타깃으로 별도 빌드 |
| `codex-sol.command` | zsh로 실행 | PowerShell 또는 CMD 런처로 연결 |
| `codex-matrix` | sh와 Mac 패키지 경로 사용 | `codex.exe` 경로와 `CODEX_MATRIX_RAIN=1` 설정 |
| `verify/validate-output-ui.py` | `pty`, `fcntl`, `termios`, Unix 프로세스 그룹 사용 | ConPTY 검증 도구 또는 Windows Terminal 실제 확인 |
| 코드비 Rust 소스·TUI 패치 | 공통 렌더 버퍼 사용 | 같은 파일과 패치 적용 후 Windows 컴파일 확인 |

## Windows 실행 절차 초안

아래 명령은 Windows에서 아직 실행하지 않은 적용·빌드 예시다.
이 폴더를 Windows로 옮긴 뒤 Rust MSVC 툴체인과 소스가 요구하는 빌드 도구를 준비한다.
Windows Terminal의 PowerShell에서 실행한다.

```powershell
git clone --depth 1 --branch rust-v0.160.1 https://github.com/openai/codex.git .build/codex
git -C .build/codex apply ../../patches/codex-0.160.1.patch
Copy-Item src/matrix_rain.rs, src/matrix_rain_tests.rs .build/codex/codex-rs/tui/src/
Push-Location .build/codex/codex-rs
cargo build -p codex-cli --bin codex --release
$env:CODEX_MATRIX_RAIN = "1"
.\target\release\codex.exe -c tui.animations=true -c tui.fullscreen_transcript=true -c tui.alternate_screen=always
Pop-Location
```

## Windows 확인 조건

- `CODEX_MATRIX_RAIN=1`에서 출력의 빈 영역에 움직이는 녹색 문자가 표시되는지 확인한다.
- `CODEX_MATRIX_RAIN=0`과 `tui.animations=false`에서는 효과가 없는지 확인한다.
- `--no-alt-screen`에서는 효과가 없는지 확인한다.
- 한글·이모지·공백·탭 출력과 입력, 커서 위치, 여러 줄 입력을 확인한다.
- 창 축소·복원, 선택·복사, 명령 팝업, 종료 후 콘솔 복원을 확인한다.
- Matrix와 텍스트 렌더링의 기존 테스트를 Windows에서도 실행한다.
- 실제 콘솔 검증이 통과한 뒤 Windows 지원을 완료로 표시한다.
