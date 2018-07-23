+++
title   = "Virtual Miachine"
chapter = true
icon    = "<b>-</b>"
+++

# Virtual Machine 관련 정리

## HVM & PV
- Hardware Virtual Machine(Full Virtualization)
- Para Virtualization

PV는 guest os 에서 hypervisor를 통해 hardware를 제어해야 하므로
guest os의 kernel에 변경이 들어감.

HVM은 CPU의 VT기능을 활용.

일반적으로 PV가 HVM 보다 성능이 좋으나 큰차이는 없음.
