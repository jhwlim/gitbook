---
description: 이펙티브 코틀린 정리하기
---

# compareTo의 규약을 지켜라

`compareTo` 메서드는 `Any` 클래스에 있는 메서드가 아니다. 이는 수학적인 부등식으로 변환되는 연산자이다.

```kotlin
obj1 > obj2 // obj1.compareTo(obj2) > 0 으로 바뀐다.
obj1 < obj2 // obj1.compareTo(obj2) < 0 으로 바뀐다.
obj1 >= obj2 // obj1.compareTo(obj2) >= 0 으로 바뀐다.
obj1 <=> obj2 // obj1.compareTo(obj2) <= 0 으로 바뀐다.
```

참고로 `compareTo` 메서드는 `Comparable<T>` 인터페이스에도 들어 있다. 어떤 객체가 이 인터페이스를 구현하고 있거나 `compareTo`라는 연산자 메서드를 갖고 있다는 의미는 해당 객체가 어떤 순서를 갖고 있으므로, 비교할 수 있다는 것이다.

`compareTo`는 다음과 같이 동작해야 한다.

- 비대칭적 동작 : a >= b 이고 b <= a 라면, a == b 여야 한다.
- 연속적 동작 : a >= b 이고 b >= c 라면, a >= c 여야 한다. 마찬가지로 a > b 이고 b > c 라면, a > c 여야 한다. 이러한 동작을 하지 못하면, 요소 정렬이 무한 반복에 빠질 수 있다.
- 코넥스적 동작(connex relation) : a >= b 또는 b >= a 중에 적어도 하나는 항상 true 여야 한다. 두 요소 사이에 관계가 없드면, 퀵 정렬과 삽입 정렬 등의 고전적인 정렬 알고리즘을 사용할 수 없다. 대신 위상 정렬과 같은 정렬 알고리즘만 사용할 수 있다.

## compareTo를 따로 정의해야 할까?

코틀린에서 `compareTo`를 따로 정의해야 하는 상황은 거의 없다. 일반적으로 어떤 프로퍼티 하나를 기반으로 순서를 지정하는 것으로 충분하기 때문이다.

### 원하는 키로 컬렉션 정렬하기

```kotlin
class User(
    val name: String,
    val surname: String,
)

val names = listOf<User>(/*...*/)
val sorted = names.sortedBy { it.surname }
```

### 여러 프로퍼티 기반으로 정렬하기

```kotlin
val sorted = names.sortedWith(compareBy({ it.surname }, { it.name })) 
```

{% hint style="success" %}
객체가 자연스러운 순서인지 확실하지 않다면, 비교기(comparator)를 사용하는 것이 좋다. 이를 자주 사용한다면, 클래스에 companion 객체로 만들어 두는 것도 좋다.

```kotlin
class User(
    val name: String,
    val surname: String,
) {
    // ...

    companion object {
        val DISPLAY_ORDER = compareBy(User::surname, User::name)
    }
}

val sorted = names.sortedWith(User.DISPLAY_ORDER)
```
{% endhint %}

## compareTo 구현하기

두 값을 단순하게 비교하기만 한다면, `compareValues` 함수를 다음과 같이 활용할 수 있다.

```kotlin
class User(
    val name: String,
    val surname: String,
) : Comparable<User> {

    override fun compareTo(other: User): Int = compareValues(surname, other.surname)

}
```

더 많은 값을 비교하거나, 선택기(selector)를 활용해서 비교하고 싶다면, `compareValuesBy`를 사용한다.

```kotlin
class User(
    val name: String,
    val surname: String,
) : Comparable<User> {

    override fun compareTo(other: User): Int = compareValuesBy(this, other, { it.surname }, { it.name })

}
```

{% hint style="success" %}
특별한 논리를 구현해야 하는 경우에는 이 함수가 다음 값을 리턴해야 한다.

- 0 : 리시버와 other가 같은 경우
- 양수 : 리시버가 other 보다 큰 경우
- 음수 : 리시버가 other 보다 작은 경우

이를 구현한 뒤에는 이 함수가 비대칭적 동작, 연속적 동작, 코넥스적 동작을 하는지 확인해야 한다.
{% endhint %}