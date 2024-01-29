---
tags:
  - Ubuntu
update: "2024-01-29"
date: "2024-01-28"
상태: "POST"
title: "우분투 용량 관리"
---
개인적으로 나스를 하나 운영하고 있다. 

이것저것 설정해서 쓰고 있는데, 어느 날 갑자기 토렌트 다운로드가 용량이 없다고 작동하지 않았다. 

용량은 다음의 명령어를 통해 확인할 수 있다. 

```bash
furychick@odroid:/HDD2/myHomePage$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           200M   29M  171M  15% /run
/dev/mmcblk1p2   15G   15G     0 100% /
tmpfs           996M     0  996M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/mmcblk1p1  128M   14M  114M  11% /media/boot
/dev/sdb1       2.7T  656G  2.0T  26% /HDD2
/dev/sda1        11T  8.3T  2.1T  80% /HDD
tmpfs           200M     0  200M   0% /run/user/1000
```

외장하든 왕 큰 놈을 달아서 사용하고 있는데, 오드로이드다보니 메인 OS가 깔리는 드라이브는 15gb짜리 작은 용량이다.. 그 드라이브가 100퍼센트 꽉 차 있었다. 

다음의 명령어를 사용해 용량이 큰 디렉토리를 검색했다. 외장하드 경로는 제외하고 검색했다. 

```bash
furychick@odroid:/$ sudo du -h / --exclude=/HDD2 --exclude=/HDD | sort -hr | head -n 10
du: cannot read directory '/proc/sys/fs/binfmt_misc': No such device
du: cannot access '/proc/19035/task/19035/fd/3': No such file or directory
du: cannot access '/proc/19035/task/19035/fdinfo/3': No such file or directory
du: cannot access '/proc/19035/fd/4': No such file or directory
du: cannot access '/proc/19035/fdinfo/4': No such file or directory
15G     /
11G     /home/furychick
11G     /home
8.7G    /home/furychick/.forever
2.2G    /var
1.7G    /usr
1.6G    /var/log
1.5G    /var/log/journal/dc87f36fc06c441a85ff7269ba4d50fb
1.5G    /var/log/journal
1.3G    /usr/lib
```

아~ 개인 홈페이지를 돌리는 로그가.. 몇 년 째 사이트를 그냥 켜둔채로 방치하다 보니 9기가에 달하게 크게 자라 내 서버를 억누르고 있었다. 

당장 로그를 삭제해주었다. 

해피엔딩~



