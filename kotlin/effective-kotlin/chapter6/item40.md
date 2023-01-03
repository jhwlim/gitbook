---
description: 이펙티브 코틀린 정리하기
---

# equals의 규약을 지켜라

Any 클래스를 상속받는 모든 메서드는 규약을 잘 지켜주는 것이 좋다. 규약을 위반하면, 일부 객체 또는 기능이 제대로 동작하지 않을 수도 있다.

## 동등성

### 구조적 동등성

`equals` 메서드와 이를 기반으로 만들어진 `==` 연산자(`!=` 포함)로 확인하는 동등성

모든 클래스의 슈퍼클래스인 Any에 구현되어 있으므로, 모든 객체에서 사용할 수 있다. 다만, 연산자를 사용해서 다른 타입의 두 객체를 비교하는 것은 허용되지 않는다.

### 레퍼런스적 동등성

`===` 연산자(`!==` 포함)로 확인하는 동등성, 

두 피연산자가 같은 객체를 가리키면, true를 리턴한다.

## equals가 필요한 이유

`equals` 메서드는 디폴트로 `===` 처럼 두 인스턴스가 완전히 같은 객체를 비교한다. 이는 모든 객체는 디폴트로 유일한 객체라는 것을 의미한다.

```kotlin
class Name(
    val name: String
)

val name1 = Name("Marcin")
val name2 = Name("Marcin")
val name1Ref = name1

name == name1       // true
name1 == name2      // false
name1 == name1Ref   // true

name1 === name1     // true
name1 === name2     // false
name1 === name1Ref  // true
```

이러한 동작은 데이터베이스 연결, 리포지토리, 스레드 등의 활동 요소를 활용할 때 굉장히 유용하다.

하지만 두 객체가 기본 생성자의 프로퍼티가 같다면, 같은 객체로 보는 형태가 있을 수 있다. `data` 한정자를 붙여서 데이터 클래스로 정의하면, 자동으로 이와 같은 동등성으로 동작한다.

```kotlin
data class FullName(
    val name: String,
    val surname: String
)
val name1 = FullName("Marcin", "Moskala")
val name2 = FullName("Marcin", "Moskala")
val name3 = FullName("Maja", "Moskala")

name1 == name1  // true
name1 == name2  // true
name1 == name3  // false

name1 === name1  // true
name1 === name2  // false
name1 === name3  // false
```

데이터 클래스는 내부에 어떤 값을 갖고 있는지가 중요하므로, 이와 같이 동작하는 것이 좋다. 그래서 일반적으로 데이터 모델을 표현할 때는 `data` 한정자를 붙인다.

데이터 클래스의 동등성은 모든 프로퍼티가 아니라 일부 프로퍼티만 비교해야 할 때도 유용하다.

```kotlin
// 동등성 확인 때 검사되지 않는 asStringCache 와 changed 프로퍼티를 갖는다.
class DateTime(
    private var millis: Long = 0L,
    private var timeZone: TimeZone? = null
) {
    private var asStringCache = ""
    private var changed = false

    override fun equals(other: Any?): Boolean = 
        other is DateTime &&
            other.millis == millis &&
            other.timeZone == timeZone

    // ...
}
```

`data` 한정자를 사용해도 같은 결과를 낼 수 있다.

```kotlin
data class DateTime(
    private var millis: Long = 0L,
    private var timeZone: TimeZone? = null
) {
    private var asStringCache = ""
    private var changed = false

    // ...
}
```

{% hint style="info" %}
기본 생성자에 선언되지 않은 프로퍼티는 `copy`로 복사되지 않는다.
{% endhint %}

`data` 한정자를 기반으로 동등성의 동작을 조작할 수 있으므로, 일반적으로 코틀린에서는 `equals`를 직접 구현할 필요가 없다.

`equals` 를 직접 구현해야 하는 경우

- 기본적으로 제공되는 동작과 다른 동작을 해야 하는 경우
- 일부 프로퍼티만으로 비교해야 하는 경우
- `data` 한정자를 붙이는 것을 원하지 않거나, 비교해야 하는 프로퍼티가 기본 생성자에 없는 경우

## equals의 규약

> **`equals` 주석 (코틀린 1.4.31 기준)**
> 어떤 다른 객체가 이 객체와 '같은지(equal to)' 확인할 때 사용한다. 
>
> 구현은 반드시 다음과 같은 요구사항을 충족해야 한다.
> - 반사적(reflexive) 동작 : x가 널이 아닌 값이라면, x.equals(x)는 true를 리턴해야 한다.
> - 대칭적(symmetric) 동작 : x와 y가 널이 아닌 값이라면, x.equals(y)는 y.equals(x)와 같은 결과를 출력해야 한다.
> - 연속적(transitive) 동작 : x, y, z가 널이 아닌 값이고, x.equals(y)와 y.equals(z)가 true라면, x.equals(z)도 true여야 한다.
> - 일관적(consistent) 동작 : x와 y가 널이 아닌 값이라면, x.equals(y)는 (비교에 사용되는 프로퍼티를 변경한 것이 아니라면) 여러 번 실행하더라도 항상 같은 결과를 리턴해야 한다.
> - 널과 관련된 동작 : x가 널이 아닌 값이라면, x.equals(null)은 항상 false를 리턴해야 한다.

추가로, 매우 빠를 거라 예측되므로, 빠르게 동작해야 한다.

### 반사적 동작을 해야 한다.

`x.equals(x)`가 true라는 것을 의미한다.

규약이 잘못되면, 컬렉션 내부에 해당 객체가 포함되어 있어도 `contains` 메서드 등으로 포함되어 있는지 확인할 수 없다.

### 대칭적 동작을 해야 한다.

`x == y`와 `y == x`가 같아야 한다는 것을 의미한다.

일반적으로 다른 타입과 동등성을 확인하려고 할 때, 이런 동작이 위반된다.

대칭적 동작을 하지 못한다는 것은 `contains` 메서드와 단위 테스트 등에서 예측하지 못한 동작이 발생할 수 있다는 것이다.

객체가 대칭적인 동작을 하지 못한다면 예상하지 못한 오류가 발생할 수 있으며, 이것을 디버깅 중에 찾기 어렵다. 따라서 동등성을 구현할 때는 항상 대칭성을 고려 해야 한다.

{% hint style="success" %}
결론적으로 다른 클래스는 동등하지 않게 만들어 버리는 것이 좋다.
{% endhint %}

### 객체 동등성이 연속적이어야 한다.

x, y, z가 널이 아닐 때, `x.equals(y)`와 `y.equals(z)`가 true라면, `x.equals(z)`도 true여야 한다는 것이다.

이러한 연속적인 동작을 설계할 때, 가장 큰 문제는 타입이 다른 경우이다.

잘못된 예

```kotlin
open class Date(
    val year: Int,
    val month: Int,
    val day: Int,
) {

    // 대칭적이지만 연속적이지 못하다.
    override fun equals(o: Any?): Boolean = when (o) {
        is DateTime -> this == o.date
        is Date -> o.day == day && o.month == month && o.year == year
        else -> false
    }

    // ...
}

class DateTime(
    val date: Date,
    val hour: Int,
    val minute: Int,
    val second: Int,
) : Date(date.year, date.month, date.day) {

    // 대칭적이지만 연속적이지 못하다.
    override fun equals(o: Any?): Boolean = when (o) {
        is DateTime -> o.date == date && o.hour == hour && o.minute == minute && o.second == second
        is Date -> o.day == day && o.month == month && o.year == year
        else -> false
    }

    // ...
}
```

```kotlin
val o1 = DateTime(Date(1992, 10, 20), 12, 30, 0)
val o2 = Date(1992, 10, 20)
val o3 = DateTime(Date(1992, 10, 20), 14, 45, 30)

o1 == o2 // true
o2 == o3 // true
o1 == o3 // false
```

날짜가 같지만 시간이 다른 두 `DateTime` 객체를 비교하면 false가 나오지만, 일한 것들과 날짜가 같은 `Date` 객체를 비교하면 true가 나온다. 즉, 연속적이지 않은 관계를 갖게 된다.

현재 `Date`와 `DateTime`이 상속 관계를 가지므로, 같은 객체끼리만 비교하게 만드는 방법은 좋지 않다. 이렇게 구현하면 리스코프 치환 원칙을 위반하기 때문이다. 따라서 처음부터 상속 대신 컴포지션을 사용하고, 두 객체를 아예 비교하지 못하게 만드는 것이 좋다.

### 일관성을 가져야 한다.

두 객체를 비교한 결과는 한 객체를 수정하지 않는 한 항상 같은 결과를 내야 한다. immutable 객체라면 결과가 언제나 같아야 한다. 즉, `equals`는 반드시 비교 대상이 되는 두 객체에만 의존하는 순수 함수여야 한다.

대표적인 예로는 `Time` 클래스와 `java.net.URL.equals()`가 있다.

### null 과 같을 수 없다.

x가 널이 아닐 때, 모든 `x.equals(null)`은 false를 리턴해야 한다.

## URL과 관련된 equals 문제

`equals`를 굉장히 잘못 설계한 예로는 `java.net.URL`이 있다. `java.net.URL` 객체 2개를 비교하면 동일한 IP 주소로 해석될 때는 true, 아닐 때는 false가 나오는데, 이 결과가 네트워크 상태에 따라서 달라진다.

```kotlin
import java.net.URL

fun main() {
    val enWiki = URL("https://en.wikipedia.org/")
    val wiki = URL("https://wikipedia.org/")
    println(enWiki == wiki) // (1)
}
```

- (1) 위의 코드는 상황에 따라서 결과가 달라진다. 일반적인 상황에서는 두 주소가 같은 IP 주소를 나타내므로 true를 출력한다. 하지만 인터넷 연결이 끊겨 있으면, false를 출력한다.

이 설계의 문제점

- 동작이 일관되지 않다. 네트워크가 정상이라면 두 URL이 같고, 문제가 있다면 다릅니다. 네트워크 설정에 따라서도 결과가 달라질 수 있다. 주어진 호스트의 IP 주소는 시간과 네트워크 상황에 따라서 다르다.
- 일반적으로 `equals`와 `hashCode` 처리는 빠를 거라 예상하지만, 네트워크 처리는 굉장히 느리다.
- 동작 자체에 문제가 있다. 동일한 IP 주소를 갖는다고, 동일한 콘텐츠를 나타내는 것이 아니다. 가상 호스팅을 한다면, 관련 없는 사이트가 같은 IP 주소를 공유할 수도 있다.

코틀린/JVM 또는 다른 플랫폼을 사용할 때는 `java.net.URL`이 아니라 `java.net.URI`를 사용해서 이런 문제를 해결한다.

## equals 구현하기

특별한 이유가 없는 이상, 직접 `equals`를 구현하는 것은 좋지 않다. 기본적으로 제공되는 것을 그대로 쓰거나, 데이터 클래스로 만들어서 사용하는 것이 좋다.

그래도 직접 구현해야 한다면, 반사적, 대칭적, 연속적, 일관적 동작을 하는지 꼭 확인해야 한다.

그리고 이러한 클래스는 `final`로 만드는 것이 좋다. 만약 상속을 한다면, 서브클래스에서 `equals`가 작동하는 방식을 변경하면 안 된다. 상속을 지원하면서도 완벽한 사용자 정의 `equals` 함수를 만드는 것은 거의 불가능에 가깝다.
