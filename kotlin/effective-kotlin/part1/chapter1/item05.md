---
description: 이펙티브 코틀린 정리하기
---

# 예외를 활용해 코드에 제한을 걸어라

{% hint style="success" %}
확실하게 어떤 형태로 동작해야 하는 코드가 있다면, 예외를 활용해 제한을 걸어주는 것이 좋다.
{% endhint %}

### 코드의 동작에 제한을 거는 방법

- `require()` : 아규먼트를 제한할 수 있다.
- `check()` : 상태와 관련된 동작을 제한할 수 있다.
- Assert 계열 함수 : 어떤 것이 true인지 확인할 수 있다. 테스트 모드에서만 동작한다.
- `return` 또는 `throw`와 함께 활용하는 Elvis 연산자

### 장점

- 제한을 걸면 문서를 읽지 않는 개발자도 문제를 확인할 수 있다.
- 문제가 있을 경우에 함수가 예상하지 못한 동작을 하지 않고 예외를 `throw` 한다. 예상하지 못한 동작을 하는 것은 예외를 `throw` 하는 것보다 굉장히 위험하며, 상태를 관리하는 것이 굉장히 힘들다. 이러한 제한으로 인해서 문제를 놓치지 않을 수 있고, 코드가 더 안정적으로 작동하게 된다.
- 코드가 어느 정도 자체적으로 검사된다.
- **스마트 캐스트를 활용할 수 있다.**

## `require()`

아규먼트와 관련된 제한을 걸 때 사용한다.

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

## `check()`

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

## Assert 계열 함수

구현 문제로 발생할 수 있는 추가적인 문제를 예방하려면 단위 테스트를 사용하는 것이 좋다. 하지만 한 경우만 테스트 해서 모든 상황에서 괜찮은지는 알 수 없다. 

함수 내부에서 Assert 계열의 함수를 사용해서 모든 케이스에서 제대로 동작하는지 확인할 수 있다. 이러한 조건은 현재 코틀린/JVM에서만 활성화되며, -ea JVM 옵션을 활성화해야 확인할 수 있다. 다만 **프로덕션 환경에서는 오류가 발생하지 않는다. 테스트를 할 때만 활성화되므로, 오류가 발생해도 사용자가 알아차릴 수 없다.**

{% hint style="success" %}
만약 정말 심각한 오류고, 심각한 결과를 초래할 수 있는 경우에는 `check()`를 사용하는 것이 좋다.
{% endhint %}

### 장점

- 코드를 자체 점검하며, 더 효율적으로 테스트할 수 있다.
- 특정 상황이 아닌 모든 상황에 대한 테스트를 할 수 있다.
- 실행 시점에 정확하게 어떻게 되는지 확인할 수 있다.
- 실제 코드가 더 빠른 시점에 실패하게 만든다. 따라서 예상하지 못한 동작이 언제 어디서 실행되었는지 쉽게 찾을 수 있다.

{% hint style="warning" %}
Assert 계열의 함수를 사용해도 여전히 단위 테스트는 따로 작성해야 한다.
{% endhint %}

## 스마트 캐스팅와 nullability

코틀린에서 `require()`와 `check()`는 어떤 조건을 확인해서 true가 나왔다면 해당 조건은 이후로도 true라고 가정한다. 

따라서 이를 활용해서 타입 비교를 했다면 스마트 캐스팅된다.

```kotlin
fun changeDress(person: Person) {
    require(person.outfit is Dress)
    val dress: Dress = person.outfit // 스마트 캐스팅
    // ...
}
```

이러한 특정은 어떤 대상이 null인지 확인할 때 유용하다. 

```kotlin
class Person(val email: String?)
fun validateEmail(email: String) { ... }

fun sendEmail(person: Person, message: String) {
    require(person.email != null)
    validateEmail(person.email) // 스마트 캐스팅
    // ...
}
```

`requireNotNull()`, `checkNotNull()`을 사용할 수도 있다. 스마트 캐스팅되기 때문에 변수를 언팩(unpack)하는 용도로 활용할 수 있다.

```kotlin
fun sendEmail(person: Person, message: String) {
    requireNotNull(person.email)
    validateEmail(person.email) // 스마트 캐스팅
    // ...
}
```

### Elvis 연산자

nullability를 목적으로 `throw` 또는 `return`과 함께 Elvis 연산자를 활용할 수 있다. 이러한 코드는 굉장히 읽기 쉽고, 유연하게 사용할 수 있다.

```kotlin
fun sendEmail(person: Person, message: String) {
    val email: String = person.email ?: return // person.email이 null 이라면 sendEmail()를 중지한다.
    // ...
}
```

프로퍼티에 문제가 있어서 null일 때 여러 처리를 해야할 때도, `return` 또는 `throw`와 `run()`을 조합하여 활용할 수 있다.

```kotlin
fun sendEmail(person: Person, text: String) {
    val email: String = person.email ?: run { 
        log("Email not sent, no email address) // person.email이 null 이라면 원인을 로그에 출력하고 sendEmail()를 중지한다.
        return
    }
    // ...
}
```

{% hint style="success" %}
`return`과 `throw`를 활용한 Elvis 연산자는 nullable을 확인할 때 굉장히 많이 사용되는 관용적인 방법으로 적극적으로 활용하는 것이 좋다.

또한, 이러한 코드는 함수의 앞부분에 넣어서 잘 보이게 만드는 것이 좋다.
{% endhint %}
