---
description: 이펙티브 코틀린 정리하기
---

# 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라

프로퍼티 위임을 사용하면 일반적인 프로퍼티의 행위를 추출해서 재사용할 수 있다.

## lazy 프로퍼티 패턴

지연 프로퍼티는 lazy 프로퍼티는 이후에 처음 사용하는 요청이 들어올 때 초기화되는 프로퍼티를 의미한다.

stdlib의 lazy 함수를 사용하여 lazy 프로퍼티 패턴을 쉽게 구현할 수 있다.

```kotlin
val value by lazy { createValue() }
```

## observable 패턴

프로퍼티 위임을 사용하면, 변화가 있을 때 이를 감지하는 observable 패턴을 쉽게 만들 수 있다.

stdlib의 observable 델리게이트를 기반으로 간단하게 구현할 수 있다.

```kotlin
var items: List<Item> by
    Delegates.observable(listOf()) { _, _, _ -> 
        notifyDataSetChanged()
    }

var key: String? by
    Delegates.observable(null) { _, old, new ->
        Log.e("key changed from $old to $new")
    }
```

## 프로퍼티 위임을 활용한 다양한 패턴

일반적으로 프로퍼티 위임 메커니즘을 활용하면, 다양한 패턴들을 만들 수 있다.

뷰, 리소스 바인딩, 의존성 주입, 데이터 바인딩 등의 이런 패턴들을 사용할 때 자바 등에서는 어노테이션을 많이 활용해야 한다. 하지만 코틀린은 프로퍼티 위임을 사용해서 간단하고 type-safe하게 구현할 수 있다.

```kotlin
// 안드로이드에서의 뷰와 리소스 바인딩
private val button: Button by bindView(R.id.button)
private val textSize by bindDimension(R.dimen.font_size)
private val doctor: Doctor by argExtra(DOCTOR_ARG)

// Koin에서의 종속성 주입
private val presenter: MainPresenter by inject()
private val repository: NetworkRepository by inject()
private val vm: MainViewModel by viewModel()

// 데이터 바인딩
private val port by bindCOnfiguration("port")
private val token: String by preferences.bind(TOKEN_KEY)
```

## 예시

아래의 두 프로퍼티는 타입은 다르지만, 내부적으로 거의 같은 처리를 한다.

```kotlin
var token: String? = null
    get() {
        println("token returned value $field")
        return field
    }
    set(value) {
        println("token changed from $field to $value")
        field = value
    }

var attempts: Int = 0
    get() {
        println("attempts returned value $field")
        return field
    }
    set(value) {
        println("attempts changed from $field to $value")
        field = value
    }
```

### 프로퍼티 위임을 활용하여 추출하기

{% hint style="info" %}
**프로퍼티 위임** 이란 다른 객체의 메서드를 활용해서 프로퍼티의 접근자(게터와 세터)를 만드는 방식이다. 이때, **다른 객체의 메서드 이름은 게터는 `getValue`, 세터는 `setValue` 함수를 사용해서 만들어야 한다.** 객체를 만든 뒤에는 `by` 키워드를 사용해서 `getValue`와 `setValue`를 정의한 클래스와 연결해주면 된다.
{% endhint %}

```kotlin
var token: String? by LogginProperty(null)
var attempts: Int by LogginProperty(0)

private class LogginProperty<T>(
    var value: T
) {
    operator fun getValue(
        thisRef: Any?,
        prop: KProperty<*>
    ): T {
        println("${prop.name} returned value $value")
        return value
    }

    operator fun setValue(
        thisRef: Any?,
        prop: KProperty<*>,
        newValue: T
    ) {
        val name = prop.name
        println("$name changed from $value to $newValue")
        value = newValue
    }
}
```

`getValue`와 `setValue` 메서드는 여러 개 있더라도 내부적으로 컨텍스트(`this`)를 활용하기 때문에 상황에 따라서 적절한 메서드가 선택된다.

객체를 프로퍼티 위임하려면 `val`의 경우 `getValue` 연산, `var`의 경우 `getValue`와 `setValue` 연산이 필요하다. 이러한 연산은 확장 함수로도 만들 수 있다.

## 알아두면 좋은 프로퍼티 델리게이터

- `lazy`
- `Delegates.observable`
- `Delegates.vetoable`
- `Delegates.notNull`

## 정리

프로퍼티 델리게이트는 프로퍼티와 관련된 다양한 조작을 할 수 있으며, 컨텍스트와 관련된 대부분의 정보를 갖는다. 이러한 특징으로 인해서 다양한 프로퍼티의 동작을 추출해서 재사용할 수 있다.

프로퍼티 위임은 프로퍼티 패턴을 추출하는 일반적인 방법으로 많이 사용되고 있다. 이를 활용하면 일반적인 패턴을 추출하거나 더 좋은 API를 만들 때 활용할 수 있다.