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

`out`은 타입 파라미터를 **covariant(공변성)** 로 만든다. 이는 A가 B의 서브타입일 때, `Cup<A>`가 `Cup<B>`의 서브타입이라는 의미이다.

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

`in` 한정자는 타입 파라미터를 **contravariant(반변성)** 으로 만든다. 이는 A가 B의 서브타입일 때, `Cup<A>`가 `Cup<B>`의 슈퍼타입이라는 것을 의미한다.

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

## 함수 타입

함수 타입은 파라미터 유형과 리턴 타입에 따라서 서로 어떤 관계를 갖는다.

코틀린 함수 타입의 모든 파라미터 타입은 contravariant 이다. 

예) Int -> Number -> Any

또한 모든 리턴 타입은 covariant 이다.

예) Any -> Number -> Int

함수 타입을 사용할 때는 이처럼 자동으로 variance 한정자가 사용된다. 코틀린에서 자주 사용되는 것으로는 covariant(`out` 한정자)를 가진 `List`가 있다.

## variance 한정자의 안전성

자바의 배열은 covariant 이다. 그런데 이 때문에 자바 배열은 큰 문제가 발생한다.

```java
Integer[] numbers = {1, 4, 2, 1};
Object[] objects = numbers;
objects[2] = "B"; // 런타임 오류
```

numbers를 Object[]로 캐스팅해도 구조 내부에서 사용되고 있는 실질적인 타입이 바뀌는 것이 아니다. (여전히 Integer이다.) 따라서 이러한 배열에 String 타입의 값을 할당하면, 오류가 발생한다.

코틀린은 이러한 결함을 해결하기 위해 Array(`IntArray`, `CharArray` 등)를 invariant로 만들었다. 따라서 `Array<Int>`를 `Array<Any>` 등으로 바꿀 수 없다.

파라미터 타입을 예측할 수 있다면 어떤 서브타입이라도 전달할 수 있다. 따라서 아규먼트를 전달할 때, 암묵적으로 업캐스팅할 수 있다.

```kotlin
open class Dog
class Puppy : Dog()
class Hound : Dog()

fun takeDog(dog: Dog) {}

takeDog(Dog())
takeDog(Puppy())
takeDog(Hound())
```

{% hint style="success" %}
코틀린은 public `in` 한정자 위치에 covariant 타입 파라미터(`out` 한정자)가 오는 것을 금지한다.
{% endhint %}

```kotlin
class Box<out T> {
    var value: T? = null // 오류

    fun set(value: T) { // 오류
        this.value = value
    }

    fun get(): T = value ?: error("Value not set")
}
```

가시성을 private로 제한하면 오류가 발생하지 않는다. 객체 내부에서는 업캐스트 객체에 covariant(`out` 한정자)를 사용할 수 없기 때문이다.

```kotlin
class Box<out T> {
    private var value: T? = null

    private set(value: T) {
        this.value = value
    }

    fun get(): T = value ?: error("Value not set")
}
```

covariant(`out` 한정자)는 public `out` 한정자 위치에서도 안전하므로 따로 제한되지 않는다. 이러한 안정성의 이유로 생성되거나 노출되는 타입에만 covariant(`out` 한정자)를 사용하는 것이다. 이러한 프로퍼티는 일반적으로 producer 또는 immutable 데이터 홀더에 많이 사용된다.

covariant(`out` 한정자) 사용 예시

- `List<T>`
- `Response`

{% hint style="success" %}
`out` 위치는 암묵적인 업캐스팅을 허용한다.
{% endhint %}

```kotlin
open class Car
interface Boat
class Amphibious : Car(), Boat

fun getAmphibious: Amphibious = Amphibious()

val car: Car = getAmphibious()
val boat: Boat = getAmphibious()
```

{% hint style="success" %}
코틀린은 contravariant 타입 파라미터(`in` 한정자)를 public `out` 한정자 위치에 사용하는 것을 금지하고 있다.
{% endhint %}

```kotlin
class Box<in T> {
    var value: T? = null // 오류

    fun set(value: T) {
        this.value = value
    }

    fun get(): T = value ?: error("Value not set") // 오류
}
```

요소가 private일 때는 문제가 없다.

```kotlin
class Box<in T> {
    private var value: T? = null

    fun set(value: T) {
        this.value = value
    }

    private fun get(): T = value ?: error("Value not set")
}
```

contravariant(`in` 한정자) 사용 예시

- 타입 파라미터
- `kotlin.coroutines.Continuation`

## variance 한정자의 위치

(1) 선언 부분 

일반적으로 이 위치에서 사용된다. 이 위치에서 사용하면 클래스와 인터페이스 선언에 한정자가 적용된다. 따라서 클래스와 인터페이스가 사용되는 모든 곳에 영향을 준다.

```kotlin
// 선언 쪽의 variance 한정자
class Box<out T>(val value: T)
val boxStr: Box<String> = Box("Str")
val boxAny: Box<Any> = boxStr
```

(2) 클래스와 인터페이스를 활용하는 위치

이 위치에 variance 한정자를 사용하면 특정한 변수에만 variance 한정자가 적용된다. 모든 인스턴스에 variance 한정자를 적용하면 안되고, 특정 인스턴스에만 적용해야할 때 사용한다.

```kotlin
class Box<T>(val value: T)
val boxStr: Box<String> = Box("Str")
// 사용하는 쪽의 variance 한정자
val boxAny: Box<out Any> = boxStr
```

{% hint style="success" %}
variance 한정자를 사용하면, 위치가 제한될 수 있다.
{% endhint %}

`MutableList<out T>`가 있다면, get으로 요소를 추출했을 때 T 타입이 나올 것이다. 하지만 set은 `Nothing` 타입의 아규먼트가 전달될 거라 예상되므로 사용할 수 없다. 이는 모든 타입의 서브타입을 가진 리스트(`Nothing` 리스트)가 존재할 가능성이 있기 때문이다.

`MutableList<in T>`를 사용할 경우, get과 set을 모두 사용할 수 있다. 하지만 get을 사용할 경우, 전달되는 자료형은 `Any?`가 된다. 이는 모든 타입의 슈퍼타입을 가진 리스트(`Any` 리스트)가 존재할 가능성이 있기 때문이다.

## 정리

코틀린의 타입 한정자

- 타입 파라미터의 기본적인 variance의 동작은 invariant 이다. A가 B의 서브타입이라고 할때, `Cup<A>`와 `Cup<B>`는 아무런 관계를 갖지 않는다.
- `out` 한정자는 타입 파라미터를 covariant하게 만든다. A가 B의 서브타입이라고 할때, `Cup<A>`는 `Cup<B>`의 서브타입이 된다.
- `in` 한정자는 타입 파라미터를 contravariant하게 만든다. A가 B의 서브타입이라고 할때, `Cup<B>`는 `Cup<A>`의 서브타입이 된다.

코틀린에서는

- List와 Set의 타입 파라미터는 covariant(`out` 한정자)이다.
- Map에서 값의 타입을 나타내는 타입 파라미터는 covariant(`out` 한정자)이다.
- Array, MutableList, MutableSet, MutableMap의 타입 파라미터는 invariant(한정자 없음)이다.
- 함수 타입의 파라미터 타입은 contravariant(`in` 한정자)이다. 그리고 리턴 타입은 contravariant(`out` 한정자)이다.
- 리턴만 되는 타입에는 covariant(`out` 한정자)를 사용한다.
- 허용만 되는 타입에는 contravariant(`in` 한정자)를 사용한다.