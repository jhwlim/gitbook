---
description: 이펙티브 코틀린 정리하기
---

# 연산자 오버로드를 할 때는 의미에 맞게 사용하라

코틀린에서 각 연산자의 의미는 항상 같게 유지된다. 이는 매우 중요한 설계 결정이다. 예를 들어 `+` 연산자가 일반적인 의미로 사용되지 않고 있다면, 연산자를 볼 때마다 연산자를 개별적으로 이해해야 하기 때문에 코드를 이해하기 어려울 것이다.

**모든 연산자는 구체적인 이름을 가진 함수이며, 이름이 나타내는 역할을 할 거라고 기대된다.**

## 분명하지 않은 경우

의미가 명확하지 않은 경우, **infix를 활용한 확장 함수를 사용하는 것이 좋다.** 일반적인 이항 연산자 형태처럼 사용할 수 있다.

```kotlin
infix fun Int.timesRepeated(operation: () -> Unit) = {
    repeat(this) { operation() }
}

val tripledHello = 3 timesRepeated { print("Hello") }
tripledHello() // HelloHelloHello 
```

**톱레벨 함수를 사용하는 것도 좋은 방법이다.**

## 규칙을 무시해도 되는 경우

도메인 특화 언어(DSL, Domain Specific Language)를 설계할 때, 연산자 오버로딩 규칙을 무시해도 된다.

```kotlin
body {
    div {
        +"Some text" // unaryPlus
    }
}
```

## 정리

{% hint style="success" %}
연산자 오버로딩은 그 이름의 의미에 맞게 사용하자.
{% endhint %}

연산자 의미가 명확하지 않다면, 연산자 오버로딩을 사용하지 않는 것이 좋다. 대신 이름이 있는 일반 함수를 사용하는 것이 좋다. 연산자 같은 형태로 사용하고 싶다면, infix 확장 함수 또는 톱레벨 함수를 활용하는 것이 좋다.