---
description: 이펙티브 코틀린 정리하기
---

# 연산 또는 액션을 전달할 때는 인터페이스 대신 함수 타입을 사용하라

{% hint style="info" %}
대부분의 프로그래밍 언어에는 함수 타입이라는 개념이 없다. 그래서 연산 또는 액션을 전달할 때 메서드가 하나만 있는 인터페이스를 활용한다. 이러한 인터페이스를 **SAM(Single-Abstract Method)**이라고 부른다.
{% endhint %}

SAM 대신에 함수 타입을 사용하는 코드로 변경하면 더 많은 자유를 얻을 수 있다.

다음과 같은 방법으로 파라미터를 전달할 수 있다.

- 람다 표현식 또는 익명 함수로 전달

```kotlin
setOnClickListener { ... }
setOnClickListener(fun(view) { ... })
```

- 함수 레퍼런스 또는 제한된 함수 레퍼런스로 전달

```kotlin
setOnClickListener(::println)
setOnClickListener(this::showUsers)
```

- 선언된 함수 타입을 구현한 객체로 전달

```kotlin
class ClickListener: (View) -> Unit {
    override fun invoke(view: View) { ... }
}

setOnClickListener(ClickListener())
```

타입 별칭(type aliiase)을 사용하면, 함수 타입도 이름을 붙일 수 있다.

```kotlin
typealias OnClick = (View) -> Unit
```

파라미터도 이름을 가질 수 있다. 이름을 붙이면, IDE의 지원을 받을 수 있다는 장점이 있다.

```kotlin
fun setOnClickListener(listener: OnClick) { ... }
typealias OnClick = (view: View) -> Unit
```

람다 표현식을 사용할 때는 아규먼트 분해도 사용할 수 있다. API를 소비하는 사용자의 관점에서는 함수 타입을 따로따로 갖는 것이 훨씬 사용하기 쉽다.

```kotlin
class CalendarView {
    var onDateClicked: ((date: Date) -> Unit)? = null
    var onPageChanged: ((date: Date) -> Unit)? = null
}
```

{% hint style="success" %}
인터페이스를 사용해야 하는 특별한 이유가 없다면, 함수 타입을 활용하는 것이 좋다.
{% endhint %}

## 언제 SAM을 사용해야 할까?

코틀린이 아닌 다른 언어에서 사용할 클래스를 설계할 때, 함수 타입으로 만들어진 클래스는 자바에서 타입 별칭과 IDE의 지원 등을 제대로 받을 수 없다.

다른 언어(자바 등)에서 코틀린의 함수 타입을 사용하려면, Unit을 명시적으로 리턴하는 함수가 필요하다.

```kotlin
// 코틀린
class CalendarView() {
    var onDateClicked: ((date: Date) -> Unit)? = null
    var onPageChanged: OnDateClicked? = null
}

interface OnDateClicked {
    fun onClick(date: Date)
}
```

```java
// 자바에서 사용할 때
CalendarView c = new CalendarView();
c.setOnDateClicked(date -> Unit.INSTANCE);
c.setOnDateChanged(date -> {});
```

{% hint style="success" %}
자바에서 사용하기 위한 API를 설계할 때는 함수 타입보다 SAM을 사용하는 것이 합리적이다.
{% endhint %}
