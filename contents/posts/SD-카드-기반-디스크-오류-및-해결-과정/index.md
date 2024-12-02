---
IDX: "NUM-222"
tags:
  - Odroid
  - Ubuntu
description: "SD 카드 기반 디스크 오류 및 해결 과정"
update: "2024-12-02T15:37:00.000Z"
date: "2024-11-27"
상태: "Ready"
title: "SD 카드 기반 디스크 오류 및 해결 과정"
---
![](image1.png)
## 서론

최근 ODROID SBC(Single Board Computer)를 사용하는 환경에서 루트 파일 시스템(`/`)이 읽기 전용(read-only) 모드로 전환되는 문제를 경험했습니다. 이는 특정 상황에서 발생할 수 있는 디스크 오류로, SD 카드나 eMMC와 같은 저장 장치를 사용하는 시스템에서 흔히 발생할 수 있습니다. 이번 글에서는 문제 발생 원인과 증상, 그리고 실제로 문제를 해결한 과정을 공유하고자 합니다.

## 문제 증상

문제는 다음과 같은 상황에서 처음 발생했습니다:

- 시스템에서 디렉토리에 접근하거나 파일을 생성하려고 했을 때 "Read-only file system" 오류가 발생.

- 예를 들어, `/HDD/develop` 디렉토리에서 Docker 이미지를 빌드하려고 했지만 권한 문제로 실패했습니다.

    ```shell
    ERROR: mkdir /home/USER/.docker: read-only file system
    ```

- 이후, 권한 문제를 해결하려고 `chmod`, `chgrp`, `usermod` 명령어를 시도했지만 여전히 동일한 오류가 발생했습니다.

    ```shell
    chmod: changing permissions of '/etc/passwd': Read-only file system
    ```

`mount | grep ' / '` 명령어를 통해 파일 시스템 상태를 확인한 결과, 루트 파일 시스템이 읽기 전용 모드로 전환된 것을 확인했습니다. ro 라고 써있는 것을 확인할 수 있습니다. 

```shell
/dev/mmcblk1p2 on / type ext4 (ro,noatime,errors=remount-ro,stripe=32753)
```

## 원인 분석

`ro`(read-only) 모드는 파일 시스템에서 심각한 오류가 발생했을 때 시스템이 자동으로 읽기 전용 모드로 전환하는 보호 메커니즘입니다. `/etc/fstab` 파일의 설정에서 `errors=remount-ro` 옵션이 이를 담당합니다. 일반적으로 이는 다음과 같은 이유로 발생할 수 있습니다:

1. 파일 시스템 손상: 예상치 못한 종료나 전원 문제로 파일 시스템이 손상됨.

1. SD 카드 불량: SD 카드나 eMMC 저장 장치의 물리적 손상.

1. 디스크 용량 부족: 루트 파티션에 남은 공간이 없을 때.

1. I/O 오류: 저장 장치와의 통신 문제.

<hr style="border: none; height: 1px; background-color: #e0e0e0; margin: 16px 0;" />
## 해결 과정

### 파일 시스템 복구

루트 파일 시스템 복구를 위해 `fsck`(파일 시스템 체크) 도구를 사용했습니다. 이를 위해 먼저 루트 파일 시스템이 마운트된 디스크를 식별했습니다. 제 경우 `/dev/mmcblk1p2`가 루트 디스크였습니다.

다음 명령어로 파일 시스템 복구를 실행했습니다:

```shell
sudo fsck -y /dev/mmcblk1p2
```

- `y` 옵션은 모든 수정 사항에 대해 자동으로 "Yes"를 선택하게 합니다.

- `fsck` 실행 중, 손상된 블록 및 파일 시스템 불일치를 수정했습니다.

### 시스템 재부팅

복구 후 시스템을 재부팅하여 파일 시스템이 정상적으로 작동하는지 확인했습니다:

```shell
sudo reboot
```

## 결과

재부팅 후 `mount` 명령어로 다시 파일 시스템 상태를 확인한 결과, 루트 파일 시스템이 읽기-쓰기 모드로 마운트된 것을 확인할 수 있었습니다:

```shell
/dev/mmcblk1p2 on / type ext4 (rw,noatime,errors=remount-ro,stripe=32753)
```

이제 권한 변경 및 Docker 이미지 빌드 작업도 정상적으로 동작했습니다:

```shell
sudo chmod g+w /HDD/develop
sudo docker build -t hotdeal_alarm .
```



다만, 해당 문제는 곧 다시 발생했으며 SD 카드 자체에 심각한 물리적인 손상이 생긴것으로 짐작되어 새로운 SD카드를 사기로 결정했습니다. 

이 글을 읽는 여러분은 위의 방법으로 해결되기를 기원합니다.. 

