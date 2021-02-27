---
layout: article
title: Python String, Stack 다루기
aside:
    toc: false
---

알고리즘 공부할 겸 배우고 있는 파이썬. 리트 코드에서 쉬운 문제 먼저 풀어 보고 있다. 그 과정에서 오늘 배운 내용들.<br/>

### number(`int`, `float`, ...) to string
```python
num = 12.25
string = str(num)
```

### string 뒤집기
[w3school](https://www.w3schools.com/python/python_howto_reverse_string.asp) 설명에 따르면, <br/>
> the slice statement `[::-1]` means start at the end of the string and end at position 0, move with the step -1, negative one, which means one step backwards.

문자열의 끝 인덱스부터 시작해서 시작 포지션 0 인덱스까지 -1 스텝씩 뒤로 움직인다는 뜻.

```python
original = "original_string"
result = original[::-1]
```

### string `list` 문자열 길이를 기준으로 정렬
```python
strs = ["flower","flow","flight"]
strs.sort(key=lambda x:len(x))
```
파이썬 `lambda` 활용은 무궁무진하던데... 이렇게 한 번씩 정리해야겠다.

### `list`가 비어 있는지 확인
명시적으로 `isEmpty()`와 같은 메소드가 있지는 않나 보다.
+ 대신 아래와 같이 길이로 확인하거나,
```python
if len(example) == 0:
    print("List is empty.")
```
+ 아래의 방법으로 확인한다.
```python
if not example:
    print("List is empty.")
```

두 번째 방법이 파이써닉(pyhonic)한 방법이라고 한다. 가끔 파이써닉, 즉 파이썬답다는 표현을 볼 때가 있는데 마침 [TCP school](http://www.tcpschool.com/python2018/python_intro_feature)에 이를 정리한 글이 있다.

### 파이썬에서 `stack` 다루기
파이썬에는 따로 `stack` 자료형이 없다. 대신 `list`를 `stack`처럼 사용할 수 있다.
```python
stack = []
stack.append('9')
stack.top()
stack.pop()
```


<!--more-->
### 참고자료
본문 속 링크 참조
