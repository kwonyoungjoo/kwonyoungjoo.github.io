---
layout: post
title:  Reddit Dynamic Product Ads 리뷰  
---

# Reddit Dynamic Product Ads 리뷰

레딧의 광고 시스템을 알아보고 우리가 적용 할만한 것들이 있는지 살펴보자.

** 레딧의 광고 목표 : right product, right person, right time on Reddit**

참고 링크 : https://www.reddit.com/r/RedditEng/comments/1gug4x9/product_candidate_generation_for_reddit_dynamic/?rdt=41426 

광고 타케팅 예시 - 곱슬머리에 대한 정보를 찾는 검색이 있으면 헤어 제품을 광고로 내보낸다.

# **Funnel Overview for DPA(Dynamic Product Ads)**

<img src="/assets/redditads/ads1.png" width="60%" height="90%" style="display: block; margin: 0 auto;" />

Targeting 

- 광고를 볼 대상자 선정
- various criteria, such as demographics, device or location.

Product Candidate Generation 

- 광고를 할 고객이 타게팅 되면 광고 제품 후보를 생성 한다
- 유저 과거 history 컨텐츠 선호도 제품 범주 등에 따라 생성

Product Select 

- 후보 랭킹
- 다양한 연관성 성과 지표에 따라 랭킹  후 TOP@N이 사용자에게 노출

Auction

- 입찰, 광고 관련성으로 제품을 선정

# **Why and What is Candidate Generation in DPA?**

정적광고와 비교 했을때 많은 제품 중에서 유저의 context와 맥락을 같이 하는 광고를 낮은 응답시간에 동적으로 생성 할수 있다. 사용자의 게시물, 맥락정보를 상품을 추천에 활용하여 가장 관련도가 높은 상품을 추천 한다. 

point → multiple candidate selectors

**Candidate Generation Approaches**

광고 후보 생성시 두가지 영역으로 구분해서 접근하고 있다.

**Modeling**

- Rule-Based Selection: 인기있는 상품, 트랜디 상품등의 점수를 바탕으로 scoring
- Contextual-Based Selection : 서브래딧내 게시물과 상품 설명의 키워드 매칭 반영
- Behavioral-Based Selection:  사용자 구매를 최대화 하기 위한 user-product interection history

Reddit은 위 방법을 결합해서 사용하고 Rule_Based 모델은 콜드스타트를 개선하고 Behavioral-Based는 개인화를 통해 구매활성도에 기여하고  Contextual-Based Selection는 사용자의 관심사를 반영하는 Selector를 구현할수 있다. 

**Serving**

- **Offline :** 빠른 검색을 위한 user-product를 배치 생성하여 데이터베이스에 저장
- **Online:**  광고 요청과 제품간의 실시간 매칭

offline, online 둘다 다들 아시는 장단점이 있죠~

# **A Closer Look: Online Approximate Nearest Neighbor Search with Behavioral-Based Two-Tower Model**

two tower model을 사용한 ANN 매칭 알고리즘 

광고 요청이 들어오면 user-tower 통해 사용자 임베딩을 생성하고 생성된 user-embedding을  ANN 알고리즘으로 product-embedding 과 매칭 한다.  

# **Model Deep Dive**

two-tower model은 추천시스템에서 흔히 사용하는 DL 아키텍처이다. 

두개의 tower이란 user-tower, product-tower를 의미하고 유저와 제품을 각각의 엔티티로 별도록 학습하고 shared embedding 공간에 매핑한다. 

## **Model Architecture, Features, and Labels**

<img src="/assets/redditada/ads2.png" width="60%" height="90%" style="display: block; margin: 0 auto;" />

### **User and Product Embeddings**:

user-specific 피쳐 : engagement, platform etc 

product-specific 피처:  price, catalog, engagement etc

user, product는 각각 high-dimensional-vector로 나타내며 user와 product가 semantic space로 나타낸다.

### **Training with Conversion Events**:

사용자 과거 전환 이벤트를 통한 학습된다.

negative sampling을 통해서 사용자의 관심이 없는 상품을 벡터가 거리를 더욱 멀어지게 학습한다.

## **Model Training and Deployment**

Ray라는 학습 파이프라인이 있음 

<img src="/assets/redditads/ads3.png" width="60%" height="90%" style="display: block; margin: 0 auto;" />

# **Serving Deep Dive**

### **Online ANN (Approximate Nearest Neighbor) Search**

전통적인 접근 방식과 달리 ANN(Approximate Nearest Neighbor)은 유사한 항목을 클러스터링 하여 빠르고 효율적이고 충분히 연관성 높은 매칭 결과를 제공 한다. 

리서치 결과 FAISS(Facebook AI Similarity Search)를 선정했고 FAISS는 인덱스생성시간, 메모리, 검색시간, recall등을 고려했을때 적합하다. 특히 높은 성능을 요구하는 유사도 검색에 적합하다.

사용자 임베딩을 바탕으로 N개의 nearest product embeddings을 검색하는 ANN sidecar를 구현하여 pod내에 패킹되어 배포하여 높은 효율과 높은 성능을 지원 

### **Product Candidate Retrieval Workflow with Online ANN**

<img src="/assets/redditads/ads4.png" width="60%" height="90%" style="display: block; margin: 0 auto;" />

이 그림은 Reddit의 동적 광고 제품 후보 검색 워크플로우를 보여주는 시스템 아키텍처입니다. 주요 구성요소와 데이터 흐름은 다음과 같습니다

- 광고 요청(Ad Request)이 들어오면 Ad Selector를 통해 사용자 쿼리 임베딩이 생성됩니다.
- Embedding Service는 Redis Cache와 Feature Server를 활용하여 광고 요청에 대한 임베딩과 제품 임베딩을 처리합니다.
- Product Ad Shard 내에서는:
    - Online Behavioral-Based Candidate Source가 쿼리 임베딩을 기반으로 관련 제품을 찾습니다
    - ANN Sidecar는 FAISS 인덱스를 사용하여 쿼리 임베딩으로 최근접 제품 임베딩을 검색합니다
    - Light Ranking이 최종 후보들의 순위를 결정합니다
- Product Metadata Delivery 시스템은 Campaign Metadata와 Catalog Service로부터 실시간 제품 정보를 받아 제품 임베딩과 메타데이터를 생성하고 Kafka를 통해 전달합니다. 신규 광고에 대한 처리를 담당합니다.

**Real-Time User Embedding Generation:**

1. 광고 요청이 **Ad Selector**는 **Embedding Service**에 사용자 임베딩 생성 요청을 보냅니다.
2. Embedding Service는 실시간 contextual feature를 더해 user-embedding을 요청한다.24시간 이내 요청에 대한 캐시기능

**Async Batch Product Embedding Generation:**

1. **Product Metadata Delivery Service**는 **Campaign Metadata Delivery Service**와 **Catalog Service**에서 실시간 캠페인의 모든 제품을 가져옵니다.
2. 예정된 시간에 **Product Metadata Delivery Service**는 배치로 제품 임베딩 생성 요청을 **Embedding Service**에 보냅니다. 
3. **Embedding Service**는 제품 타워 모델에서 평가된 배치 제품 임베딩을 반환합니다.
4. **Product Metadata Delivery Service**는 실시간 제품 메타데이터와 제품 임베딩을 **Kafka**에 게시하여 **Product Ad Shard**에서 소비할 수 있도록 합니다.

**Async ANN Index Building**

제품 인덱스는 **Product Ad Shard** 내의 **ANN Sidecar**에 저장됩니다. **ANN Sidecar**는 **PMD**에서 가져온 모든 실시간 제품 임베딩으로 초기화되며, 30초마다 새로 고쳐져 인덱스 공간을 최신 상태로 유지하기 위해 제품 임베딩을 추가, 수정 또는 삭제합니다.

**Candidate Generation and Light Ranking**

1. Product Ad Shard는 upstream으로 부터 맥락, user-embedding 을 받아서 모든 selector에게 요청을 보낸다. 
2. online behavioral-based selector는 ANN Sidcar에 local request를 보내고 ANN 검색을 통해서 빠르게 매칭 결과를 반환한다. 이때 user-embedding과 product-embedding 버전 확인이 중요(당연 버전에 따라 vector space가 완전 달라짐)
3. 모든 후보는 union 되고 랭킹 단계 통해 최종 후보로 정렬 되고 DPA 광고를 구성하고 auction에 참여 

## Additional Information

Two Tower Model 은 하이브리드 추천 모델, 주로 추천에서 사용되는 모델로 두개의 모델(네트워크)가 다른 유형의 입력을 처리하고 이를 결합하여 유사도를 구하는 방식 

임베딩의 특성상 유사한 유저 혹은 아이템은 비슷한 위치로 클러스터링 된다.

user-tower  : 나이, 성별, 취향을 input 으로 사용자의 특성을 학습 

item-tower : 가격, 리뷰, 특성을 input으로 아이템의 특성을 학습 

두개의 타워를  처리해서 사용자가 제품을 좋아할 확률 == 사용자와 제품의 벡터 공간상의 유사도 

<img src="/assets/redditads/ads5.png" width="60%" height="90%" style="display: block; margin: 0 auto;" />

특징

- 병렬 학습
- 각 타워는 다른 종류의 데이터를 입수
- 두 타워의 유사도 측정이 용이

user-item 의 정답셋을 필요함