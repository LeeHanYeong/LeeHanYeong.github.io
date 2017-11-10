---
layout: post
title:  "클린 코드를 위한 테스트 주도 개발"
categories: ['TDD']
---

이 포스팅에서는 클린 코드를 위한 테스트 주도 개발 [e-book](http://chimera.labs.oreilly.com/books/1234000000754/index.html){:target="_blank"}을 진행하며 책과 최신버전에서의 차이점이나, 설명이 생략되어 있는 부분들에 대해 기술한다.  
[한국어 번역 책](http://books.11st.co.kr/product/SellerProductDetail.tmall?method=getSellerProductDetail&prdNo=1225272081&trTypeCd=9p&trCtgrNo=63517){:target="_blank"}

---

## 시작 전 필요사항
이 책에서 진행하는 TDD프로젝트는 `pyenv`와 `pyenv-virtualenv`를 사용하는 환경에서 진행했다.  
자세한 내용은 [pyenv에 대한 포스팅](https://lhy.kr/configuring-the-python-development-environment-with-pyenv-and-virtualenv)을 참조한다.

프로젝트는 홈 폴더의 `projects`폴더에 `tdd`폴더를 만들고, 그 내부에 가상환경을 생성한 후 진행한다. 파이썬은 `3.6.1`버전을 사용한다.

```
➜ cd ~/
➜ mkdir projects
➜ cd projects
➜ mkdir tdd
➜ cd tdd
➜ pyenv virtualenv 3.6.1 tdd
➜ pyenv local tdd
```

이후 `pip`를 이용해 `django`, `selenium`을 설치하고, `brew`를 이용해 `Chromedriver`를 설치한다. `django`는 `1.11.x`버전을 사용한다.

```
(tdd) ➜ pip install 'django<1.12'
(tdd) ➜ pip install selenium
(tdd) ➜ brew install chromedriver
```

---

# Chapter01

## 첫 번째 functional_tests.py

그리고 첫 번째`functional_tests.py`는 아래와 같이 `FireFox`를 `Chrome`으로 변경한다. 본인은 파일이름이 너무 길다고 생각해 앞으로 `ft.py`파일명을 사용한다.

```python
# ft.py
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('http://localhost:8000')

assert 'Django' in browser.title
```

`pyenv`를 사용할 경우, `python3 manage.py ...`라고 실행할 필요없이 `python manage.py ...`또는 `./manage.py ...`로 같은 명령을 실행 가능하다.


## Starting Git Repository
책에서는 `.gitignore`파일을 생성하면서 `git`에서 파일을 추가하고 `unstage`로 바꾸는 여러 작업을 한다. 헷갈릴 수 있으니 `.gitignore`파일은 [gitignore.io(macOS, Python, Pycharm, Django, Linux)](https://www.gitignore.io/api/macos%2Clinux%2Cdjango%2Cpython%2Cpycharm)의 내용을 그대로 사용하며, 이 파일의 맨 상단에 `.idea/`를 추가해준다 (`Pycharm`에서 사용하는 폴더명)  
[생성될 .gitignore의 내용](https://gist.github.com/LeeHanYeong/463837a5c5338fa745c0f3b9b2fba5f0)

이후 `git`명령어는 아래와 같이 실행한다.

```
git init
git add -A
git commit -m 'First commit: First FT and basic Django config'
```


## 그 외
이후 `webdriver.Firefox()`를 사용하는 부분들은 전부 `webdriver.Chrome()`을 사용한다.

---

# Chapter02
이 챕터에는 특별한 추가 요소가 없다

---

# Chapter03
## superlists/urls.py에 내용을 추가한 이후

```python
from django.conf.urls import url

from lists import views

urlpatterns = [
    url(r'^$', views.home_page, name='home'),
    # url(r'^admin/', include(admin.site.urls)),
]
```
위 내용을 추가한 후 `./manage.py test`를 실행했을 때의 결과가 다르다.

```
# 책
[...]
AttributeError: 'NoneType' object has no attribute 'rindex'

# 실제 결과
[...]
TypeError: view must be a callable or a list/tuple in the case of include().
```

---

# Chapter04
이 챕터에는 특별한 추가 요소가 없다

---

# Chapter05
## Wiring Up Our Form to Send a POST Request
> list/templates/home.html을 수정한 후

{% raw %}
`{% csrf_token %}`을 넣은 후, `test_home_page_returns_correct_html()`에서 `AssertionError`가 발생한다.
{% endraw %}

```
FAIL: test_home_page_returns_correct_html (lists.tests.HomePageTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/Users/lhy/projects/tdd/superlists/lists/tests.py", line 17, in test_home_page_returns_correct_html
    self.assertEqual(response.content.decode(), expected_html)
AssertionError: '<htm[134 chars]t\t\t<input type=\'hidden\' name=\'csrfmiddlew[259 chars]tml>' != '<htm[134 chars]t\t\t\n\t\t\t<input type="text" name="item_tex[130 chars]tml>'
```

{% raw %}
이 오류는 `{% csrf_token %}`에 의해 생성된 `input`요소의 값이 매번 달라서 발생하며, 해당 부분을 정규식으로 삭제처리해주면 정상작동한다.
{% endraw %}

```python
# lists/tests.py

class HomePageTest(TestCase):
    pattern_input_csrf = re.compile(r'<input[^>]*csrfmiddlewaretoken[^>]*>')
    
    ...
    
    def test_home_page_returns_correct_html(self):
      request = HttpRequest()
      response = home_page(request)
      expected_html = render_to_string('home.html')
      self.assertEqual(
          re.sub(self.pattern_input_csrf, '', response.content.decode()),
          re.sub(self.pattern_input_csrf, '', expected_html)
      )
```


## Passing Python Variables to Be Rendered in the Template
> list/tests.py에서 assertEqual로 비교할 때

{% raw %}
위와 같은 오류이며, 역시 정규식을 이용해 `{% csrf_token %}`에 의해 생성된 `input`요소를 삭제해준다.
{% endraw %}

```python
# lists/tests.py
class HomePageTest(TestCase):
  ...
  
  def test_home_page_can_save_a_POST_request(self):
    request = HttpRequest()
    request.method = 'POST'
    request.POST['item_text'] = 'A new list item'

    response = home_page(request)

    self.assertIn('A new list item', response.content.decode())
    expected_html = render_to_string(
        'home.html',
        {
            'new_item_text': 'A new list item',
        }
    )
    self.assertEqual(
        re.sub(self.pattern_input_csrf, '', response.content.decode()),
        re.sub(self.pattern_input_csrf, '', expected_html)
    )
```
