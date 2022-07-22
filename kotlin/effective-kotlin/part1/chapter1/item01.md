---
description: 이펙티브 코틀린 정리하기
---

# 가변성을 제한하라

상태(state)를 갖는 것은 시간의 변화에 따라서 변하는 요소를 표현할 수 있다는 점에서 유용하지만, 상태를 적절하게 관리하는 것이 어렵다.

- 프로그램을 이해하고 디버그하기 어렵다.
- 가변성이 있으면, 코드의 실행을 추론하기 어렵다.
- 멀티스레드 프로그램일 때는 적절한 동기화가 필요하다.
- 모든 상태를 테스트해야하기 때문에 테스트하기 어렵다.
- 상태 변경이 일어날 때, 이러한 변경을 다른 부분에 알려야 하는 경우가 있다.

가변성은 시스템의 상태를 나타내기 위한 중요한 방법이다. 하지만 변경이 일어나야 하는 부분을 신중하고 확실하게 결정하고 사용해야 한다.

## 코틀린에서 가변성 제한하기

### 읽기 전용 프로퍼티(val)

`val`을 사용해 읽기 전용 프로퍼티를 만들 수 있다. 이렇게 선언된 프로퍼티는 마치 값(value)처럼 동작하며, 일반적인 방법으로는 값이 변하지 않는다.

{% hint style="warning" %}
읽기 전용 프로퍼티가 완전히 변경 불가능한 것은 아니다.

읽기 전용 프로퍼티가 mutable 객체를 담고 있다면, 내부적으로 변할 수 있다.
{% endhint %}

<br>

읽기 전용 프로퍼티는 다른 프로퍼티를 활용하는 사용자 정의 게터로도 정의할 수 있다.

```kotlin
var name = "Marcin"
var surname = "Moskala"
val fullName
    get() = "$name $surname"
```

```kotlin
// main
println(fullName) // Marcin Moskala

name = "Maja"
println(fullName) // Maja Moskala
```

게터를 활용하여 정의한 경우, 값을 사용하는 시점의 name에 따라서 다른 결과가 나올 수 있기 때문에 스마트 캐스트할 수 없다.

```kotlin
val name: String? = "Marton"
val surname: String? = "Braun"
val fullName: String?
    get() = name?.let { "$it $surname" }
val fullName2: String? = name?.let { "$it $surname" }
```

`fullName`은 게터로 정의했기 때문에 스마트 캐스트할 수 없다.

```kotlin
// main
if (fullName != null) {
    println(fullName.length) // 컴파일 오류, 안전연산자(?.)을 사용하여 호출해야 한다.
}
```

`fullName2`처럼 지역 변수가 아닌 프로퍼티(non-local property)가 `final`이고, 사용자 정의 게터를 갖지 않을 경우 스마트 캐스트할 수 있다. 

```kotlin
// main
if (fullName2 != null) {
    println(fulName2.length) // Marton Braun, null이 아니기 때문에 안전연산자(?.)를 사용하지 않아도 된다.
}
```

<br>

읽기 전용 프로퍼티 레퍼런스 자체를 변경할 수 있는 없기 때문에 동기화 문제 등을 줄일 수 있다. 그래서 일반적으로 `var`보다 `val`을 많이 사용한다.

### 가변 컬렉션과 읽기 전용 컬렉션 구분하기

### 데이터 클래스의 copy

## 다른 종류의 변경 가능 지점

## 변경 가능 지점 노출하지 말기

## 정리

코틀린은 가변성을 제한하기 위해 다양한 도구들을 제공하며, 이를 활용하여 가변 지점을 제한하며 코드를 작성하자.

### 코드 작성 규칙

- `var`보다는 `val`을 사용하는 것이 좋다.
- mutable 프로퍼티보다는 immutable 프로퍼티를 사용하는 것이 좋다.
- mutable 객체와 클래스보다는 immutable 객체와 클래스를 사용하는 것이 좋다.
- 변경이 필요한 대상을 만들어야 한다면, immutable 데이터 클래스로 만들고 copy를 활용하는 것이 좋다.
- 컬렉션에 상태를 저장해야 한다면, mutable 컬렉션보다는 읽기 전용 컬렉션을 사용하는 것이 좋다.
- 변이 지정을 적절하게 설계하고, 불필요한 변이 지점은 만들지 않는 것이 좋다.
- mutable 객체를 외부에 노출하지 않는 것이 좋다.

### 예외

- 효율성 때문에 immutable 객체보다 mutable 객체를 사용하는 것이 좋을 때가 있다. 단, 이러한 최적화는 코드에서 성능이 중요한 부분에서만 사용하는 것이 좋다.
- immutable 객체를 사용할 때는 언제나 멀티스레드 때에 더 많은 주의를 기울여야 한다.
