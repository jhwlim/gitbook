---
description: 이펙티브 코틀린 정리하기
---

# 적절하게 null을 처리하라

null은 값이 부족하다는 것을 나타낸다. 프로퍼티가 null이라는 것은 값이 설정되지 않았거나, 제거되었다는 것을 나타낸다.

null은 최대한 명확한 의미를 갖는 것이 좋다.

기본적으로 nullable 타입은 3가지 방법으로 처리한다.

- 안전 호출(`.?`), 스마트 캐스팅, Elvis 연산자(`?:`) 등을 활용한다.
- 오류를 `throw` 한다.
- 함수 또는 프로퍼티를 리팩터링해서 nullable 타입이 나오지 않도록 바꾼다.

## null을 안전하게 처리하기

### 안전 호출과 스마트 캐스팅 사용하기

null을 안전하게 처리하는 방법 중 널리 사용되는 방법으로는 안전 호출과 스마트 캐스팅이 있다.

```kotlin
// 안전 호출
printer?.print() 

// 스마트 캐스팅
if (printer != null) {
    printer.print() 
}
```

스마트 캐스팅은 코틀린의 규약 기능을 지원한다.

```kotlin
println("What is your name?")
val name = readLine()
if (!name.isNullOrBlank()) {
    println("Hello ${name.toUpperCase()})
}

val news: List<News>? = getNews()
if (!news.isNullOrEmpty()) {
    news.forEach { notifyUser(it) }
}
```

### Elvis 연산자 사용하기

다른 방법으로는 Elvis 연산자를 사용하는 것이다. Elvis 연산자는 오른쪽에 `return` 또는 `throw` 을 포함한 모든 표현식이 허용된다.

```kotlin
val printerName1 = printer?.name ?: "Unnamed"
val printerName2 = printer?.name ?: return
val printerName3 = printer?.name ?: throw Error("Printer must be named")
```

### 방어적 프로그래밍과 공격적 프로그래밍

**방어적 프로그래밍**은 프로덕션 환경으로 들어갔을 때 발생할 수 있는 수많은 것들로부터 프로그램을 방어해서 안정성을 높이는 방법을 나타내는 포괄적인 용어이다. 상황을 처리할 수 있는 올바른 방법이 있을 때는 굉장히 좋다.

**공격적 프로그래밍**은 예상하지 못한 상황이 발생했을 때, 이러한 문제를 개발자에게 알려서 수정하게 만드는 것이다.

**둘은 코드의 안전을 위해 모두 필요하다. 둘을 모두 이해하고 적절하게 사용할 수 있어야 한다.**
