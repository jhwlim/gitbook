---
description: 이펙티브 코틀린 정리하기
---

# 인라인 클래스의 사용을 고려하라

하나의 값을 보유한 객체도 inline으로 만들 수 있다.

기본 생성자 프로퍼티가 하나인 클래스 앞에 `inline`을 붙이면, 해당 객체를 사용하는 위치가 모두 해당 프로퍼티로 교체된다. 이러한 `inline` 클래스는 타입만 맞다면, 그냥 값을 곧바로 집어 넣는 것도 허용된다.

```kotlin
inline class Name(
    private val value: String,
) { ... }

val name: Name = Name("Marcin")

// 컴파일
val name: String = "Marcin"
```

`inline` 클래스의 메서드는 모두 정적 메서드로 만들어진다.

```kotlin
inline class Name(
    private val value: String,
) {
    // ...

    fun greet() {
        print("Hello, I am $value")
    }
}

val name: Name = Name("Narcin")
name.greet()

// 컴파일
val name: String = "Marcin"
Name.`greet-imple`(name)
```

인라인 클래스는 다른 자료형을 래핑해서 새로운 자료형을 만들 때 많이 사용된다. 이때, 어떠한 오버헤드도 발생하지 않는다.

## inline 클래스를 사용하는 경우

### 측정 단위를 표현할 때

측정 단위 혼동은 굉장히 큰 문제를 초래할 수 있다.

이러한 문제를 해결할 수 있는 가장 쉬운 방법은 파라미터 이름에 측정 단위를 붙여 주는 것이다. 하지만 함수를 사용할 때 프로퍼티 이름이 표시되지 않을 수 있으므로, 여전히 실수 할 수 있다. 또한 파라미터는 이름을 붙일 수 있지만, 리턴 값은 이름을 붙일 수 없다. 물론 함수에 이름을 붙여서, 어떤 단위로 리턴하는지 알려 줄 수 있다. 하지만 이러한 해결 방법은 함수를 더 길게 만들고, 필요 없는 정보까지도 전달해 줄 가능성이 있으므로, 실제로는 거의 사용되지 않는다.

더 좋은 해결 방법은 타입에 제한을 거는 것이다. 제한을 걸면 제네릭 유형을 잘못 사용하는 문제를 줄일 수 있다. 그리고 이때 코드를 더 효율적으로 만들려면, 인라인 클래스를 활용한다.

```kotlin
inline class Minutes(val minutes: Int) {
    fun toMillis(): Millis = Millis(minutes * 60 * 1000)
    // ...
}

inline class Millis(val milliseconds: Int) { ... }

interface User {
    fun decideAboutTime(): Minutes
    fun wakeUp()
}

interface Timer {
    fun callAfter(timeMillis: Millis, callback: () -> Unit)
}
```

프런트엔드 개발자에서는 px, mm, dp 등의 다양한 단위를 사용하는데, 이러한 단위를 제한할 때 활용하면 좋다. 또한 객체 생성을 위해서 DSL-like 확장 프로퍼티를 만들어 두어도 좋다.

### 타입 오용으로 발생하는 문제를 막을 때

SQL 데이터베이스는 일반적으로 ID를 사용해서 요소를 식별하고, ID는 일반적으로 단순한 숫자이다.

```kotlin
@Entity(tableName = "grades")
class Grades(
    @ColumnInfo(name = "studentId")
    val studentId: Int
    @ColumnInfo(name = "teacherId")
    val teacherId: Int
    @ColumnInfo(name = "schoolId")
    val schoolId: Int
)
```

위 코드의 문제점

- 모든 ID가 Int 자료형이므로, 실수로 잘못된 값을 넣을 수 있다.
- 또한, 이러한 문제가 발생했을 때 어떠한 오류도 발생하지 않으므로, 문제를 찾는 게 힘들어진다.

```kotlin
inline class StudentId(val studentId: Int)
inline class TeacherId(val teacherId: Int)
inline class SchoolId(val schoolId: Int)

@Entity(tableName = "grades")
class Grades(
    @ColumnInfo(name = "studentId")
    val studentId: StudentId
    @ColumnInfo(name = "teacherId")
    val teacherId: TeacherId
    @ColumnInfo(name = "schoolId")
    val schoolId: SchoolId
)
```

- ID를 사용하는 것이 굉장히 안전해지며, 컴파일할 때 타입이 Int로 대체되므로 코드를 바꾸어도 별도의 문제가 발생하지 않는다.

{% hint style="success" %}
인라인 클래스를 사용하면, 안전을 위해 새로운 타입을 도입해도 추가적인 오버헤드가 발생하지 않는다.
{% endhint %}

## 인라인 클래스와 인터페이스

인라인 클래스도 다른 클래스와 마찬가지로 인터페이스를 구현할 수 있다.

하지만 인터페이스를 통해서 타입을 나타내려면, 객체를 래핑해서 사용해야 하기 때문에 클래스를 `inline`으로 만들었을 때 얻을 수 있는 장점이 하나도 없다.

## typealias

`typealias`를 사용하면, 타입에 새로운 이름을 붙여 줄 수 있다.

이러한 `typealias`는 길고 반복적으로 사용해야 할 때 많이 유용하다.

하지만 `typealias`는 안전하지 않다. 실수로 혼용해서 잘못 입력하더라도, 어떠한 오류도 발생하지 않는다. 하지만 명확하게 타입 이름이 구분되어 있으므로, 안전할 거라는 착각을 하게 만든다. 이는 오히려 문제가 발생했을 때, 문제 찾는 것을 어렵게 만든다.

```kotlin
typealias Seconds = Int
typealias Millis = Int

fun getTime(): Millis = 10
fun setUpTimer(time: Seconds) {}

fun main() {
    val seconds: Seconds = 10
    val millis: Millis = seconds // 컴파일 오류가 발생하지 않는다.

    setUpTimer(getTime())
}
```

{% hint style="warning" %}
`typealias`를 사용하지 않는 것이 오히려 오류를 쉽게 찾을 수 있다. 따라서 이런 형태로 `typealias`를 사용하면 안 된다.
{% endhint %}

단위 등을 표현하려면, 파라미터 이름 또는 클래스를 사용하는 것이 좋다. 이름은 비용이 적게 들고, 클래스는 안전하다.

{% hint style="success" %}
인라인 클래스를 사용하면, 비용은 적게 들면서 안전하다.
{% endhint %}

## 정리

인라인 클래스를 사용하면 성능적인 오버헤드 없이 타입을 래핑할 수 있다.

인라인 클래스는 타입 시스템을 통해 실수로 코드를 잘못 작성하는 것을 막아주므로, 코드의 안정성을 향상시켜 준다.

의미가 명확하지 않은 타입, 특히 여러 측정 단위들을 함께 사용하는 경우에는 인라인 클래스를 활용하는 것이 좋다.
