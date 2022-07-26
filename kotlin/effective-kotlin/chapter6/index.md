---
description: 이펙티브 코틀린 정리하기
---

# 6장. 클래스 설계

이번 장에서는 **클래스를 설계하는 방법**에 대해서 다룬다.

코틀린에서 자주 볼 수 있는 클래스를 사용하는 패턴을 보고, 이를 어떻게 활용하고, 활용할 때 무엇을 기대할 수 있는지 등의 규약에 대해서 알아본다.

- 상속은 언제 어떻게 활용해야 하는지
- 데이터 클래스는 어떤 형태로 사용해야 하는지
- 언제 하나의 메서드를 가진 인터페이스 함수 타입을 사용해야 하는지
- `equals`, `hashCode`, `compareTo`는 어떤 규약을 가지고 있는지
- 멤버와 확장 함수는 어떻게 구분해서 사용해야 하는지

## 목차

* [아이템 36. 상속보다는 컴포지션을 사용하라](./item36.md)
* [아이템 37. 데이터 집합 표현에 data 한정자를 사용하라](./item37.md)
* [아이템 38. 연산 또는 액션을 전달할 때는 인터페이스 대신 함수 타입을 사용하라](./item38.md)
* [아이템 39. 태그 클래스보다는 클래스 계층을 사용하라](./item39.md)
* [아이템 40. equals의 규약을 지켜라](./item40.md)
* [아이템 41. hashCode의 규약을 지켜라](./item41.md)
* [아이템 42. compareTo의 규약을 지켜라](./item42.md)
* [아이템 43. API의 필수적이지 않는 부분을 확장 함수로 추출하라](./item43.md)
* [아이템 44. 멤버 확장 함수의 사용을 피하라](./item44.md)