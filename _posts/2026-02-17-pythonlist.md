---
title: "[Python] 리스트(list)"
date: 2026-02-18 21:00:00 +0900
categories: [computer language,programming language]
tags: [python]
---

> phython의 리스트 문법

<br>

## 리스트 - 정의 Definition
***

<br>

`ListName` : 리스트의 이름을 지정합니다\
`Element` : 리스트에 포함시킬 요소를 지정합니다

```python
ListName = [ Element,Element,Element,Element ]  
```

```python
# For Example
list1 = [1,2,4,7]
list2 = ['a','b','c']
```

<br>

비어있는 리스트를 `[]` 또는 `list` 코드를 이용해 생성할 수 있습니다
```python
emptylist1 = []
emptylist2 = list()
```

<br>

리스트 내부에 리스트를 넣을 수 있습니다
```python
list1 = [1,2,[3,4,[5,6]]]
```

<br>

## 리스트 - 함수 function

<br>

***
> `len` : 리스트의 길이를 나타내는 함수  

```python
ListName = [ Element,Element,Element,Element ]    
len(ListName)
```

```python
# For Example
>>> list1 = [1,2,4,7]
>>> print(len(list1))
4
```
<br>
<br>

***
>`max` : 리스트의 최댓값을 구하는 함수  

```python
ListName = [ Element,Element,Element,Element ]    
max(ListName)
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

***
>`min` : 리스트의 최솟값을 구하는 함수  

```python
ListName = [ Element,Element,Element,Element ]  
min(ListName)
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

***
>`sum` : 리스트의 값을 모두 더하는 함수

```python
ListName = [ Element,Element,Element,Element ]  
sum(ListName)
```

```python
# For Example
>>> list1 = [1,2,4,7]
>>> print(sum(list1))
14          #1+2+4+7=14
```

<br>
<br>

***
>`sorted` : 정렬한 리스트를 구하는 함수  

```python
ListName = [ Element,Element,Element,Element ]  
sorted(ListName)
sorted(ListName, reverse=booltype)   #세부사항  
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

<br>

***
>`append` : 리스트의 끝에 요소를 추가

```python
ListName = [ Element,Element,Element,Element ]  
ListName.append(element)
```

```python
# For Example
>>> list1 = [1,2,4,7]
>>> list1.append(10)
>>> print(list1)
[1, 2, 4, 7, 10]
```


