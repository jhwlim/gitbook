---
description: 이펙티브 코틀린 정리하기
---

# 생성자 대신 팩토리 함수를 사용하라

## 팩토리 함수

생성자의 역할을 대신 해주는 함수

### 장점

- 함수에 이름을 붙일 수 있다. 이름은 객체가 생성되는 방법과 아규먼트로 무엇이 필요한지 설명할 수 있다. 이름이 붙어 있다면, 이해하기 쉽다. 또한, 동일한 파라미터 타입을 갖는 생성자의 충돌을 줄일 수 있다.
- 함수가 원하는 형태의 타입을 리턴할 수 있다. 따라서 다른 객체를 생성할 때 사용할 수 있다. 인터페이스 뒤에 실제 객체의 구현을 숨길 때 유용하게 사용할 수 있다.
- 호출될 때마다 새 객체를 만들 필요가 없다. 함수를 사용해서 객체를 생성하면 싱글턴 패턴처럼 객체를 하나만 생성하게 강제하거나, 최적화를 위해 캐싱 매커니즘을 사용할 수 있다. 또한, 객체를 만들 수 없을 경우, `null`을 리턴하게 만들 수도 있다.
- 아직 존재하지 않는 객체를 리턴할 수도 있다. 이를 활용하면 프로젝트를 빌드하지 않고도 앞으로 만들어질 객체를 사용하거나, 프록시를 통해 만들어지는 객체를 사용할 수 있다.
- 객체 외부에 팩토리 함수를 만들면, 그 가시성을 원하는 대로 제어할 수 있다. 예를 들어 톱레벨 팩토리 함수를 같은 파일 또는 같은 모듈에서만 접근하게 할 수 있다.
- 인라인으로 만들 수 있으며, 그 파라미터들은 `reified`로 만들 수 있다.
- 팩토리 함수는 생성자로 만들기 복잡한 객체도 만들어 낼 수 있다.
- 원하는 때에 생성자를 호출할 수 있다. (생성자는 즉시 슈퍼클래스 또는 기본 생성자를 호출해야 한다.)

<br>

다만 팩토리함수로 클래스를 생성할 때 서브클래스 생성에는 슈퍼클래스의 생성자가 필요하기 때문에, 서브클래스를 만들어낼 수 없다.

```kotlin
class IntLinkedList: MyLinkedList<Int>() { // MyLinkedList가 open이라면

    constructor(vararg ints: Int): myLinkedListOf(*ints) // 오류

}
```

하지만 팩토리 함수로 슈퍼클래스를 만들기로 했다면, 그 서브클래스에도 팩토리 함수를 만들면 된다.

```kotlin
class MyLinkedIntList(
    head: Int,
    tail: MyLinkedIntList?
) : MyLinkedList<Int>(head, tail)

fun myLinkedIntListOf(vararg elements: Int): MyLinkedIntList? {
    if (elements.isEmpty()) {
        return null
    }

    val head = elements.first()
    val elementsTail = elements.copyOfRange(1, elements.size)
    val tail = myLinkedIntListOf(*elementsTail)
    return MyLinkedIntList(head, tail)
}
```

<br>

팩토리 함수는 기본 생성자가 아닌 추가적인 생성자(secondary constructor)와 경쟁 관계이며, 추가적인 생성자보다 팩토리 함수를 많이 사용한다. 팩토리 함수는 다른 종류의 팩토리 함수와 경쟁 관계에 있다고 할 수 있다.

### 종류

- companion 객체 팩토리 함수
- 확장 팩토리 함수
- 톱레벨 팩토리 함수
- 가짜 생성자
- 팩토리 클래스의 메서드

## Companion 객체 팩토리 함수

팩토리 함수를 정의하는 가장 일반적인 방법

### 특징

인터페이스에도 구현할 수 있다.

```kotlin
class MyLinkedList<T>(
    val head: T,
    val tail: MyLinkedList<T>?
) : MyList<T> { ... }

interface MyList<T> {
    // ...

    companion object {

        fun <T> of(vararg elements: T): MyList<T>? { ... } 

    }
}

// 사용
val list = MyList.of(1, 2)
```

### 많이 사용하는 이름

|이름|설명|
|:-:|:--|
|from|파라미터를 하나 받고, 같은 타입의 인스턴스 하나를 리턴하는 타입 변환 함수
|of|파라미터를 여러 개 받고, 이를 통합해서 인스턴스를 만들어 주는 함수
|valueOf|`from` 또는 `of`와 비슷한 기능을 하면서도, 의미를 조금 더 쉽게 읽을 수 있게 이름을 붙인 함수
|instance<br>getInstance|싱글턴으로 인스턴스 하나를 리턴하는 함수<br>파라미터가 있을 경우, 아규먼트를 기반으로 하는 인스턴스를 리턴한다. 일반적으로 같은 아규먼트를 넣으면, 같은 인스턴스를 리턴하는 형태로 작동한다.
|createInstance<br>newInstance|`getInstance`처럼 동작하지만, 함수를 호출할 때마다 새로운 인스턴스를 만들어서 리턴하는 함수
|getType|`getInstance`처럼 동작하지만, 팩토리 함수가 다른 클래스에 있을 때 사용하는 이름 (`Type`은 팩토리 함수에서 리턴하는 타입)
|newType|`newInstance`처럼 동작하지만, 팩토리 함수가 다른 클래스에 있을 때 사용하는 이름 (`Type`은 팩토리 함수에서 리턴하는 타입)

{% hint style="info" %}
**companion object** 객체는 인터페이스를 구현할 수 있으며, 클래스를 상속 받을 수 있다.
{% endhint %}

일반적으로 다음과 같은 형태로 companion 객체를 만드는 팩토리 함수를 만든다.

```kotlin
abstract class ActivityFactory {
    abstract fun getIntent(context: Context): Intent

    fun start(context: Context) {
        val intent = getIntent(context)
        context.startActivity(intent)
    }

    fun startForResult(activity: Activity, requestCode: Int) {
        val intent = getIntent(activity)
        activity.startActivityForResult(intent, requestCode)
    }

}

class MainActivity : AppCompatActivity() {
    // ...

    companion object: ActivityFactory() {
        override fun getIntent(context: Context): Intent = Intent(context, MainActivity::class.java)
    }
}

// 사용
val intent = MainActivity.getIntent(context)
MainActivity.start(context)
MainActivity.startForResult(activity, requestCode)
```

## 확장 팩토리 함수

이미 companion 객체가 존재하고 직접 수정할 수 없을 때, 이 객체의 함수처럼 사용할 수 있는 팩토리 함수를 만들어야 할 때, 확장 함수를 활용하면 된다.

```kotlin
fun Tool.Companion.createBigTool( /*...*/ ): BigTool { ... }

// 사용
Tool.createBigTool()
```

외부 라이브러리를 확장할 수 있다.

다만, companion 객체를 확장하려면, (적어도 비어 있는) 컴페니언 객체가 필요하다.

```kotlin
interfact Tool {
    companion object {}
}
```

## 톱레벨 팩토리 함수

대표적인 예 : `listOf`, `setOf`, `mapOf`

### 단점

- public 톱레벨 함수는 모든 곳에서 사용할 수 있으므로, IDE가 제공하는 팁을 복잡하게 만든다.
- 톱레벨 함수의 이름을 클래스 메서드 이름처럼 만들면, 다양한 혼란을 일으킬 수 있다.


{% hint style="warning" %}
톱레벨 함수를 만들 때는 꼭 **이름**을 신중하게 생각해서 잘 지정해야 한다.
{% endhint %}


## 가짜 생성자

```kotlin
public inline fun <T> List(
    size: Int,
    init: (index: Int) -> T
): List<T> = MutableList(size, init)

public inline fun <T> MutableList(
    size: Int,
    init: (index: Int) -> T
): MutableList<T> {
    val list = ArrayList<T>(size)
    repeat(size) { index -> list.add(init(index)) }
    return list
}
```

이러한 톱레벨 함수는 생성자처럼 보이고, 생성자처럼 작동한다. 하지만 팩토리 함수와 같은 모든 장점을 갖는다. 이것을 **가짜 생성자**라고 부른다.

### 가짜 생성자를 만드는 이유

- 인터페이스를 위한 생성자를 만들고 싶을 때
- `reified` 타입 아규먼트를 갖게 하고 싶을 때

이를 제외하면, 가짜 생성자는 진짜 생성자처럼 동작해야 한다. 생성자처럼 보여야 하며, 생성자와 같은 동작을 해야 한다.


{% hint style="success" %}
캐싱, nullable 타입 리턴, 서브클래스 리턴 등의 기능을 포함해서 객체를 만들고 싶다면, companion 객체 팩토리 메서드처럼 다른 이름을 가진 팩토리 함수를 사용하는 것이 좋다.
{% endhint %}

### 가짜 생성자를 선언하는 또 다른 방법

`invoke` 연산자를 갖는 companion 객체를 사용하면, 비슷한 결과를 얻을 수 있다.

```kotlin
class Tree<T> {

    companion object {
        operator fun <T> invoke(size: Int, generator: (Int) -> T): Tree<T> { ... }
    }

}

// 사용
Tree(10) { "$it" }
```

{% hint style="danger" %}
다만 이와 같은 방식은 거의 사용되지 않으며, 필자도 추천하지 않는 방법이다. 이는 '아이템 12: 연산자 오버로드를 할 때는 의미에 맞게 하라'에 위배되기 때문이다.
{% endhint %}

{% hint style="success" %}
가짜 생성자는 톱레벨 함수를 사용하는 것이 좋다. <u>기본 생성자를 만들 수 없는 상황 또는 생성자가 제공하지 않는 기능(예: `reified` 타입 파라미터 등)으로 생성자를 만들어야 하는 상황에만</u> 가짜 생성자를 사용하는 것이 좋다.
{% endhint %}

## 팩토리 클래스의 메서드

팩토리 클래스와 관련된 추상 팩토리, 프로토타입 등의 수많은 생성 패턴이 존재하는데, 이러한 패턴 중 일부는 코틀린에서는 적합하지 않다. (예: 점층적 생성자 패턴, 빌더 패턴 등)

팩토리 클래스는 클래스의 상태를 가질 수 있다는 특징 때문에 팩토리 함수보다 다양한 기능을 갖는다.

```kotlin
data class Student(
    val id: Int,
    val name: String,
    val surname: String,
)

class StudentsFactory {
    var nextId = 0
    fun next(name: String, surname: String) = Student(nextId++, name, surname)
}
```

팩토리 클래스는 프로퍼티를 가질 수 있다. 이를 활용하면 다양한 종류로 최적화하고, 다양한 기능을 도입할 수 있다. 예를 들어 캐싱을 활용하거나, 이전에 만든 객체를 복제해서 객체를 생성하는 방법으로 객체 생성 속도를 높일 수 있다.

## 정리

{% hint style="success" %}
코틀린은 팩토리 함수를 만들 수 있는 다양한 방법들을 제공하고 있으며, 각각 여러 특징을 갖고 있다. 객체를 생성할 때는 이러한 특징을 잘 파악하고 사용해야 한다. 
{% endhint %}

{% hint style="warning" %}
가짜 생성자, 톱레벨 팩토리 함수, 확장 팩토리 함수 등 일부는 신중하게 사용해야 한다.
{% endhint %}