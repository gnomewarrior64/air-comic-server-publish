# Synology DSM 7.4에서 systemd 서비스로 실행

[← Air Comic Server 사용법](../README.md)

DSM 7.4에서 CLI를 systemd 서비스로 등록하면 부팅 자동 시작, 프로세스 관리, 비정상 종료 후 재시작을 처리할 수 있습니다. `nohup`, `&`, PID 파일은 필요하지 않습니다. 아래는 **직접 설치하는 관리자용 서비스 예시**이며, Synology 패키지 센터용 SPK 설치 방식은 아닙니다. 실제 NAS에서 검증한 설치 절차는 아니므로 먼저 다음 명령으로 설치 환경을 확인하세요.

```sh
cat /etc.defaults/VERSION
uname -m
ps -p 1 -o comm=
command -v systemctl
systemctl --version
```

PID 1이 `systemd`이고 `systemctl`을 사용할 수 있는 환경을 전제로 합니다. 명령이 없다면 DSM 버전과 서비스 관리 도구를 먼저 확인하세요. 패키지 관리용 `synosystemctl`에 아래 명령을 그대로 대입하지 마세요.

DS916+는 `linux_amd64` CLI를 사용합니다. 다른 모델은 `uname -m`이 `x86_64`이면 `linux_amd64`, `aarch64`이면 `linux_arm64`를 선택합니다. 32비트 ARM 모델용 파일은 현재 제공하지 않습니다.

## 1. 파일과 실행 계정 준비

이하 예시는 다음 경로와 계정을 사용합니다. 실제 환경에 맞게 모든 경로를 바꾸세요.

| 항목 | 예시 |
|---|---|
| 실행 계정 | `aircomic` |
| 실행파일 | `/volume1/apps/air-comic-server/air-comic-server-cli` |
| 읽기 기록 폴더(실행파일과 같은 폴더) | `/volume1/apps/air-comic-server` |
| 만화 폴더 | `/volume1/comics` |

DSM **제어판 → 사용자 및 그룹**에서 일반 사용자 `aircomic`을 만듭니다. 만화 폴더는 읽기, 실행파일과 그 상위 폴더는 읽기·탐색이 가능하도록 DSM/File Station 권한을 설정합니다. 현재 앱은 실행파일 옆에 읽기 기록을 저장하므로 실행파일이 있는 폴더에도 쓰기 권한이 필요합니다. `/etc`에 등록하는 서비스 설정과 비밀번호 파일은 관리자가 관리합니다.

File Station으로 압축을 푼 CLI 파일을 위 경로에 복사합니다. 이후 SSH에 관리자 계정으로 접속해 준비합니다.

```sh
sudo chmod 755 /volume1/apps/air-comic-server/air-comic-server-cli
sudo chown aircomic /volume1/apps/air-comic-server
sudo chmod 750 /volume1/apps/air-comic-server
```

DSM 공유폴더 ACL도 함께 적용되므로 `chown`만으로 모든 권한이 해결되는 것은 아닙니다. 접근 오류가 나면 File Station의 권한도 확인하세요. 예전에 작업 스케줄러로 서버를 실행했다면 해당 자동 실행 작업을 끄고 기존 서버를 종료해 포트 충돌을 방지합니다.

## 2. 비밀번호 설정

다음 파일을 편집해 한 줄을 넣습니다. 비밀번호 파일은 관리자만 읽을 수 있게 만듭니다.

```sh
sudo touch /etc/air-comic-server.env
sudo chmod 600 /etc/air-comic-server.env
sudo vi /etc/air-comic-server.env
```

```ini
AIR_COMIC_PASSWORD="원하는비밀번호"
```

비밀번호 없이 사용할 경우 `AIR_COMIC_PASSWORD=""`로 설정합니다. systemd가 이 파일을 읽어 실행 계정에 환경변수로 전달하며, CLI에는 `--password` 인자로 전달됩니다.

## 3. 서비스 파일 등록

```sh
sudo mkdir -p /etc/systemd/system
sudo vi /etc/systemd/system/air-comic-server.service
```

아래 내용을 저장합니다. `ExecStart`는 한 줄로 입력하세요.

```ini
[Unit]
Description=Air Comic Server
Wants=network-online.target
After=network-online.target
RequiresMountsFor=/volume1/apps/air-comic-server /volume1/comics

[Service]
Type=simple
User=aircomic
WorkingDirectory=/volume1/apps/air-comic-server
EnvironmentFile=/etc/air-comic-server.env
ExecStart=/volume1/apps/air-comic-server/air-comic-server-cli --dir /volume1/comics --port 21002 --password ${AIR_COMIC_PASSWORD}
Restart=on-failure
RestartSec=10
TimeoutStopSec=30
UMask=0077
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

`RequiresMountsFor`는 해당 경로에 필요한 systemd 마운트의 시작 순서를 지정합니다. 암호화 공유폴더의 잠금 해제까지 대신하지는 않습니다. 부팅 후 만화 폴더가 자동으로 마운트·잠금 해제되는지도 확인하세요. 여러 폴더를 공유하려면 `ExecStart`에 `--dir /다른/폴더`를 추가하고 `RequiresMountsFor`에도 해당 경로를 추가합니다.

현재 앱의 읽기 기록 저장 위치는 **실행파일이 있는 폴더**입니다. `WorkingDirectory`를 변경해도 저장 위치는 바뀌지 않으며, 별도의 `data` 폴더나 저장 경로 지정 옵션은 없습니다. 기존 읽기 기록을 유지하면서 실행파일 위치를 옮긴다면 서버를 중지한 상태에서 기존 실행파일 옆의 `AirComicServer_History_*.db`도 새 실행파일 폴더로 복사하고 실행 계정의 읽기·쓰기 권한을 맞춥니다.

## 4. 시작과 확인

```sh
sudo systemctl daemon-reload
sudo systemctl enable air-comic-server.service
sudo systemctl start air-comic-server.service
sudo systemctl status air-comic-server.service --no-pager
sudo systemctl is-enabled air-comic-server.service
```

`active (running)`과 `enabled`를 확인합니다. Android 앱에는 NAS의 IP, 포트 `21002`, 위에서 설정한 비밀번호를 입력합니다. DSM 방화벽을 사용한다면 접속할 네트워크에서 TCP `21002`로 들어오는 연결을 허용하세요. 재부팅 후에도 같은 명령과 앱 접속으로 자동 시작을 확인합니다.

```sh
# 최근 로그 / 실시간 로그 (실시간 보기 종료: Ctrl+C)
sudo journalctl -u air-comic-server.service -n 100 --no-pager
sudo journalctl -u air-comic-server.service -f

# 재시작 / 중지 / 부팅 자동 시작 해제
sudo systemctl restart air-comic-server.service
sudo systemctl stop air-comic-server.service
sudo systemctl disable air-comic-server.service
```

`systemctl stop`으로 의도적으로 중지하면 `Restart=on-failure`가 다시 시작하지 않습니다. `disable`은 자동 시작만 해제하므로 현재 프로세스까지 종료하려면 `stop`도 실행하세요. `journalctl`의 로그 보존 기간은 NAS의 journal 설정을 따릅니다.

## 5. 설정 변경·업데이트·등록 해제

- 만화 폴더·포트 등 unit 내용을 변경했다면 `sudo systemctl daemon-reload` 후 서비스를 재시작합니다. 비밀번호 파일만 변경했다면 재시작하면 됩니다.
- 앱 업데이트는 서비스를 중지하고 실행파일을 교체한 뒤 실행 권한을 확인하고 다시 시작합니다. 같은 폴더의 `AirComicServer_History_*.db` 파일은 유지합니다.
- `status=203/EXEC`는 실행파일 경로·권한·아키텍처, `status=200/CHDIR`는 작업 폴더 접근 권한, `status=217/USER`는 실행 계정을 먼저 확인합니다.
- DSM 업데이트 후 직접 추가한 unit과 자동 시작 설정이 유지되는지는 이 문서에서 보장하지 않습니다. 서비스 파일과 비밀번호 설정의 별도 백업을 보관하고, 업데이트 후 `systemctl status`와 `is-enabled`를 다시 확인하세요. 파일이 사라졌다면 백업을 복원하고 `daemon-reload`, `enable`, `start`를 다시 실행합니다.

서비스 등록만 해제하고 실행파일·읽기 기록은 남기려면:

```sh
sudo systemctl stop air-comic-server.service
sudo systemctl disable air-comic-server.service
sudo rm /etc/systemd/system/air-comic-server.service
sudo systemctl daemon-reload
```

참고: [Synology systemd 패키지 연동 문서](https://help.synology.com/developer-guide/resource_acquisition/systemd_user_unit.html), [systemd 서비스 설정](https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html), [systemd 실행 환경](https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html). Synology 문서는 SPK 패키지 연동을 설명하며, 위 수동 설치 예시 자체를 공식 지원 절차로 보증하는 문서는 아닙니다.
