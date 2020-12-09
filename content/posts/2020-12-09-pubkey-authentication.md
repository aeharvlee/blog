---
title: "Pubkey Authentication not working"
date: 2020-12-09
---

> 리모트 서버와 패스워드 입력 없이 비대칭 키로 원격 접속을 하고 싶은데, 키를 이용한 접속이 되지 않고, 프롬프트에서 패스워드를 요구하는 경우 어떻게 대처하면 좋을지를 알아봅시다.
>
> 저자의 로컬 클라이언트 운영체제는 MacOS Bigsur, 리모트 서버의 경우 CentOS 7으로 되어 있다는 것을 먼저 알려드립니다.

퍼블릭 키를 원격 서버에 잘 두었고, 로컬에 짝이 맞는 개인 키도 있는 상태에서 **키를 지정하여 통신하려 했음에도 실패한 경험**이 있으시다면 다음과 같은 것들을 점검해보시기 바랍니다.

## 1. Check remote server

원격 서버의 `/etc/ssh/ssh/sshd_config` 파일에 `PubkeyAuthentication yes` 항목이 있는지 확인합니다. sshd 설정에서 아예 이를 막아둔 경우 키를 활용한 접속이 불가능합니다.
만약 설정을 변경해주었으면 `service sshd restart` 해주시기 바랍니다. 변경된 설정을 로드해야하니까요. :v:

### 1.1 Let's Add Pubkey

키로 접속하려는 리모트 서버의 `.ssh/authorized_keys` 파일을 직접 확인하셔서 내가 갖고 있는 개인키와 매칭되는 공개키가 제대로 등록되어 있는지 확인해볼 수도 있겠지만, 확실하게 하기 위해 키를 직접 추가해주도록 합시다. 수작업 보다는 그래도 커맨드로 한 번 더 해주는 것이 좋습니다. :keyboard:

 `ssh-copy-id -i ~/.ssh/id_rsa_macbook_pro15.pub user@123.456.789.123 -p 28888` 

로컬에서 위와 같은 커맨드를 입력하여 리모트 서버에 키를 추가해주도록 합시다.. 위 커맨드의 가정은 아래와 같습니다.

* 접속하려는 리모트 server의 아이피 주소가 123.456.789.123, ssh 접속할 때 사용하는 포트는 28888
* 로컬의 private key 파일은 현재 `/Users/aeharvlee/local/.ssh/id_rsa_macbook_pro15` 로 되어 있으며 리모트 서버의 user 계정으로 접근시도

## 2. Check local and remote server both

1번에서 매칭되는 공개 키를 잘 추가해주셨다면, 이제 디버깅할 시간입니다. 접속을 시도하는 로컬의 로그와 동시에 타겟 서버인 리모트 서버의 로그를 살펴보면 됩니다.

```bash
# 클라이언트 측에서 서버에 연결할 때의 과정을 상세히 출력해보도록 합니다.
$ ssh -i ~/.ssh/id_rsa_macbook_pro15 user@123.456.789.123 -p 7788 -v

# 리모트 서버측의 로그를 확인하도록 합니다.
sudo tail -f /var/log/secure
```

먼저 클라이언트 측 ssh 접속 시도 로그를 살펴보시면, 중간에 키로 접속을 시도하는 구간이 잘 되었는지, 아니면 해당 구간이 제대로 통과되지 않아 결국 패스워드 인증으로 넘어갔는지를 살펴볼 수 있습니다.

만약 패스워드 인증으로 넘어갔다면 그에 대한 이유가 바로 리모트 서버의 로그에 남습니다. 저의 경우 리모트 서버의 로그를 확인해보니, 접속하려는 계정의 디렉토리 권한이 맞지 않다는 이유로 접속이 안된다는 것을 확인했습니다. 따라서 그에 맞게 아래와 같이 조치를 치해주었고 그 결과 이제는 정상 접속이 되고 있습니다. :heavy_check_mark:

```
chmod g-w /home/user/
chmod 700 .ssh/
chmod 600 .ssh/authorized_keys
```

크게 어렵진 않지만 한 번 발을 잘못 디디면 시간을 많이 허비할 수 있는 문제라서, 저처럼 헤매시는 분들이 많이 없길 바라는 마음에서 글을 작성했습니다. 도움이 되길 바랍니다. :high_brightness: