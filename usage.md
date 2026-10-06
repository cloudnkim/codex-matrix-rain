# 적용·빌드·실행

## 적용 기준

- OpenAI Codex `rust-v0.160.1`에 적용한다.
- 대화 출력 영역의 비어 있는 셀에 코드비를 표시한다.
- 입력창, 출력 텍스트, 한글·이모지, 선택 영역을 보존한다.
- `CODEX_MATRIX_RAIN=1`과 TUI 애니메이션이 켜져 있어야 한다.
- TrueColor 또는 ANSI 256색 터미널을 사용한다.
- `--no-alt-screen`에서는 코드비를 표시하지 않는다.
- `patches/codex-0.160.1.patch`와 `src/`의 두 Rust 파일을 함께 적용한다.
- 새 Codex 버전에서는 해당 태그의 렌더 호출부와 패치 적용 여부를 확인한다.

## 소스 적용

이 폴더에서 실행한다.

```sh
git clone --depth 1 --branch rust-v0.160.1 https://github.com/openai/codex.git .build/codex
git -C .build/codex apply ../../patches/codex-0.160.1.patch
cp src/matrix_rain.rs src/matrix_rain_tests.rs .build/codex/codex-rs/tui/src/
```

## 빌드

Codex 소스의 Rust `1.95.0` 툴체인을 사용한다.

```sh
export PATH="$HOME/.cargo/bin:$PATH"
cd .build/codex/codex-rs
cargo build -p codex-cli --bin codex --release
CODEX_MATRIX_RAIN=1 ./target/release/codex
```

독립 실행 패키지가 필요하면 Codex의 `just assemble-codex-package`에 빌드한 CLI와 해당 버전의 보조 실행 파일을 전달한다.

## 검증

Codex 소스 루트에서 관련 기존 테스트를 실행한다.

```sh
just test -p codex-tui -E 'test(matrix_rain) | test(transcript_view::text::)'
```

이 폴더에서 `pyte`가 설치된 Python으로 실제 터미널 검증을 실행한다.

```sh
python verify/validate-output-ui.py .build/codex/codex-rs/target/release/codex
```

효과 켜짐·꺼짐, 애니메이션 꺼짐, 일반 터미널 모드, 기본 모드를 확인한다.
한글·이모지 입력, 커서 위치, 여러 줄 입력, 창 크기 변경, 명령 팝업을 확인한다.
검증 중에는 프롬프트를 전송하지 않는다.

## 보관

원본 Codex 소스와 빌드 산출물은 `.build/`에 둔다.
버전별 패치와 코드비 소스, 검증 도구, 라이선스를 함께 보관한다.
