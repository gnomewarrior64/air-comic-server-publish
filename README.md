# Air Comic Server

Android 만화 뷰어 앱에서 PC의 만화 파일을 읽을 수 있도록 제공하는 이미지 서버입니다. 공유할 폴더를 지정하면 폴더 안의 이미지와 압축파일 내부 이미지를 앱에서 탐색하고 볼 수 있습니다.

**[최신 버전 다운로드](https://github.com/gnomewarrior64/air-comic-server-publish/releases/latest)** · [전체 릴리스](https://github.com/gnomewarrior64/air-comic-server-publish/releases)

## 1. 다운로드

Releases의 **Assets**에서 운영체제와 CPU에 맞는 파일을 내려받고 압축을 풉니다. `Source code` 파일은 실행 프로그램이 아닙니다.

| 운영체제 | CPU | CLI(터미널 실행) | GUI(창으로 실행) |
|---|---|---|---|
| Windows | amd64: 일반적인 Intel·AMD 64비트 PC | ZIP | ZIP |
| Windows | arm64: ARM 기반 Windows PC | ZIP | ZIP |
| Linux | amd64 | tar.gz | 미제공 |
| Linux | arm64 | tar.gz | 미제공 |
| macOS | arm64: Apple Silicon Mac | tar.gz | 미제공 |

파일명 예시:

- `air-comic-server-gui_2.1.0_windows_amd64.zip`: 일반 Windows PC용 GUI
- `air-comic-server-cli_2.1.0_linux_arm64.tar.gz`: Linux ARM64용 CLI
- `air-comic-server-cli_2.1.0_darwin_arm64.tar.gz`: Apple Silicon Mac용 CLI

`darwin`은 macOS를 뜻합니다. Intel Mac용 파일은 제공하지 않습니다. Windows에서 터미널 사용이 익숙하지 않다면 GUI를 선택하세요.

## 2. Windows GUI 사용법

1. 압축을 푼 폴더에서 `air-comic-server-gui.exe`를 실행합니다.
2. **Add Directory**를 눌러 공유할 만화 폴더를 추가합니다. 여러 폴더를 추가할 수 있습니다.
3. **TCP Port**를 지정합니다. 기본값은 `21002`입니다.
4. 필요한 경우 **Password**에 접속 비밀번호를 입력합니다. 비워두면 비밀번호 없이 접속할 수 있습니다.
5. **Start Server**를 누릅니다. `Server is running.`이 표시되면 앱에서 접속할 수 있습니다.
6. 서버를 중지하려면 **Stop Server**를 누릅니다. 폴더를 제거하려면 서버를 중지한 뒤 목록에서 선택하고 **Remove**를 누릅니다.

폴더와 포트 설정은 서버를 시작할 때 저장됩니다. 비밀번호와 자동 시작 옵션의 변경은 즉시 저장됩니다.

### 자동 실행과 종료

| 설정 | 동작 |
|---|---|
| **부팅 시 앱 시작** | 현재 사용자가 Windows에 로그인하면 GUI를 실행합니다. 로그인 전 실행되는 서비스는 아닙니다. |
| **Start server on app startup** | GUI 실행 시 저장된 설정으로 서버도 시작합니다. |
| **Minimize on app startup** | GUI를 창 없이 시스템 트레이에서 시작합니다. |

Windows 로그인 후 서버가 자동으로 동작하게 하려면 **부팅 시 앱 시작**과 **Start server on app startup**을 함께 켭니다. 백그라운드로 실행하려면 **Minimize on app startup**도 켭니다.

**창의 닫기(X)는 프로그램 종료가 아니라 트레이로 숨기기입니다.** 작업 표시줄 알림 영역의 Air Comic Server 아이콘 메뉴에서 **Show**로 창을 다시 열고, **Quit**으로 서버와 프로그램을 종료합니다.

실행파일 위치를 옮겼다면 새 위치에서 실행한 뒤 **부팅 시 앱 시작**을 다시 설정하세요. 작업 관리자에서 시작 앱을 사용 안 함으로 설정한 경우에는 해당 설정도 확인하세요.

## 3. CLI 사용법

압축을 푼 폴더에서 터미널을 열고 실행합니다. 공유할 폴더는 최소 한 개 필요하며, 공백이 포함된 경로는 따옴표로 감싸세요.

### Windows PowerShell

```powershell
.\air-comic-server-cli.exe --dir "D:\Comics"
```

### Linux

```bash
./air-comic-server-cli --dir "/home/user/Comics"
```

### macOS

```bash
./air-comic-server-cli --dir "$HOME/Comics"
```

Linux·macOS에서 실행 권한이 없다는 오류가 나오면 다음 명령을 한 번 실행합니다.

```bash
chmod +x ./air-comic-server-cli
```

### 실행 옵션

| 옵션 | 설명 | 기본값 |
|---|---|---|
| `-d`, `--dir` | 공유할 폴더. 여러 번 지정 가능 | 최소 한 개의 유효한 폴더 필요 |
| `-p`, `--port` | 서버 TCP 포트, `1`~`65535` | `21002` |
| `--password` | 앱 접속 비밀번호 | 없음 |
| `-h`, `--help` | 도움말 표시 | — |

여러 폴더와 비밀번호를 지정하는 예시:

```powershell
# Windows PowerShell
.\air-comic-server-cli.exe --dir "D:\Comics" --dir "E:\Manga" --port 21002 --password "my-password"
```

```bash
# Linux / macOS — 경로를 실제 폴더에 맞게 바꾸세요.
./air-comic-server-cli --dir "$HOME/Comics" --dir "$HOME/Manga" --port 21002 --password "my-password"
```

CLI는 실행 중인 터미널을 유지해야 합니다. 중지할 때는 **Ctrl+C**를 누릅니다. GUI의 로그인 자동 실행 설정은 CLI에는 적용되지 않습니다.

## 4. Android 앱에서 접속

1. 먼저 PC에서 서버를 시작합니다.
2. 처음 연결할 때는 PC와 Android 기기를 같은 공유기·네트워크에 연결합니다.
3. PC의 내부 IP 주소를 확인합니다. Windows에서는 `ipconfig`의 해당 네트워크 어댑터 **IPv4 주소**, Linux·macOS에서는 네트워크 설정에서 확인할 수 있습니다.
4. 앱의 서버 연결 설정에 PC의 IP 주소와 서버 포트를 입력합니다. 주소와 포트가 별도 입력란이면 각각 입력하고, URL 입력란이면 `http://192.168.0.10:21002`처럼 입력합니다. 이 주소는 예시이므로 실제 PC 주소로 바꾸세요.
5. 서버에서 비밀번호를 설정했다면 앱에도 같은 비밀번호를 입력합니다.
6. 연결 후 공유 폴더에서 만화를 선택합니다.

휴대폰에서 `localhost`나 `127.0.0.1`을 입력하면 PC가 아니라 휴대폰 자신을 가리킵니다. PC의 IP 주소를 사용하세요.

같은 네트워크 안에서 연결할 때는 공유기 포트포워딩이 필요하지 않습니다. 외부 네트워크에서 접속하려면 공유기와 회선의 외부 접속 설정이 필요합니다. 서버는 UPnP 포트포워딩을 자동으로 시도하지만, 공유기 지원 여부와 회선 구성에 따라 외부 접속이 되지 않을 수 있습니다.

## 사용자 히스토리와 로그 저장 위치

**사용자별 읽기 히스토리는 실행파일이 있는 디렉토리에 `AirComicServer_History_*.db` 파일로 저장됩니다.** 파일 확장자는 `.db`이지만 내용은 JSON입니다. 서버 실행 계정에 해당 디렉토리의 쓰기 권한이 필요합니다. 앱을 업데이트할 때는 히스토리 파일을 유지하고, 실행파일을 다른 디렉토리로 옮길 때는 히스토리 파일도 함께 옮겨야 기존 기록을 유지할 수 있습니다.

**현재 앱은 로그 파일을 자동으로 생성하지 않습니다.** 로그는 기본적으로 표준 오류(stderr)로 출력되며, 실행 방식에 따라 터미널이나 systemd journal에 기록됩니다. 로그 파일도 실행파일과 같은 디렉토리에 보관하려면 해당 디렉토리에서 다음과 같이 출력을 파일로 리다이렉트합니다.

```sh
# Linux / macOS / Synology — 경로와 옵션은 실제 환경에 맞게 변경하세요.
cd /volume1/apps/air-comic-server || exit 1
./air-comic-server-cli --dir /volume1/comics --port 21002 >>server.log 2>&1
```

위 실행 예시에서는 실행파일 디렉토리에 **로그 파일 `server.log`와 사용자 히스토리 파일 `AirComicServer_History_*.db`가 함께 저장됩니다.** `>>`는 기존 로그 뒤에 추가하며, 로그 파일 크기를 자동으로 제한하거나 오래된 로그를 삭제하지 않습니다. [Synology systemd 설치 가이드](docs/synology-dsm-7.4-systemd.md)의 예시는 파일 리다이렉트를 사용하지 않으므로 로그는 `journalctl`로 확인합니다. `WorkingDirectory`만 설정한다고 로그 파일이 생성되는 것은 아닙니다.

## 지원 파일

- 이미지: JPG, JPEG, BMP, GIF, PNG, WEBP
- 압축파일: ZIP, CBZ, RAR, CBR
- 압축파일 내부 이미지는 미리 압축을 풀지 않아도 탐색할 수 있습니다.

## 연결·실행이 안 될 때

| 증상 | 확인할 내용 |
|---|---|
| 앱에서 서버에 연결되지 않음 | 서버가 실행 중인지, IP·포트가 맞는지, PC와 휴대폰이 같은 네트워크인지 확인하세요. PC 방화벽에서 서버의 TCP 포트 접속도 허용해야 합니다. |
| 포트를 사용할 수 없다는 오류 | 다른 프로그램이나 이미 실행 중인 서버가 같은 포트를 사용하는지 확인하고, 필요하면 포트를 변경하세요. 앱의 포트도 함께 변경해야 합니다. |
| 유효한 폴더가 없다는 오류 | 지정한 폴더가 실제로 존재하고 서버 실행 계정으로 접근 가능한지 확인하세요. |
| 인증 실패 | 서버와 앱의 비밀번호가 같은지 확인하세요. |
| GUI 창이 보이지 않음 | 시스템 트레이에서 **Show**를 선택하고 **Minimize on app startup** 설정을 확인하세요. |
| Windows GUI 그래픽 초기화 오류 | GUI는 OpenGL 지원이 필요합니다. 그래픽 드라이버를 확인하거나 CLI를 사용하세요. |
| 실행파일을 실행할 수 없음 | 운영체제와 CPU에 맞는 파일을 받았는지 확인하세요. macOS 파일은 Apple Silicon 전용입니다. |

Windows 실행파일에는 배포 인증서 서명이 없고, macOS CLI에는 Apple 공증이 적용되어 있지 않습니다. Linux CLI는 CGO 없이 빌드되어 glibc에 의존하지 않으며, 최소 커널 지원 범위는 사용된 Go 버전을 따릅니다.

## 다운로드 무결성 확인

릴리스의 `SHA256SUMS`에는 배포 파일별 SHA-256 값이 있습니다. 다운로드한 파일의 해시와 해당 파일명 행의 값을 비교할 수 있습니다.

```powershell
# Windows — 실제 다운로드한 파일명으로 바꾸세요.
Get-FileHash -Algorithm SHA256 .\air-comic-server-gui_2.1.0_windows_amd64.zip
```

```bash
# Linux
sha256sum air-comic-server-cli_2.1.0_linux_amd64.tar.gz

# macOS
shasum -a 256 air-comic-server-cli_2.1.0_darwin_arm64.tar.gz
```

## Synology NAS에서 서비스로 실행

DSM 7.4에서 부팅 자동 시작과 비정상 종료 후 자동 재시작을 설정하는 방법은 별도 문서를 참고하세요.

**[Synology DSM 7.4 systemd 서비스 설치 가이드](docs/synology-dsm-7.4-systemd.md)**
