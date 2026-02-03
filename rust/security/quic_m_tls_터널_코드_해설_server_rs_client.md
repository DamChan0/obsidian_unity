# 목표
WSL 환경에서 **QUIC + mTLS**로 “키(클라이언트 인증서)를 가진 클라이언트만 접속” 가능한 **보안 터널의 뼈대**를 만든다.

- **server.rs**: QUIC 서버를 띄우고, **mTLS(클 인증서 필수)**로 핸드셰이크를 통과한 연결만 받아서 **bi-stream echo**를 수행한다.
- **client.rs**: 서버에 QUIC로 붙을 때 **서버 인증라서 검증(CA 루트)** + **클라이언트 인증서 제시(mTLS)**를 수행하고, **ping → echo**를 확인한다.

---

# 공통 핵심 개념
## 1) QUIC(=UDP + TLS 1.3 + 스트림)
- QUIC은 UDP 위에서 **연결(Connection)**을 만들고, 그 안에 **여러 스트림(Stream)**을 멀티플렉싱한다.
- 핸드셰이크는 TLS 1.3 기반이다.

## 2) mTLS(서버+클라 상호 인증)
- 서버는 **클라이언트 인증서**를 요구한다.
- 클라이언트는 **자기 인증서+개인키**로 자신을 증명한다.
- 서버는 **너의 CA(루트)**로 클라이언트 인증서를 검증한다.

## 3) ALPN
- TLS 핸드셰이크에서 “우리 앱 프로토콜”을 문자열로 합의한다.
- `qtunnel/1`을 클라/서버 양쪽에서 동일하게 설정해야 한다.

---

# server.rs

## 전체 코드
```rust
use anyhow::{Context, Result};
use quinn::crypto::rustls::QuicServerConfig;
use rustls::pki_types::{CertificateDer, PrivateKeyDer};
use std::{fs, net::SocketAddr, sync::Arc};

fn load_certs_pem(path: &str) -> Result<Vec<CertificateDer<'static>>> {
    let bytes = fs::read(path).with_context(|| format!("read cert pem: {path}"))?;
    let certs = rustls_pemfile::certs(&mut &*bytes)
        .collect::<std::result::Result<Vec<_>, _>>()
        .context("parse cert pem")?;
    Ok(certs)
}

fn load_private_key_pem(path: &str) -> Result<PrivateKeyDer<'static>> {
    let bytes = fs::read(path).with_context(|| format!("read key pem: {path}"))?;
    let key = rustls_pemfile::private_key(&mut &*bytes)
        .context("parse private key pem")?
        .context("no private key found")?;
    Ok(key)
}

#[tokio::main]
async fn main() -> Result<()> {
    // ===== 설정값(하드코딩 최소) =====
    let listen: SocketAddr = "0.0.0.0:4433".parse().unwrap();

    // mTLS용 파일들
    let ca_crt = "certs/out/ca.crt"; // 클라 인증서가 이 CA로 서명돼야만 허용
    let server_crt = "certs/out/server.crt";
    let server_key = "certs/out/server.key";

    // ===== (1) 서버가 제시할 인증서/키 로드 =====
    let cert_chain = load_certs_pem(server_crt)?;
    let key = load_private_key_pem(server_key)?;

    // ===== (2) 클라이언트 인증서 검증용 Root CA 로드 =====
    let ca_chain = load_certs_pem(ca_crt)?;
    let mut roots = rustls::RootCertStore::empty();
    for c in ca_chain {
        roots.add(c).context("add ca cert to root store")?;
    }

    // ===== (3) "클라 인증서 필수" verifier 구성 =====
    // rustls의 webpki 검증기를 사용한다. (allow_unauthenticated()를 호출하지 않으면 필수이다)
    let client_verifier = rustls::server::WebPkiClientVerifier::builder(roots.into())
        .build()
        .context("build client cert verifier")?;

    // ===== (4) rustls ServerConfig 생성 (mTLS) =====
    let mut server_crypto = rustls::ServerConfig::builder()
        .with_client_cert_verifier(client_verifier)
        .with_single_cert(cert_chain, key)
        .context("build rustls server config")?;

    // ALPN은 우리 앱 프로토콜 식별자이다(임의 문자열). 클라/서버가 동일해야 한다.
    server_crypto.alpn_protocols = vec![b"qtunnel/1".to_vec()];

    // ===== (5) quinn ServerConfig로 래핑하고 listen =====
    let mut server_config =
        quinn::ServerConfig::with_crypto(Arc::new(QuicServerConfig::try_from(server_crypto)?));

    // (옵션) 유니 스트림은 사용 안 한다.
    Arc::get_mut(&mut server_config.transport)
        .unwrap()
        .max_concurrent_uni_streams(0u8.into());

    let endpoint = quinn::Endpoint::server(server_config, listen)?;
    eprintln!("server listening on {}", endpoint.local_addr()?);

    // ===== (6) 연결 수락 + 스트림 echo =====
    while let Some(incoming) = endpoint.accept().await {
        tokio::spawn(async move {
            let conn = match incoming.await {
                Ok(c) => c,
                Err(e) => {
                    eprintln!("handshake failed: {e}");
                    return;
                }
            };
            let remote = conn.remote_address();
            eprintln!("connected: {remote}");

            loop {
                let Ok((mut send, mut recv)) = conn.accept_bi().await else {
                    eprintln!("connection closed: {remote}");
                    return;
                };

                let msg = match recv.read_to_end(64 * 1024).await {
                    Ok(b) => b,
                    Err(e) => {
                        eprintln!("read failed: {e}");
                        return;
                    }
                };

                // echo
                if let Err(e) = send.write_all(&msg).await {
                    eprintln!("write failed: {e}");
                    return;
                }
                let _ = send.finish();
            }
        });
    }

    Ok(())
}
```

## 구조/흐름(큰 그림)
1. **PEM 로더 함수 2개**로 (인증서/키) 파일을 rustls 타입으로 변환한다.
2. 서버가 제시할 **서버 인증서+키**를 로드한다.
3. 클라 인증서를 검증할 **CA 루트 저장소**를 만든다.
4. `WebPkiClientVerifier`를 만들어 **클라 인증서 필수(mTLS)**로 만든다.
5. rustls 서버 설정을 만들고, 이를 quinn 서버 설정으로 래핑한다.
6. 엔드포인트를 띄우고 연결을 accept한 뒤, 스트림을 받아 **echo**한다.

## 라인별 해설(핵심 줄 중심)
### import 블록
- `anyhow::{Context, Result}`: 에러에 문맥을 붙여 디버깅하기 쉽게 한다.
- `QuicServerConfig`: rustls 설정을 QUIC용으로 포장한다.
- `CertificateDer / PrivateKeyDer`: rustls가 쓰는 인증서/키 바이너리 표현이다.

### `load_certs_pem`
- `fs::read`: PEM 파일을 바이트로 읽는다.
- `rustls_pemfile::certs`: PEM에서 cert들을 파싱한다.
- `collect::<Result<Vec<_>, _>>()`: iterator 결과를 Vec로 모으고 파싱 에러를 처리한다.

### `load_private_key_pem`
- `rustls_pemfile::private_key`: PEM에서 private key 1개를 꺼낸다.
- `.context("no private key found")?`: 파일에 키가 없으면 즉시 실패한다.

### `main` 설정
- `listen = 0.0.0.0:4433`: 모든 인터페이스에서 UDP 4433 수신이다.
- `ca_crt/server_crt/server_key`: mTLS 구성 요소 파일 경로다.

### mTLS의 ‘결정적’ 구성
- `roots.add(c)`: 신뢰할 CA를 등록한다.
- `WebPkiClientVerifier::builder(...).build()`: **서버가 클라 인증서 검증기를 가진다**.
- `with_client_cert_verifier(client_verifier)`: 여기서 **클라 인증서 요구가 사실상 강제**된다.

### ALPN
- `server_crypto.alpn_protocols = ...`: `qtunnel/1`로 앱 프로토콜 합의.

### QUIC 서버 구동
- `Endpoint::server(server_config, listen)`: 서버 엔드포인트 생성 + bind.
- `endpoint.accept().await`: 새 인바운드 연결을 기다린다.
- `incoming.await`: 핸드셰이크 완료 대기(여기서 mTLS 검증 실패하면 Err로 떨어진다).

### 스트림 echo
- `conn.accept_bi().await`: 클라이언트가 연 bi-stream을 받는다.
- `recv.read_to_end(...)`: 스트림 끝까지 읽는다(상한 64KB).
- `send.write_all(&msg)`: 받은 데이터를 그대로 돌려준다.
- `send.finish()`: 송신 스트림을 종료해 상대의 read가 끝나게 한다.

---

# client.rs

## 전체 코드
```rust
use anyhow::{Context, Result};
use quinn::crypto::rustls::QuicClientConfig;
use rustls::pki_types::{CertificateDer, PrivateKeyDer};
use std::{fs, net::SocketAddr, sync::Arc};

fn load_certs_pem(path: &str) -> Result<Vec<CertificateDer<'static>>> {
    let bytes = fs::read(path).with_context(|| format!("read cert pem: {path}"))?;
    let certs = rustls_pemfile::certs(&mut &*bytes)
        .collect::<std::result::Result<Vec<_>, _>>()
        .context("parse cert pem")?;
    Ok(certs)
}

fn load_private_key_pem(path: &str) -> Result<PrivateKeyDer<'static>> {
    let bytes = fs::read(path).with_context(|| format!("read key pem: {path}"))?;
    let key = rustls_pemfile::private_key(&mut &*bytes)
        .context("parse private key pem")?
        .context("no private key found")?;
    Ok(key)
}

#[tokio::main]
async fn main() -> Result<()> {
    // ===== 설정값 =====
    // <PUBLIC_IP> 를 실제 공인 IP로 바꿔라
    let server_ip = "<PUBLIC_IP>";
    let remote: SocketAddr = format!("{server_ip}:4433").parse().unwrap();

    let ca_crt = "certs/out/ca.crt";       // 서버 인증서 검증용 CA
    let client_crt = "certs/out/client.crt";
    let client_key = "certs/out/client.key";

    // ===== (1) 서버 CA 신뢰 설정 =====
    let mut roots = rustls::RootCertStore::empty();
    for c in load_certs_pem(ca_crt)? {
        roots.add(c).context("add ca cert to root store")?;
    }

    // ===== (2) 클라 인증서/키 로드 (mTLS) =====
    let client_cert_chain = load_certs_pem(client_crt)?;
    let client_key = load_private_key_pem(client_key)?;

    let mut client_crypto = rustls::ClientConfig::builder()
        .with_root_certificates(roots)
        .with_client_auth_cert(client_cert_chain, client_key)
        .context("build rustls client config")?;

    client_crypto.alpn_protocols = vec![b"qtunnel/1".to_vec()];

    let client_config = quinn::ClientConfig::new(
        Arc::new(QuicClientConfig::try_from(client_crypto)?)
    );

    // ===== (3) 엔드포인트 생성 및 기본 설정 =====
    let mut endpoint = quinn::Endpoint::client("0.0.0.0:0".parse().unwrap())?;
    endpoint.set_default_client_config(client_config);

    // ===== (4) connect =====
    // 여기의 server_name(host)는 인증서 검증에 쓰인다.
    // "공인 IP SAN"을 넣었으니, host에도 IP 문자열을 넣는다.
    let conn = endpoint
        .connect(remote, server_ip)?
        .await
        .context("connect")?;

    eprintln!("connected to {remote}");

    // ===== (5) ping/echo =====
    let (mut send, mut recv) = conn.open_bi().await.context("open bi")?;
    let msg = b"ping";
    send.write_all(msg).await.context("send")?;
    let _ = send.finish();

    let resp = recv.read_to_end(64 * 1024).await.context("recv")?;
    eprintln!("resp: {}", String::from_utf8_lossy(&resp));

    conn.close(0u32.into(), b"done");
    endpoint.wait_idle().await;
    Ok(())
}
```

## 구조/흐름(큰 그림)
1. 서버 인증서 검증용 **CA 루트 저장소**를 만든다.
2. 클라이언트 인증서+키를 로드해서 **mTLS로 제시**한다.
3. QUIC 클라이언트 엔드포인트를 만들고 기본 설정을 세팅한다.
4. `connect(remote, server_ip)`로 핸드셰이크한다(서버 인증서 SAN 매칭 포함).
5. bi-stream을 열어 `ping`을 보내고 echo를 받는다.
6. 연결을 닫고 종료한다.

## 라인별 해설(핵심 줄 중심)
### 서버 검증(서버 인증서가 진짜인지)
- `with_root_certificates(roots)`: 클라이언트가 **서버 인증서**를 검증할 때 쓰는 루트 CA를 지정한다.

### 클라 인증(mTLS에서 ‘내가 부여한 키’)
- `with_client_auth_cert(client_cert_chain, client_key)`: 클라이언트가 **자기 인증서+개인키**를 서버에 제시한다.
- 서버는 이 인증서를 CA로 검증하고, 키 소유 증명을 확인한다.

### server_name이 중요한 이유
- `connect(remote, server_ip)`: 두 번째 인자는 TLS의 **Server Name(검증 기준)**이다.
- 너는 서버 인증서를 **IP SAN**으로 발급했으니, 여기에도 **IP 문자열**을 넣어야 검증이 맞아떨어진다.

### ping/echo
- `open_bi()`: 연결 안에 새 양방향 스트림을 연다.
- `send.write_all(b"ping")`: ping 전송.
- `send.finish()`: 송신 종료(상대 read_to_end가 끝을 감지).
- `recv.read_to_end(...)`: echo 수신.

---

# 학습 체크(1문항)
server.rs에서 **mTLS를 “필수”로 만드는 결정적 설정 블록**은 아래 중 어디인가?
1) `roots.add(...)` 루프
2) `WebPkiClientVerifier::builder(...).build()`
3) `with_client_cert_verifier(client_verifier)`

하나만 골라라.

