# Spring Cloud MSA Sample

Spring Cloud 기반으로 구성한 마이크로서비스 학습 프로젝트입니다.  
Eureka Server를 중심으로 서비스 디스커버리를 구성하고, Spring Cloud Gateway를 통해 진입점을 단일화했으며, Auth / Product / Order 서비스를 분리해 인증, 상품 관리, 주문 기능을 각각 독립적으로 구현했습니다.

또한 JWT 기반 인증 흐름, Feign Client를 통한 서비스 간 통신, QueryDSL 기반 조회 구조, Zipkin 트레이싱 설정까지 반영해 MSA의 핵심 요소를 실습할 수 있도록 구성했습니다.

---

## 1. 프로젝트 소개

이 프로젝트는 단일 애플리케이션이 아닌 여러 개의 독립 서비스로 기능을 분리해 구성한 Spring Cloud 기반 MSA 샘플입니다.

각 서비스는 Eureka에 등록되고, Gateway는 등록된 서비스 정보를 기반으로 요청을 라우팅합니다.  
인증은 Auth Service에서 JWT를 발급하는 방식으로 처리했고, Gateway에서는 JWT를 검증한 뒤 사용자 정보를 헤더에 담아 하위 서비스로 전달합니다.

Order Service는 Product Service와 직접 연결하지 않고 Feign Client를 통해 통신하며, 주문 생성 시 상품 수량을 확인하고 차감하는 흐름으로 서비스 간 협업 구조를 구성했습니다.

---

## 2. 주요 기능

### 서비스 디스커버리
- Eureka Server 실행
- 각 마이크로서비스를 Eureka Client로 등록
- Gateway 및 서비스 간 논리적 서비스명 기반 호출

### 인증 기능
- 회원가입
- 로그인
- JWT 발급
- Gateway에서 JWT 검증
- 인증 성공 시 `X-User-Id`, `X-Role` 헤더 전달

### 상품 기능
- 상품 등록
- 상품 단건 조회
- 상품 목록 조회
- 상품 수정
- 상품 삭제
- 상품 수량 차감

### 주문 기능
- 주문 생성
- 주문 단건 조회
- 주문 목록 조회
- 주문 수정
- 주문 삭제
- 주문 생성 시 Product Service에 상품 재고 확인 요청
- 주문 생성 시 상품 수량 차감

### 운영/관찰 기능
- Spring Actuator 적용
- Micrometer + Zipkin 기반 분산 트레이싱 설정

---

## 3. 서비스 구성

### Eureka Server
- 서비스 등록/조회 서버
- 각 서비스의 위치를 관리

### Gateway Service
- 외부 요청의 진입점
- JWT 검증 처리
- 서비스별 라우팅 처리

### Auth Service
- 사용자 회원가입 및 로그인 처리
- JWT 액세스 토큰 발급

### Product Service
- 상품 CRUD
- 상품 검색/조회
- 재고 관리

### Order Service
- 주문 CRUD
- Product Service와 연동한 주문 처리
- 관리자 권한 기반 주문 목록 조회 구조 반영

---

## 4. 기술 스택

### Backend
- Java 17
- Spring Boot 3.3.1
- Spring Cloud 2023.0.2
- Spring Web
- Spring Data JPA
- Spring Security
- Spring Cloud Gateway
- Spring Cloud Netflix Eureka
- Spring Cloud OpenFeign
- QueryDSL
- JWT (jjwt)
- Spring Boot Actuator
- Micrometer Tracing
- Zipkin Reporter
- Lombok

### Database
- HSQLDB

### Dev / Build
- Gradle 8.x

---

## 5. 핵심 구현 포인트

### 5-1. Eureka 기반 서비스 디스커버리
- 각 서비스는 Eureka Server에 자신을 등록합니다.
- Gateway와 서비스 간 통신은 실제 IP/포트가 아니라 서비스 이름(`lb://service-name`)을 기준으로 동작합니다.

### 5-2. Gateway 중심 진입 구조
- 외부 요청은 Gateway를 통해 진입합니다.
- Gateway는 요청 경로에 따라 `auth-service`, `product-service`, `order-service`로 라우팅합니다.
- 서비스 디스커버리 locator를 활성화해 등록된 서비스 기반의 동적 라우팅 구조도 반영했습니다.

### 5-3. JWT 기반 인증 흐름
- Auth Service에서 로그인 성공 시 JWT를 발급합니다.
- Gateway의 글로벌 필터에서 JWT를 검증합니다.
- `/auth/signIn`, `/auth/signUp` 요청은 인증 없이 통과시키고, 나머지 요청은 토큰 검증 후 처리합니다.
- 검증에 성공하면 사용자 정보(`X-User-Id`, `X-Role`)를 헤더에 담아 하위 서비스로 전달합니다.

### 5-4. 서비스 간 통신
- Order Service는 Product Service를 직접 호출하지 않고 Feign Client를 사용해 통신합니다.
- 주문 생성 시:
  1. 상품 존재 여부 및 수량 확인
  2. 재고 차감 요청
  3. 주문 저장
- 이를 통해 서비스 간 역할을 분리하면서도 협업 흐름을 구성했습니다.

### 5-5. QueryDSL 기반 조회 확장
- Product, Order 서비스에서 QueryDSL 기반 조회 구조를 사용했습니다.
- 단순 CRUD를 넘어서 검색 조건과 페이징을 확장할 수 있는 형태로 구성했습니다.

### 5-6. 역할 기반 접근 제어
- Product 등록은 `MANAGER` 역할만 가능하도록 제한했습니다.
- Order 목록 조회도 `MANAGER` 권한이 있을 때만 허용하도록 처리했습니다.

### 5-7. 분산 트레이싱
- Product / Order 서비스에 Micrometer Tracing과 Zipkin 연동 설정을 반영했습니다.
- 서비스 간 호출 흐름을 추적할 수 있도록 구성했습니다.

---

## 6. 서비스 포트 정보

| Service | Port |
|---|---:|
| Eureka Server | `19090` |
| Gateway Service | `19091` |
| Order Service | `19092` |
| Product Service | `19093` |
| Auth Service | `19095` |

---

## 7. 요청 흐름

```text
Client
  ↓
Gateway Service (JWT 검증 / 라우팅)
  ↓
┌───────────────┬───────────────┬───────────────┐
│ Auth Service  │ Product Service │ Order Service │
└───────────────┴───────────────┴───────────────┘

Order Service
  ↓ (Feign Client)
Product Service
```
---

## 8. 프로젝트 구조
com.spring-cloud.sample
├── com.spring-cloud.eureka.server
│   └── src/main/java/com/spring_cloud/eureka/server
│       └── ServerApplication.java
├── com.spring-cloud.eureka.client.gateway
│   └── src/main/java/com/spring_cloud/eureka/client/gateway
│       ├── GatewayApplication.java
│       ├── LocalJwtAuthenticationFilter.java
│       ├── CustomPreFilter.java
│       └── CustomPostFilter.java
├── com.spring-cloud.eureka.client.auth
│   └── src/main/java/com/spring_cloud/eureka/client/auth
│       ├── AuthApplication.java
│       ├── AuthConfig.java
│       ├── AuthController.java
│       ├── AuthService.java
│       ├── UserRepository.java
│       └── core
│           └── User.java
├── com.spring-cloud.eureka.client.product
│   └── src/main/java/com/spring_cloud/eureka/client/product
│       ├── ProductApplication.java
│       ├── ProductApplicationQueryDslConfig.java
│       ├── core
│       │   └── Product.java
│       └── products
│           ├── ProductController.java
│           ├── ProductRepository.java
│           ├── ProductRepositoryCustom.java
│           ├── ProductRepositoryImpl.java
│           ├── ProductRequestDto.java
│           ├── ProductResponseDto.java
│           ├── ProductSearchDto.java
│           └── ProductService.java
└── com.spring-cloud.eureka.client.order
    └── src/main/java/com/spring_cloud/eureka/client/order
        ├── OrderApplication.java
        ├── OrderApplicationQueryDslConfig.java
        ├── core
        │   ├── client
        │   │   ├── ProductClient.java
        │   │   └── ProductResponseDto.java
        │   ├── domain
        │   │   └── Order.java
        │   └── enums
        │       └── OrderStatus.java
        └── orders
            ├── OrderController.java
            ├── OrderRepository.java
            ├── OrderRepositoryCustom.java
            ├── OrderRepositoryImpl.java
            ├── OrderRequestDto.java
            ├── OrderResponseDto.java
            ├── OrderSearchDto.java
            └── OrderService.java
---

## 9. API 명세

### 9-1. Auth API

#### 회원가입
- `POST /auth/signUp`

#### 로그인
- `POST /auth/signIn`

---

### 9-2. Product API

#### 상품 등록
- `POST /products`

#### 상품 목록 조회
- `GET /products`

#### 상품 단건 조회
- `GET /products/{productId}`

#### 상품 수정
- `PUT /products/{productId}`

#### 상품 삭제
- `DELETE /products/{productId}?deletedBy=...`

#### 상품 수량 차감
- `GET /products/{id}/reduceQuantity?quantity=...`

---

### 9-3. Order API

#### 주문 생성
- `POST /orders`

#### 주문 목록 조회
- `GET /orders`

#### 주문 단건 조회
- `GET /orders/{orderId}`

#### 주문 수정
- `PUT /orders/{orderId}`

#### 주문 삭제
- `DELETE /orders/{orderId}?deletedBy=...`

---

## 10. 실행 방법

### 10-1. 프로젝트 클론

```bash
git clone <YOUR_REPOSITORY_URL>
cd com.spring-cloud.sample
