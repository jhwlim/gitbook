---
description: 이펙티브 코틀린 정리하기
---

# 적절하게 null을 처리하라

{% hint style="info" %}
프로퍼티가 null이라는 것은 값이 설정되지 않았거나, 제거되었다는 것을 나타낸다.
{% endhint %}

기본적으로 nullable 타입은 3가지 방법으로 처리한다.

- 안전 호출(`.?`), 스마트 캐스팅, Elvis 연산자(`?:`) 등을 활용한다.
- 오류를 `throw` 한다.
- 함수 또는 프로퍼티를 리팩터링해서 nullable 타입이 나오지 않도록 바꾼다.

## null을 안전하게 처리하기

### 안전 호출과 스마트 캐스팅 사용하기

null을 안전하게 처리하는 방법 중 널리 사용되는 방법으로는 안전 호출과 스마트 캐스팅이 있다.

```kotlin
// 안전 호출
printer?.print() 

// 스마트 캐스팅
if (printer != null) {
    printer.print() 
}
```

스마트 캐스팅은 코틀린의 규약 기능을 지원한다.

```kotlin
println("What is your name?")
val name = readLine()
if (!name.isNullOrBlank()) {
    println("Hello ${name.toUpperCase()})
}

val news: List<News>? = getNews()
if (!news.isNullOrEmpty()) {
    news.forEach { notifyUser(it) }
}
```

### Elvis 연산자 사용하기

다른 방법으로는 Elvis 연산자를 사용하는 것이다. Elvis 연산자는 오른쪽에 `return` 또는 `throw` 을 포함한 모든 표현식이 허용된다.

```kotlin
val printerName1 = printer?.name ?: "Unnamed"
val printerName2 = printer?.name ?: return
val printerName3 = printer?.name ?: throw Error("Printer must be named")
```

### 방어적 프로그래밍과 공격적 프로그래밍

**방어적 프로그래밍**은 프로덕션 환경으로 들어갔을 때 발생할 수 있는 수많은 것들로부터 프로그램을 방어해서 안정성을 높이는 방법을 나타내는 포괄적인 용어이다. 상황을 처리할 수 있는 올바른 방법이 있을 때는 굉장히 좋다.

**공격적 프로그래밍**은 예상하지 못한 상황이 발생했을 때, 이러한 문제를 개발자에게 알려서 수정하게 만드는 것이다.

**둘은 코드의 안전을 위해 모두 필요하다. 둘을 모두 이해하고 적절하게 사용할 수 있어야 한다.**

## 오류 throw 하기

{% hint style="success" %}
다른 개발자가 어떤 코드를 보고 선입견처럼 '당연히 그럴 것이다'라고 생각하게 되는 부분이 있고, 그 부분에서 문제가 발생할 경우에는 개발자에게 오류를 강제로 발생시켜 주는 것이 좋다.
{% endhint %}

오류를 강제로 발생시킬 때는 `throw`, `!!`, `requireNotNull()`, `checkNotNull()` 등을 활용한다.

## not-null assertion(`!!`)과 관련된 문제

not-null assertion(`!!`)은 사용하기 쉽지만 nullable을 처리하는 좋은 해결 방법이 아니다. **어떤 설명도 없는 제네릭 예외가 발생한다.** 또한, **코드가 짧고 너무 사용하기 쉽다 보니 남용하게 되는 문제도 있다.** `!!`은 타입이 nullable이지만, null이 나오지 않는다는 것이 거의 확실한 상황에서 많이 사용되는데, 현재 확실하다고 미래에 확실한 것은 아니기 때문에 **미래의 어느 순간에 문제가 발생할 수 있다.**

변수를 선언하고 이후에 사용하기 전에 값을 할당해서 사용해야 하는 경우에, 변수를 null로 설정하고 이후에 `!!` 연산자를 사용하는 방법은 **프로퍼티를 계속 언팩(unpack)해야 하므로 사용하기 귀찮다. 또한, 의미 있는 null 값을 가질 가능성 자체를 차단해 버린다.** (올바른 방법은 `lateinit` 또는 `Delegates.notNull` 을 사용하는 것이다.)

{% hint style="success" %}
예외는 예상하지 못한 잘못된 부분을 알려 주기 위해서 발생하는 것이다. 명시적 오류는 제네릭 NPE보다는 훨씬 더 많은 정보를 제공해 줄 수 있기 때문에 `!!` 연산자를 사용하는 것보다 훨씬 좋다.
{% endhint %}

{% hint style="danger" %}
`!!` 연산자가 의미 있는 경우는 굉장히 드물다. 일반적으로 nullability가 제대로 표현되지 않는 라이브러리를 사용할 때 정도에만 사용해야 한다. 코틀린을 대상으로 설계된 API를 활용한다면, `!!` 연산자를 사용하는 것을 이상하게 생각해야 한다.
{% endhint %}

## 의미 없는 nullability 피하기

{% hint style="success" %}
nullability는 어떻게든 적절하게 처리해야 하므로 추가 비용이 발생한다. 따라서 필요한 경우가 아니라면, nullability 자체를 피하는 것이 좋다.

null은 중요한 메시지를 전달하는 데 사용될 수 있다. 따라서 다른 개발자가 보기에 의미가 없을 때는 null을 사용하지 않는 것이 좋다. 
{% endhint %}

### 방법

- 클래스에서 nullability에 따라 여러 함수를 만들어서 제공할 수 있다. 예) `List<T>`의 `get()`과 `getOrNull()`
- 어떤 값이 클래스 생성 이후에 확실하게 설정된다는 보장이 있다면 `lateinit` 프로퍼티와 notNull 델리게이트를 사용하자.
- 빈 컬렉션 대신 null을 리턴하지 말자. null은 컬렉션 자체가 없다는 것을 나타낸다. 요소가 부족하다는 것을 나타내려면 빈 컬렉션을 사용하자.
- nullable enum과 None enum 값은 완전히 다른 의미이다. null enum은 별도로 처리해야 하지만, None enum 정의에 없으므로 필요한 경우에 사용하는 쪽에서 추가해서 활용할 수 있다는 의미이다.

## lateinit 프로퍼티와 nutNull 델리게이트

클래스는 클래스 생성 중에 초기화할 수 없는 프로퍼티를 가질 수 있다. 이러한 프로퍼티는 사용 전에 반드시 초기화해서 사용해야 한다.

### lateinit

`lateinit` 한정자는 **프로퍼티가 이후에 설정될 것임을 명시하는 한정자**이다. 만약, 초기화 전에 값을 사용하려고 하면 예외가 발생한다.

nullable과 비교해서 다음과 같은 차이가 있다.

- `!!` 연산자로 언팩하지 않아도 된다.
- 이후에 어떤 의미를 나타내기 위해서 null을 사용하고 싶을 때, nullable로 만들 수 있다.
- 프로퍼티가 초기화된 이후에는 초기화되지 않은 상태로 돌아갈 수 없다.

**Int, Long, Double, Boolean과 같은 기본 타입과 연결된 타입으로 프로퍼티를 초기화해야 하는 경우 `lateinit`을 사용할 수 없다.**

### Delegates.notNull

```kotlin
class DoctorActivity: Activity() {
    private var doctorId: Int by Delegates.notNull()
    private var fromNotification: Boolean by Delegates.notNull()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        doctorId = intent.extras.getInt(DOCTOR_ID_ARG)
        fromNotification = intent.extras.getBoolean(FROM_NOTIFICATION_ARG)
    }
}
```

지연 초기화하는 형태로 다음과 같이 프로퍼티 위임을 사용할 수도 있다.

```kotlin
class DoctorActivity: Activity() {
    private var doctorId: Int by arg(DOCTOR_ID_ARG)
    private var fromNotification: Boolean by arg(FROM_NOTIFICATION_ARG)
}
```

프로퍼티 위임을 사용하면 nullability로 발생하는 여러 가지 문제를 안전하게 처리할 수 있다.