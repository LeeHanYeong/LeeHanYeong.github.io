## Introduction

Channels에 오신 것을 환영합니다! Channels는 Django의 synchronous core를 변경하여 비동기 코드를 만들며, Django프로젝트가 HTTP뿐만 아니라 long-running connection이 필요한 WebSocket, MQTT, 챗봇, 라디오 등의 프로토콜을 처리할 수 있습니다.

Django의 동기적이고 사용하기 쉬운 특성을 유지하면서, 코드를 작성하는 방법을 선택할 수 있습니다. Django views와 같은 synchronous(동기적인) 스타일과 완전 비동기식(fully asynchronous), 또는 그 둘의 혼합 방식이 있습니다. 또한, Django의 인증과 세션 시스템등과 통합되어 HTTP전용 프로젝트를 다른 프로토콜로 쉽게 확장 할 수 있습니다.

또, 이벤트 중심 아키텍쳐를 프로세스 간 쉽게 통신하고 프로젝트를 다른 프로세스로 분리할 수 있는 시스템인 channel layers를 번들로 제공합니다.

### Turtles All The Way Down

Channels는 "거북이는 언제나 거기에 있어"라는 원칙에 따라 작동합니다. 우리는 channels "application"이 무엇인지에 대한 단일 아이디어를 가지고 있으며, 가장 단순한 consumers(Django views와 동일)조차도 스스로 실행 할 수 있는 완전히 유효한 ASGI애플리케이션입니다.

> ASGI는 Channels가 구축된 비동기 서버 specification의 이름입니다. WSGI와 마찬가지로 Channels와 Daphne에 고정되지 않고, 다른 서버와 프레임워크 중에서 선택할 수 있도록 설계되었습니다.  
> [http://asgi.readthedocs.io](http://asgi.readthedocs.io)

Channels는 이러한 basic consumers (채팅 메시징 또는 알림을 처리할 수 있는 개별 조각)를 작성하고, URL라우팅, 프로토콜 탐지 및 기타 편리한 기능을 함께 묶어 완전한 애플리케이션을 만드는 도구를 제공합니다.

Channels에서는 HTTP와 기존 Django views를 전체의 일부로 취급합니다. 전통적인 Django views는 여전히 Channels와 함께 사용 가능하며(`channels.http.AsgiHandler`라는 ASGI애플리케이션으로 되어 있습니다), 그 외에 사용자가 정의한 HTTP 롱 폴링 핸들링 또는 WebSocket수신자 등의 기존 코드를 함께 배치할 수도 있습니다.

우리는 대부분의 코드에 대해 Django views와 같은 안전하고 동기적인 기술을 사용할 수 있는 기능을 원하지만, 복잡한 작업을 위해 보다 직접적인 비동기식 인터페이스로 drop down할 수 있는 옵션을 가지고 있습니다.

### Scopes and Events

Channels와 ASGI는 들어오는 연결들을 scope와 일련의 events라는 두 가지 구성요소로 나눕니다.

scope는 하나의 들어오는 연결(incoming connection)에 대한 세부정보 집합이며, 연결 전체에서(throughout the connection) 유지됩니다.

> 세부정보: 웹 요청의 path또는 WebSocket의 origin IP, 사용자가 챗봇에게 메시징하는 등

HTTP의 경우 scope는 단일 요청동안 지속됩니다. WebSocket의 경우, 소켓의 수명동안 지속됩니다(단, 소켓을 닫았다가 다시 연결하면 변경됨). 다른 프로토콜의 경우, 프로토콜의 ASGI spec작성 방법에 따라 다릅니다. 예를 들어, 챗봇 프로토콜은 기본 채팅 프로토콜이 상태가 없는 경우에도 봇과의 사용자 대화 전체에 대해 하나의 scope를 유지할 수 있습니다.

이 scope의 수명 동안, 일련의 이벤트들이 발생합니다. 이는 HTTP요청 작성 또는 WebSocket프레임 전송과 같은 사용자 상호작용을 나타냅니다. Channels또는 ASGI 애플리케이션은 **scope당 한 번 인스턴스화**되며, 해당 scope내에서 발생하는 events의 스트림이 공급되어 처리 방법을 결정합니다.

HTTP를 사용한 예:

- 사용자가 HTTP요청을 만듭니다
- 요청 경로, 메서드, 헤더 등의 세부 정보가 포함된 새로운 `http`유형의 scope를 엽니다
- HTTP body content와 함께 `http.request`이벤트를 보냅니다
- Channels또는 ASGI애플리케이션은 이를 처리하고 `http.response`이벤트를 생성하여 브라우저로 보내고 연결을 닫습니다
- HTTP 요청/응답이 완료되고 scope가 삭제됩니다

chatbot을 사용한 예:

- 사용자는 첫 번째 메시지를 chatbot에 보냅니다
- 사용자의 username, chosen name, userID를 포함한 scope를 엽니다
- 응용프로그램에는 이벤트 텍스트가 있는 `chat.received_message`이벤트가 제공됩니다. 응답을 할 필요는 없지만, 원하는 경우 하나 이상의 채팅 메시지를 `chat.received_message`이벤트로 다시 보낼 수 있습니다
- 사용자가 더 많은 메시지를 챗봇에 보내고, 더 많은 `chat.received_message`이벤트가 생성됩니다
- 시간초과 또는 응용 프로그램 프로세스가 다시 시작되면 scope가 닫힙니다

scope의 수명(채팅, HTTP요청, 소켓 연결 등) 내에모든 이벤트를 처리하는 하나의 애플리케이션 인스턴스가 있게되며, 모든 항목들을 해당 인스턴스에서 유지 할 수 있습니다. 원하는 경우 raw ASGI애플리케이션을 작성할 수도 있지만, Channels는 consumers라는 사용하기 쉬운 추상화를 제공합니다.

### What is Consumer?

consumer는 Channels코드의 기본 단위입니다. 우리는 events를 소비하는 것을 consumer라고 부릅니다. 하지만 이를 작은 응용프로그램으로 생각 할 수 있습니다. 요청이나 새 소켓이 들어오면 Channels는 라우팅 테이블을 따라가며, 들어온 연결에 적합한 consumer를 찾아서 복사본을 시작합니다.

이는 Django views와 달리, consumers가 long-running한다는 것을 의미합니다. 또한 실행시간이 짧을 수도 있지만(이것은 결국 consumer가 HTTP요청을 제공할 수도 있다는 뜻입니다), consumer는 little while동안 살아있을 것이라는 아이디어를 바탕으로 구축되었습니다. (위에서 설명한 대로, consumer는 scope동안 살아있습니다)

기본 consumer는 다음과 같습니다:

```python
class ChatConsumer(WebsocketConsumer):
    def connect(self):
        self.username = "Anonymous"
        self.accept()
        self.send(text_data="[Welcome %s!]" % self.username)

    def receive(self, *, text_data):
        if text_data.startswith("/name"):
            self.username = text_data[5:].strip()
            self.send(text_data="[set your username to %s]" % self.username)
        else:
            self.send(text_data=self.username + ": " + text_data)

    def disconnect(self, message):
        pass
```

각각의 서로 다른 프로토콜들은 발생하는 이벤트들이 다르며, 각 유형들은 서로 다른 방법으로 표현됩니다. 각 이벤트를 처리하는 코드를 작성하면, Channles에서 이벤트들을 스케쥴링하여 병렬로 실행합니다.

내부적으로, Channels는 완전 비동기 이벤트 루프(fully asynchronous event loop)에서 실행되며, 위와 같은 코드를 작성하면 동기 스레드에서 호출됩니다. 따라서 Django ORM호출과 같은 차단 작업을 안전하게 수행 할 수 있습니다:

```python
class LogConsumer(WebsocketConsumer):
    def connect(self, message):
        Log.objects.create(
            type="connected",
            client=self.scope["client"],
        )
```

그럼에도 더 많은 제어 기능을 원하고, 비동기 함수에서만 작업하려는 경우 완전 비동기 consumer를 작성할 수 있습니다:

```python
class PingConsumer(AsyncConsumer):
    async def websocket_connect(self, message):
        await self.send({
            "type": "websocket.accept",
        })

    async def websocket_receive(self, message):
        await asyncio.sleep(1)
        await self.send({
            "type": "websocket.send",
            "text": "pong",
        })
```

You can read more about consumers in [Consumers](https://channels.readthedocs.io/en/latest/topics/consumers.html).

### Routing and Multiple Protocols

여러 Consumers(자신의 ASGI앱)를 라우팅을 사용하여 프로젝트를 나타내는 하나의 큰 앱으로 결합 할 수 있습니다:

```python
application = URLRouter([
    url(r"^chat/admin/$", AdminChatConsumer),
    url(r"^chat/$", PublicChatConsumer),
])
```

Channels는 HTTP및 WebSocket만을 위해 만들어지지 않았습니다. 이러한 프로토콜들과 유사한 이벤트 세트에 맵핑하는 서버를 만들어 Django환경에 빌드하여 적용할 수 있습니다. 예를 들어, 유사한 스타일로 챗봇을 만들 수 있습니다:

```python
class ChattyBotConsumer(SyncConsumer):
    def telegram_message(self, message):
        """
        Simple echo handler for telegram messages in any chat.
        """
        self.send({
            "type": "telegram.message",
            "text": "You said: %s" % message["text"],
        })
```

그리고 다른 라우터를 사용하여 한 프로젝트에서 WebSocket과 채팅 요청을 모두 처리할 수 있습니다:

```python
application = ProtocolTypeRouter({
    "websocket": URLRouter([
        url(r"^chat/admin/$", AdminChatConsumer),
        url(r"^chat/$", PublicChatConsumer),
    ]),
    "telegram": ChattyBotConsumer,
})
```

Channels의 목표는 최신 웹에서 접할 수 있는 모든 프로토콜 또는 transport에서 작동하도록 Django 프로젝트를 구축하고, 익숙한 구성 요소 및 코딩 스타일로 작업 할 수 있도록 하는 것입니다.

For more information about protocol routing, see [Routing](https://channels.readthedocs.io/en/latest/topics/routing.html).

### Cross-Process Communication

표준 WSGI서버와 마찬가지로, 프로토콜 이벤트를 처리하는 응용프로그램 코드는 서버 프로세스 자체 내에서 실행됩니다. 예를 들어, WebSocket처리 코드는 WebSocket서버 프로세스 내에서 실행됩니다.

전체 응용프로그램에 대한 각 소켓 또는 연결(connection)은 이러한 서버들 중 하나의 응용프로그램 인스턴스에 의해 처리됩니다. 그들은 호출되어 클라이언트에게 직접 데이터를 보낼 수 있습니다.

하지만, 보다 복잡한 응용프로그램 시스템을 구축 할 때에는 다른 응용프로그램 인스턴스간에 통신이 필요합니다. 예를 들어 대화방을 구축하는 경우, 하나의 응용프로그램 인스턴스가 메시지를 수신하면 이를 대화방에 속한 다른 사람들을 나타내는 다른 인스턴스들에 배포해야 합니다.

데이터베이스를 폴링하여 이 작업을 수행 할 수도 있지만, Channels에서는 여러 프로세스간에 정보를 전송할 수 있는 전송 집합에 대한 저수준 추상화인 channels layer개념을 도입했습니다. 각 응용프로그램 인스턴스는 고유한 channel name이 있으며, groups에 들어가 지점 간(point-to-point)와 브로드캐스트 메시징을 모두 허용할 수 있습니다.

>Channels layers는 Channels의 선택적인 부분이며, 원하는 경우 `CHANNELS_LAYERS`설정을 빈 값으로 설정하여 비활성화 할 수 있습니다

(insert cross-process example here)

또한, 고정된 채널 이름을 듣고 있는 전용 프로세스로 메시지를 보낼 수도 있습니다.

```python
# In a consumer
self.channel_layer.send(
    "myproject.thumbnail_notifications",
    {
        "type": "thumbnail.generate",
        "id": 90902949,
    },
)
```

You can read more about channel layers in [Channel Layers](https://channels.readthedocs.io/en/latest/topics/channel_layers.html).

### Django Integration

Channels는 세션 및 인증과 같은 일반적인 Django기능을 쉽게 지원합니다. 적절한 미들웨어를 추가하여 인증을 WebSocket views와 결합 할 수 있습니다:

```python
application = ProtocolTypeRouter({
    "websocket": AuthMiddlewareStack(
        URLRouter([
            url(r"^front(end)/$", consumers.AsyncChatConsumer),
        ])
    ),
})
```

For more, see [Sessions](https://channels.readthedocs.io/en/latest/topics/sessions.html) and [Authentication](https://channels.readthedocs.io/en/latest/topics/authentication.html).

