---
description: 이펙티브 코틀린 정리하기
---

# 문서를 규약을 정의하라

함수가 무엇을 하는지 명확하게 설명하고 싶다면, KDoc 주석을 붙여 주는 것이 좋다.

일반적으로 대부분의 함수와 클래스는 이름만으로 예측할 수 없는 세부 사항들을 갖고 있다.

일반적인 문제는 행위가 문서화되지 않고, 요소의 이름이 명확하지 않다면 이를 사용하는 사용자는 우리가 만들려고 했던 추상화 목표가 아닌, 현재 구현에만 의존하게 된다는 것이다. 이러한 문제는 예상되는 행위를 문서로 설명함으로써 해결한다.

## 규약

어떤 행위를 설명하면 사용자는 이를 일종의 약속으로 취급하며, 이를 기반으로 스스로 자유롭게 생각하던 예측을 조정한다. 이처럼 예측되는 행위를 **요소의 규약**이라고 부른다. 규약의 당사자들은 서로 상대방이 규약을 안정적으로 계속 지킬 거라 믿는다.

규약이 적절하게 정의되어 있다면, 클래스를 만든 사람은 클래스가 어떻게 사용될지 걱정하지 않아도 된다. 따라서 규약만 지킨다면 원하는 부분을 마음대로 수정할 수 있다.

클래스를 사용하는 사람은 클래스가 내부적으로 어떻게 구현되어 있는지를 걱정하지 않아도 된다. 클래스의 구현을 믿을 수도 있으므로, 이를 의존해서 다른 무언가를 만들 수도 있다. 클래스를 만드는 사람과 사용하는 사람 모두 미리 정의된 규약에 따라 독립적으로 작업할 수 있다.

만약 규약을 설정하지 않는다면, 클래스를 사용하는 사람은 스스로 할 수 있는 것과 할 수 없는 것을 모르기 때문에 구현의 세부적인 정보에 의존하게 된다. 클래스를 만든 사람은 사용자가 대체 무엇을 할지 알 수가 없으므로 사용자의 구현을 망칠 위험이 있다. 따라서 규약을 설정하는 것은 중요하다.

## 규약 정의하기

### 규약을 정의하는 방법

- 이름 : 일반적인 개념과 관련된 메서드는 이름만으로 동작을 예측할 수 있다.
- 주석과 문서 : 필요한 모든 규약을 적을 수 있다.
- 타입 : 어떤 함수의 선언에 있는, 리턴 타입과 아규먼트 타입은 굉장히 큰 의미가 있다. 자주 사용되는 타입의 경우에는 타입만 보아도 어떻게 사용하는지 알 수 있지만, 일부 타입은 문서에 추가로 설명해야 할 의무가 있다.

## 주석을 써야 할까?

코드만 읽어도 어느 정도 알 수 있는 코드를 만들어야 한다는 데 절대적으로 동의한다. 하지만 주석을 함께 사용하면 요소(함수 또는 클래스)에 더 많은 내용의 규약을 설명할 수 있다.

물론 대부분의 기능은 이름 등으로도 무엇을 하는지 확실하게 알 수 있으므로, 주석을 활용한 추가적인 설명이 필요 없다. 이런 경우에는 주석을 다는 것은 코드를 산만하게 만드는 노이즈이다. 함수 이름과 파라미터만으로 정확하게 표현되는 요소에는 따로 주석을 넣지 않는 것이 좋다.

{% hint style="success" %}
주석을 다는 것보다 함수로서 추출하는 것이 훨씬 좋다.
{% endhint %}

함수로 추출하면, 주석이 없어도 이해하기 쉬운 코드를 만들 수 있다.

하지만 주석은 굉장히 유용하고 중용하다. 규약을 잘 정리해 주므로, 사용자에게 자유를 준다.

## KDoc 형식

주석으로 함수를 문서화할 때 사용되는 공식적인 형식을 **KDoc**이라고 부른다.

모든 KDoc 주석은 `/**`로 시작해서 `*/`로 끝난다. 또한 이 사이의 모든 줄은 일반적으로 `*`로 시작한다. 설명은 KDoc 마크다운이라는 형식으로 작성한다.

### KDoc 주석의 구조

- 첫 번째 부분 : 요소에 대한 요약 설명
- 두 번째 부분 : 상세 설명
- 이어지는 줄 : 모두 태그로 시작한다. 이러한 태그는 추가적인 설명을 위해 사용된다.

{% hint style="success" %}
모든 것을 설명할 필요는 없다. 짧으면서 명확하지 않은 부분을 자세하게 설명하는 문서가 좋은 문서이다.
{% endhint %}

## 타입 시스템과 예측

타입 계층(type hierarchy)은 객체와 관련된 중요한 정보이다.

인터페이스는 우리가 구현해야 한다고 약속한 메서드 목록 이상의 의미를 갖는다. 클래스가 어떤 동작을 할 것이라 예측되면, 그 서브클래스도 이를 보장해야 한다. 이를 **리스코프 치환 원칙**이라 부른다. 기본적으로 이는 'S가 T의 서브타입이라면, 별도의 변경이 없어도 T 타입 객체를 S 타입 객체로 대체할 수 있어야 한다'라고 이야기 한다. 

{% hint style="warning" %}
클래스가 어떻게 동작할 거라는 예측 자체에 문제가 있으면, 이 클래스와 관련된 다양한 상속 문제가 발생할 수 있다.
{% endhint %}

{% hint style="success" %}
사용자가 클래스의 동작을 확실하게 예측할 수 있게 하려면, 공개 함수에 대한 규약을 잘 지정해야 한다. 
{% endhint %}

표준 라이브러리와 인기 있는 라이브러리에 있는 대부분의 클래스는 그 서브클래스(와 요소)에 대한 자세한 설명과 규약을 갖고 있다. 이를 기반으로 사용자는 해당 클래스에 대한 예측을 쉽게 할 수 있다. 이러한 설명과 규약은 인터페이스를 유용하게 만든다. 규약이 지켜지는 범위에서는 이를 구현하는 클래스를 자유롭게 만들어도 된다.

## 조금씩 달라지는 세부 사항

구현의 세부사항은 조금씩 다르다. 어떤 식으로 작동해도 괜찮지만, 좋은 방식들을 기억하고 이를 적용해서 사용하는 것이 좋다.

{% hint style="success" %}
구현의 세부 사항은 항상 달라질 수 있지만, 최대한 많이 보호하는 것이 좋다. 
{% endhint %}

일반적으로 캡슐화를 통해서 이를 보호한다. 캡슐화는 '허용하는 범위'를 지정하는 데 도움을 주는 도구이다. 캡슐화가 많이 적용될수록, 사용자가 구현에 신경을 많이 쓸 필요가 없어지므로, 더 많은 자유를 갖게 된다.

## 정리

요소, 특히 외부 API를 구현할 때는 규약을 잘 정의해야 한다. 이러한 규약은 이름, 문서, 주석, 타입을 통해 구현할 수 있다. 규약은 사용자가 객체를 사용하는 방법을 쉽게 이해하는 등 **요소를 쉽게 예측할 수 있게 해준다.**

규약은 요소가 현재 어떻게 동작하고, 앞으로 어떻게 동작할지를 사용자에게 전달해 준다. 이를 기반으로 **사용자는 요소를 확실하게 사용할 수 있고, 규약에 없는 부분을 변경할 수 있는 자유를 얻는다.**