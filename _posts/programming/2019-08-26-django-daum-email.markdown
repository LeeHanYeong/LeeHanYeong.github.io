---
layout: post
title:  "Django에서 Daum메일을 사용한 SMTP설정"
categories: ['Django', 'Python']
---

대부분의 예제가 Gmail을 사용하는 것인데, 개인적으로 Daum메일을 연동해야 할 일이 생겨 기록을 남겨놓습니다

### 설정

메일에서 IMAP사용 설정후, 아래 설정을 `settings.py`에 추가합니다.  
`EMAIL_PORT = 465` 설정이 필수입니다

```python
EMAIL_HOST = 'smtp.daum.net'
EMAIL_HOST_USER = '<Daum ID>'
EMAIL_HOST_PASSWORD = '<Daum Password>'
EMAIL_USE_SSL = True
EMAIL_PORT = 465
```



### 기록

다음 메일의 사용설명에는 외부 클라이언트와 IMAP/SMTP연동시, 반드시 SSL을 사용하도록 설명되어있습니다.

Django의 Settings Reference의 [EMAIL_USE_SSL](<https://docs.djangoproject.com/en/2.2/ref/settings/#email-use-ssl>) 항목을 True로 설정할 경우, SMTP서버와 보안 연결을 사용하며, 일반적으로 465번 포트를 사용한다고 설명되어 있습니다.

허나 EMAIL_PORT설정 없이 이메일 전송을 시도하면 아래와 같은 오류를 보게됩니다

```python
Internal Server Error: /password-reset/
Traceback (most recent call last):
  File "/python3.7/site-packages/django/core/handlers/exception.py", line 34, in inner
    response = get_response(request)
  File "/python3.7/site-packages/django/core/handlers/base.py", line 115, in _get_response
    response = self.process_exception_by_middleware(e, request)
  File "/python3.7/site-packages/django/core/handlers/base.py", line 113, in _get_response
    response = wrapped_callback(request, *callback_args, **callback_kwargs)
  File "/python3.7/site-packages/django/views/generic/base.py", line 71, in view
    return self.dispatch(request, *args, **kwargs)
  File "/python3.7/site-packages/django/utils/decorators.py", line 45, in _wrapper
    return bound_method(*args, **kwargs)
  File "/python3.7/site-packages/django/utils/decorators.py", line 142, in _wrapped_view
    response = view_func(request, *args, **kwargs)
  File "/python3.7/site-packages/django/contrib/auth/views.py", line 220, in dispatch
    return super().dispatch(*args, **kwargs)
  File "/python3.7/site-packages/django/views/generic/base.py", line 97, in dispatch
    return handler(request, *args, **kwargs)
  File "/python3.7/site-packages/django/views/generic/edit.py", line 142, in post
    return self.form_valid(form)
  File "/python3.7/site-packages/django/contrib/auth/views.py", line 233, in form_valid
    form.save(**opts)
  File "/python3.7/site-packages/django/contrib/auth/forms.py", line 295, in save
    email, html_email_template_name=html_email_template_name,
  File "/python3.7/site-packages/django/contrib/auth/forms.py", line 250, in send_mail
    email_message.send()
  File "/python3.7/site-packages/django/core/mail/message.py", line 291, in send
    return self.get_connection(fail_silently).send_messages([self])
  File "/python3.7/site-packages/django/core/mail/backends/smtp.py", line 103, in send_messages
    new_conn_created = self.open()
  File "/python3.7/site-packages/django/core/mail/backends/smtp.py", line 63, in open
    self.connection = self.connection_class(self.host, self.port, **connection_params)
  File "/python3.7/smtplib.py", line 1031, in __init__
    source_address)
  File "/python3.7/smtplib.py", line 251, in __init__
    (code, msg) = self.connect(host, port)
  File "/python3.7/smtplib.py", line 336, in connect
    self.sock = self._get_socket(host, port, self.timeout)
  File "/python3.7/smtplib.py", line 1037, in _get_socket
    self.source_address)
  File "/python3.7/socket.py", line 727, in create_connection
    raise err
  File "/python3.7/socket.py", line 716, in create_connection
    sock.connect(sa)
TimeoutError: [Errno 60] Operation timed out
```

에러 로그를 보면 smtplib.py의 소켓 커넥션에서 에러가 발생하며, 이 부분에서 디버거를 실행해봅니다.

![port25]({{ site.url }}/images/django-daum-email/port25.png)

EMAIL_USE_SSL의 설명에서 일반적으로 사용한다하여  당연히 465번 포트를 사용해 줄 것으로 생각했으나, 해당 설정과는 별개로 EMAIL_PORT를 설정하지 않으면 기본 포트인 25번을 사용합니다. ([Settings Reference - EMAIL_PORT](<https://docs.djangoproject.com/en/2.2/ref/settings/#email-port>))



`EMAIL_PORT = 465` 설정을 추가 한 후에는 465번 포트를 사용하며, 메일도 정상적으로 전송됩니다

![port465]({{ site.url }}/images/django-daum-email/port465.png)



