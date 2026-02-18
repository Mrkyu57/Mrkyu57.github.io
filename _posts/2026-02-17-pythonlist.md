---
title: "[Phython] 리스트(list)"
date: 2026-02-18 21:00:00 +0900
categories: [programming,phython]
tags: [programming]
excerpt: " "
---

> phython의 리스트 문법

## 리스트

<br>

`listname` : 리스트의 이름을 지정합니다\
`element` : 리스트에 포함시킬 요소를 지정합니다

```python
listname = [ element,element,element,element ]  
```

```python
# For Example
list1 = [1,2,4,7]
list2 = ['a','b','c']
```

<br>

## 함수

<br>

> `len` : 리스트의 길이를 나타내는 함수  

```python
listname = [ element,element,element,element ]  
len(listname)
```

```python
# For Example
>>> list1 = [1,2,4,7]
>>> print(len(list1))
4
```
<br>
<br>

>`max` : 리스트의 최댓값을 구하는 함수  

```python
listname = [ element,element,element,element ]  
max(listname)
```

```python
# For Example
>>> list1 = [1,2,4,7]
>>> print(max(list1))
7
>>> list2 = ['a','b','c']
>>> print(max(list2))
c
```
<br>
<br>

>`min` : 리스트의 최솟값을 구하는 함수  

```python
listname = [ element,element,element,element ]  
max(listname)
```

```python
# For Example
>>> list1 = [1,2,4,7]
>>> print(min(list1))
1
>>> list2 = ['a','b','c']
>>> print(min(list2))
a
```
<br>
<br>

>`sum` : 리스트의 값을 모두 더하는 함수

```python
listname = [ element,element,element,element ]  
sum(listname)
```

```python
# For Example
>>> list1 = [1,2,4,7]
>>> print(sum(list1))
14          #1+2+4+7=14
```

<br>
<br>

>`sorted` : 정렬한 리스트를 구하는 함수  

```python
listname = [ element,element,element,element ]  
sorted(listname)
sorted(listname, reverse=booltype)   #세부사항  
```
booltype=`True` : 내림차순\
booltype=`False` : 올림차순
```python
# For Example
>>> list1 = [4,2,7,1]
>>> print(sorted(list1))
[1, 2, 4, 7]

>>> list1 = [4,2,7,1]
>>> print(sorted(list1, reverse=True))
[7, 4, 2, 1]
```
<br>

## 매서드

