---
layout: post
title:  "POST메서드를 처리하는 view에 대한 login_required 데코레이터 동작 변경"
categories: ['Django']
---

**참고문서**
- [login_required](https://docs.djangoproject.com/ko/1.11/topics/auth/default/#django.contrib.auth.decorators.login_required)
- [request.META['HTTP_REFERER']](https://docs.djangoproject.com/en/1.11/ref/request-response/#django.http.HttpRequest.META)

---

`Django`가 `auth`애플리케이션에서 제공하는 `login_required()`데코레이터는 특정 뷰에 요청시 사용자 인증이 되어있지 않은 경우, 로그인 페이지로 리다이리렉트 응답을 보내면서 `next`파라미터에 기존 `view`의 주소를 보내주어 로그인 페이지에 해당하는 뷰에서 로그인 완료 후 기존에 요청한 `URL`로 리다이렉트 처리를 도와주는 역할을 한다.

이 때, 댓글을 단다던가 하는 `POST`메서드 요청에 대해서만 응답을 제공하는 뷰의 경우에는 로그인 완료 후 `HttpResponse`를 보내주지 않으며, 실제로는 `POST`요청을 처리하는 뷰가 호출되기 이전의 뷰로 되돌아가야 하나 `login_required()` 데코레이터는 이러한 응답을 처리하지 못한다.

이러한 요청을 매번 해당 뷰에서 처리하는게 번거로워 `POST`요청시 로그인 후 `next`값을 `request`의 `HTTP_REFERER`의 값으로 지정하는 커스텀 데코레이터를 만들어 사용했다.

## 이슈 상황

<center><b>인스타그램 프로젝트</b></center>

![index]({{ site.url }}/images/django-post-decorator/index.png)

<center>
  <b>비로그인 상태에서 글 작성 화면으로 접근하려고 할 경우</b>
  <div>`next`파라미터의 값이 글 작성 (`/post/create/`)임을 볼 수 있다</div>
</center>

![login_post_create]({{ site.url }}/images/django-post-decorator/login_post_create.png)

<center>
  <b>비로그인 상태에서 댓글 작성 요청 (POST)을 한 경우</b>
</center>

![comment]({{ site.url }}/images/django-post-decorator/comment.png)

<center>
  <b>`next`파라미터의 값이 `POST`요청만 처리 가능한 댓글 작성 (`/post/5/comment/create/`)임을 볼 수 있다</b>
</center>

![next_invalid]({{ site.url }}/images/django-post-decorator/next_invalid.png)

<center><b>이 상태로 로그인을 실행하면, `next`값의 URL로 리다이렉트되며 <br>댓글 작성 뷰는 `GET`요청을 처리하지 못하므로 에러를 출력한다</b></center>

![post_view]({{ site.url }}/images/django-post-decorator/post_view.png)

## POST요청시 `HTTP_REFERER`를 사용하는 데코레이터 구현

```python
from functools import wraps
from urllib.parse import urlparse

from django.contrib.auth.decorators import login_required as django_login_required
from django.contrib.auth.views import redirect_to_login
from django.core.handlers.wsgi import WSGIRequest


def login_required(view_func):
    @wraps(view_func)
    def decorator(*args, **kwargs):
        if args:
            request = args[0]
            # Django의 view에서 첫 번째 매개변수는 HttpRequest타입의 변수이며, 이를 확인한다
            # 또한 요청 메서드가 POST인지 확인한다
            if isinstance(request, WSGIRequest) and request.method == 'POST':
                # request의 user가 존재하며 인증되어있는지 확인한다
                user = getattr(request, 'user')
                if user and user.is_authenticated:
                    return view_func(*args, **kwargs)
                # 인증되지 않았을 경우, HTTP_REFERER의 path를 가져온다
                path = urlparse(request.META['HTTP_REFERER']).path
                # 로그인 뷰로 이동하며 GET파라미터의 next값을 path로 지정해주는
                # redirect_to_login함수를 되돌려준다
                return redirect_to_login(path)
        # 위에 해당하지 않는 경우, Django에서 제공하는 기본 login_required를 데코레이터로 사용한다
        return django_login_required(view_func)(*args, **kwargs)
    return decorator
```

위 데코레이터를 기본 `login_required()`대신 사용하면 `POST`요청만 처리하는 뷰의 경우 로그인페이지의 `next`파라미터에 이전 페이지로 돌아가는 URL이 생성된다.