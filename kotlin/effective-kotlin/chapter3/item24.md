---
description: 이펙티브 코틀린 정리하기
---

# 제네릭 타입과 variance 한정자를 활용하라

## variance 한정자

타입 파라미터에 variance 한정자(`out` 또는 `in`)가 없는 경우 기본적으로 **invariant(불공변성)** 이다.

제네릭 타입으로 만들어지는 타입들이 서로 관련성이 없다는 의미이다.

```kotlin
fun main() {
    val anys: Cup<Any> = Cup<Int>() // 오류 : Type mismatch
    val nothings: Cup<Nothing> = Cup<Int>() // 오류
}
```

만약에 어떤 관련성을 원한다면 `out` 또는 `in` 이라는 variance 한정자를 붙인다. 

### out

`out`은 타입 파라미터를 **covariant(공변성)**로 만든다. 이는 A가 B의 서브타입일 때, `Cup<A>`가 `Cup<B>`의 서브타입이라는 의미이다.

```kotlin
class Cup<out T>
open Class Dog
class Puppy: Dog()

fun main() {
    val a: Cup<Dog> = Cup<Puppy>() // OK
    val b: Cup<Puppy> = Cup<Dog>() // 오류
}
```

### in

`in` 한정자는 타입 파라미터를 **contravariant(반변성)**으로 만든다. 이는 A가 B의 서브타입일 때, `Cup<A>`가 `Cup<B>`의 슈퍼타입이라는 것을 의미한다.

```kotlin
class Cup<in T>
open class Dog
class Puppy(): Dog()

fun main() {
    val a: Cup<Dog> = Cup<Puppy>() // 오류
    val b: Cup<Puppy> = Cup<Dog>() // OK

    val anys: Cup<Any> = Cup<Int>() // 오류
    val nothings: Cup<Nothing> = Cup<Int> // OK
}
```