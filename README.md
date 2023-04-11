![header](https://capsule-render.vercel.app/api?type=cylinder&color=auto&customColorList=19&text=MINE&fontAlignY=45&fontSize=40&height=150&animation=twinkling&desc=Discover👀%20|%20Bid💸%20|%20Mine🎁&descAlignY=70)

## About
* Mine은 경매 서비스를 제공하기 위한 프로젝트입니다.
* C2C 거래 형태를 지원합니다.
* 상품을 구매하기 위해서는 입찰을 하고 낙찰되어야 합니다. 수동 입찰, 자동 입찰을 지원합니다.
* 자동 입찰을 설정할 수 있습니다. 지불할 의사가 있는 최대 금액을 입력하면 사용자를 대신하여 증분 입찰가로 응찰합니다.

## Prototype
카카오 오븐을 사용해서 제작한 화면 프로토타입입니다.  
👉 [**More Info**](https://ovenapp.io/view/k2rFY8UlHtr6riDPqkIHbIMFPgvRjml7/)  
![프로토타입](https://user-images.githubusercontent.com/76784643/231027721-1cc1b019-8521-4d68-baa6-7729b2b521ee.png)

## System Design
![Mine Cloud](https://user-images.githubusercontent.com/76784643/231027770-17216e94-5ae1-46ad-b701-33b85199c160.png)

## Tech Stack
Java, Spring Boot, JPA, MariaDB, Redis  
AWS: ECS, Fargate, EventBridge, Lambda, SES, S3, RDS

## Functional Requirement
* 회원가입
* 로그인
* 경매 생성
* 경매 카탈로그 조회
* Exception Handling
* 수동 입찰
* 자동 입찰 등록 / 자동 입찰
* 입찰 증분 계산
* 실시간 최고 입찰가
* 경매 마감 일괄 처리

## Non-Functional Requirement
* 시스템에 의해 관리될 수 있는 자동 입찰은 경매 당 하나다.
* 자동 입찰의 낙찰 우선순위를 보장한다.
* 관리 중인 자동 입찰이 존재하는 상태에서 자동 입찰 등록이 요청될 경우, 기존 자동 입찰 한도와 신규 자동 입찰 한도를 비교해 관리할 자동 입찰을 선택한다.
* 기존 자동 입찰의 한도가 높아서 신규 자동 입찰을 등록할 수 없는 경우, 등록은 하지 않지만 한도로 입찰한다. 한도로 입찰함으로써 최종 낙찰 금액을 높이고 경매 생성자와 기업의 이익을 최대화한다.
* 최고 입찰가에 증분을 더한 금액 이상으로 다음 입찰을 수행한다. 1원 단위 입찰이 가능할 경우, 과도한 트래픽과 장난 입찰 발생 가능성이 증가한다.
* 페이지 새로고침 없이 실시간으로 갱신되는 최고 입찰가를 통해 편의성을 제공하고 역동성을 부여해 경매의 묘미를 제공한다.
* 경매는 자동으로 마감 처리되도록 한다. 경매가 마감되면 낙찰자에게 이메일 알림을 보낸다.

## 입찰(수동 입찰/자동 입찰 등록/자동 입찰)
![Facade](https://user-images.githubusercontent.com/76784643/231029003-89df1f48-0b5b-4ebe-96a3-3c8be3d349da.png)

* ‘수동 입찰’, ‘자동 입찰 등록’은 사용자가 요청한다.
* ‘자동 입찰’은 수동 입찰 요청, 자동 입찰 등록 요청에 의해 함께 수행된다.
* 자동 입찰 등록 시 지불할 의사가 있는 최대 금액을 입력하면 사용자를 대신해 증분 입찰가로 응찰한다.
* 자동 입찰 등록 과정에서 신규 자동 입찰, 기존 자동 입찰에 대한 입찰이 수행될 수 있으며 입찰 금액을 기반으로 재산정한 증분 입찰가와 한도를 비교해 등록 여부를 결정한다.

## 입찰 Swimlane Diagram
### 입찰 가능한 시간/금액 확인
![validation](https://user-images.githubusercontent.com/76784643/231029303-71f0af80-ab7f-4330-8d72-b2e7d15f17e8.png)

### 자동 입찰 등록 요청
* ‘증분 입찰가’란 입찰 금액에 일정 증분을 더한 금액이다. (증분 입찰가 = 입찰 가능한 최소 금액)  

![AutoBidRegister](https://user-images.githubusercontent.com/76784643/231029474-b10d9d0a-f7d6-4e1c-bbed-d5b32d8128dc.png)

### 수동 입찰 요청
![ManualBid](https://user-images.githubusercontent.com/76784643/231029694-c7793531-70de-4870-8773-d410eed16e5e.png)

## 입찰 Class Diagram
![BidClassDiagram](https://user-images.githubusercontent.com/76784643/231029783-d606b340-ec9a-4df0-b640-3009b8d574df.png)
* ManualBidServiceImpl(수동 입찰), AutoBidRegisterServiceImpl(자동 입찰 등록), AutoBidServiceImpl(자동 입찰) 클래스는 입찰 관련 비즈니스 로직을 포함한다.
* AbstractBid 추상 클래스는 3가지 입찰 관련 클래스의 공통 멤버를 관리해 중복을 제거한다.
* AbstractBidRequest 추상 클래스는 사용자가 요청하는 수동 입찰, 자동 입찰 등록을 처리하는 2가지 클래스의 공통 멤버를 관리해 중복을 제거한다.
* 3가지 입찰 관련 클래스에서 각 클래스 고유 멤버는 개별 인터페이스에 정의해 관리한다.

![Template](https://user-images.githubusercontent.com/76784643/231029843-eaf2e17a-6946-463a-93fd-e8f8a3fb501d.png)
* 입찰은 부가 기능을 제외하면 총 3가지 단계로 구성되며, 이를 템플릿화한 입찰 메서드를 사용한다.
* 템플릿 메서드 내에서 호출하는 메서드는 필요에 따라 Overriding한다.

## 실시간 최고 입찰가
![websocket](https://user-images.githubusercontent.com/76784643/231030023-e8afe574-8bfe-40ee-b929-b8adcbbf6977.png)
* 경매 카탈로그 조회 시 WebSocket 연결 설정, 경매 Topic 구독을 수행한다.
* 입찰 요청 시 최고 입찰가 정보를 포함한 메시지를 Redis Pub/Sub ‘highest_bid’ Topic에 게시한다.
* ‘highest_bid’ Topic을 구독하는 모든 서버에 메시지를 전달한다.
* 메시지를 전달 받은 각 서버는 In-memory Message Broker를 통해 WebSocket 연결이 설정된 클라이언트 측으로 메시지를 전달한다.

## 입찰 증분 계산
![increment](https://user-images.githubusercontent.com/76784643/231030197-5895345c-7ddb-4c60-9a5c-29868dccc5c7.png)
* 입찰이 수행될 때마다 입찰 금액을 기반으로 증분 입찰가(입찰 가능한 최소 금액)를 재산정한다.
* 입찰 금액이 800만 원일 경우 증분 입찰가는 815만 원이다.

<img width="396" alt="result" src="https://user-images.githubusercontent.com/76784643/231030391-de6820f7-10ff-44b7-9579-a4a1cfc9b477.png">  

![incrementCalc](https://user-images.githubusercontent.com/76784643/231030471-90a05b9b-0817-42d2-846d-d01706753530.png)  

* JMH(Java Microbenchmark Harness)를 이용한 메서드 성능 측정 결과에 기반해 입찰 증분을 구하기 위한 5가지 방법 중 If문을 사용한다.  
* 백분율 계산은 기본적으로 O(1) 상수 시간에 계산되며 입찰 금액이 속하는 범위를 찾을 필요가 없기 때문에 입찰 증분을 가장 빠르게 구할 수 있다. 그러나 입찰 금액이 높아질수록 필요 이상으로 입찰 증분이 높아져 적정 증분을 구할 수 없다고 판단해 사용하지 않는다.
