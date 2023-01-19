---
description: 이펙티브 코틀린 정리하기
---

# 함수 타입 파라미터를 갖는 함수에 inline 한정자를 붙여라

`inline` 한정자의 역할은 컴파일 시점에 '함수를 호출하는 부분'을 '함수의 본문'으로 대체하는 것이다.

```kotlin
repeat(10) {
    print(it)
}

// 컴파일 시점
for (index in 0 until 10) {
    print(index)
}
```

일반적인 함수를 호출하면 함수 본문으로 점프하고, 본문의 모든 문장을 호출한 뒤에 함수를 호출했던 위치로 다시 점프하는 과정을 거친다.

하지만 '함수를 호출하는 부분'을 '함수의 본문'으로 대체하면 이러한 점프가 일어나지 않는다.

장점

- 타입 아규먼트에 `reified` 한정자를 붙여서 사용할 수 있다.
- 함수 타입 파라미터를 가진 함수가 훨씬 빠르게 동작한다.
- 비지역(non-local) 리턴을 사용할 수 있다.

단점

- `inlin` 한정자를 붙였을 때 발생하는 비용이 존재한다.

## 타입 아규먼트를 reified로 사용할 수 있다

JVM 바이트 코드에는 제네릭이 존재하지 않는다. 따라서 컴파일을 하면, 제네릭 타입과 관련된 내용이 제거 된다.

```kotlin
any is List<Int>    // 오류
any is List<*>      // OK

fun <T> printTypeName() {
    print(T::class.simpleName)  // 오류
}
```

함수를 인라인으로 만들면, 이러한 제한을 무시할 수 있다. 함수 호출이 본문으로 대체되므로, `reified` 한정자를 지정하면, 타입 파라미터를 사용한 부분이 타입 아규먼트로 대체된다.

```kotlin
inline fun <reified T> printTypeName() {
    print(T::class.simpleName)
}

// 사용
printTypeName<Int>()    // Int
printTypeName<Char>()   // Char
printTypeName<String>() // String
```

컴파일하는 동안 `printTypeName`의 본문이 실제로 대체된다.

```kotlin
print(Int::class.simpleName)    // Int
print(Char::class.simpleName)   // Char
print(String::class.simpleName) // String
```

## 함수 타입 파라미터를 가진 함수가 훨씬 빠르게 동작한다

모든 함수는 `inline` 한정자를 붙이면 조금 더 빠르게 동작한다. 함수 호출과 리턴을 위해 점프하는 과정과 백스택을 추적하는 과정이 없기 때문이다. (그래서 표준 라이브러리에 있는 간단한 함수들에는 대부분 inline 한정자가 붙어 있다.)

하지만 함수 파라미터를 가지지 않는 함수에서는 이러한 차이가 큰 성능 차이를 발생시키지 않는다.

함수 타입은 단순한 인터페이스로 컴파일러에 의해서 생성된다. 이때, 함수 본문을 객체로 랩(wrap)하게 되기 때문에 코드의 속도가 느려진다.

```kotlin
// (1)
inline fun repeat(times: Int, action: (Int) -> Unit) {
    for (index in 0 until times) {
        action(index)
    }
}

// (2)
fun repeatNoinline(times: Int, action: (Int) -> Unit) {
    for (index in 0 until times) {
        action(index)
    }
}
```
- (1) : 숫자로 반복을 돌면서, 빈 함수를 호출한다.
- (2) : 숫자로 반복을 돌면서, 객체를 호출하고, 이 객체가 빈 함수를 호출한다.
- (1) 이 (2) 보다 더 빠르다.

### 더 중요한 차이는 함수 리터럴 내부에서 지역 변수를 캡처할 때 확인할 수 있다

```kotlin
var l = 1L
noinlineRepeat(100_000_000) {
    l += it
}
```

위의 코드는 컴파일 과정 중에 아래와 같이 레퍼런스 객체로 래핑되고, 람다 표현식 내부에서는 이를 사용한다.

```kotlin
val a = Ref.LongRef()
a.element = 1L
noinlineRepeat(100_00_000) {
    a.element = a.element + it
}
```

함수가 객체로 컴파일되고, 지역 변수가 래핑되기 때문에 성능에 차이가 발생한다.

{% hint style="success" %}
일반적으로 함수 타입의 파라미터가 어떤 식으로 동작하는지 이해하기 어려우므로, 함수 타입 파라미터를 활용해서 유틸리티 함수를 만들 때는 그냥 인라인을 붙여 준다 생각하는 것도 좋다.
{% endhint %}

## 비지역적 리턴(non-local return)을 사용할 수 있다

```kotlin
fun main() {
    repeatNoinline(10) {
        print(it)
        return // 오류 : 허용되지 않는다.
    }
}
```

이는 함수 리터럴이 컴파일될 때, 함수가 객체로 래핑되어서 발생하는 문제이다. 함수가 다른 클래스에 위치하므로, return을 사용해서 main으로 돌아올 수 없다.

인라인 함수라면 함수가 main 함수 내부에 박히기 때문에 이러한 제한이 없다. 

```kotlin
fun main() {
    repeat(10) {
        print(it)
        return // OK
    }
}

fun getSomeMoney(): Money? {
    repeat(100) {
        val money = searchForMoney()
        if (money != null) return money
    }
    return null
}
```

## inline 한정자의 비용

`inline` 한정자는 모든 곳에 사용할 수는 없다.

### 대표적인 예로 인라인 함수는 재귀적으로 동작할 수 없다

재귀적으로 사용하면, 무한하게 대체되는 문제가 발생한다. 

{% hint style="danger" %}
이러한 문제는 인텔리제이가 오류로 잡아 주지 못하므로 굉장히 위험하다.
{% endhint %}

### 인라인 함수는 더 많은 가시성 제한을 가진 요소를 사용할 수 없다.

public 인라인 함수 내부에서는 private와 internal 가시성을 가진 함수와 프로퍼티를 사용할 수 없다.

{% hint style="info" %}
인라인 함수는 구현을 숨길 수 없으므로, 클래스에 거의 사용되지 않는다.
{% endhint %}

### inline 한정자를 남용하면 코드의 크기가 쉽게 커진다.

{% hint style="danger" %}
서로 호출하는 인라인 함수가 많아지면, 코드가 기하급수적으로 증가하므로 위험하다.
{% endhint %}

## crossinline과 noinline

함수를 인라인으로 만들고 싶지만, 어떤 이유로 일부 함수 타입 파라미터는 `inline`으로 받고 싶지 않은 경우가 있을 수 있다. 이러한 경우에 다음과 같은 한정자를 사용한다.

### crossinline 

아규먼트로 인라인 함수를 받지만, 비지역적 리턴을 하는 함수는 받을 수 없게 만든다.

인라인으로 만들지 않은 다른 람다 표현식과 조합해서 사용할 때 문제가 발생하는 경우 활용한다.

### noinline

아규먼트로 인라인 함수를 받을 수 없게 만든다.

인라인 함수가 아닌 함수를 아규먼트로 사용하고 싶을 때 활용한다.

{% hint style="info" %}
인텔리제이 IDEA가 필요할 때 알아서 제안을 해 주므로 대충 알아 두기만 해도 괜찮다.
{% endhint %}

## 정리

인라인 함수가 사용되는 주요 사례

- print 함수처럼 매우 많이 사용되는 경우
- filterIsInstance 함수처럼 타입 아규먼트로 reified 타입을 전달받는 경우
- 함수 타입 파라미터를 갖는 톱레벨 함수를 정의해야 하는 경우, 특히 컬렉션 처리 함수와 같은 헬퍼 함수(map, filter, flatMap, joinToString 등), 스코프 함수(also, apply, let 등), 톱 레벨 유틸리티 함수(repeat, run, with)의 경우

API를 정의할 때 인라인 함수를 사용하는 경우는 거의 없다. 

또한 한 인라인 함수가 다른 인라인 함수를 호출하는 경우, 코드가 기하급수적으로 많아질 수 있으므로 주의해야 한다.