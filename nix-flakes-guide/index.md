# Nix Flakes로 재현 가능한 개발 환경 만들기

"내 컴퓨터에서는 되는데?"라는 문제를 근본적으로 해결하는 방법이 있다. Nix Flakes를 사용하면 개발 환경의 모든 의존성을 해시로 고정하여 100% 재현 가능한 빌드를 달성할 수 있다.

## Nix와 Flakes 소개

Nix는 순수 함수형 패키지 매니저다. 모든 패키지가 입력(소스, 의존성)의 해시로 고유하게 식별되므로, 같은 입력이면 항상 같은 결과가 나온다.

Flakes는 Nix 프로젝트의 표준 구조를 정의하는 최신 인터페이스다. `flake.nix`와 `flake.lock` 파일로 프로젝트의 입력과 출력을 명시한다.

### Nix 설치

```bash
# 공식 설치 스크립트 (멀티유저 설치)
sh <(curl -L https://nixos.org/nix/install) --daemon

# Flakes 활성화
mkdir -p ~/.config/nix
echo "experimental-features = nix-command flakes" >> ~/.config/nix/nix.conf

# 설치 확인
nix --version
# nix (Nix) 2.24.0
```

Arch Linux에서는 `pacman -S nix`로도 설치할 수 있지만, 공식 설치 스크립트를 권장한다. 데몬 모드로 설치해야 빌드 샌드박싱과 멀티유저 지원이 가능하다.

## flake.nix 작성

프로젝트 루트에 `flake.nix`를 작성한다. 기본 구조는 `inputs`(의존성)와 `outputs`(결과물)로 나뉜다.

```nix
{
  description = "My Rust project";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    rust-overlay.url = "github:oxalica/rust-overlay";
    flake-utils.url = "github:numtide/flake-utils";
  };

  outputs = { self, nixpkgs, rust-overlay, flake-utils }:
    flake-utils.lib.eachDefaultSystem (system:
      let
        overlays = [ (import rust-overlay) ];
        pkgs = import nixpkgs { inherit system overlays; };
        rustToolchain = pkgs.rust-bin.stable.latest.default.override {
          extensions = [ "rust-src" "rust-analyzer" ];
        };
      in {
        devShells.default = pkgs.mkShell {
          buildInputs = [
            rustToolchain
            pkgs.pkg-config
            pkgs.openssl
          ];

          shellHook = ''
            echo "Rust $(rustc --version) ready"
          '';
        };
      }
    );
}
```

`nix develop` 명령을 실행하면 이 `devShell`에 정의된 모든 도구가 PATH에 추가된 셸이 열린다.

### flake.lock과 재현성

처음 `nix develop`을 실행하면 `flake.lock`이 생성된다. 이 파일에는 모든 입력의 정확한 커밋 해시와 NAR 해시가 기록된다.

```bash
# 현재 고정된 버전 확인
nix flake metadata
# └───nixpkgs: github:NixOS/nixpkgs/abc1234...

# 입력 업데이트
nix flake update
# 또는 특정 입력만 업데이트
nix flake update nixpkgs
```

`flake.lock`을 VCS에 커밋하면, 모든 팀원과 CI가 동일한 버전의 도구를 사용하게 된다.

## devShell 활용 패턴

### 언어별 개발 환경

```nix
# Python 프로젝트
devShells.default = pkgs.mkShell {
  buildInputs = [
    (pkgs.python312.withPackages (ps: [
      ps.fastapi
      ps.uvicorn
      ps.sqlalchemy
      ps.pytest
    ]))
    pkgs.postgresql
    pkgs.redis
  ];
};
```

Python 패키지를 `withPackages`로 관리하면 `pip install`없이 즉시 사용할 수 있다. 시스템 라이브러리(PostgreSQL, Redis)도 함께 제공되므로, 외부 설치가 필요 없다.

### direnv 통합

`direnv`와 `nix-direnv`를 사용하면 디렉토리 진입 시 자동으로 Nix 환경이 로드된다.

```bash
# .envrc
use flake

# direnv 허용
direnv allow
```

에디터에서 프로젝트를 열 때도 자동으로 올바른 도구 체인이 활성화된다. VS Code의 `direnv` 확장이나 네오빔의 `direnv.vim`과 함께 사용하면 완벽하다.

## 패키지 빌드

Flakes로 프로젝트 자체를 Nix 패키지로 빌드할 수도 있다. `packages` 출력을 정의하면 된다.

```nix
packages.default = pkgs.rustPlatform.buildRustPackage {
  pname = "my-app";
  version = "0.1.0";
  src = ./.;
  cargoLock.lockFile = ./Cargo.lock;

  nativeBuildInputs = [ pkgs.pkg-config ];
  buildInputs = [ pkgs.openssl ];
};
```

`nix build`로 빌드하면 `result/bin/my-app`에 바이너리가 생성된다. 이 빌드는 샌드박스 안에서 수행되므로, 호스트 시스템의 상태에 전혀 의존하지 않는다.

### Docker 이미지 생성

Nix로 빌드한 패키지를 Docker 이미지로 감쌀 수도 있다. Dockerfile 없이, 필요한 런타임 의존성만 포함하는 최소 이미지를 만들 수 있다.

```nix
packages.docker = pkgs.dockerTools.buildImage {
  name = "my-app";
  tag = "latest";
  copyToRoot = [ self.packages.${system}.default ];
  config.Cmd = [ "/bin/my-app" ];
};
```

결과 이미지에는 Nix 스토어의 필요한 클로저만 포함되어, 일반 Docker 빌드보다 훨씬 작은 이미지가 만들어진다.
