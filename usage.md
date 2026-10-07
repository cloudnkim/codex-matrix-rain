# 적용·빌드·실행

## 적용 기준

- OpenAI Codex `rust-v0.160.1`에 적용한다.
- 대화 출력 영역의 비어 있는 셀에 코드비를 표시한다.
- 입력창, 출력 텍스트, 한글·이모지, 선택 영역을 보존한다.
- `CODEX_MATRIX_RAIN=1`과 TUI 애니메이션이 켜져 있어야 한다.
- TrueColor 또는 ANSI 256색 터미널을 사용한다.
- `--no-alt-screen`에서는 코드비를 표시하지 않는다.
- 새 버전은 [공식 소스 포크](https://github.com/cloudnkim/codex)의 `matrix` 브랜치에서 관리한다.
- 이 저장소의 `0.160.1` 패치와 소스는 최초 버전 기록으로 보존한다.

## 소스 받기

이 폴더에서 실행한다.

```sh
git clone --branch matrix https://github.com/cloudnkim/codex.git .build/codex
```

## 빌드

해당 소스의 `rust-toolchain.toml`에 고정된 Rust 툴체인을 사용한다.

```sh
export PATH="$HOME/.cargo/bin:$PATH"
cd .build/codex/codex-rs
cargo build --locked --release -p codex-cli --bin codex
CODEX_MATRIX_RAIN=1 ./target/release/codex
```

새 공식 릴리즈의 macOS Apple Silicon 패키지는 [포크 릴리즈](https://github.com/cloudnkim/codex/releases)에 게시한다.
패키지의 `bin/codex-matrix`로 효과를 켜고 기존 CLI 인자를 전달한다.
다른 Apple Silicon 맥에서는 압축을 풀고 `sh ./install.command`로 설치한다.
설치한 `~/.codex/bin/codex-matrix-X.Y.Z`를 실행한다.
설치에는 Rust·Node.js·Python이 필요하지 않고 기존 Codex 설정·인증은 보존한다.
예약·병합·실패 처리는 [포크의 자동 업데이트 문서](https://github.com/cloudnkim/codex/blob/matrix/matrix-rain.md)를 따른다.

## 검증

CLI 버전이 포크의 `matrix-upstream-version.txt`와 일치하는지 확인한다.

```sh
./target/release/codex --version
```

효과 켜짐·꺼짐, 애니메이션 꺼짐, 일반 터미널 모드, 기본 모드를 확인한다.
한글·이모지 입력, 커서 위치, 여러 줄 입력, 창 크기 변경, 명령 팝업을 확인한다.
검증 중에는 프롬프트를 전송하지 않는다.

## 보관

포크 소스와 로컬 빌드 산출물은 `.build/`에 둔다.
최초 버전의 패치·코드비 소스·라이선스는 이 저장소에 보존한다.
