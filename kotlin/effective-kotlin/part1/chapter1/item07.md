---
description: 이펙티브 코틀린 정리하기
---

# 결과 부족이 발생할 경우 null과 Failure를 사용하라

## 함수가 원하는 결과를 만들 수 없을 때 처리하는 메커니즘

- null 또는 '실패를 나타내는 sealed 클래스(일반적으로 Failure라는 이름을 붙인다.)'를 리턴한다.
- 예외를 `throw` 한다.

### 예외를 `throw`하는 방식의 문제점

- 많은 개발자가 예외가 전파되는 과정을 제대로 추적하지 못한다.
- 코틀린의 모든 예외는 unchecked 예외이기 때문에 사용자가 예외를 처리하지 않을 수도 있으며, 이와 관련된 내용은 문서에도 제대로 드러나지 않는다.
- 예외는 예외적인 상황을 처리하기 위해서 만들어졌으므로 명시적인 테스트만큼 빠르게 동작하지 않는다.
- try ~ catch 블록 내부에 코드를 배치하면, 컴파일러가 할 수 있는 최적화가 제한된다.

### null과 Failure를 사용했을 때의 장점

try ~ catch 블록보다 효율적이며, 사용하기 쉽고 더 명확하다.

null 값과 sealed result 클래스는 명시적으로 처리해야 하며, 애플리케이션의 흐름을 중지하지도 않는다. (예외는 놓칠 수도 있으며, 전체 애플리케이션을 중지시킬 수도 있다.)

{% hint style="success" %}
충분이 예측할 수 있는 범위의 오류는 null과 Failure를 사용하고,

예측하기 어려운 예외적인 범위의 오류는 `throw`해서 처리하는 것이 좋다.
{% endhint %}

## null과 Failure 사용하기

### null 사용하기

```kotlin
inline fun <reified T> String.readObjectOrNull(): T? {
    // ...
    if (incorrectSign) {
        return null
    }
    // ...
    return result
}
```

null을 처리해야 한다면, 사용자는 안전 호출(safe call) 또는 Elvis 연산자 같은 다양한 널 안정성(null-safety) 기능을 활용한다.

```kotlin
val age = userText.readObjectOrNull<Person>()?.age ?: -1
```

### Failure 사용하기

```kotlin
inline fun <reified T> String.readObject(): Result<T> {
    // ...
    if (incorrectSign) {
        return Failure(JsonParsingException())
    }
    // ...
    return Success(result)
}

sealed class Result<out T>
class Success<out T>(val result: T): Result<T>()
class Failure(val throwable: Throwable): Result<Nothing>()

class JsonParsingException: Exception()
```

Result와 같은 공용체(union type)을 리턴하기로 했다면, when 표현식을 사용해서 처리할 수 있다.

```kotlin
val person = userText.readObjectOrNull<Person>()
val age = when(person) {
    is Success -> person.age
    is Failure -> -1
}
```

{% hint style="success" %}
Failure는 처리할 때 필요한 정보를 가질 수 있기 때문에, 추가적인 정보를 전달해야 한다면 sealed result를 사용하고, 그렇지 않으면 null을 사용하는 것이 일반적이다.
{% endhint %}

<br>

일반적으로 두 가지 형태의 함수를 사용한다.

- get : 특정 위치에 있는 요소를 추출할 때 사용한다. 만약 요소가 해당 위치에 없다면 `IndexOutOfBoundsException`을 발생시킨다.
- getOrNull : out of range 오류가 발생할 수 있는 경우에 사용하며, 발생한 경우에는 null을 리턴한다.

일부 상황에서 getOrDefault와 같은 다른 선택지도 있지만, getOrNull 또는 Elvis 연산자를 사용하는 것이 쉽다.

{% hint style="success" %}
개발자에게 null이 발생할 수 있다는 경고를 주려면 getOrNull 등을 사용해서 무엇이 리턴되는지 예측할 수 있게 하는 것이 좋다.
{% endhint %}
