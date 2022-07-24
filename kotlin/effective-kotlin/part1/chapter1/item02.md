---
description: 이펙티브 코틀린 정리하기
---

# 변수의 스코프를 최소화하라

상태를 정의할 때는 변수와 프로퍼티의 스코프를 최소화하는 것이 좋다.

- 프로퍼티보다는 지역 변수를 사용하는 것이 좋다.
- 최대한 좁은 스코프를 갖게 변수를 사용하는 것이 좋다.

{% hint style="info" %}
요소의 **스코프**란?

요소를 볼 수 있는(visible) 컴퓨터 프로그램 영역이다. 코틀린의 스코프는 기본적으로 중괄호로 만들어지며, 내부 스코프에서 외부 스코프에 있는 요소에만 접근할 수 있다.
{% endhint %}

```kotlin
// bad
var user: User
for (i in users.indices) {
    user = users[i]
    // ...
}
```

```kotlin
// better
for (i in users.indices) {
    val user = users[i]
    // ...
}
```

```kotlin
// best
for ((i, user) in users.withIndex()) {
    // ...
}
```

<br>

스코프를 좁게 만드는 것이 좋은 가장 중요한 이유는 **프로그램을 추적하고 관리하기 쉽기 때문이다.** 좁은 스코프에 걸쳐 있을수록, 그 변경을 추적하는 것이 쉽다. 이렇게 추적이 되어야 코드를 이해하고 변경하는 것이 쉽다.

{% hint style="success" %}
변수는 읽기 전용 또는 일고 쓰기 전용 여부와 상관 없이, 변수를 정의할 때 초기화되는 것이 좋다. if, when, try~catch, Elvis 표현식 등을 활용하면, 최대한 변수를 정의할 때 초기화할 수 있다.
{% endhint %}

```kotlin
// bad
val user: User
if (hasValue) {
    user = getValue()
} else {
    user = User()
}
```

```kotlin
// good
val user: User = if(hasValue) getValue() else User()
```

{% hint style="success" %}
여러 프로퍼티를 한꺼번에 설정해야 하는 경우에는 구조분해 선언을 활용하는 것이 좋다.
{% endhint %}

```kotlin
// bad
fun updateWeather(degrees: Int) {
    val description: String
    val color: Int
    if (degrees < 5) {
        description = "cold"
        color = Color.BLUE
    } else if (degrees < 23) {
        description = "mide"
        color = Color.YELLO
    } else {
        description = "hot"
        color = Color>RED
    }
}
```

```kotlin
// good
fun updateWeather(degrees: Int) {
    val (description, color) = when {
        degrees < 5 -> "cold" to Color.BLUE
        degrees < 23 -> "mild" to Color.YELLOW
        else -> "hot" to Color.RED
    }
}
```

## 캡처링

### 에라토스테네스의 체 구현하기

1. 2부터 시작하는 숫자 리스트를 만든다.
2. 첫 번째 요소를 선택한다. 이는 소수이다.
3. 남아 있는 숫자 중에서 2번에서 선택한 소수로 나눌 수 있는 모든 숫자를 제거한다.

```kotlin
var numbers = (2..30).toList()
val primes = mutableListOf<Int>()
while (numbers.isNotEmpty()) {
    val prime = numbers.first()
    primes.add(prime)
    numbers = numbers.filter { it % prime != 0 }
}
println(primes) // [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```

### 시퀀스를 활용하여 구현하기

```kotlin
val primes: Sequence<Int> = sequence {
    var numbers = generateSequence(2) { it + 1 }
    while (true) {
        val prime = numbers.first()
        yield(prime)
        numbers = numbers.drop(1)
            .filter { it % prime != 0 }
    }
}
println(primes.take(10).toList()) // [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```

### 캡처 문제가 발생하는 경우

```kotlin
val primes: Sequence<Int> = sequence {
    var numbers = generateSequence(2) { it + 1 }

    var prime: Int // 반복문에 진입하기 전에 한 번만 생성한다.
    while (true) {
        prime = numbers.first()
        println(prime)
        yield(prime)
        numbers = numbers.drop(1)
            .filter { it % prime != 0 }
    }
}
println(primes.take(10).toList()) // [2, 3, 5, 6, 7, 8, 9, 10, 11, 12]
```

위와 같이 실행 결과가 다르게 나오는 이유는 시퀀스를 활용하므로 필터링이 지연되기 때문이다. 따라서 최종적인 prime 값으로만 필터링 된다. 즉, prime이 2로 설정되어 있을 때 필터링된 4만 제외되고 그냥 연속된 숫자가 나와버린다.

이러한 문제가 발생할 수 있으므로, **항상 잠재적인 캡처 문제를 주의해야 한다. 가변성을 피하고 스코프 범위를 좁게 만들면, 이런 문제를 간단하게 피할 수 있다.**

## 정리

- 변수의 스코프는 좁게 만들어서 활용하는 것이 좋다.
- `var`보다는 `val`을 사용하는 것이 좋다.

## 더 알아보기

- 시퀀스
- 지연 계산
