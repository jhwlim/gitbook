---
description: 마크다운 문법을 이용하여 GitBook 문서를 작성하는 방법에 대하여 정리한다.
---

# Guide

## Guide

### 제목

```md
# Heading 1
## Heading2
### Heading 3
#### Heading 4
##### Heading 5
###### Heading 6
```

## Heading 1

### Heading2

#### Heading 3

**Heading 4**

**Heading 5**

**Heading 6**

### 텍스트 스타일

```md
**굵게**, __굵게__
*기울게*, _기울게_
<u>밑줄</u>
~~취소선~~
```

**굵게**, **굵게** _기울게_, _기울게_ 밑줄 ~~취소선~~

### 목록

1. Parent 1
   * Child 1
   * Child 2
   * Child 3
2. Parent 2
   * Child 1
   * Child 2
   * Child 3

### 인용

> Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book.

### Hint

{% hint style="info" %}
Info
{% endhint %}

{% hint style="success" %}
Success
{% endhint %}

{% hint style="warning" %}
Warning
{% endhint %}

{% hint style="danger" %}
Danger
{% endhint %}

### 표

```md
|기본 정렬|왼쪽 정렬|가운데 정렬|오른쪽 정렬|
|---|:--|:-:|--:|
|생생하며,|많이 무엇을 과실이 사랑의 뭇 사랑의 아니다.|것은 그들의 피가 거친 있다.|황금시대의 무엇이 곳으로 더운지라 그리하였는가?|
|미인을 끓는 것이다.|얼마나 이상 든 아니한 소리다.|보라, 그리하였는가?|우리는 가치를 아니한 같은 새가 인생에 보라.|
|이것은 인간의 발휘하기 힘있다.|별과 생생하며,|얼음이 천자만홍이 따뜻한 더운지라 풀이 구하지 타오르고 이것이다.|것은 사는가 타오르고 노년에게서 봄바람이다.|
```

| 기본 정렬             | 왼쪽 정렬                     |                가운데 정렬                |                     오른쪽 정렬 |
| ----------------- | ------------------------- | :----------------------------------: | -------------------------: |
| 생생하며,             | 많이 무엇을 과실이 사랑의 뭇 사랑의 아니다. |           것은 그들의 피가 거친 있다.           | 황금시대의 무엇이 곳으로 더운지라 그리하였는가? |
| 미인을 끓는 것이다.       | 얼마나 이상 든 아니한 소리다.         |              보라, 그리하였는가?             |  우리는 가치를 아니한 같은 새가 인생에 보라. |
| 이것은 인간의 발휘하기 힘있다. | 별과 생생하며,                  | 얼음이 천자만홍이 따뜻한 더운지라 풀이 구하지 타오르고 이것이다. |   것은 사는가 타오르고 노년에게서 봄바람이다. |

### 인라인 블록

`인라인 블록` 입니다.

### 코드 블록

````md
```kotlin
fun printHelloWorld() {
    println("Hello World!")
}
```
````

```kotlin
fun printHelloWorld() {
    println("Hello World!")
}
```

### 코드 탭

{% tabs %}
{% tab title="첫번째 탭" %}
첫번째 탭 내용입니다.
{% endtab %}

{% tab title="두번째 탭" %}
두번째 탭 내용입니다.
{% endtab %}
{% endtabs %}
