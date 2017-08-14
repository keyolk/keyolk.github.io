+++
title  = "regular expression"
toc    = true
weight = 0
+++

## Type
- Posix Rerular Expression
  - BRE: Basic Regular Expression
  - ERE: Extened Regular Expression
- PCRE: Perl Compatible Regular Expression

## Posix Regular Expression
- IEE std 1003.1

|           |        |                                      |
|-----------|--------|--------------------------------------|
| 문자지정 	| .      | 임의의 문자 한개                     |
| 반복 지정	| ?      | 선행 문자 패턴이 0개 혹은 1개        |
| 				  | +      | 선행 문자 패턴이 1개 이상 반복       |
| 				  | *      | 선행 문자 패턴이 0개 이상 반복       |
| 				  | {m, n} | 반복수 지정                          |
| 위치지정  | ^      | 라인의 앞부분                        |
|           | $      | 라인의 끝부분                        |
| 그룹 지정 | [...]  | 그룹 중 한 문자                      |
|           | [^...] | 그룹 내 문자들을 제외한 나머지       |
| 기타      | \      | escape                               |
|           | \|     | OR                                   |
|           | ()     | pattern group                        |
