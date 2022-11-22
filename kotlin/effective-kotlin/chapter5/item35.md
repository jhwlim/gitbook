---
description: 이펙티브 코틀린 정리하기
---

# 복잡한 객체를 생성하기 위한 DSL을 정의하라

DSL은 복잡한 객체, 계층 구조를 갖고 있는 객체들을 정의할 때 굉장히 유용하다. DSL을 만드는 것은 약간 힘든 일이지만, 한 번 만들고 나면 보일러플레이트와 복잡성을 숨기면서 개발자의 의도를 명확하게 표현할 수 있다.

{% hint style="info" %}
보일러플레이트는 자주 사용되는 상용구적인 코드를 의미한다.
{% endhint %}

DSL을 활용하면 복잡하고 계층적인 자료 구조를 쉽게 만들 수 있다. DSL 내부에서도 코틀린이 제공하는 모든 것을 활용할 수 있다. 코틀린은 type-safe이기 때문에 여러 가지 유용한 힌트를 활용할 수 있다.

## 사용자 정의 DSL 만들기

### Receiver를 사용하는 함수 타입에 대한 개념 이해하기

|함수 타입|설명|
|:--|:--|
() -> Unit|아규먼트를 갖지 않고, Unit을 리턴하는 함수
(Int) -> Unit|Int를 아규먼트로 받고, Unit을 리턴하는 함수
(Int) -> Int|Int를 아규먼트로 받고, Int를 리턴하는 함수
(Int, Int) -> Int|Int 2개를 아규먼트로 받고, 다른 함수를 리턴하는 함수
(Int) -> () -> Unit|Int를 아규먼트로 받고, 다른 함수를 리턴하는 함수. 이때, 다른 함수는 아규먼트로 아무것도 받지 않고, Unit을 리턴한다.
(() -> Unit) -> Unit|다른 함수를 아규먼트로 받고, Unit을 리턴하는 함수. 이때, 다른 함수는 아규먼트로 아무것도 받지 않고, Unit을 리턴한다.

### 함수 타입을 만드는 기본적인 방법

- 람다 표현식
- 익명 함수
- 함수 레퍼런스

```kotlin
fun plus(a: Int, b: Int) = a + b

// 유사함수
val plus1: (Int, Int) -> Int = { a, b -> a + b }
val plus2: (Int, Int) -> Int = fun(a, b) = a + b
val plus3: (Int, Int) -> Int = ::plus
val plus4 = { a: Int, b: Int -> a + b }
val plus5 = fun(a: Int, b: Int) = a + b
```

{% hint style="info" %}
익명 함수를 만들 때는 일반 함수처럼 만들고, 이름만 빼면 된다.
{% endhint %}

### 리시버를 가진 함수 타입

일반적인 함수 타입과 비슷하지만, 라파미터 앞에 리시버 타입이 추가되어 있으며, 점(`.`) 기호로 구분되어 있다. (확장 함수를 나타내는 특별한 타입)

```kotlin
val myPlus: Int.(Int) -> Int = fun Int.(other: Int) = this + other
```

이와 같이 함수는 람다식, 구체적으로 리시버를 가진 람다 표현식을 사용해서 정의할 수 있다. 이렇게 하면 **스코프 내부에 `this` 키워드가 확장 리시버를 참조하게 된다.**

```kotlin
// 사용
myPlus.invoke(1, 2) // 일반적인 객체처럼 invoke 메서드를 사용
myPlus(1, 2)        // 확장 함수가 아닌 함수처럼 사용
1.myPlus(2)         // 일반적인 확장 함수처럼 사용
```

리시버를 가진 함수타입은 코틀린 DSL을 구성하는 가장 기본적인 블록이다.

### 예시

HTML 표를 생성하기 위한 DSL 정의하기

```kotlin
fun table(init: TableBuilder.() -> Unit): TableBuilder { ... }

class TableBuilder {
    fun tr(init: TrBuilder.() -> Unit) { ... }
}

class TrBuilder {
    fun td(init: TdBuilder.() -> Unit) { ... }
}

class TdBuilder {
    var text = ""

    operator fun String.unaryPlus() {
        text += this
    }
}
```

사용

```kotlin
fun createTable(): TableDsl = table {
    tr {
        for (i in 1..2) {
            td {
                +"This is column $i"
            }
        }
    }
}
```

## 단점

DSL은 여러 종류의 정보를 표현할 수 있지만, 사용자 입장에서는 이 정보가 어떻게 활용되는지 명확하지 않다. 내부적으로 얼마나 정확하게 만들어지는지 알 수 없다. 또한 DSL의 복잡한 사용법은 찾기 힘들 수도 있다. 해당 DSL에 익숙하지 않은 사람에게 DSL은 혼란은 줄 수 있다. DSL을 정의한다는 것은 개발자의 인지적 혼란과 성능이라는 비용이 모두 발생할 수 있다. 

{% hint style="warning" %}
단순한 기능까지 DSL을 사용한다는 것은 바람직하지 않다.
{% endhint %}

## 언제 사용해야 할까?

- 복잡한 자료 구조
- 계층적인 구조
- 거대한 양의 데이터

DSL은 많이 사용되는 구조의 반복을 제거할 수 있게 해준다. 많이 사용되는 반복되는 코드가 있고, 이를 간단하게 만들 수 있는 별도의 코틀린 기능이 없다면, DSL 사용을 고려해 보는 것이 좋다.

## 정리

DSL은 언어 내부에서 사용할 수 있는 특별한 언어이다. 복잡한 객체는 물론이고, 계층 구조를 갖는 객체를 간단하게 표현할 수 있게 해준다.

하지만 DSL 구현은 해당 DSL이 익숙하지 않은 개발자에게 혼란과 어려움을 줄 수 있다.

따라서 DSL은 복잡한 객체를 만들거나, 복잡한 계층 구조를 갖는 객체를 만들 때만 활용하는 것이 좋다.