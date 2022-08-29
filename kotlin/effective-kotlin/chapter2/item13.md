---
description: 이펙티브 코틀린 정리하기
---

# Unit?을 리턴하지 말라

Unit?은 Unit 또는 null 이라는 값을 가질 수 있습니다.

따라서 Boolean과 Unit? 타입은 서로 바꿔서 사용할 수 있다.

```kotlin
fun keyIsCorrect(key: String): Boolean { ... }

if (!keyIsCorrect(key)) return
```

다음 코드처럼 사용할 수 있다.

```kotlin
fun verifyKey(key: String): Unit? { ... }

verifyKey(key) ?: return
```

이러한 트릭은 코드를 작성할 때는 멋있게 보일 수도 있겠지만, 읽을 때는 그렇지 않다. Unit?으로 Boolean을 표현하는 것은 오해의 소지가 있으며, 예측하기 어려운 오류를 만들 수 있다.

{% hint style="success" %}
Unit?은 오해를 불러 일으키기 쉽다. 따라서 Boolean을 사용하는 형태로 변경하는 것이 좋다.

기본적으로 Unit?을 리턴하거나, 이를 기반으로 연산하지 않는 것이 좋다.
{% endhint %}
