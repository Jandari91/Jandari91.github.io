---
title: Rust 설치 및 VS Code 셋팅(Linux, Debian)
author: Jandari
date: 2023-01-06 22:00:00 +09:00
categories: [Rust, 설치]
tags: [Rust, Linux, VSCode, 설치]
math: true
mermaid: true
---

# 소개

최근 Rust에 관심이 생겨 Rust를 공부하고 있습니다.

제가 집에서 사용하는 PC는 Mint Linux로 Debian 계열의 리눅스를 사용하고 있습니다.

Linux에 Rust를 설치하는 방법과 VS Code에서 Rust를 사용하는 방법에 대해 정리하겠습니다.

## Rust 설치

리눅스에서 Rust를 설치하는 방법은 3가지가 있습니다.

1. rustup 사용 : 공식 사이트에서 가장 추천하는 방법 입니다.
1. 패키지 매니저 사용 : 편리하지만 오래 된 버전의 Rust가 설치 됩니다.
1. 직접 소스코드 빌드 : 특이한 경우가 아니라면 비추입니다.

저는 이 중 가장 추천하는 rustup을 사용하여 설치하겠습니다.

### Rustup으로 설치하기

먼저 터미널을 실행 시킵니다.(Ctrl + Alt + T)

아래 명령어를 사용합니다.

```
bak:~$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

info: downloading installer

Welcome to Rust!

This will download and install the official compiler for the Rust
programming language, and its package manager, Cargo.

Rustup metadata and toolchains will be installed into the Rustup
home directory, located at:

  /home/bak/.rustup

This can be modified with the RUSTUP_HOME environment variable.

The Cargo home directory is located at:

  /home/bak/.cargo

This can be modified with the CARGO_HOME environment variable.

The cargo, rustc, rustup and other commands will be added to
Cargo's bin directory, located at:

  /home/bak/.cargo/bin

This path will then be added to your PATH environment variable by
modifying the profile files located at:

  /home/bak/.profile
  /home/bak/.bashrc

You can uninstall at any time with rustup self uninstall and
these changes will be reverted.

Current installation options:


   default host triple: x86_64-unknown-linux-gnu
     default toolchain: stable (default)
               profile: default
  modify PATH variable: yes

1) Proceed with installation (default)
2) Customize installation
3) Cancel installation
1
```

위와 같은 글씨가 출력되면 1 을 입력한다.

```
info: profile set to 'default'
info: default host triple is x86_64-unknown-linux-gnu
info: syncing channel updates for 'stable-x86_64-unknown-linux-gnu'
info: latest update on 2022-12-15, rust version 1.66.0 (69f9c33d7 2022-12-12)
info: downloading component 'cargo'
info: downloading component 'clippy'
info: downloading component 'rust-docs'
info: downloading component 'rust-std'
info: downloading component 'rustc'
 68.0 MiB /  68.0 MiB (100 %)  55.3 MiB/s in  1s ETA:  0s
info: downloading component 'rustfmt'|-|-|
info: installing component 'rust-std'
 29.7 MiB /  29.7 MiB (100 %)  14.0 MiB/s in  2s ETA:  0s
info: installing component 'rustc'
 68.0 MiB /  68.0 MiB (100 %)  15.5 MiB/s in  4s ETA:  0s
info: installing component 'rustfmt'
info: default toolchain set to 'stable-x86_64-unknown-linux-gnu'

  stable-x86_64-unknown-linux-gnu installed - rustc 1.66.0 (69f9c33d7 2022-12-12)


Rust is installed now. Great!

To get started you may need to restart your current shell.
This would reload your PATH environment variable to include
Cargo's bin directory ($HOME/.cargo/bin).

To configure your current shell, run:
source "$HOME/.cargo/env"
```

위와 같이 cargo나 rustc 등등 필요한 패키지들이 설치 된다.

위 이야기를 보면 최근 shell을 다시 시작하라고 나온다. 그래서 설치한 터미널에서는 `rustc`를 입력해도 찾을수 없다고 나온다.

```
bak:~$ rustc --version
명령어 'rustc' 을(를) 찾을 수 없습니다. 그러나 다음을 통해 설치할 수 있습니다:
sudo apt install rustc
```

새로운 터미널을 열어 다시 입력하면 아래와 같이 정상적으로 출력되는 것을 알 수 있다.

```
bak:~$ rustc --version
rustc 1.66.0 (69f9c33d7 2022-12-12)
```

rust는 아래와 같이 세 가지 명령행 도구가 있습니다.

| 도구   | 설명                                    |
| ------ | --------------------------------------- |
| cargo  | 전체 크레이트를 관리합니다.             |
| rustup | 러스트 설치를 관리합니다.               |
| rustc  | 러스트 소스 코드의 컴파일을 관리합니다. |

### Rust 완전 삭제 방법

삭제하는 방법은 간단 한데 아래와 같이 명령어만 입력하면 환경변수까지 모두 삭제가 된다.

```
bak:~$ rustup self uninstall
```

## VS Code에서 Rust 디버깅

VS Code에서 Rust를 개발하기 위해서는 아래와 같이 세 가지 extension을 설치해주면 좋습니다.

- [VS code Install Rust](https://code.visualstudio.com/docs/languages/rust#_installation)

| Extension     | 설명                                       |
| ------------- | ------------------------------------------ |
| rust-analyzer | rust의 자동완성, 구문강조 등 어시스트 기능 |
| CodeLLDB      | 디버깅을 하기 위해 필요합니다.             |
| Better TOML   | Rust의 설정 파일인 TOML 파일을 지원합니다. |

### hello-world 만들기

rust는 cargo를 사용해서 프로젝트를 생성합니다.

```
bak:~$ cargo new hello_world
Created binary (application) `hello_world` package
```

`VS Code로 rust를 실행시키기 위해서는 프로젝트가 VS code의 root 디렉터리가 되어야 합니다.`

그래서 아래와 같이 해당 프로젝트로 들어가 VS Code를 실행시켜야 합니다.

```
bak:~$ code hello_world
```

VS Code를 실행시키면 아래와 같이 `src` 디렉터리에 `main.rs` 파일이 있는 것을 볼 수 있습니다.

![image](/assets/img/post/2023-12-06-Rust_Install/1.jpg)

### 디버깅

2번째 라인에 Break Point가 찍히는 것을 볼 수 있습니다.

![image](/assets/img/post/2023-12-06-Rust_Install/2.jpg)

여기서 F5를 누르면 launch.json이 셋팅되지 않았다고 하며, 파일을 생성합니다.

![image](/assets/img/post/2023-12-06-Rust_Install/3.jpg)

![image](/assets/img/post/2023-12-06-Rust_Install/4.jpg)

`중요한 것을 지금 launch.json을 만들어도 F5로는 디버깅이 되지 않습니다.`

왜냐하면 일반 계정의 vscode에서 cargo를 사용하기 위한 셋팅이 되지 않았기 때문입니다.

터미널을 띄워 아래와 같이 입력합니다.

```
bak:~$ sudo ln -s /home/your_user_name/.cargo/bin/cargo /bin/cargo
```

저의 사용자 계정은 bak이기 때문에 `your_user_name`에 bak을 입력하였습니다.

이제 다시 F5를 눌러 launch.json을 만들게 되면 아래와 같이 만들어집니다.

```
{
    // IntelliSense를 사용하여 가능한 특성에 대해 알아보세요.
    // 기존 특성에 대한 설명을 보려면 가리킵니다.
    // 자세한 내용을 보려면 https://go.microsoft.com/fwlink/?linkid=830387을(를) 방문하세요.
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "Debug executable 'hello_world'",
            "cargo": {
                "args": [
                    "build",
                    "--bin=hello_world",
                    "--package=hello_world"
                ],
                "filter": {
                    "name": "hello_world",
                    "kind": "bin"
                }
            },
            "args": [],
            "cwd": "${workspaceFolder}"
        },
        {
            "type": "lldb",
            "request": "launch",
            "name": "Debug unit tests in executable 'hello_world'",
            "cargo": {
                "args": [
                    "test",
                    "--no-run",
                    "--bin=hello_world",
                    "--package=hello_world"
                ],
                "filter": {
                    "name": "hello_world",
                    "kind": "bin"
                }
            },
            "args": [],
            "cwd": "${workspaceFolder}"
        }
    ]
}
```

## 참고

- https://github.com/rust-lang/vscode-rust/issues/708
- https://rust-kr.org/pages/install/
