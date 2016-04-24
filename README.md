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
OpenIRC 서버는 OAuth를 사용한 인증과 이메일 인증 방식, 그리고 발급받은 토큰을 사용하는 인증방식을 제공합니다.

#### OAuth 절차
**TODO**

#### 이메일 인증 절차
**TODO**

#### 토큰 인증 절차
POST 메서드로 url은 `http://<OpenIRC 서버 주소>/login/`으로, 요청인자는 `{ "method": "token", "token": "<발급받은 토큰>" }`으로 호출하면 서버는 다음과 같은 json 객체를 응답합니다:
```json
{
    "connections": [
        {
            "name": "오징오징",
            "url": "ocarina.irc.ozinger.org",
            "port": 16667,
            "ssl": true,
            "connid": 0,
            "seq": 0,
            "user": {
                "nickname": "<사용자 대화명>",
                "realname": "<사용자명>",
                "username": "<사용자 계정명>",
                "password": "<사용자 비밀번호>"
            },
            "channels": [
                {
                    "name": "<채널 이름>",
                    "topic": "<채널 토픽>",
                    "users": []
                }
            ]
        }
    ],
    "ws": "<웹소켓 연결 주소>"
}
```

커넥션마다 `seq`라는 시퀀스 번호가 들어있는데,
이 시퀀스 번호보다 작은 채팅 이벤트 로그는 커넥션의 상태를 조작하지 말라는 의미입니다.
여기서 커넥션의 상태란 채널 목록과 채널별 토픽 및 유저목록 등을 의미합니다.


### 채팅 메세지
[웹소켓](http://tools.ietf.org/html/rfc6455)을 통해서 데이터를 주고받습니다.

로그인 절차를 거치면 사용자 기본 정보와 웹소켓 연결 주소(`res.ws`)를 얻을 수 있습니다.
사용자 인증된 웹소켓 연결 주소이기 때문에 채팅 메세지에는 사용자 인증 정보를 따로 담을 필요가 없습니다.

그 외 사용자 정보는 RESTful api를 통해서 얻어야 합니다.


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

`rid`가 `int32` 타입이 아니거나(예: `12.34`) 음수값을 가질 경우,
서버는 `rid`가 `-1`인 응답을 돌려주어야합니다:
```json
{
    "rid": -1,
    "given_rid": 12.34,
    "kind": "error",
    "list": [
        {
            "kind": "invalid_type",
            "context": "rid",
            "given_type": "number",
            "required_type": "int32"
        }
    ]
}
```

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

response[rid = -1] is {
    given_rid: *
}

response[kind = 'error'] is {
    list: [error]
}

error is {
    kind: ('invalid_type')
}

error[kind = 'invalid_type'] is {
    context: string, // 'key.array["0"].key2'...
    given_type: string,
    required_type: string
}
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
    connid: int32,
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
