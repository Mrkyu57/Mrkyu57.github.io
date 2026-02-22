---
title: "[Python] 함수(function)"
date: 2026-02-19 12:00:00 +0900
categories: [computer language,programming language]
tags: [python]
excerpt: " "
---

> phython의 함수

<br>

## 함수 - 정의 Definition

<br>

`FunctionName` : 함수의 이름을 지정합니다\
`Patameter` :  매개변수, 함수에 입력으로 전달된 값을 받는 변수를 의미한다\
`LogicCode` : 함수 호출시 수행되는 코드\
`ResuiltCode` : 함수의 반환값\
`Argument` : 인수, 함수를 호출할 때 전달하는 입력값을 의미한다

```python
def FunctionName(Parameter,Parameter,Parameter): 
    LogicCode
    return ResultCode

FunctionName(Argument,Argument,Argument)
```

```python
# For Example
def add(a,b): 
    return a+b

>>> print(add(3,4))
7
```

<br>

다만, 수학에서의 함수와 달리 입력값과 출력값이 존재하지 않을 수 있습니다