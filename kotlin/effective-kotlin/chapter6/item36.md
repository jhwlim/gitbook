---
description: 이펙티브 코틀린 정리하기
---

# 상속보다는 컴포지션을 사용하라 

상속은 `is-a` 관계의 객체 계층 구조를 만들기 위해 설계되었다. 상속은 관계가 명확하지 않을 때 사용하면, 여러 가지 문제가 발생할 수 있다.

{% hint style="success" %}
따라서 단순하게 코드 추출 또는 재사용을 위해 상속을 하려고 한다면, 조금 더 신중하게 생각해야 한다. 일반적으로 이러한 경우에는 상속보다 컴포지션을 사용하는 것이 좋다.
{% endhint %}

## 간단한 행위 재사용

### 상속의 단점

- 상속은 하나의 클래스만을 대상으로 할 수 있다. 상속을 사용해서 행위를 추출하다 보면, 많은 함수를 갖는 거대한 BaseXXX 클래스를 만들게 되고, 굉장히 깊고 복잡한 계층 구조가 만들어진다.
- 상속은 클래스의 모든 것을 가져오게 된다. 따라서 불필요한 함수를 갖는 클래스가 만들어질 수 있다. (인터페이스 분리 원칙을 위반하게 된다.)
- 상속은 이해하기 어렵다. 일반적으로 개발자가 메서드를 읽고, 메서드의 작동 방식을 이해하기 위해 슈퍼클래스를 여러 번 확인해야 한다면, 문제가 있는 것이다.


이러한 이유 떄문에 다른 대안을 사용하는 것이 좋다. 대표적인 대안이 컴포지션이다. 

### 컴포지션

컴포지션을 사용한다는 것은 **객체를 프로퍼티로 갖고, 함수를 호출하는 형태로 재사용하는 것을 의미한다.**

객체를 다른 모든 객체에서 갖고 활용하는 추가 코드가 필요하다. 이러한 추가 코드를 적절하게 처리하는 것이 조금 어려울 수도 있어 컴포지션보다 상속을 선호하는 경우도 많다.

하지만 이런 추가 코드로 인해서 코드를 읽는 사람들이 코드의 실행을 더 명확하게 예측할 수 있다는 장점도 있고, 훨씬 자유롭게 사용할 수 있다는 장점이 있다. 또한 하나의 클래스 내부에서 여러 기능을 재사용할 수 있게 된다.

## 모든 것을 가져올 수 밖에 없는 상속

상속은 슈퍼클래스의 메서드, 제약, 행위 등 모든 것을 가져온다. 따라서 상속은 객체의 계층 구조를 나타낼 때 굉장히 좋은 도구이다. 하지만 일부분을 재사용하기 위한 목적으로는 적합하지 않다. 

{% hint style="success" %}
일부분만 재사용하고 싶다면, 컴포지션을 사용하는 것이 좋다. 컴포지션은 우리가 원하는 행위만 가져올 수 있기 때문이다.
{% endhint %}

```kotlin
abstract class Dog {
    open fun bark() { ... }
    open fun sniff() { ... }
}

class RobotDog : Dog() {
    override fun sniff() {
        throw Error("Operation not supported")
    }
}
```
- `RobotDog`는 필요도 없는 메서드를 갖기 때문에, 인터페이스 분리 원칙에 위반된다.
- 또한, 슈퍼클래스의 동작을 서브클래스에서 깨버리므로, 리스코프 치환 원칙에도 위반된다.

만약 타입 계층 구조를 표현해야 한다면, 인터페이스를 활용해서 다중 상속을 하는 것이 좋을 수도 있다.

## 캡슐화를 깨는 상속

상속을 활용할 때는 외부에서 이를 어떻게 활용하는지도 중요하지만, 내부적으로 이를 어떻게 활용하는지도 중요하다. 내부적인 구현 방법 변경에 의해 클래스의 캡슐화가 깨질 수 있기 때문이다.

### 상속을 사용했을 때 문제

```kotlin
class CounterSet<T> : HashSet<T>() {
    var elementsAdded: Int = 0
        private set

    override fun add(element: T): Boolean {
        elementsAdded++
        return super.add(element)
    }

    override fun addAll(elements: Collection<T>): Boolean {
        elementsAdded += elements.size
        return super.addAll(elements)
    }

}

// 사용
val counterList = CounterSet<String>()
counterList.addAll(listOf("A", "B", "C"))
print(counterList.elementsAdded) // 6
```

위 코드의 문제점

- HashSet의 `addAll()` 내부에서 `add()`를 사용했기 때문에 `counterList.elementsAdded`는 3이 아닌 6이 된다.

위의 문제를 해결하기 위해 아래와 같이 구현한 `addAll()`를 삭제하면 의도한 대로 동작할 것이다.

```kotlin
class CounterSet<T>: HashSet<T>() {
    var elementsAdded: Int = 0
        private set

    override fun add(element: T): Boolean {
        elementsAdded++
        return super.add(element)
    }
}
```

위 코드의 문제점

하지만 이후에 내부적으로 `add()`를 호출하지 않는 방식으로 구현이 변경된다면, `addAll()`은 다시 의도한대로 동작하지 않을 것이고, 해당 클래스를 활용한 곳에도 연쇄적으로 문제가 발생할 것이다.

### 컴포지션 사용하기

컴포지션을 사용하면 이러한 문제가 발생할 가능성을 막을 수 있다.

```kotlin
class CounterSet<T> {
    private val innerSet = HashSet<T>()
    var elementsAdded: Int = 0
        private set
    
    fun add(element: T) {
        elementsAdded++
        innerSet.add(element)
    }

    fun addAll(elements: Collection<T>) {
        elementsAdded += elements.size
        innerSet.addAll(elements)
    }
}

// 사용
val counterList = CounterSet<String>()
counterList.addAll(listOf("A", "B", "C"))
print(counterList.elementsAdded) // 3
```

위 코드의 문제점

- 다형성이 사라진다. (`CounterSet`은 더 이상 Set이 아니다!)

### 위임 패턴 사용하기

{% hint style="info" %}
**위임 패턴**은 클래스가 인터페이스를 상속받게 하고, 포함한 객체의 메서드들을 활용해서, 인터페이스에서 정의한 메서드를 구현하는 패턴이다. 이렇게 구현된 메서드를 **포워딩 메서드**라고 한다.
{% endhint %}

```kotlin
class CounterSet<T> : MutableSet<T> {
    private val innerSet = HashSet<T>()
    var elementsAdded: Int = 0
        private set

    override fun add(element: T): Boolean {
        elementsAdded++
        return innerSet.add(element)
    }

    override fun addAll(elements: Collection<T>): Boolean {
        elementsAdded += elements.size
        return innerSet.addAll(elements)
    }

    override val size: Int
        get() = innerSet.size

    override fun contains(element: T): Boolean = innerSet.contains(element)

    override fun containsAll(elements: Collection<T>): Boolean = innerSet.containsAll(elements)

    override fun isEmpty(): boolean = innerSet.iterator()

    override fun clear() = innerSet.clear()

    override fun remove(element: T): Boolean = innerSet.remove(element)

    override fun removeAll(elements: Collection<T>): Boolean = innerSet.removeAll(elements)

    override fun retainAll(elements: Collection<T>): Boolean = innerSet.retainAll(elements)
}
```

위 코드의 문제점

- 구현해야 하는 포워딩 메서드가 너무 많아진다.

### 위임 패턴 사용하기 (코틀린 버전)

```kotlin
class CounterSet<T>(
    private val innerSet = HashSet<T>() = mutableSetOf()
) : MutableSet<T> by innerSet {
    
    var elementsAdded: Int = 0
        private set

    override fun add(element: T): Boolean {
        elementsAdded++
        return innerSet.add(element)
    }

    override fun addAll(elements: Collection<T>): Boolean {
        elementsAdded += elements.size
        return innerSet.addAll(elements)
    }

}
```

상속된 메서드를 직접 활용하는 것이 위험할 때는 이와 같은 위임 패턴을 사용하는 것이 좋다. 하지만 일반적으로 다형성이 그렇게까지 필요한 경우는 없다. 그래서 단순하게 컴포지션을 활요하면 해결되는 경우가 굉장히 많다.

상속으로 캡슐화를 깰 수 있다는 사실은 보안 문제이다. 하지만 대부분의 경우에 이러한 행위는 규약으로 지정되어 있거나, 서브클래스에 의존할 필요가 없는 경우이다. (일반적으로 메서드가 상속을 위해서 설계된 경우이다.)

## 오버라이딩 제한하기

상속용으로 설계되지 않은 클래스를 상속하지 못하게 하려면, `final`을 사용하면 된다.

만약 어떤 이유로 상속은 허용하지만, 메서드는 오버라이드하지 못하게 만들고 싶은 경우는 `open` 키워드를 사용한다. `open` 클래스는 `open` 메서드만 오버라이드할 수 있다. 상속용으로 설계된 메서드에만 `open`을 붙이면 된다.


```kotlin
open class Parent {
    fun a() {}
    open fun b() {}
}

class Child: Parent() {
    override fun a() {} // 오류
    override fun b() {}
}
```

메서드를 오버라이드할 때, 서브클래스에서 해당 메서드에 `final`을 붙일 수도 있다. 이를 활용하면 서브클래스에서 오버라이드할 수 있는 메서드를 제한할 수 있다.

```kotlin
open class ProfileLoader: InternetLoader() {

    final override fun loadFromInterner() { ... }

}
```

## 정리

컴포지션 vs 상속

- 컴포지션은 더 안전하다. 다른 클래스의 내부적인 구현에 의존하지 않고, 외부에서 관찰되는 동작에만 의존하므로 안전하다.
- 컴포지션은 더 유연하다. 상속은 한 클래스만을 대상으로 할 수 있지만, 컴포지션은 여러 클래스를 대상으로 할 수 있다. 상속은 모든 것을 받지만, 컴포지션은 필요한 것만 받을 수 있다.
- 컴포지션은 더 명시적이다. 상속을 이용하여 슈퍼 클래스의 메서드를 사용할 때는 리시버(`this`)를 지정하지 않아도 된다. 코드가 더 짧아 질 수 있지만 메서드가 어디서 왔는지 혼동될 수 있기 때문에 위험할 수 있다.
- 컴포지션은 생각보다 번거롭다. 대상 클래스에 일부 기능을 추가할 때 이를 포함하는 객체의 코드를 변경해야 한다.
- 상속은 다형성을 활용할 수 있다. 상속을 사용할 경우 슈퍼클래스와 서브클래스의 규약을 항상 잘 지켜서 코드를 작성해야 한다.

일반적으로 OOP에서는 상속보다 컴포지션을 사용하는 것이 좋다.

상속은 명확한 `is-a` 관계일 때 상속을 사용하는 것이 좋다. 슈퍼클래스를 상속하는 모든 서브클래스는 슈퍼클래스로도 동작할 수 있어야 한다. 슈퍼클래스의 모든 단위 테스트는 서브클래스로도 통과할 수 있어야 한다. (리스코프 치환 원칙)

또한, 상속을 위해 설계되지 않은 메서드는 `final`로 만들어 두는 것이 좋다.