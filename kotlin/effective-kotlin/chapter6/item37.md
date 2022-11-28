---
description: 이펙티브 코틀린 정리하기
---

# 데이터 집합 표현에 data 한정자를 사용하라

`data` 한정자를 붙이면, 다음과 같은 몇 가지 함수가 자동으로 생성된다.

- `toString`
- `equals`와 `hashCode`
- `copy`
    - immutable 데이터 클래스를 만들 때 편리하다. 기본 생성자 프로퍼티가 같은 새로운 객체를 복제한다. (객체를 얕은 복사한다.)
- `componentN`
    - 위치를 기반으로 객체를 해제할 수 있게 해준다.
    - 가장 큰 장점은 변수의 이름을 원하는 대로 지정할 수 있다는 것이다.
    - 하지만 위치를 잘못지정하면, 다양한 문제가 발생할 수 있어서 위험하기 때문에 기본 생성자에 붙어있는 프로퍼티 이름과 같은 이름을 사용하는 것이 좋다. (순서를 잘못 지정했을 때, IDE에서 경고를 준다.)
    - 값을 하나만 갖는 데이터 클래스는 읽는 사람에게 혼동을 줄 수 있기 때문에 해제하지 않는 것이 좋다.

```kotlin
data class User(
    val name: String
)

fun main() {
    val user = User("John")
    user.let { a -> print(a) } // User(name=John)

    // 이렇게 하지 마세요.
    user.let { (a) -> print(a) } // John
}
```

## 튜플 대신 데이터 클래스 사용하기

코틀린의 튜플은 `Serializable`을 기반으로 만들어지며, `toString`을 사용할 수 있는 제네릭 데이터 클래스이다. 예) `Pair`, `Triple`

튜플은 데이터 클래스와 같은 역할을 하지만, 튜플만 보고는 어떤 타입을 나타내는지 예측할 수 없다.

### 튜플을 사용하는 경우

- 값에 간단하게 이름을 붙일 때

```kotlin
val (description, color) = when {
    degrees < 5 -> "cold" to Color.BLUE
    degrees < 23 -> "mild" to Color.YELLOW
    else -> "hot" to Color.RED
}
```

- 표준 라이브러리에서 볼 수 있는 것처럼 미리 알 수 없는 aggregate(집합)를 표현할 때

```kotlin
val (odd, even) = numbers.partition { it % 2 == 1 }
val map = mapOf(1 to "San Francisco", 2 to "Amsterdam")
```

{% hint style="success" %}
위의 경우들을 제외하면 무조건 데이터 클래스를 사용하는 것이 좋다.
{% endhint %}