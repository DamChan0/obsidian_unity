# crates
```toml
anyhow = "1"
tokio = { version = "1", features = ["macros", "rt-multi-thread", "io-util"] }
quinn = "0.11"
rustls = "0.23"
rustls-pemfile = "2"
```



## anyhow 

### with_context :  Error시 context를 출력함
``` rust
    fn with_context<C, F>(self, context: F) -> Result<T, Error>
    where
        C: Display + Send + Sync + 'static,
        F: FnOnce() -> C,
    {
        match self {
            Ok(ok) => Ok(ok),
            Err(error) => Err(error.ext_context(context())),
        }
    }
```

## rustls 

### rustls_pemfile::cert

> **PEM 형식 안에서 “CERTIFICATE 블록들”만 찾아서 파싱한다**
- 반환값 : Iterator 
	- 즉 각 아이템은 `Result<CertificateDer<'_>, Error>` 형태를 띈다
- collect의 의미
- 이건 Rust의 정석 패턴이다:
	- `Iterator<Item = Result<T, E>>`  → `Result<Vec<T>, E>`
	- 즉
		- 하나라도 실패하면 → 즉시 Err
		- 전부 성공하면 → Vec로 묶음