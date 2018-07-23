+++
title  = "raid"
toc    = true
weight = 1
+++

## RAID
Redundat Array of Inexpensive Disk

## Hardware RAID & Software RAID

## Striping & Mirroring

Striping: 논리적으로 연속된 data segment를 물리적으로 여러개의 디스크에 RR 방식으로 나눠 저장.

Mirroring: Data Replicatin

## Architecture
### RAID 0
Striped disk array without fault tolerance

![img](../raid.files/raid0.png)

### RAID 1
Mirrroing and duplexing

![img](../raid.files/raid1.png)

### RAID 2
Hamming code ECC

![img](../raid.files/raid2.png)

### RAID 3 
Parallel transfer with parity

하나의 디스크를 Parity 용으로 사용

![img](../raid.files/raid3.png)

### RAID 4
Independent data disk with shared parity disk

하나의 디스크를 Parity 용으로 사용하 되,
블락 단위로 분산 저장.

![img](../raid.files/raid4.png)

### RAID 5
Independent data disk with distributed parity blocks

3개 이상의 디스크를 붙여서 하나의 디스크로 사용하고
각각에서 parity를 저장.

1개의 디스크 에러는 다른 2개의 디스크를 통해 복구 됨.

![img](../raid.files/raid4.png)

### RAID 6

Parity를 2개씩 사용,
2개의 디스크 에러도 복구 될 수 있음.

![img](../raid.files/raid6.png)

### RAID 0+1

![img](../raid.files/raid01.png)

### RAID 1+0

![img](../raid.files/raid10.png)

## Reference
- http://blog.acronym.co.kr/487
- 
{{%attachments title="Related files" pattern=".*"/%}}
