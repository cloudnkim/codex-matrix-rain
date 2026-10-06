# 검증 결과

- 검증 시각: 2026-10-06 10:08 KST.
- 기준 소스: OpenAI Codex `rust-v0.160.1` (`d27764b82`).
- 실행 환경: macOS Apple Silicon.

| 항목 | 실제 결과 |
| --- | --- |
| 코드비·텍스트 렌더링 테스트 | `just test -p codex-tui -E 'test(matrix_rain) \| test(transcript_view::text::)'`: 39개 통과, 종료 코드 0 |
| CLI 릴리스 빌드 | `cargo build -p codex-cli --bin codex --release`: 종료 코드 0, 18분 34초 |
| 독립 패키지 조립 | `0.160.1-matrix-aarch64-apple-darwin`: 종료 코드 0 |
| 설치 명령 | `codex-sol --version`: `codex-cli 0.160.1`, 종료 코드 0 |
| 실제 PTY UI 검증 | `verify/validate-output-ui.py`: 5조건 통과, 종료 코드 0 |
| 소스 보관 | Rust 파일 일치, 내보낸 패치 적용 검사 통과 |
| Rust 포맷 | `just fmt`의 Rust 단계 통과 |
| 전체 포맷 | Python·Bazel 단계는 `uv`·`dotslash` 미설치로 실패 |
| Windows | 소스·호출 경로 정적 검토 완료, Windows 빌드·실행 미검증 |

UI 검증은 효과 꺼짐, 효과 켜짐, 애니메이션 꺼짐, `--no-alt-screen`, 기본 화면 모드를 확인했다.
효과가 켜진 두 화면 모드에서 서로 다른 6프레임을 확인했다.
한글·이모지 입력, 커서 위치, 여러 줄 입력, 창 크기 변경, 명령 팝업을 확인했다.
검증 중에는 프롬프트를 전송하지 않았다.
