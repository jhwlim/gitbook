---
description: 이펙티브 코틀린 정리하기
---

# 요소의 가시성을 최소화하라

## 간결한 API를 선호하는 이유

- 작은 인터페이스는 배우기 쉽고 유지하기 쉽다. 보이는 요소 자체가 적다면, 유지보수하고 테스트할 것이 적다.
- 변경을 가할 때는 기존의 것을 숨기는 것보다 새로운 것을 노출하는 것이 쉽다. 일반적으로 공개적으로 노출되어 있는 요소들은 공개 API의 일부이며, 외부에서 사용할 수 있다. 이런 요소들을 변경하면, 이 코드를 사용하는 모든 부분이 영향을 받는다. 따라서 작은 API로서 개발을 하도록 강제하는 것이 더 좋을 수 있다.
- 클래스의 상태를 나타내는 프로퍼티를 외부에서 변경할 수 있다면, 클래스는 자신의 상태를 보장할 수 없다. 
    - 세터만 private으로 만드는 코드는 굉장히 많이 사용되므로 기억하자.
    ```kotlin
    class CounterSet<T>(
        private val innerSet: MutableSet<T> = setOf()
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
    - 일반적으로 코틀린에서는 구체 접근자의 가시성을 제한해서 모든 프로퍼티를 캡슐화하는 것이 좋다.
- 서로서로 의존하는 프로퍼티가 있을 때는 객체 상태를 보호하는 것이 더 중요해진다.
    ```kotlin
    class MutableLazyHolder<T>(
        val initializer: () -> T
    ) {

        private var value: Any = Any()
        private var initialized = false

        override fun get(): T {
            if (!initialized) {
                value = initializer()
                initialized = true
            }
            return value as T
        }

        override fun setValue() {
            this.value = value
            initialized = true
        }

    }
    ```

{% hint style="success" %}
가시성이 제한될수록 클래스의 변경을 쉽게 추적할 수 있으며, 프로퍼티의 상태를 더 쉽게 이해할 수 있다. 
{% endhint %}

이는 동시성을 처리할 때 중요하다. 상태 변경은 병렬 프로그래밍에서 문제가 된다. 따라서 **많은 것을 제한할수록 병렬 프로그래밍을 할 때 안전해진다.**

## 가시성 한정자 사용하기

내부적인 변경 없이 작은 인터페이스를 유지하고 싶다면, 가시성을 제한하면 된다.

기본적으로 클래스와 요소를 외부에 노출할 필요가 없다면, 가시성을 제한해서 외부에서 접근할 수 없게 만드는 것이 좋다.

### 클래스 멤버의 가시성 한정자

- public : 어디에서나 볼 수 있다. (디폴트)
- private : 클래스 내부에서만 볼 수 있다.
- protected : 클래스와 서브클래스 내부에서만 볼 수 있다.
- internal : 모듈 내부에서만 볼 수 있다.

### 톱레벨 요소의 가시성 한정자

- public : 어디에서나 볼 수 있다. (디폴트)
- private : 같은 파일 내부에서만 볼 수 있다.
- internal : 모듈 내부에서만 볼 수 있다.

모듈이 다른 모듈에 의해서 사용될 가능성이 있다면, internal을 사용해서 공개하고 싶지 않은 요소를 숨긴다.

{% hint style="info" %}
코틀린에서 **모듈**이란 함께 컴파일되는 코틀린 소스를 의미한다.

- Gradle 소스 세트
- Maven 프로젝트
- IntelliJ IDEA 모듈
- Ant 태스크 한 번으로 컴파일되는 파일 세트
{% endhint %}

요소가 상속을 위해 설계되어 있고, 클래스와 서브클래스에서만 사용되게 만들고 싶다면 protected를 사용한다.

동일한 파일 또는 클래스에서만 요소를 사용하게 만들고 싶다면 private을 사용한다.

{% hint style="success" %}
이러한 규칙은 데이터를 저장하도록 설계된 클래스(데이터 모델 클래스, DTO)에는 적용하지 않는 것이 좋다.
{% endhint %}

데이터를 저장하도록 설계된 클래스는 숨길 이유가 없기 때문이다. 따라서 프로퍼티를 사용할 수 있게 눈에 띄게 만드는 것이 좋으며, 필요하지 않은 경우 그냥 프로퍼티를 제거하는 것이 좋다.

```kotlin
class User(
    val name: String,
    val surname: String,
    val age: Int,
)
```

**한 가지 큰 제한은 API를 상속할 때 오버라이드해서 가시성을 제한할 수는 없다.** 이는 서브클래스가 슈퍼클래스로도 사용될 수 있기 때문이다. 이것이 상속보다 컴포지션을  선호하는 대표적인 이유이다.

## 정리

요소의 가시성을 최대한 제한적인 것이 좋다. 보이는 요소들은 모두 public API로서 사용되며, 아래와 같은 이유로 최대한 단순한 것이 좋다.

- 인터페이스가 작을수록 이를 공부하고 유지하는 것이 쉽다.
- 최대한 제한이 되어 있어야 변경하기 쉽다.
- 클래스의 상태를 나타내는 프로퍼티가 노출되어 있다면, 클래스가 자신의 상태를 책임질 수 없다.
- 가시성이 제한되면 API의 변경을 쉽게 추적할 수 있다.