---
title:  "Open VPN을 사용하여 DIY VPN만들기. 네 VPN은 네가 만들어서 써라."
date:   2021-08-21 08:15:00 +0900
permalink: ":categories/security/:title"
---

>원문 출처 [Wolfgang's Channel](https://www.youtube.com/watch?v=gxpX_mubz2A&t=107s&ab_channel=Wolfgang%27sChannel)

### 왜 VPN을 직접 만들어서 써야 할까?

1. 인터넷이 발전함에 따라 인터넷상 개인에대한 국가의 검열이 심해지고있다.
1. Nord VPN, Pure VPN 등 해외 내로라 하는 유료 VPN 제공자들도 'no log policy'라고 해서 로그를 가지고있지 않다곤 하는데 그건 그들의 말뿐이다. 실제로 [Pure VPN은 FBI에게 클라이언트의 로그를 제공한적 있다](https://www.zdnet.com/article/cyberstalker-thwarted-by-vpn-logs-gets-17-years-in-prison/). 내가 직접 서버를 세팅하고 로그를 저장하지 않도록 세팅을 해주는게 확실히 안전하다.

    >범죄 행위를 두둔하는것은 아님을 명확하게 짚고 넘어가자.

1. 대부분의 고퀄 VPN은 유료다. 굳이 돈을 써야한다면 내 서버를 구축하는데 쓰는게 덜 아깝다.

### 클라우드 서버 세팅

이 강의에서 사용한 클라우드는 `linode`이다. 아마존의 `aws`와 비슷하지만 사용이 간편하고 서포트 팀이 24시간 대기하고 있어서 언제든지 채팅을하던 전화를하던 직접 서포트 팀과 통화할 수 있다. 더 신기한건 이 회사는 2003년도에 서비스를 시작했다. 심지어 아마존의 `aws`보다 전에 서비스를 시작했고 그때부터 쭉 고퀄의 서비스를 제공하고 있다.

1. [linode](https://www.linode.com/) 에서 계정을 생성한다.
1. 로그인 후  화면 좌측 패널에 `Linodes`를 선택후 `Create Linode`버튼을 클릭한다.
1. `Images` 는 가장 최신 버전 `Ubuntu 20.04 LTS`를 선택한다.
1. `Region` 은 한국과 물리적으로 가까운 `Japan`을 선택한다.
1. `Linode Plan`은 `Nanode 1 GB`를 선택한다. 월 $5정도에 사용할 수 있다고 한다.
1. `Linode Label`은 `(사용자명 또는 아이디)VPN`으로 적어준다.
1. `Root Password`는 자신이 사용하는 비밀번호를 적어주면 된다.
1. 다른건 건드리지 않고 `Add-ons`에서 `Private IP`를 선택해 준다.
1. `Create Linode` 버튼을 눌러준다.

이제 클라우드 서버가 세팅이 시작이되고 이 프로세스는 최대 몇 분정도 소요될 수 있다.

### SSH Keys 생성하기

이 강의를 진행하는 개발 환경은 MacOS이다. 터미널을 열고 다음과 같이 입력한다.

```zsh
  ssh-keygen -t ras -b 4096
```

이후 패스워드를 입력하고 디폴트 디렉토리에 ssh key를 저장해 준다.

이제 다시 `linode`로 돌아가 세팅이 완료된 서버의 `ip`주소를 복사한다. 그리고 터미널에 다음과 같이 입력한다.

입력:

```zsh
  ssh root@139.162.103.189<복사한 자신의 ip 주소>
```

출력:

```zsh
  The authenticity of host '139.162.103.189 (139.162.103.189)' can't be established.
  ECDSA key fingerprint is SHA256:8CE2Lqk2Nd444CIzrrgdkyTdf1pAsFkoFCVP2Hm8NQ8.
  Are you sure you want to continue connecting (yes/no/[fingerprint])? 
```

`yes`를 입력하고 `linode`서버 세팅시 설정한 비밀번호를 입력해 준다.

```zsh
  root@localhost:~# 
```

위 처럼 방금 만든 서버에 접근한것을 확인할 수 있다.

### 패키지 업데이트 하기

다음의 커멘드를 입력하여 서버 패키지를 업데이트 해 준다.

입력:

```zsh
  apt-get update && apt-get upgrade
```

출력:

```zsh
  Unpacking grub-pc (2.04-1ubuntu26.13) over (2.04-1ubuntu26.12) ...
  Preparing to unpack .../13-grub-pc-bin_2.04-1ubuntu26.13_amd64.deb ...

  ...

  I: (UUID=8cd34d89-9891-4e4e-a050-559242151b9f)
  I: Set the RESUME variable to override this.
  root@localhost:~# 
```

동의가 필요한 경우 `y` 또는 `yes`를 입력하고 모든 설치를 완료해 준다.

### non-root 유저 만들기

서버에 접근을 할때마다 `root` 비밀번호를 입력하는 것은 보안상 좋지 않기 때문에 새로운 어카운트를 만들어 준다. 커멘드에 다음과 같이 입력한다.

```zsh
  useradd -G sudo -m <username> -s /bin/bash
```

그리고 방금 생성한 유저의 비밀번호도 세팅을 해 준다.

입력:

```zsh
  root@localhost:~# passwd mannerism 
```

출력:

```zsh
  New password: 
  Retype new password: 
```

비밀번호를 두번 입력해주면 세팅이 완료된다.

### SSH 세팅하기

지금까지 사용한 `root@localhost:~#` 터미널은 그대로 두고 새로운 터미널은 한개 더 띄워서 컴퓨터에 저장되어있는 `public ssh key`를 서버에 복사해 준다.

새로운 터미널에 입력:

```zsh
  ssh-copy-id <username>@139.162.103.189
```

출력:

```zsh
  mannerism@139.162.103.189's password: 
  Number of key(s) added:        1

  Now try logging into the machine, with:   "ssh 'mannerism@139.162.103.189'" and check to make sure that only the key(s) you wanted were added.
```

`root@localhost:~#` 터미널로 돌아가서 `public_key`로만 접근 가능하도록 `Authentication` 세팅을 수정해준다.

```zsh
   vi /etc/ssh/sshd_config
```

를 입력하면 `vim` 텍스트 에디터로 세팅 파일이 열리게 된다. `vim`사용법을 모르는 사람은 좀 당황할 수 있는데 어려울거 없다. 파일이 열리면 키보드 `j`과 `k`버튼을 누르면 커서가 위/아래로 움직이고 `h`와 `l`을 누르면 좌/우로 움직인다. 커서를 움직여서 `port`로 이동하여 `i` 버튼을 누르면 이제 텍스트를 수정할 수 있게된다.
  
1. `#Port 22`를 `Port 69`으로 바꾸어 준다.
1. `PasswordAuthentication yes`를 `PasswordAuthentication no`로 바꾸어준다.
1. `PermitRootLogin yes`를 `PermitRootLogin no`로 바꾸어준다.
1. `esc` 버튼을 한번 눌러서 `insert`모드에서 빠져나온다.
1. `:wq`를 입력하고 엔터를 눌러서 변경사항을 저장 후 `vim`에서 빠져나온다.
1. `sudo systemctl restart sshd`를 커멘드에 입력하여 ssh세팅을 적용해준다.

이제 다시 로컬 터미널로 돌아가서 `ssh`를 사용하여 서버에 접근이 가능한지 테스트를 해 본다.

```zsh
  ssh -i ~/.ssh/id_rsa mannerism@139.162.103.189 -p 69 
```

비밀번호를 입력하라는 메세지가 나오면 비밀번호를 입력한다. 그리고나서 서버에 접근이 가능한 것을 확인할 수 있다.

### 기억하기 쉬운 Alias 커멘드 만들기

```zsh
  ssh -i ~/.ssh/id_rsa mannerism@139.162.103.189 -p 69 
```

이런방식으로 접근을해도 상관이 없지만 상당히 귀찮다는걸 알 수 있다. 따라서 커멘드를 간소화하기 위해 `Alias`를 만들어보자. 다음의 커멘드를 쳐서 `config`라는 파일을 `.ssh`폴더에 만들어준다.

```zsh
  vim ~/.ssh/config 
```

그리고 나서 `vim`에디터를 사용하여 다음과 같이 적어준다. 앞서 말했듯이 `i`를 눌러줘야 `insert`모드 즉 편집모드로 들어간다.

```vim
  Host mannerismvpn
        User mannerism
        Port 69
        IdentityFile ~/.ssh/id_rsa
        HostName 139.162.103.189
```

이제 서버에 접근할 때마다 `ssh mannerismvpn` 커멘드를 사용하여 쉽게 접근할 수 있다.

>팁: 로그인을 할때마다 나오는 웰컴 텍스트를 없애기 위해 `touch .hushlogin`을 하면 된다.

### Open VPN 설치하기

`mannerism@localhost:~$`로 돌아가서 `wget`을 설치해 준다.

```zsh
  sudo apt install wget
```

Open VPN을 설치하는 스크립트로 [OpenVPN road warrior](https://github.com/Nyr/openvpn-install/blob/master/openvpn-install.sh)를 사용할 것이다.

해당 깃헙에 있는 스크립트의 `raw link`를 복사해 준다.

```zsh
  https://raw.githubusercontent.com/Nyr/openvpn-install/master/openvpn-install.sh
```

`wget`을 사용하여 스크립트를 서버에 복사해 준다.

```zsh
  wget https://raw.githubusercontent.com/Nyr/openvpn-install/master/openvpn-install.sh
```

복사된 스크립트를 실행시켜 준다.

```zsh
  sudo bash openvpn-install.sh
```

스크립트를 실행시키면 다음과 같은 문항이 제공된다. 그대로 적으면 된다.

```zsh
  Welcome to this OpenVPN road warrior installer!

Which IPv4 address should be used?
     1) 139.162.103.189
     2) 192.168.129.15
IPv4 address [1]: 1

Which protocol should OpenVPN use?
   1) UDP (recommended)
   2) TCP
Protocol [1]: 1

What port should OpenVPN listen to?
Port [1194]: 443

Select a DNS server for the clients:
   1) Current system resolvers
   2) Google
   3) 1.1.1.1
   4) OpenDNS
   5) Quad9
   6) AdGuard
DNS server [1]: 3

Enter a name for the first client:
Name [client]: mac 

OpenVPN installation is ready to begin.
Press any key to continue...
```

스크립트 실행이 완료되면 `open vpn config`파일이 생성이 되고 서버의 `root`에 저장이 된다. 다음의 커멘드를 실행시켜 `root`에 있는 파일은 옮겨와 준다.

```zsh
  sudo mv /root/mac.ovpn .
```

그리고 파일의 권한을 변경해 준다.

```zsh
  sudo chown mannerism mac.ovpn
```

### No Log Policy 세팅하기

`mannerism@localhost:~$`로 돌아가서 `open vpn config`파일을 수정해 준다.

`config`파일을 열고,

```zsh
  sudo vim /etc/openvpn/server/server.conf
```

`verb 3`를 `verb 0`으로 수정해 준다.

그리고 `openvpn`을 재부팅해준다.

```zsh
  sudo systemctl restart openvpn-server@server.service
```

그리고 마지막으로 서버명으로 `localhost`에서 다른이름으로 변경해 준다.

```zsh
  sudo hostnamectl set-hostname mannerismvpn
```

### 서버에서 OpenVPN Config파일 가져오기

`sftp`커멘드와 `alias`를 사용하여 서버에 접근해 준다.

```zsh
  sftp mannerismvpn
```

그리고 스크립트를 사용하여 만든 `ovpn` config파일을 다운받아 준다.

```zsh
  sftp> get mannerismmac.ovpn
```

커멘드에 `exit`을 쳐서 서버접근을 종료한다.

### Tunnel Blick

[TunnelBlick](https://tunnelblick.net/downloads.html)을 다운받고 위에서 받은 `ovpn`을 추가하여 실행시키면 바로 VPN이 작동되는걸 확인할 수 있다.

끝.
