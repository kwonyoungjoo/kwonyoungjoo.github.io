---
layout: post
title: 이벤트스토밍 진행하기 
---
이벤트스토밍은 왜?? : 도메인주도 개발을 진행하기 위해서 현재 도메인에 대한 구성원간의(개발자외 전무 포함) 이해도를 높이고
유비쿼터스 용어를 정리하기 위한 워크샵이라고 볼수 있다. 

용어가 익숙하지 않고 경계가 불분명애서 도메인을 만드는것 조차 어렵다.

이벤트 스토밍 진행

모든 사람의 적극적인 참여 필요

개발수준의 코드를 제거하고 비지니스 로직 중심으로 진행

준비물 : 포스트잇, 마커펜

**1단계**

도메인 이벤트 작성, 자신의 방식으로 기록해서 화이트 보드에 붙임

적극적인 의견 기재, 개발용어를 배제하고 비지니스로직 중심으로 작성

이벤트 정의(주황)

상태 변경되는 모든 행위,  유의미한 모든 행위

ex) 수강생이 등록되었다. 종목이 선택되었다. 금액이 입력되었다. 관심주식이 삭제되었다.

이벤트 표현 방법

발생한 일을 과거형으로 표현

이벤트에 Read 항목을 없앰, 특수하게 Read가 필요한 경우 Read 모델을 작성

**2단계**

모든 도메인 이벤트 타임라인순으로 정렬

가로- 시리얼한 타임라인, 세로 - 동일 타임라인

시작시에 타임라인 정렬이 어려워서 스토리보드 기준으로 먼저 정리 하고 진행해도 나쁘지 않다.

중복제거는 신중하게 확신이 들때 제거한다.

중복제거 하면서 겹치는 용어를 유비쿼터스 용어로 정리 한다.

애그리거트 단위로 같은 용어이더라도 그 의미가 달라질수 있다.

2단계에서 발생한 갈등 표기(보라색)

**3단계**

엑터 색상(노랑)

구체적으로 표기, 사용자, 관리자, 개발자

이벤트를 트리커 하는 대상자

커멘드(파랑)

현재형으로 표기

엑터가 실행하는 명령

호출되면 이벤트 생성, 커멘드가 발생 되있다고 해서 이벤트가 발생 하는것은 아님

외부 시스템(분홍)

디비, 원장, 외부 데이터 제공자

애그리게이트(노랑)

내부 트렌잭션 실행 단위 , 일관성유지 단위 (비지니스 규칙)

도메인 객체들의 집합

**4단계**

바운디드 컨텍스트 분리

리드모델, 정보(초록)

액터가 커맨드 하는데 도움이 되는 정보

정책(연보라)

~할떄 마다 , 도메인이벤트와 커멘드 사이에 위치

이벤트에 연쇄적으로 발생 하는 이벤트

리뷰

처음 할때는 많이 헤메었는데 이것도 하다 보니 점점 기술이 늘어가는것 같다.

참고자료

- [Alberto Brandolini - Introducing Event Storming](https://ziobrando.blogspot.com/2013/11/introducing-event-storming.html)
- [Event Storming - Alberto Brandolini - DDD Europe 2019](https://www.youtube.com/watch?v=mLXQIYEwK24)
- [KCD 2020 [Track 2] 도메인 지식 탐구를 위한 이벤트 스토밍 Event Storming (추천, 17분 가량의 강의 내용)](https://www.youtube.com/watch?v=hUcpv5fdCIk)
- [SK C&C 블로그 - 마이크로서비스 모델링 ④ : 이벤트 스토밍을 통한 마이크로서비스 도출](https://engineering-skcc.github.io/microservice%20modeling/Event-Storming/)
- [Mathias Verraes - Domain Events](https://verraes.net/2014/11/domain-events/)
- [Gitbook: Microservice Architecture - Domain Event Pattern](https://badia-kharroubi.gitbooks.io/microservices-architecture/content/patterns/tactical-patterns/domain-event-pattern.html)
- [Event Storming, Storytelling, Visualisations](https://verraes.net/2015/03/event-storming-storytelling-visualisations/)
- [Mathias Verraes - Facilitating Event Storming](https://verraes.net/2013/08/facilitating-event-storming/)
- [Nick Tune - EventStorming Modelling Tips to Facilitate Microservice Design](https://medium.com/nick-tune-tech-strategy-blog/eventstorming-modelling-tips-to-facilitate-microservice-design-1b1b0b838efc)
- [Barryosull - Event Granularity: Modelling events in event driven applications](https://dev.to/barryosull/event-granularity-modelling-events-in-event-driven-applications-e50)