---
layout: post
title:  "리눅스에서 3TB이상의 USB외장하드 마운트시키기"
categories: ['Linux']
---

## 디스크 파티션 나누기

### `fdisk -l`로 외장하드 인식 확인

```
➜  ~ sudo fdisk -l
Disk /dev/sdc: 3.7 TiB, 4000787029504 bytes, 7814037167 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 33553920 bytes
Disklabel type: gpt
Disk identifier: BF1EB814-80D7-4DB9-9485-D772CF6AF7CA

Device     Start        End    Sectors  Size Type
/dev/sdc1  65535 7814000189 7813934655  3.7T Linux filesystem
```

`/dev/sdc1`에 외장하드가 있는 것을 확인한다.

### `parted`사용

`fdisk`는 1.1TB이상의 파티션을 나누지 못하므로, `parted`를 사용한다.

```shell
➜  ~ sudo parted /dev/sdc
GNU Parted 3.2
Using /dev/sdc
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) 
```

위와 같이 `parted`의 셸에서 다음 명령어 입력

```
unit a
rm 1
mkpart LVM ext4 0% 100%
print
```

## 디스크 포맷, 마운트

```
mkfs.ext4 /dev/sdc1
sudo mkdir /hdd
sudo mount -t ext4 /dev/sdc1 /hdd
```


`/hdd`폴더에 `/dev/sdc1`에 해당하는 외장하드가 마운트된다.


#### 참조

- https://blog.hqcodeshop.fi/archives/273-GNU-Parted-Solving-the-dreaded-The-resulting-partition-is-not-properly-aligned-for-best-performance.html
- https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Disks/ko
- https://www.linux.co.kr/home/lecture/index.php?cateNo=&secNo=&theNo=&leccode=10966
- http://plming.tistory.com/m/68
                                                                                                                                                                                                                                          