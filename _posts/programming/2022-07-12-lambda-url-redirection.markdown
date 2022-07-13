---
layout: post
title: 'AWS Lambda와 API Gateway를 사용한 URL 리다이렉션'
categories: ['AWS']
---

Bitly, Short URL, Cuttly같은 서비스를 사용하면 긴 URL에 연결되는 짧은 URL을 생성할 수 있습니다. 하지만 대부분의 경우 원하는 URL을 사용 할 수는 없고, 서비스에서 제공하는 임의의 주소가 할당됩니다.

AWS의 서비스를 사용해서 임의의 URL대신 원하는 URL을 직접 지정하고, 소유한 도메인에 연결하는 과정을 설명합니다.

> 소유한 도메인이 있으며, Route53에 등록되어 있다는 가정 하에 진행합니다.

## ACM 인증서 발급

AWS Certificate Manager는 SSL/TLS인증서 프로비저닝, 관리, 배포할 수 있는 서비스입니다. Route53에 도메인을 등록해 놓았다면, 쉽게 SSL인증서를 발급 받을 수 있습니다.

예제에서는 Route53에 **lhy.kr**도메인을 소유하고 있으며, 이 도메인의 서브도메인인 **url.lhy.kr**을 사용해 URL redirection을 구현합니다.

[ACM Console](https://console.aws.amazon.com/acm)에서 Route53에 등록된 도메인의 서브도메인으로 인증서 발급을 요청합니다.

![acm1](../../images/2022-07-12-lambda-url-redirection/acm1.png)

DNS검증을 사용하면 자동으로 Route 53에 검증에 필요한 레코드를 생성 할 수 있습니다.

![acm2](../../images/2022-07-12-lambda-url-redirection/acm2.png)

![acm3](../../images/2022-07-12-lambda-url-redirection/acm3.png)

Route53에 레코드를 생성하고 잠시 기다리면 검증 상태가 **발급됨**으로 변경됩니다.

![acm4](../../images/2022-07-12-lambda-url-redirection/acm4.png)

서브도메인의 인증서를 발급받았습니다.



## Lambda 생성

Lambda는 별도의 서버 없이 코드를 실행할 수 있는 서버리스 컴퓨팅 서비스입니다. 외부에서 온 URL을 분석해 새 URL로 redirection하는 응답을 돌려줄 함수를 작성합니다.

![lambda1](../../images/2022-07-12-lambda-url-redirection/lambda1.png)

함수이름으로 url-redirection을 사용합니다. 예제에서 런타임은 Python을 사용하지만, 다른 언어를 사용해도 무관합니다.

![lambda2](../../images/2022-07-12-lambda-url-redirection/lambda2.png)

함수를 생성 완료했으면, 외부에서 요청을 받아 Lambda에 전달할 API Gateway를 생성합니다. 위 화면의 + 트리거 추가 버튼으로도 추가 할 수 있지만, 저는 경로 설정에서 문제를 겪어 직접 [API Gateway Console](https://console.aws.amazon.com/apigateway/home)에서 생성합니다.

## API Gateway 생성

API Gateway는 개발자가 API를 손쉽게 생성, 게시, 유지 관리, 모니터링 할 수 있는 서비스입니다. 외부 요청으로부터의 '정문'역할을 하며, REST또는 WebSocket형식의 API를 작성 할 수 있습니다.

API Gateway를 생성하고, Route53에 등록된 URL로 오는 요청을 받도록 설정해봅니다.

> **[API Gateway Console](https://console.aws.amazon.com/apigateway) → API 생성 → HTTP API** 로 API 생성 과정에 진입합니다.

**Step1. API생성**  
**통합:** Lambda를 선택하고, 이전에 생성한 Lambda 함수를 선택해줍니다.  
**API이름:** url-redirect-api를 사용합니다.  
![apigateway1](../../images/2022-07-12-lambda-url-redirection/apigateway1.png)

**Step2. 경로 구성**  
리소스 경로에 $default를 입력합니다. 통합 대상의 이름이 생성한 Lambda함수의 이름인지 확인합니다.  
![apigateway2](../../images/2022-07-12-lambda-url-redirection/apigateway2.png)

**Step3. 스테이지 구성**  
특별한 작업 없이 다음으로 넘어갑니다.  
![apigateway3](../../images/2022-07-12-lambda-url-redirection/apigateway3.png)

**Step4. 검토 및 생성**  
API이름, 통합, 경로 내용을 확인합니다.  
![apigateway4](../../images/2022-07-12-lambda-url-redirection/apigateway4.png)

## API Gateway에 사용자 지정 도메인 이름 연결

생성한 API Gateway는 AWS에서 임의의 URL을 할당받습니다. 소유한 도메인과 연결하는 설정을 추가합니다.

1. **API Gateway Console → 사용자 지정 도메인 이름 → 생성** 에 진입합니다.  
    ![apigateway-domain1](../../images/2022-07-12-lambda-url-redirection/apigateway-domain1.png)  
    도메인 이름에 ACM인증서를 받은 도메인명을 입력, ACM인증서에서 발급받은 인증서를 선택하고 도메인 이름을 생성해줍니다.

2. **API Gateway Console → 사용자 지정 도메인 이름 → 좌측의 도메인 이름에서 생성한 도메인 이름 선택 → 우측의 API 매핑 탭 클릭 → API 매핑 구성** 에 진입합니다.  
   ![apigateway-domain2](../../images/2022-07-12-lambda-url-redirection/apigateway-domain2.png)  
   이 API가 사용자 지정 도메인(예제에서는 url.lhy.kr)을 사용하도록 설정했습니다.  
   ![apigateway-domain3](../../images/2022-07-12-lambda-url-redirection/apigateway-domain3.png)

이제 Route53에서 도메인이 이 API와 연결되는 설정을 추가합니다.



## Route53의 도메인과 API Gateway연결

API Gateway의 API가 사용자 지정 도메인을 사용하도록 설정 한 것 과는 별개로, Route53에서 도메인으로의 연결을 정의해야합니다. 레코드를 추가합니다.

![route1](../../images/2022-07-12-lambda-url-redirection/route1.png)

ACM에서 지정했던 서브도메인에 대한 레코드를 생성하며, 라우팅 대상에 별칭을 사용합니다. API Gateway API와 API를 생성했던 리전을 선택하면 엔드포인트 목록이 나타나며, API Gateway Console에서 사용자 정의 도메인을 생성 한 후 나타났던 API Gateway 도메인 이름과 같은지 확인합니다.

레코드를 생성하고 연결되는데는 약간의 시간이 걸립니다. 빠르면 1~2분 내에 연결되며, 연결 완료 시 지정한 도메인으로 접속하면 "Hello from Lambda!"메시지가 출력됩니다.

![route2](../../images/2022-07-12-lambda-url-redirection/route2.png)

## Lambda함수에 redirection설정

Lambda함수로 돌아와보면 **+ 트리거 추가** 버튼 위에 생성한 API Gateway가 연결 된 것을 볼 수 있습니다.

![lambda-function1](../../images/2022-07-12-lambda-url-redirection/lambda-function1.png)

아래쪽 **코드 소스**를 수정합니다.

```python
mapping = {
    '/django': 'https://docs.djangoproject.com/en/dev/',
    '/naver': 'https://naver.com',
}

def lambda_handler(event, context):
    path = event['rawPath']
    location = mapping.get(path, 'https://lhy.kr')
    
    return {
        'statusCode': 302,
        'headers': {'location': location}
    }

```

위 handler함수는 연결된 사용자 지정 도메인에 mapping사전 객체에 정의된 경로가 전달되면 그 값으로, 정의된 값이 없다면 제 블로그로 redirect하는 응답을 돌려줍니다. 코드를 수정하고 코드 위쪽의 Deploy버튼을 눌러 변경사항을 배포해줍니다.

https://url.lhy.kr/django로 접속하면 Django공식문서로 이동하며, https://url.lhy.kr/naver로 접속하면 네이버 메인 페이지로 이동하고, 그 외의 URL이나 경로를 입력하지 않으면 https://lhy.kr(제 블로그)로 연결됩니다.





