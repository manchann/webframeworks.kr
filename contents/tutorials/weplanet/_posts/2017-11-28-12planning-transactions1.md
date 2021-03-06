---
layout : tutorials
category : tutorials
title : VueJS 가이드 11 - Transaction 설계하기
subcategory : setlayout
summary : VueJS를 통한 애플리케이션 개발에 대해 알아봅니다.
permalink : /tutorials/weplanet/12planning-transactions1
author : danielcho
tags : vue javascript
title\_background\_color : F1F71A
---



> 본 포스팅은 [Matthias Hager](https://matthiashager.com) 의 [Vue.js Application Tutorial - Step 12: Planning for Transactions](https://matthiashager.com/complete-vuejs-application-tutorial/planning-transactions)를 저자의 허락하에 번역한 글입니다. 오탈자, 오역 등이 있다면 연락부탁드립니다.



여기까지 오는데 꽤나 많은 시간이 걸렸지만 마침내 마지막 모듈인 트랜잭션을 진행할 준비가 되었다. 일단 예산 및 계정은 끝났다. 이 둘은 서로 독립적이였지만 트랜잭션은 모든 것을 하나로 묶을 모듈이다. 한 계정에서 각 트랜잭션을 기록하고, 특정 예산 카테고리에 할당하며, 사용자가 대부분의 시간을 보낼 곳이다.각 로그가 기록될 때마다 애플리케이션은 모든 관련 정보를 최신 상태로 유지해야한다.

 

소득은 계정의 잔액을 증가시킨다. 송금은 한 계좌의 잔액을 감소시키는 동시에 다른 계좌의 잔액을 증가시킨다. 지불은 계정의 잔액을 줄이고 예산 카테고리의 지출액을 늘리면서 결과적으로는 해당 월의 예산 지출을 증가시킨다.

 

이 모듈을 계획하면서 이 업데이트를 만드는 건 누구의 책임이여야 할지 잘 생각해봐야 한다. 트랜잭션 모듈이 예산 카테고리에서지출액을 직접 변경해야할까 아니면 예산 모듈이 트랜잭션 모듈과 인터페이스 할 일종의 API 또는 이벤트 시스템을 제공해야할까? 어떻게 해야 우리의 모든 값을 지속적으로 업데이트하여 동기화를시키고 사용자에게 혼란과 에러를 주지 않을까?

 

##사용자 스토리

사용자는 트랜잭션을 입력해야한다. 그들이 대부분의 시간을 여기서 보낼 것이기 때문에 인터페이스는 아주 매끄럽고 간편해 보여야 한다. 필자의 경험상 개인 금융 트랜잭션을 추적하는 것은 시간을 추적하는 것과 거의 같다. 마찰이 조금이라도 있으면 쉽게 사용을 중단한다. 약간의 성가심은 지속적인 불편함이 된다. 예를 들면 긴 드롭다운 목록에서 카테고리를 선택하기, 긴 비즈니스 이름을 반복해서쓰기, 저장 버튼을 클릭하려고 계속 키보드에서 마우스로 바꾸기. 사용자는 아마도 한 번에 몇 개의 트랜잭션을 입력 할 것이다. 무언가를 살 때마다 애플리케이션을 키는 게 아니다. 그들은 일주일에 한 두 번 들어가서 모든 걸 업데이트 할 것이다.

 

이것은 인터페이스가 어떻게 작동해야 할지에 대한 좋은 그림을 그려준다. 이제 사용자 스토리 몇 가지를 만들 수 있다.

사용자는 다음을 할 수 있다:

- 계정, 날짜, 예산 카테고리, 비즈니스 이름, 노트 및 금액이 포함된 트랜잭션 추가
- 계속 음수 부호를 입력하지 않고도 비용 추가
- 엔터를 눌러서 트랜잭션을 저장하고 즉시 다음 트랜잭션 추가로 이동
- 이전에 입력한 비즈니스 이름 목록보기 및 선택
- 잔액을 쉽게 송금하기 위해 비즈니스 이름에서 계정 선택
- 기존 비즈니스를 선택하면 마지막으로 사용한 예산 카테고리 자동으로 선택
- 거래를 입력 하면 계정 잔액과 예산값이 변경되는 것 보기



여기서 몇 아이템들은 최소 실행 가능 범위에 맞지 않는다고 생각할 수도 있다. 필자도 그 생각을 하고 있었다. 그러나 사용 편의성은 우리의 핵심 기능 중 하나이기 때문에 이 목록으로 계속 진행하려고 한다. (나중에 후회하는 일이 없기를!)

 

##작업 목록

위에 사용자 스토리 목록을 몇 가지 작업으로 나눌 수 있다.

1. 모든 트랜잭션을 보여주는 트랜잭션 목록 페이지를 추가하기. (나중에 사용자가 필터링하고 제한하게 할 수 있다.)


2. 나중에 쉽게 편집할 수 있게 목록의 각 트랜잭션에 컴포넌트를 사용하기. (우린 과거의 결정으로부터 배우고 있다!)


3. 인라인 트랜잭션-추가 컴포넌트 만들기.


4. 자동완성 드롭다운에서 쓸 비즈니스 이름들 데이터베이스에 저장하기.


5. 각 비즈니스 이름에서 사용된 가장 최근의 예산 카테고리를 저장하고, 있으면 자동으로 선택하기.


6. 사용자가 트랜잭션을 추가하거나 편집할 때 그 달의 예산 카테고리 업데이트하기.


7. 사용자가 트랜잭션을 추가하거나 편집할 때 계정 잔액을 업데이트하기.


8. 예산하고 계정에서 쓸 분별있는 API를 만들어 트랜잭션과 너무 밀접하게 연결되지 않도록 하기.


9. 사용자가 계정을 비즈니스로 선택하면 그 계정에 소득 트랜잭션 추가하기.


10. -$10.53 대신10.53을 입력해도 된다는 걸 사용자에게 알리기



나쁘지 않다.적어도 표면에서는. 데이터베이스 구조가 어떻게 생겼을지 간단히 살펴보자.이쯤 되면 이건 익숙해야 한다.

 

```
'transactions': {
  'idj384jidi': {
    'id': 'idj384jidi',
    'date': '2017-04-16T22:48:39.330Z',
    'amount': -10.53,
    'note': 'tomato and pepper seedlings',
    'business': 'djinbck8',
    'budget': 'de7ednve',
    'category': 'jcijeojde88',
    'account': 'eorcndl2m'
  }
}

'businesses': {
  'djinbck8': {
    'id': 'djinbck8',
    'name': 'Garden Surplus',
    'lastCategory': 'jcijeojde88'
  }
}
```



우리는 벌서 첫 질문에 도달했다. 트랜잭션이 `budgetCategory`를 참조해야 할까 아니면 원래 `category` 객체만 참조해야할까? 나중에 사용자가 시간 경과에 따라 각 카테고리에서 얼마를 썼는지에 대한 보고서를 볼 수 있게 해주고 싶을 수도 있다. 예산 객체가 직접 참조되고 트랜잭션을 넣어야 예산 카테고리 잔액이 업데이트 될 것이다. 이걸 다 고려하면 category가 논리적인 선택이다.

 

트랜잭션 모듈 계획하기는 이게 다이다. 다음에는 우리한테 익숙한 상용구 코드를 써내려가고 트랜잭션의 핵심을 다룰 것이다.

 

