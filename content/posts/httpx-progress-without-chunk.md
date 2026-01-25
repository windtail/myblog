---
title: "Httpx Progress Without Chunk"
date: 2026-01-25T14:21:44+08:00
draft: false
tags:
  - python
---

httpx 一般put/post/patch，又想要进度，一般都会使用chunked方式发送content，
下面介绍了一种方式，content传一个coroutine，可以慢慢地发，同时可以更新进度，
但是`Content-Length`得自己设置，因为httpx不能从coroutine中得到。这样发送的请求，
不是chunked的方式。

```python

_CHUNK_SIZE = 1024 * 500


@define
class ApiClient:
    client: httpx.AsyncClient
    ip: str

    async def put_with_progress(
        self,
        path: str,
        content: bytes,
        progress: Callable[[int], None],
        chunk_size: int = _CHUNK_SIZE,
        headers: Mapping[str, str] | None = None,
    ):
        content_length = len(content)

        async def sender():
            sent = 0
            for k in itertools.count():
                chunk = content[k * chunk_size : (k + 1) * chunk_size]
                if not chunk:
                    break
                yield chunk
                sent += len(chunk)
                progress(sent * 100 // content_length)

        # 设置请求头（关键：手动指定Content-Length）
        _headers = {"Content-Length": str(content_length)}
        if headers is not None:
            _headers.update(headers)

        response = await self.put(path, headers=_headers, content=sender())
        response.into_no_content()

    async def get_with_progress(
        self,
        path: str,
        progress: Callable[[int], None],
        chunk_size: int = _CHUNK_SIZE,
        headers: Mapping[str, str] | None = None,
    ) -> bytes:
        async with self.client.stream("GET", path, headers=headers) as response:
            total_size = int(response.headers.get("Content-Length", 0))

            received = 0
            buf = BytesIO()
            async for chunk in response.aiter_bytes(chunk_size):
                buf.write(chunk)
                received += len(chunk)
                if total_size > 0:
                    progress(received * 100 // total_size)
                else:
                    progress(received // chunk_size)  # chunked encoding，每读到一个chunk，+1

            content = buf.getvalue()
            if response.is_error:
                raise ApiError(kebab_serde.from_json(json.loads(content), ApiErrorData))
            else:
                return content
```