# systemctl 사용법
본 문서는 시스템 부팅/재부팅 시, 자동으로 서비스를 시작하기 위해 사용하는 systemctl 사용법에 대해 기술한 문서이다.



## [Ubuntu 16.04]

### 공통

우선은 재부팅 후, 실행하고자 하는 파일을 작성한다.

```bash
sudo vi /etc/init.d/test.sh
```

```bash
# tesh.sh 내용
#!/bin/bash

source /home/secy/catkin_ws/devel/setup.bash
roslaunch secy_proj view_secy_bomb.launch

exit 0
```

이후, 작성한 파일을 실행가능하고, 사용자가 원활히 사용가능토록, 권한을 수정한다.

```bash
sudo chmod 755 /etc/init.d/test.sh
```

------

### systemctl 서비스 등록 방법 1

다음은 rc의 각 단계에서 서비스 수행 가능하도록 등록하는 방법이다.

> 작성자의 경우, default 런레벨은 graphical 이었으나, 이 방법을 사용한 경우, CLI 모드에서 서비스가 실행되는 것을 확인함.

먼저 /etc/rc.local 에 서비스를 등록한다.

```bash
sudo vi /etc/rc.local
```

```bash
# /etc/rc.local 내용
#!/bin/bash

/etc/init.d/test.sh

exit 0
```

이후, /etc/rc.local에 실행권한과, 사용자가 원활히 사용가능토록 권한을 수정한다.

```bash
sudo chmod 755 /etc/rc.local
```

다음은 서비스 등록을 위해 사용한다.

> 설명을 봤을 땐, WantedBy를 통해 실행되는 런레벨을 바꿀 수 있는 것으로 이해했는데, 직접해보니까 무조건 CLI모드로만 실행된다... 하... ㅋㅋㅋ;;

```bash
sudo vi /lib/systemd/system/rc-local.service
```

제일 아래에 [Install] 부분을 추가해준다.

```bash
[Unit]
Description=Poratiner Service
Requires=docker.service
After=docker.service

[Service]
Type=simple
ExecStart=/opt/portainer/portainer

[Install]
WantedBy=multi-user.target
```

이후, 서비스 등록, 시작 및 현재 서비스 등록 상태 확인을 위한 명령어를 입력하면 작업 끝.

```bash
systemctl enable rc-local.service (systemctl disable rc-local.service)

systemctl start rc-local.service

systemctl status rc-local.service
```

------

### systemctl 서비스 등록 방법 2

> 본 방법은 작성자가 사용한 방법이며, 방법 1에서 잘 되지 않던 GUI 모드를 본 방법으로 하니 정상작동하는 것을 확인함.

먼저 /etc/rc5.d로 이동한 후, 실행할 파일을 심볼릭 링크로 만들면 끝이다.

```bash
cd /etc/rc5.d

sudo ln -s /etc/init.d/test.sh S01test
```



누가봐도 후자의 방법이 서비스 등록할 때 더 편하다고 생각될 것이다.

하지만 만약 GUI 모드가 불가능한 상황이라면 전자의 방법을 사용하겠지?