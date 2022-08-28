---
description: 이펙티브 코틀린 정리하기
---

# 변수 타입이 명확하지 않은 경우 확실하게 지정하라

코틀린은 개발자가 타입을 지정하지 않아도, 타입을 지정해서 넣어 주는 굉장히 수준 높은 타입 추론 시스템을 가지고 있다. 이는 개발 시간을 줄여 줄 뿐만 아니라 유형이 명확할 때 코드가 짧아지므로 코드의 가독성이 크게 향상된다. 

하지만 유형이 명확하지 않을 때는 남용하면 좋지 않다. **가독성을 위해 코드를 설계할 때 읽는 사람에게 중요한 정보를 숨겨서는 안 된다. 가독성 향상 이외에 안전을 위해서도 타입을 지정하는 것이 좋다.**

타입을 무조건 지정하라는 것이 아니다. 상황에 맞게 사용하자.