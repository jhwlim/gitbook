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
if (fullName != null) {
    println(fullName.length) // 컴파일 오류, 안전연산자(?.)을 사용하여 호출해야 한다.
}
```

`fullName2`처럼 지역 변수가 아닌 프로퍼티(non-local property)가 `final`이고, 사용자 정의 게터를 갖지 않을 경우 스마트 캐스트할 수 있다.

```kotlin
if (fullName2 != null) {
    println(fulName2.length) // Marton Braun, null이 아니기 때문에 안전연산자(?.)를 사용하지 않아도 된다.
}
```

<br>

읽기 전용 프로퍼티 레퍼런스 자체를 변경할 수 있는 없기 때문에 동기화 문제 등을 줄일 수 있다. 그래서 일반적으로 `var`보다 `val`을 많이 사용한다.

### 가변 컬렉션과 읽기 전용 컬렉션 구분하기

코틀린은 읽고 쓸 수 있는 컬렉션과 읽기 전용 컬렉션을 구분한다.

- 읽기 전용 컬렉션 : `Iterable`, `Collectin`, `Set`, `List`
- 읽고 쓸 수 있는 컬렉션 : `MutableIterable`, `MutableCollection`, `MutableSet`, `MutableList`

mutable이 붙은 인터페이스는 대응되는 읽기 전용 인터페이스를 상속 받아서, 변경을 위한 메서드를 추가한 것이다. 즉, 코틀린은 내부적으로 immutable하지 않은 컬렉션을 외부적으로 immutable하게 보이게 만드는 것이다.

읽기 전용 컬렉션이 내부의 값을 변경할 수 없다는 의미는 아니다. 대부분의 경우에는 변경할 수 있다. 하지만 읽기 전용 인터페이스가 이를 지원하지 않으므로 변경할 수 없고, mutable 컬렉션으로 다운캐스팅해서 값을 변경할 수 있다.

하지만 다운캐스팅은 읽기 전용으로 사용해야 한다는 규약을 위반하는 것이고, 추상화를 무시하는 행위이다. 안전하지 않고, 예측하지 못한 결과를 초래한다.

```kotlin
// bad
val list = listOf(1, 2, 3)
if (list is MutableList) {
    list.add(4)
}
```

따라서 코틀린에서 읽기 전용 컬렉션을 mutable 컬렉션으로 다운캐스팅하면 안 된다.

만약, 읽기 전용에서 mutable로 변경해야 한다면, 복제(copy)를 통해서 새로운 mutable 컬렉션을 만드는 `toMutableList()`를 활용해야 한다.

```kotlin
// good
val list = listOf(1, 2, 3)
val mutableList = list.toMutableList()
mutableList.add(4)
```

### 데이터 클래스의 copy

**immutable 객체를 사용했을 때의 장점**

- 한 번 정의된 상태가 유지되므로, 코드를 이해하기 쉽다.
- immutable 객체는 공유했을 때도 충돌이 따로 이루어지지 않으므로, 병렬 처리를 안전하게 할 수 있다.
- immutable 객체에 대한 참조는 변경되지 않으므로, 쉽게 캐시할 수 있다.
- immutable 객체는 방어적 복사본(defensive copy)을 만들 필요가 없다. 또한, 객체를 복사할 때 깊은 복사를 따로 하지 않아도 된다.
- immutable 객체는 다른 mutable 또는 immutable 객체를 만들 때 활용하기 좋다. 또한, immutable 객체는 실행을 더 쉽게 예측할 수 있다.
- immutable 객체는 `Set` 또는 `Map의 키`로 사용할 수 있다.

{% hint style="info" %}
mutable 객체는 `Set` 또는 `Map의 키`로 사용할 수 없다. `Set`과 `Map`이 내부적으로 해시 테이블을 사용하고, 해시 테이블은 처음 요소를 넣을 때 요소의 값을 기반으로 버킷을 결정하기 때문이다. 따라서 요소에 수정이 일어나면 해시 테이블 내부에서 요소를 찾을 수 없게 된다.
{% endhint %}

<br>

mutable 객체는 예측하기 어려우며 위험하다는 단점이 있다. 반면에 immutable 객체는 변경할 수 없다는 단점이 있다. 따라서 **immutable 객체는 자신의 일부를 수정한 새로운 객체를 만들어 내는 메서드를 가져야 한다.**

```kotlin
class User(
    val name: String,
    val surname: String
) {
    fun withSurname(surname: String) = User(name, surname)
}
```

```kotlin
var user = User("Maja", "Markiewicz")
user = user.withSurname("Moskala")
println(user) // User(name=Maja, surname=Moskala)
```

<br>

다만 모든 프로퍼티를 대상으로 이런 함수를 만드는 것은 굉장히 귀찮은 일이다. **데이터 모델 클래스은 `copy`라는 이름의 메서드를 만들어 주는데, 이를 활용하면 모든 기본 생성자 프로퍼티가 같은 새로운 객체를 만들어 낼 수 있다.**

```kotlin
data class User(
    val name: String,
    val surname: String
)
```

```kotlin
var user = User("Maja", "Markiewicz")
user = user.copy(username = "Moskala")
println(user) // User(name=Maja, surname=Moskala)
```

이렇게 데이터 모델 클래스를 만들어 immutable 객체로 만드는 것이 더 많은 장점을 가지므로, 기본적으로는 이렇게 만드는 것이 더 좋다.

## 다른 종류의 변경 가능 지점

변경할 수 있는 리스트를 만드는 방법

- mutable 컬렉션을 만드는 것
- mutable 프로퍼티를 만드는 것

### mutable 컬렉션을 만드는 것

구체적인 리스트 구현 내부에 변경 가능 지점이 있다.

```kotlin
val list1: MutableList<Int> = mutableListOf()
list1.add(1)
assertThat(list1.size).isEqualTo(1)
assertThat(list1[0]).isEqualTo(1)

list1 += 2 // list1.plusAssign(1)
assertThat(list1.size).isEqualTo(2)
assertThat(list1[0]).isEqualTo(1)
assertThat(list1[1]).isEqualTo(2)
```

{% hint style="warning" %}
멀티스레드 처리가 이루어질 경우, 내부적으로 적절한 동기화가 되어 있는지 확실하게 알 수 없으므로 위험하다.
{% endhint %}

```kotlin
var list = listOf<Int>()
for (i in 1..1000) {
    thread {
        list = list + i
    }
}
Thread.sleep(1000)
assertThat(list.size).isNotEqualTo(1000) // 1000이 되지 않을 수 있다.
```

### mutable 프로퍼티를 만드는 것

프로퍼티 자체가 변경 가능 지점이다.

```kotlin
var list2: List<Int> = listOf()
list2 = list2 + 1
assertThat(list2.size).isEqualTo(1)
assertThat(list2[0]).isEqualTo(1)

list2 += 2 // list2.plus(2); 실제로 내부 로직을 살펴보면 새로운 리스트를 생성한다.
assertThat(list2.size).isEqualTo(2)
assertThat(list2[0]).isEqualTo(1)
assertThat(list2[1]).isEqualTo(2)
```

{% hint style="success" %}
멀티스레드 처리의 안정성이 더 좋다고 할 수 있다. (물론 잘못 만들면 일부 요소가 손실될 수도 있다.)
{% endhint %}

<br>

**mutalbe 프로퍼티를 사용하는 형태는 사용자 정의 세터를 활용하여 변경을 추적할 수 있다.**

`Delegates.observable()`을 사용하면, 리스트에 변경이 있을 때 로그를 출력할 수 있다.

```kotlin
var names by Delegates.observable(listOf<String>()) { _, old, new -> println("Names changed from $old to $new")}

names += "Fabio" // Names changed from [] to [Fabio]
assertThat(names.size).isEqualTo(1)
assertThat(names[0]).isEqualTo("Fabio")

names += "Bill" // Names changed from [Fabio] to [Fabio, Bill]
assertThat(names.size).isEqualTo(2)
assertThat(names[0]).isEqualTo("Fabio")
assertThat(names[1]).isEqualTo("Bill")
```

mutable 컬렉션도 위와 같이 관찰(observe)할 수 있게 만들려면, 추가적인 구현이 필요하다.

따라서 mutable 프로퍼티에 읽기 전용 컬렉션을 넣어 사용하는 것이 쉽다. 여러 객체를 변경하는 여러 메서드 대신 세터를 사용할 수 있고, private으로 만들 수도 있다는 장점이 있다.

```kotlin
var announcements = listOf<Announcement>()
    private set(value) { ... }
```

{% hint style="success" %}
mutable 프로퍼티를 사용하면 객체 변경을 제어하기가 더 쉽다.
{% endhint %}

{% hint style="danger" %}
프로퍼티와 컬렉션을 모두 변경 가능한 지점으로 만들지 마라.

- 변경될 수 있는 두 지점 모두에 대한 동기화를 구현해야 한다.
- 또한 모호성이 발생해서 `+=`을 사용할 수 없다.

```kotlin
var list3 = mutableListOf<Int>()
```

{% endhint %}

<br>

상태를 변경할 수 있는 불필요한 방법은 만들지 않아야 한다. 상태를 변경하는 모든 방법은 코드를 이해하고 유지해야 하므로 비용이 발행한다. 따라서 **가변성을 제한하는 것이 좋다.**

## 변경 가능 지점 노출하지 말기

### mutable 객체를 외부에 노출하는 것은 돌발적인 수정이 일어날 때 위험할 수 있다.

```kotlin
data class User(val name: String)

class UserRepository {

    private val storedUsers: MutableMap<Int, String> = mutableMapOf()

    fun loadAll(): MutableMap<Int, String> = storedUsers

}
```

```kotlin
val userRepository = UserRepository()

val storedUsers = userRepository.loadAll()
storedUsers[4] = "Kirill"

assertThat(storedUsers.size).isEqualTo(1)
assertThat(storedUsers[4]).isEqualTo("Kirill")
```

### 방법 (1) 리턴되는 mutable 객체를 복제하기

```kotlin
class UserHolder {

    private val user: MutableUser()

    fun get(): MutableUser = user.copy()

}
```

### 방법 (2) 컬렉션은 객체를 읽기 전용 슈퍼타입으로 업캐스트하기

```kotlin
class UserRepository {

    private val storedUsers: MutableMap<Int, String> = mutableMapOf()

    fun loadAll(): Map<Int, String> = storedUsers

}
```

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
