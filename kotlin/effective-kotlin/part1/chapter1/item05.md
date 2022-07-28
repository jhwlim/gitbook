---
description: 이펙티브 코틀린 정리하기
---

# 예외를 활용해 코드에 제한을 걸어라

{% hint style="success" %}
확실하게 어떤 형태로 동작해야 하는 코드가 있다면, 예외를 활용해 제한을 걸어주는 것이 좋다.
{% endhint %}

코드의 동작에 제한을 거는 방법

- `require()` : 아규먼트를 제한할 수 있다.
- `check()` : 상태와 관련된 동작을 제한할 수 있다.
- `assert()` : 어떤 것이 true인지 확인할 수 있다. 테스트 모드에서만 동작한다.
- `return` 또는 `throw`와 함께 활용하는 Elvis 연산자

장점

- 제한을 걸면 문서를 읽지 않는 개발자도 문제를 확인할 수 있다.
- 문제가 있을 경우에 함수가 예상하지 못한 동작을 하지 않고 예외를 `throw` 한다. 예상하지 못한 동자을 하는 것은 예외를 `throw` 하는 것보다 굉장히 위험하며, 상태를 관리하는 것이 굉장히 힘들다. 이러한 제한으로 인해서 문제를 놓치지 않을 수 있고, 코드가 더 안정적으로 작동하게 된다.
- 코드가 어느 정도 자체적으로 검사된다.
- 스마트 캐스트 기능을 활용할 수 있게 된다.

## `require()`는 아규먼트와 관련된 제한을 걸 때 사용할 수 있다.

조건을 만족하지 못할 때 무조건적으로 `IllegalArgumentException`을 발생시키므로 제한을 무시할 수 없다.

```kotlin
// 숫자를 아규먼트로 받아서 팩토리얼을 계산할 때, 숫자는 양의 정수여야 한다.
fun factorial(n: Int): Long {
    require(n >= 0)
    return if (n <= 1) 1 else factorial(n - 1) * n
}

// 좌표들을 아규먼트로 받아서 클러스터를 찾을 때, 좌표 목록은 비어있으면 안된다.
fun findClussters(points: List<Point>): List<Cluster> {
    require(points.isNotEmpty())
    // ...
}

// 사용자로부터 이메일 주소를 입력받을 때, 값이 있어야 하고 올바른 이메일 형식이어야 한다.
fun sendEmail(user: User, message: String) {
    requireNotNull(user.email)
    require(isValidEmail(user.email))
    // ...
}
```

람다를 활용해서 지연 메시지를 정의할 수 있다.

```kotlin
fun factorial(n: Int): Long {
    require(n >= 0) { "Cannot calculate factorial of $n because it is smaller than 0" }
    return if (n <= 1) 1 else factorial(n - 1) * n
}
```

{% hint style="info" %}
코드를 읽지 않는 사람이 있을 수도 있으므로 반드시 문서에도 이러한 제한이 있다고 별도로 표시해야 한다.
{% endhint %}

## `check()`는 상태와 관련된 동작을 제한할 수 있다.

상태가 올바른지 확인할 때 사용한다. `require()`과 비슷하지만, 지정된 예측을 만족하지 못할 때, `IllegalStateException`을 `throw`한다.

```kotlin
// 어떤 객체가 미리 초기화되어 있어야만 처리를 하게 하고 싶은 함수
fun speak(text: String) {
    check(isInitialized)
    // ...
}

// 사용자가 로그인했을 때만 처리를 하게 하고 싶은 함수
fun getUserInfo(): UserInfo {
    checkNotNull(token)
    // ...
}

// 객체를 사용할 수 있는 시점에 사용하고 싶은 함수
fun next() {
    check(isOpen)
    // ...
}
```

예외 메시지는 `require()`와 마찬가지로 지연 메시지를 사용해서 변경할 수 있다.

함수 전체에 대한 어떤 예측이 있을 때는 일반적으로 `require()` 뒤에 배치한다.

이러한 확인은 사용자가 규약을 어기고, 사용하면 안되는 곳에 함수를 호출하고 있다고 의심될 때 한다. 사용자가 코드를 제대로 사용할 거라고 믿고 있는 것보다는 항상 문제 상황을 예측하고, 문제 상황에 예외를 `throw`하는 것이 좋다.
