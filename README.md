OpenIRC protocol <sup>v0.0.0</sup>
========
> *This document is stil work in progress*

OpenIRC에서 서버와 클라이언트가 어떤 것을 얼마나 어떻게 서로 주고받을지를 기술합니다.

### Table of contents
1. [방법](#1-방법)
2. [채팅 메세지 포맷](#2-채팅-메세지-포맷)
3. [요청/응답 메세지 포맷](#3-요청응답-메세지-포맷)
4. [서버에서 클라이언트로 보내는 메세지 포맷](#4-서버에서-클라이언트로-보내는-메세지-포맷)

<br>

1. 방법
--------

### 로그인
RESTful api를 사용하여 로그인을 수행합니다.

### 채팅 메세지
[웹소켓](http://tools.ietf.org/html/rfc6455)을 통해서 데이터를 주고받습니다.

사용자 계정은 로그인 세션을 통해 정보를 얻을 수 있기 때문에 채팅 메세지에는 사용자 계정정보를 포함할 필요가 없습니다.

로그아웃 돼있을 경우 웹소켓 연결을 맺지 않습니다.

2. 채팅 메세지 포맷
--------

[메세지팩](http://msgpack.org/index.html)을 사용합니다.

주고받는 메세지 데이터는 무조건 `object` 형태여야 합니다.

이후로 작성되는 메세지 데이터 포맷은 [makise](https://github.com/disjukr/makise) 문법을 사용하여 기술됩니다.

서버는 클라이언트의 모든 요청에 대해서 응답을 해줄 의무가 있습니다.

클라이언트에서 서버로 보내는 모든 채팅 메세지는 요청 id(이하 `rid`)를 갖습니다.
서버의 모든 응답 메세지도 `rid`를 갖습니다.
클라이언트는 이 `rid`를 이용해서 요청에 상응하는 응답을 알아낼 수 있습니다.

`rid`는 `int32` 타입을 가지며, 클라이언트에서 직접 관리(`2147483647`을 넘어가면 `0`부터 다시 시작하는 등의 처리)하면 됩니다.

(클라이언트가 서버로 보내는) 요청 메세지의 기본 포맷은 다음과 같습니다:
```makise
request is {
    rid: int32,
    kind: request_kind
}

request_kind is (
    'join', 'leave', 'message'
)
```

(서버가 클라이언트로 보내는) 응답 메세지의 기본 포맷은 다음과 같습니다:
```makise
response is {
    rid: int32,
    kind: response_kind
}

response_kind is (
    'ok',
    'error'
)

response[kind = 'error'] is {
    type: error_type
}

error_type is () // TODO
```

응답이 아닌 경우에, 서버가 클라이언트로 보내는 메세지는 클라이언트가 응답할 의무가 없습니다.

서버가 클라이언트로 보내는 메세지의 기본 포맷은 다음과 같습니다:
```
message is {
    kind: message_kind
}

message_kind is () // TODO
```

3. 요청/응답 메세지 포맷
--------
```makise
target is {
    kind: ('channel', 'user'),
    name: string
}

// 채널 접속
request[kind = 'join'] is {
    target: target
}

// 채널 퇴장
request[kind = 'leave'] is {
    target: target
}

// 대화
request[kind = 'message'] is {
    target: target,
    content: string
}
```


4. 서버에서 클라이언트로 보내는 메세지 포맷
--------

**TODO**
