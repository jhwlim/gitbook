---
description: 이펙티브 코틀린 정리하기
---

# 멤버 확장 함수의 사용을 피하라

어떤 클래스에 대한 확장 함수를 정의할 때, 이를 멤버로 추가하는 것은 좋지 않다.

```kotlin
class PhoneBookIncorrect {
    fun String.isPhoneNumber(): Boolean = length == 7 && all { it.isDigit() }
}
```

{% hint style="warning" %}
DSL을 만들 때를 제외하면 이를 사용하지 않는 것이 좋다. 
{% endhint %}

## 멤버 확장을 피해야 하는 이유

- 가시성을 제한하지 못하고, 단순하게 확장 함수를 사용하는 형태를 어렵게 만들 뿐이다. 

{% hint style="success" %}
확장 함수의 가시성을 제한하고 싶다면, 멤버로 만들지 말고, 가시성 한정자를 붙여 주면 된다.
{% endhint %}

```kotlin
private fun String.isPhoneNumber() = length == 7 && all { it.isDigit() }
```

- 레퍼런스를 지원하지 않는다.

```kotlin
val refX = PhoneBookIncorrect::isPhoneNumber // 오류

val book = PhoneBookIncorrect()
val boundedRefX = book::isPhoneNumber // 오류
```

- 암묵적 접근을 할 때, 두 리시버 중에 어떤 리시버가 선택될지 혼동된다.

```kotlin
class A {
    val a = 10
}

class B {
    val a = 20
    val b = 30

    fun A.test() = a + b // 40일까요? 50일까요?
}
```

- 확장 함수가 외부에 있는 다른 클래스를 리시버로 받을 때, 해당 함수가 어떤 동작을 하는지 명확하지 않다.

```kotlin
class A {
    // ...
}

class B {
    // ...

    fun A.update() = ... // A 와 B 중에서 어떤 것을 업데이트할까요?
}
```

## 정리

멤버 확장 함수를 사용하는 것이 의미가 있는 경우에는 사용해도 괜찮다. 하지만 일반적으로는 그 단점을 인지하고, 사용하지 않는 것이 좋다. 가시성을 제한하려면, 가시성을 제한하는 확장자를 사용하는 것이 좋다.