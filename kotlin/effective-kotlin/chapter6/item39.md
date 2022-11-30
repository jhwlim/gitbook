---
description: 이펙티브 코틀린 정리하기
---

# 태그 클래스보다는 클래스 계층을 사용하라

{% hint style="info" %}
상수(constant) 모드를 **태그**라고 부르며, 태그를 포함한 클래스를 **태그 클래스**라고 부른다.
{% endhint %}

태그 클래스는 다양한 문제를 내포하고 있다.

이러한 문제는 서로 다른 책임을 한 클래스에 태그로 구분해서 넣는다는 것에서 시작한다.

```kotlin
class ValueMatcher<T> private constructor(
    private val value: T? = null,
    private val matcher: Matcher
) {
    fun match(value: T?) = when(matcher) {
        Matcher.EQUAL -> value == this.value
        Matcher.NOT_EQUAL -> value != this.value
        Matcher.LIST_EMPTY -> value is List<*> && value.isEmpty()
        Matcher.LIST_NOT_EMPTY -> value is List<*> && value.isNotEmpty()
    }

    enum class Matcher {
        EQUAL,
        NOT_EQUAL,
        LIST_EMPTY,
        LIST_NOT_EMPTY
    }

    companion object {
        fun <T> equal(value: T) = ValueMatcher<T>(value = value, matcher = Matcher.EQUAL)

        fun <T> notEqual(value: T) = ValueMatcher<T>)(value = value, matcher = Matcher.NOT_EQUAL)

        fun <T> emptyList() = ValueMatcher<T>(matcher = Matcher.LIST_EMPTY)

        fun <T> notEmptyList() = ValueMatcher<T>(matcher = Matcher.LIST_NOT_EMPTY)
    }
}
```

위 코드의 단점

- 한 클래스에 여러 모드를 처리하기 위한 상용구(boilerplate)가 추가된다.
- 여러 목적으로 사용해야 하므로 프로퍼티가 일관적이지 않게 사용될 수 있으며, 더 많은 프로퍼티가 필요하다.
- 요소가 여러 목적을 가지고, 요소를 여러 방법으로 설정할 수 있는 경우에는 상태의 일관성과 정확성을 지키기 어렵다.
- 팩토리 메서드를 사용해야 하는 경우가 많다.

코틀린은 그래서 일반적으로 태그 클래스보다 sealed 클래스를 많이 사용한다. 한 클래스에 여러 모드를 만드는 방법 대신에, 각각의 모드를 여러 클래스에 만들고 타입 시스템과 다형성을 활용하는 것이다. 그리고 이러한 클래스에는 `sealed` 한정자를 붙여서 서브 클래스 정의를 제한한다.

```kotlin
sealed class ValueMatcher<T> {
    abstract fun match(value: T): Boolean

    class Equal<T>(
        val value: T
    ) : ValueMatcher<T>() {
        override fun match(value: T): Boolean = value == this.value
    }

    class NotEqual<T>(
        val value: T
    ) : ValueMatcher<T>() {
        override fun match(value: T): Boolean = value != this.value
    }

    class EmptyList<T>() : ValueMatcher<T>() {
        override fun match(value: T) = value is List<*> && value.isEmpty()
    }

    class NotEmptyList<T>() : ValueMatcher<T>() {
        override fun match(value: T) = value is List<*> && value.isNotEmpty()
    }

}
```

위 코드의 장점

- 책임이 분산되어 훨씬 깔끔하다.
- 각각의 객체들은 자신에게 필요한 데이터만 있으며, 적절한 파라미터만 갖는다.

이와 같이 계층을 사용하면, 태그 클래스의 단점을 모두 해소할 수 있다.

## sealed 한정자

`sealed` 한정자는 외부 파일에서 서브클래스를 만드는 행위 자체를 모두 제한한다. 외부에서 추가적인 서브클래스를 만들 수 없으므로, 타입이 추가되지 않을 거라는 게 보장된다. 따라서 `when`을 사용할 때 `else` 브랜치를 따로 만들 필요가 없다. 이러한 장점을 이용해서 새로운 기능을 쉽게 추가할 수 있으며, `when` 구문에서 이를 처리하는 것을 잊어버리지 않을 수도 있다.

`when`은 모드를 구분해서 다른 처리를 만들 때 굉장히 편리하다. 예를 들어 어떤 처리를 각각의 서브클래스에 구현할 필요 없이, when을 활용하는 확장 함수로 정의하면 한번에 구현할 수 있다.

```kotlin
fun <T> ValueMatcher<T>.resersed(): ValueMatcher<T> = when (this) {
    is ValueMatcher.EmptyList -> ValueMatcher.NotEmptyList<T>()
    is ValueMatcher.NotEmptyList -> ValueMatcher.EmptyList<T>()
    is ValueMatcher.Equal -> ValueMatcher.NotEqual(value)
    is ValueMatcher.NotEqual -> ValueMatcher.Equal(value)
}
```

반드시 `sealed` 한정자를 사용해야 하는 것은 아니다. 대신 `abstract` 한정자를 사용할 수도 있다.

하지만 `abstract` 키워드를 사용하면, 다른 개발자가 새로운 인스턴스를 만들어서 사용할 수도 있다. 이러한 경우에는 함수를 `abstract`로 선언하고, 서브클래스 내부에 구현해야 한다. `when`을 사용하면, 프로젝트 외부에서 새로운 클래스가 추가될 때 함수가 제대로 동작하지 않을 수 있기 때문이다.

`sealed` 한정자를 사용하면, 확장 함수를 사용해서 클래스에 새로운 함수를 사용해서 클래스에 새로운 함수를 추가하거나, 클래스의 다양한 변경을 쉽게 처리할 수 있다.

`abstract` 클래스는 계층에 새로운 클래스를 추가할 수 있는 여지를 남긴다. 

{% hint style="success" %}
클래스의 서브 클래스를 제어하려면, `sealed` 한정자를 사용해야 한다. `abstract`는 상속과 관련된 설계를 할 때 사용한다.
{% endhint %}

## 태그 클래스와 상태 패턴의 차이

### 상태 패턴

객체의 내부 상태가 변화할 때, 객체의 동작이 변하는 소프트웨어 디자인 패턴

프런트엔드 각각 MVC, MVP, MVVM 아키텍처에서 컨트롤러(controller), 프레젠터(presenter), 뷰(view) 모델을 설계할 때 많이 사용된다. 

상태 패턴을 사용한다면, 서로 다른 상태를 나타내는 클래스 계층 구조를 만들게 된다. 그리고 현재 상태를 나타내기 위한 읽고 쓸 수 있는 프로퍼티도 만들게 된다.

```kotlin
sealed class WorkoutState

class PrepareState(
    val exercise: Exercise
) : WorkoutState()

class ExerciseState(
    val exercise: Exercise
) : WorkoutState()

object DoneState : WorkoutState()

fun List<Exercise>.toStates(): List<WorkoutState> = flatMap { exercise -> 
    listOf(PrepareState(exercise), ExerciseState(exercise)) 
} + DoneState

class WorkoutPresenter( /* ... */ ) {
    private var state: WorkoutState = states.first()
}
```

차이점

- 상태는 더 많은 책임을 가진 큰 클래스이다.
- 상태는 변경할 수 있다.

구체 상태는 객체를 활용해서 표현하는 것이 일반적이며, 태그 클래스보다는 sealed 클래스 계층으로 만든다. 또한 이를 immutable 객체로 만들고 변경해야 할 때마다 `state` 프로퍼티를 변경하게 만든다. 그리고 뷰에서 이러한 `state`의 변화를 관찰한다.

```kotlin
private var state: WorkoutState by
    Delegates.observable(states.first()) { _, _, _ -> 
        updateView()
    }
```

## 정리

코틀린에서는 태그 클래스보다 타입 계층을 사용하는 것이 좋다. 그리고 일반적으로 이러한 타입 계층을 만들 때는 sealed 클래스를 사용한다.

타입 계층과 상태 패턴은 실질적으로 함께 사용하는 협력 관계라고 할 수 있다. 하나의 뷰를 가지는 경우보다는 여러 개의 상태로 구분할 수 있는 뷰를 가질 때 많이 활용된다.