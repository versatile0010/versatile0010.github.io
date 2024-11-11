---
title: " 🔐 Enhancing API Security with RSA Encryption in Spring/Java and Redis "
layout: single
classes: wide
categories:
  - Security
---

### 서비스를 개발할 때, 성능적인 고민도 분명 필요하지만 API 통신에 이용되는 고객의 민감정보를 어떻게 하면 더 잘 지켜낼 수 있을지도 매우 중요한 고민 포인트이다.

가령, 로그인 기능을 구현했다고 가정해보자.

`Client` 에서는 유저의 `id` 와 `password` 를 받은 뒤, `payload` 에 담아 로그인 API 를 호출할 것이다.

보통 아래와 같이 비밀번호와 같은 민감정보들은 `masking` 처리되어 보여지겠지만

<img width="364" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/0f058b69-22e5-45d3-aacb-aa0210cc1431">

별도의 보안적인 처리를 하지 않으면 개발자도구의 `payload` 에서 `rawdata` 가 전부 노출된다.

반대로 서버에서 클라이언트로 정보를 내려줄 때, 보안처리를 신경쓰지 않으면

`response` 탭에서 노출되면 안될 정보가 그대로 노출될 수 있다.

이러한 상황을 인지하게 된다면 고객은 우리 서비스를 믿고 사용하지 못할 수 있고,

계속 방치된다면 가능성이 낮더라도 심각한 문제가 발생할 수도 있다.

---

### 어떻게 해결할 수 있을까?

사실 해결법은 쉽게 떠올릴 수 있다.

민감한 정보는 제 3자가 알 수 없도록 암호화 처리하면 된다.

러프하게 요구사항을 정리해보자.

- 클라이언트에서 민감 정보를 서버로 보낼 때에는 암호화하고, 서버에서 복호화한 뒤 로직을 수행하자.

위 요구사항을 만족할 수 있다면 기존에 제기한 문제는 해결될 것이다.

문제가 정의되었으니, 어떤 도구로 이를 해결할 지 고민해보자.

---

### 어떤 암호화 알고리즘을 사용할 것인가?

암호화 알고리즘은 크게 아래와 같이 분류된다.

```java
1. 단방향 암호화
2. 대칭키 암호화
3. 비대칭키 암호화
```
단방향 암호화는 `plain text` 를 `hashing` 처리하며, 복호화를 할 수 없기 때문에 `password` 암호화에 주로 사용된다.

우리가 정의한 요구사항을 만족하려면 복호화가 가능해야 하므로 `단방향 암호화` 는 후보에서 제외한다.

다음은 대칭키 암호화, 비대칭키 암호화이다.

러프하게 설명하면,

- 대칭키 암호화는 암호화 할 때 사용한 key 로만 복호화가 가능하다.


<img width="442" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/6d2c49a4-292b-4bd9-a437-7462acb63c8a">



대표적으로 AES 암호화 알고리즘이 있다.


- 비대칭키 암호화는 암호화 할 때 사용한 key 와 복호화할 때 사용가능한 key 를 서로 다르게 할 수 있다.


<img width="441" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/ff7cbb58-2e2c-4646-95ba-2d6d2ab77c25">


대표적으로 RSA 암호화 알고리즘이 있다.

대칭키 암호화와 비대칭키 암호화는 둘다 복호화가 가능하다.

둘 중 어느 암호화 알고리즘을 사용하는 게 좋을까?

구현 난이도는 대칭키 암호화 알고리즘인 AES 가 비대칭키 암호화 알고리즘인 RSA 보다 훨씬 용이하다.

- ( 심지어 프로젝트 내에 AES Utility 클래스가 구현되어 있었기 때문에 그냥 가져다가 사용하면 되는 상황이었다. )

그럼에도 RSA 암호화 알고리즘을 선택하였는데, 그 이유는 아래와 같다.

```java
1. 대칭키 암호화(AES)를 사용하려면 key 를 공유해야 한다. Server side 에서는 db 처리를 해서 가지고 있으면 되지만 Client side 에서는 key 를 안전하게 저장할 곳이 마땅치 않다. 
2. 비대칭키 암호화(RSA)를 사용하면 같은 key 를 공유할 필요가 없어진다. 암호화는 누구나 할 수 있지만, 복호화는 마음대로 하지 못하게 하는 것이 가능하다.
```

초기에 정의한 요구사항을 RSA 암호화로 해결할 수 있는지 확인해보자.

- 클라이언트에서 민감 정보를 서버로 보낼 때에는 암호화하고, 서버에서 복호화한 뒤 로직을 수행하자.
  - 클라이언트에서 public key 로 암호화하여 서버에 요청하면, 서버에서는 해당 public key 에 대한 private key 를 알고있으면, 가능하다.


즉, `클라이언트에서 public key 로 암호화하여 서버에 요청하면, 서버에서는 해당 public key 에 대한 private key 를 알고있으면` 된다.

그럼 어떻게 할 수 있을까?

기본적으로 `RSA keypair` 는 안전하게 서버에서 관리하는 게 좋아보인다.

그럼 아래와 같이 하면 어떨까?

1. 클라이언트에서 암호화 할 일이 생긴다면 서버로 `RSA public key` 를 하나 요청한다.
2. 서버에서 `RSA Keypair` 를 하나 생성하고, `public key` 를 내려준다.
3. 클라이언트에서 `public key` 로 암호화를 수행한 뒤, `public key` 와 함께 암호문을 서버로 보낸다.
   - 이 과정에서 제 3 자가 탈취하더라도 `public key` 에 매칭되는 `private key` 를 모르니 안전하다.
4. 서버에서는 `public key` 에 매칭되는 `private key` 를 찾아서 복호화한 뒤 로직을 수행한뒤에 해당 `key pair` 를 제거한다.


이를 위해서 클라이언트가 암호화 한 `public key` 에 대한 `private key` 를 조회할 수 있어야 한다.

rdb 에 저장해둘 수도 있겠지만, 인메모리 기반으로 더 빠른 I/O 를 제공하며

ttl 을 지정할수도 있고 public key 에 대한 private key 를 찾는 연산이

자주 발생할 것으로 예상되는데, 

key-value 형식의 data storage 인 `redis` 가 더 적합하다고 판단하였다.

하지만, 하나만 더 고려해보자.

위 방법대로면 요청마다 서버에서 `RSA Keypair` 를 생성해야 한다.

그렇게 해도 괜찮을 만큼 `RSA Keypair 생성 연산이 가벼운 지` 찾아본 결과,

```java
Finding two large prime numbers	
  : O((log n)^2)
Computing the modulus	
  : O(log^2 n)
Computing modular inverses	
  : O(log^3 n)
Total
  : O((log n)^3)
```
대략 `O((log n)^3)` 정도인 것 같다. ( n = 1024, 2048, ... )

굳이 매 요청마다 `RSA Keypair` 를 새로 생성해도록 해야할까?

`Thread pool` 처럼 `RSA Keypair` 를 여러개 서버에서 미리 만들어두고

클라이언트로부터 요청이 들어오면 하나 랜덤하게 `pick` 해서 내려주는 방법은 어떨까?

그러면 RSA Keypair 를 매번 새롭게 만드는 데에 발생하는 cost 를 제거할 수 있다.

<img width="1015" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/ec6ccf90-dbef-4fbf-bbda-7b7ece012da2">


1. 클라이언트에서 암호화 할 일이 생긴다면 서버로 `RSA public key` 를 하나 요청한다.
2. 서버에서는 미리 생성해둔 `RSA Keypair` 중 하나를 골라서, `public key` 를 내려준다.
3. 클라이언트에서 `public key` 로 암호화를 수행한 뒤, `public key` 와 함께 암호문을 서버로 보낸다. 
   - 이 과정에서 제 3 자가 탈취하더라도 `public key` 에 매칭되는 `private key` 를 모르니 안전하다.
4. 서버에서는 `public key` 에 매칭되는 `private key` 를 찾아서 복호화하여 이후 로직을 수행한다.


하지만 또.. 하나 더 고려해야할 점이 있다.

위 로직은 Redis 에 상당한 의존도를 가진다.

만약 Redis 서버가 다운된다면?

이에 대한 고려가 전혀 되어 있지 않으므로, Redis 를 사용하는 API 에서 모두 장애가 발생할 것이다.

Redis 서버가 다운되는 상황이 흔치 않다고는 하지만 가능성이 없는 것은 아니다.

이러한 상황까지 커버하려면 어떻게 해야할까?

Redis 서버가 정상적이지 않은 경우에 대해서도 잘 동작할 수 있도록

별도의 fallback 을 파두어야 한다.

이를 `Circuit breaker` 이라고 하며, 해당 내용은 별도의 포스팅에서 다루려고 한다.

---
## References
- https://www.baeldung.com/java-rsa
- https://www.geeksforgeeks.org/how-to-generate-large-prime-numbers-for-rsa-algorithm/
- https://dev.gmarket.com/47