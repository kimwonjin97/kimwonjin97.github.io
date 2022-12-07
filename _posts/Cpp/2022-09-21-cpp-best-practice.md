---
title:  "Cpp Best Practices"
excerpt: "No Discard"

categories:
  - Programming
  - Cpp
tags:
  - [Cpp]

toc: true
toc_sticky: true

date: 2022-12-06
last_modified_at: 2022-12-06
---

# Nodiscard 





```c++
struct Vector
{
  bool empty() const;
};

int main()
{
  Vector vec;
  vec.empty();
}
```

만약 위에 코드와 같이 리턴값에 대해 아무것도 하지 않았을때 이것은 버그일까? 

알고보면 이것은 c++ standard library 사용자들이 가장 흔히 마주하게되는 버그이다. 왜냐하면 사용자들은 Vector안에 있는 empty 함수가 비어있을때 true를 반환하길 기대하기 때문이다. 하지만 현실은 vec.empty()는 Vector가 비어있든 아니든 true를 반환한다. 

```c++
struct Vector
{
  [[nodiscard("You probably meant to call clear()")]]bool empty() const;
  void clear();
};

int main()
{
  Vector vec;
  vec.empty();
}
```

**따라서 어떠한 함수가 리턴값을 무시하고 호출했을때 버그인가를 생각하고 적절하게 nodiscard를 추가해주는것이 필요하다.**



<br>

타입또한 nodiscard로 마킹될 수 있다.

```c++
struct [[nodiscard("This is an error type!!")]] Error {};

Error do_thing();

int main()
{
	do_thing();
}
```

