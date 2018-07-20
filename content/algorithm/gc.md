+++
title  = "gc"
toc    = true
weight = 2
+++

## Referenche Counting
순환 참조 시 문제가 됨.

## Mark and Sweep
root set을 두고 reference의 관계를 추적하면서 garbage를 수집.

bitmap table이나 root object의 header에 reference에 대한 mark를 남김.
이를 통해 garbage 여부를 판단.

Memory 파편화 위험이 있음.

## Mark and Compaction
파편화를 막기 위해 Compaction을 사용.

## Copying
Heap을 Active/InActive로 나눠서 다룸.

## Generational
Copying에서 더 나아가 보다 다양한 방식으로 sub heap을 나눠서 사용함.
각각 sub heap 마다 별개의 GC 방식 적용이 가능함.

## Train/Incremental
Heap을 Block으로 나눠서 다룸.

## Adaptive

