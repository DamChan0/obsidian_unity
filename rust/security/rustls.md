# rustls란
- TLS 프로토콜 흐름(핸드세이크,레코드 처리)를 담당하는 **<mark style="background: #ADCCFFA6;">TLS 프로토콜 엔진</mark>** 
- `rustls`는 TLS 규칙 자체를 구현함
- 실제 암호 계산로직은 CryptoProvider가 담당한다
- CryptoProvider = 암호 백엔드 구현 묶음을 주입한다 

- 소켓(TCP) 자체를 열어주진 않음  
    (소켓은 `tokio`, `std::net` 같은 쪽이 담당)
- 그 소켓 위에 보안 채널(TLS 계층)을 올려주는 역할을 함

## Crypto란

- TSL에서 Crypto 는 아래 요소가 필요하다
	- 키 교환(Key Exchange): ECDHE 같은 방식으로 세션 키 재료 생성
	- 인증(Authentication): 인증서 체인 검증, 서명 검증
	- 무결성(Integrity): 메시지가 변조되지 않았는지 확인
	- 기밀성(Confidentiality): AES-GCM/ChaCha20-Poly1305로 데이터 암호화
	- 난수(Randomness): nonce, 키 생성용 안전한 난수
- 이런 것들을 수행하는데 필요한 기능들을 라이브러리로 제공한다

### `CryptoProvider`가 왜 필요한가

`rustls`는 암호 알고리즘을 하드코딩하지 않고, 공급자 패턴으로 분리한다

- 구현 교체 가능 (`ring`, `aws-lc-rs` 등)
- 정책/규정(FIPS 등)에 맞춰 백엔드 선택 가능
- 플랫폼/성능 특성에 따라 선택 가능

쉽게 비유하면:

- `rustls` = 운전 규칙을 아는 운전자(TLS 프로토콜 엔진)
- `CryptoProvider` = 실제 엔진/변속기(암호 계산 구현)

## CryptoProvider

- get_default 
	- ``` rust
		pub fn get_default() -> Option<&'static Arc<Self>> {
			static_default::get_default()
		}
		``` 
- option이 있기때문에 값이없으면 none을 반환함 