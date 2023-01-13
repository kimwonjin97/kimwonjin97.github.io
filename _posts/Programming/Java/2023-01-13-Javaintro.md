---
title:  "Introduction to Java"
excerpt: "Important basic concepts in Java"

categories:
  - Programming
tags:
  - [OOP, Java]

toc: true
toc_sticky: true

date: 2023-01-13
last_modified_at: 2022-01-13
---

### Java 소개

1991년에 첫 등장, 썬 마이크로시스템즈의 제임스 고슬링과 동료들이 개발.

#### Java개발 배경

원래목적: 셋톱 박스 등의 임베디드 시스템에서 사용하기 위해
임베디드 시스템은 다양한 기기와 OS를 가지고 있음
기존 언어는 각 플랫폼마다 컴파일을 해야했음.
**한번만 빌드하면 어떤 플랫폼에서든 작동하는 언어를 만들게 됨.**

하지만 갑자기 고속 성장하던 인터넷에 맞춰 그쪽으로 방향을 바꾸며 대중화
**매니지드** 언어기에 메모리 관리를 덜 신경써도됨
따라서 기계와 아주 가깝지 않은 개념을 코드로 옮기기에 사용하기 적합

## Lecture2: Java 기본문법

### 메인 함수:

```java
public class HelloWorld
{
	public static void main(String[] args)
	{
		System.out.println("Hello POCU");
	}
}
```

`<접근 제어자> class <클래스명> { ... }`

- Java 에서는 언제나 클래스가 필요함.
- 한 .java 파일에는 최고레벨 public 클래스가 하나만 있어야함

#### 내포 클래스

1. 클래스 안에 다른 클래스를 넣을수 있음
   - 안에 있는 클래스를 내포(nested) 클래스라 부름
   - 중첩 클래스, 내부 클래스라고도 부름
2. 내포 클래스는 public 이어도 상관 없음

#### main 함수

- 프로그램의 시작점(entry point)
- `public static void main(String[] args)`
  - 반드시 이 시그내처(signature) 대로 main 함수를 

### 출력문과 가변인자

#### System.out.println("Hello World");

```java
System.out.println(12345);
System.out.println("Your Score is" + 100);
```

- 표준 출력(standard output)으로 한 줄을 출력하는 함수
  - ln은 line을 의미
- 문자열 뿐만 아니라 숫자도 출력 -> 함수오버로딩

System은 클래스
out은 system클래스의 정적(static) 맴버 변수
	- 이때 out 의 자료형은 `PrintStream` 클래스, 즉 out은 개체(object), PrintStream은 Java의 표준 출력 스트림

Java에도 printf()가 있다
`public PrintStream printf(String format, Object... args);

이때 `Object... args` 은 가변인자다. c와 다르게 ...앞에 <자료형>을 반드시 넣어야한다. 해당 자료형의 데이터만 이자로 전달 가능하다. 하지만 Object라는 자료형을 쓸 경우 모든 자료형을 넣을 수 있다.


```java
String name = "Mumu";
int score = 64;
System.out.printf("%s's score: %d\n", name, score);
```

원래 println()만 있다가 나중에 추가되었다.

새줄 문자를 반환하는 매서드는 플래폼마다 다르다.

- Linux: \n반환
- Windows: \r\n 반환

이는 `public static String lineSeparator();`를 통해서 다음과 같이 해결 가능하다.

`System.out.printf("%s's score: %d%s", name, score, System.lineSeparator());` 

### Package

```java
package <패키지 경로>;
```

- 연관된 클래스들 끼리 묶는 기법.
- 마지 디스크 상의 폴더와 같은 역할
  - e.g. 사진 폴더는 사진파일에, 동영상 폴더는 동영상 파일에 저장

#### 패키지 종류

- 자바 기본(built-in) 패키지
  - 이름이 java로 시작하는 패키지들(java.lang, java.util);

- 프로그래머가 직접 만든(user-defined) 패키지

#### 패키지의 목적

이름 충돌 문제를 피할 수 있게 해줌
다만 패키 이름의 중복을 최소화해야함
그래서 보통 회사의 도메인 명을 패키지 이름에 사용(단, 역순으로)

- e.g., 삼성(samsung.com)에서 패키지를 만들 경우 -> `package com.samsung.<패키지 이름>;`

```java
package code.sample;

//이 코드가 위에있는 패키지에 속한다는 뜻 
public class HelloWorld
{
	public static void main(String[] args)
	{
		System.out.println("Hello POCU");
	}
}
```

> 주의: 패키지 이름만 적으면 안됨, 패키지명과 똑같은 폴더 트리에 .java 파일을 넣어야함 (IDE 사용하면 알아서 해줌)