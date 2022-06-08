---
title: "[Algorithm] 파이썬(Python)으로 조합(Combination)과 순열(Permutation) 구하기"
categories:
  - Algorithm
tags:
  - [study, java]
toc: true
toc_sticky: true
sidebar: 
  nav: "docs"
---
python에는 유용한 라이브러리인 itertools에 내장함수로 combination과 permutaion이 있어 조합,순열을 쉽게 구할 수 있다.       
      
하지만 혹시 코딩테스트에 itertools를 사용하지 못하게 할 경우가 있다고 하니      
조합, 순열을 어떻게 구하는지 정리해볼려고 한다.      
<https://www.acmicpc.net/problem/15649>   
라이브러리를 사용하면 이런 문제는 엄청 쉽기는 하다.    
하지만 내 실력을 키우기 위해서 그리고 혹시 모르니까 알아두는 것이 좋을 것 같다.

순열고 조합 둘 다 아이디어는 비슷비슷하다.     
그리고 확장해 나가면 중복순열, 중복조합까지도 충분히 구현할 수 있다.

# 조합
```python
def combinations(arr,r):
    for i in range(len(arr)):  // 함수에서 지금할 일
        if r == 1:  // 종료조건
            yield [arr[i]]
        else:
            for next in combinations(arr[i+1:],r-1): // 다음에 할 일
                yield [arr[i]] + next

// 아래는 함수를 실행하기 위한 사용법입니다.
for combi in combinations([1,2,3,4,5],2):
    print(combi)
```

# 중복조합
```python
def combinations_with_replacement(arr,r):
    for i in range(len(arr)):
        if r == 1:
            yield [arr[i]]
        else:
            for next in combinations_with_replacement(arr[i:],r-1):
                yield [arr[i]] + next
```
# 순열
```python
def permutation(arr,r):
    for i in range(len(arr)):
        if r == 1:
            yield [arr[i]]
        else:
            for next in permutation(arr[:i]+arr[i+1:],r-1):
                yield [arr[i]] + next

```

# 중복순열
```python
def product(arr,r):
    for i in range(len(arr)):
        if r == 1:
            yield [arr[i]]
        else:
            for next in product(arr,r-1):
                yield [arr[i]] + next
```