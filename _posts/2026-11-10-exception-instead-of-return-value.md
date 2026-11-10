---
title: 실패를 return 0으로 알리면 생기는 문제
date: 2026-11-10 22:00:00 +0900
categories: [Java, Exception]
tags: [java, exception, try-catch]
---

## 1. 문제 상황
계산기를 만들면서 계산은 Calculator가, 출력은 App이 맡도록 나눴다.
0으로 나누면 안내 메시지만 출력하고, 계산은 else로 건너뛰도록 설계했다.
그런데 실행해 보니 메시지 다음에 계산 결과까지 출력됐다.

```
두번째 정수에 0이 입력될 수 없습니다.
10 / 0 = 0
```

else로 막았으니 결과 줄은 나오지 않을 거라고 생각했다.

## 2. 원인 — 실패와 정상 결과를 구분할 수 없는 return 값
이 현상은 Calculator가 실패를 따로 알리지 않고 0을 return하기 때문에 발생한다.

- Calculator: else 덕분에 계산은 건너뛰지만, 메서드는 끝까지 실행되어 result의 처음 값 0을 return한다.
- App: 받은 값이 실패인지 모른 채 그대로 출력한다.

```java
// Calculator
int result = 0;
if (num2 == 0) {
    System.out.println("두번째 정수에 0이 입력될 수 없습니다.");
} else {
    result = num1 / num2;
}
return result;   // 0으로 나누면 처음 값 0이 그대로 return된다

// App
int result = calculator.calculate(num1, num2, operator);
System.out.println(num1 + " " + operator + " " + num2 + " = " + result);   // 조건 없이 출력
```

예를 들어 `5 - 5`의 결과도 0이므로, App 입장에서는 실패의 0과 정상 결과 0을 구분할 수 없다.

## 3. 해결 — return 대신 예외로 알리기
Calculator에서 0으로 나누면 return 대신 ArithmeticException을 던진다.
예외가 나면 메서드는 그 자리에서 멈추고 return까지 가지 않는다.

App은 try-catch로 잡는다.
예외가 나면 try 안의 결과 출력 줄은 건너뛰고, catch에서 메시지만 출력한다.

```java
// Calculator
if (num2 == 0) {
    throw new ArithmeticException("두번째 정수에 0이 입력될 수 없습니다.");
}

// App
try {
    int result = calculator.calculate(num1, num2, operator);
    System.out.println(num1 + " " + operator + " " + num2 + " = " + result);
} catch (ArithmeticException e) {
    System.out.println(e.getMessage());
}
```

고친 뒤 `10 / 0`을 넣으면 결과 줄 없이 메시지만 출력된다.

```
두번째 정수에 0이 입력될 수 없습니다.
```

## 4. 배운 점
- 실패를 return 값으로 알리면, 받는 쪽은 실패와 정상 결과를 구분할 수 없다.
- 실패는 return 대신 예외로 알려야 한다.
- 결과가 이상하면 출력한 곳만 보지 말고, 그 값이 어디서 와서 어디로 갔는지 따라가 봐야 한다.
