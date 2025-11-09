---
title: "Axum Codec"
date: 2025-11-09T13:07:13+08:00
draft: false
tags: ["rust"]
---

## [axum_codec](https://docs.rs/axum-codec/latest/axum_codec/)

axum_codec 是一个用于处理 axum 请求和响应的编解码器。
可用根据客户端的accept头，自动处理 JSON、XML、YAML、TOML、MsgPack格式。

```rust
#[axum_codec::apply(encode, decode)]
struct User {
  name: String,
  age: u8,
}

async fn me() -> Codec<User> {
  Codec(User {
    name: "Alice".into(),
    age: 42,
  })
}
```

通过包装成 `Codec<User>` 自动客户端请求的格式。

## extract时错误处理

为了统一格式，我们需要在extract出错时，也返回客户端期望的格式，如msgpack。应该如何处理呢。
首先我们需要定义一个错误类型，如 `AppError`，并让它实现 `IntoCodecResponse` trait，然后如下

```rust
use axum::extract::{FromRequestParts, Path, Query};
use axum::http::request::Parts;
use axum::response::IntoResponse;
use axum_codec::IntoCodecResponse;

pub struct ApiErrorWithAccept {
    error: ApiError,
    accept: axum_codec::Accept,
}

impl IntoResponse for ApiErrorWithAccept {
    fn into_response(self) -> axum::response::Response {
        let ApiErrorWithAccept { error, accept } = self;
        let response = error.into_codec_response(accept.into());
        response.into_response()
    }
}

pub struct AppPath<T>(pub T);

impl<S, T> FromRequestParts<S> for AppPath<T>
where
    T: serde::de::DeserializeOwned + Send,
    S: Send + Sync + 'static,
{
    type Rejection = ApiErrorWithAccept;

    async fn from_request_parts(parts: &mut Parts, state: &S) -> Result<Self, Self::Rejection> {
        let accept = axum_codec::Accept::from_request_parts(parts, state)
            .await
            .unwrap();

        match Path::<T>::from_request_parts(parts, state).await {
            Ok(Path(value)) => Ok(AppPath(value)),
            Err(rejection) => Err(ApiErrorWithAccept {
                error: ApiError::new(
                    ApiErrorCode::ParamOrQuery,
                    format!("Invalid path parameter: {}", rejection),
                ),
                accept,
            }),
        }
    }
}
```

实现 `FromRequestParts`，并自定义 `Rejection` 类型，使其中包含 `ApiError` 及 `axum_codec::Accept`，
例如上面的 `ApiErrorWithAccept`，并为其实现 `IntoResponse` trait，也可通过相似的方法实现 `AppQuery`

## 404时返回格式

```rust
use axum_codec::{Accept, IntoCodecResponse};
use tower_http::trace::TraceLayer;

async fn not_found(accept: Accept) -> impl IntoResponse {
    ApiError::not_found().into_codec_response(accept.into())
}

let app = Router::new()
    .route("/health", get(health_check).into())
    .nest("/api/v1", api_v1)
    .fallback(not_found)
    .layer(TraceLayer::new_for_http().on_request(
        |req: &axum::http::Request<axum::body::Body>, _span: &tracing::Span| {
            info!("{} {}", req.method(), req.uri());
        },
    ));
```

`accept: Accept`，可以直接声明，自动extract，使用一个实现了 `IntoCodecResponse` 的对象，
转换时使用`accept`。最后在`route`中使用`fallback`即可。

> 上述代码最后面是用来打开每个请求的日志，方便调试。
