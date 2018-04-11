---
layout: post
title:  "DRF에서 CamelCase JSON을 적용 및 테스트에 참고할 점"
categories: ['Python', 'Django', 'DRF']
---

DRF(Django REST framework)의 파싱과 렌더링은 기본값으로 JSON형식을 사용하나, 키 값에 파이썬 변수 명명규칙인 `lower_case_with_underscores(Snake Case)`를 사용합니다.

DRF의 서드파티 파서 라이브러리인 [DRF JSON CamelCase](https://github.com/vbabiy/djangorestframework-camel-case)를 사용하면, 클라이언트측에서 CamelCase로 요청과 응답을 처리할 수 있습니다.

---

## 기본 설정

### 패키지 설치 및 파서, 렌더러 설정

설치법은 GitHub저장소에서 확인할 수 있습니다. [https://github.com/vbabiy/djangorestframework-camel-case](https://github.com/vbabiy/djangorestframework-camel-case)

`settings.py`에서 DRF관련 내용을 아래와 같이 설정합니다.

```python
REST_FRAMEWORK = {
    # CamelCaseJSON관련 설정
    'DEFAULT_PARSER_CLASSES': (
        'djangorestframework_camel_case.parser.CamelCaseJSONParser',
        'rest_framework.parsers.JSONParser',
        'rest_framework.parsers.MultiPartParser',
    ),
    'DEFAULT_RENDERER_CLASSES': (
        'djangorestframework_camel_case.render.CamelCaseJSONRenderer',
    ),
}
```


### 응답 확인

응답결과가 아래와 같이 CamelCase처리된 키의 JSON형식임을 확인합니다.

```json
{
    "user": {
        "pk": 3,
        "name": "이한영",
        "phoneNumber": "010-1234-1234",
        "email": "dev3@lhy.kr",
        "imgProfile": null
    },
    "token": "563ee9f243d8b79c13a7d83ca0ae0492a0eee2e3"
}
```

## 테스트에서의 문제

DRF에서의 [테스팅 문서](http://www.django-rest-framework.org/api-guide/testing/#testing-responses)를 보면, 완전히 렌더링 된 응답보다는 응답이 생성되는데 사용된 `data`속성을 비교하는것이 좀 더 편리하다고 알려줍니다.

하지만 위 방식을 테스트 코드에서 사용 시, `response.data`의 키 값들에는 CamelCase가 미적용 상태입니다.

```
proj ❯ ./manage.py test members
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..{'user': {'pk': 1, 'name': 'TestName', 'phone_number': '010-1234-1234', 'email': 'TestEmail@test.com', 'img_profile': None}, 'token': '89bbbc613a84bfdd908a0446a00c1d25425a3fee'}
.
```

따라서 테스트 코드에서 응답을 검사해야 할 경우, `response.data`대신 `response.content`를 파싱한 결과를 사용해야 합니다.

```python
response_data = json.loads(response.content)
self.assertEqual(response_data['user']['phoneNumber'], self.TEST_PHONE_NUMBER)
```
