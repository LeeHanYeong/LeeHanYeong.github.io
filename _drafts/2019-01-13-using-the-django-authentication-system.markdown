---
layout: post
title:  'Using the Django authentication system'
categories: ['Django', 'Python', '번역'']
---

이 문서는 장고 기본 설정에서의 인증 시스템 사용법을 설명합니다. 이 구성은 가장 일반적인 프로젝트 요구를 처리하고 합리적으로 광범위한 작업을 처리하며 암호와 사용 권한을 신중하게 구현하도록 발전했습니다. Django는 인증 요구 사항이 기본값과 다른 프로젝트의 경우 광범위한 인증 확장 및 사용자 정의를 지원합니다.

Django 인증은 인증과 권한 부여를 함께 제공하며 일반적으로 인증 시스템이라고합니다. 이러한 기능은 다소 결합되어 있기 때문입니다.

## User objects

**[User](https://docs.djangoproject.com/en/dev/ref/contrib/auth/#django.contrib.auth.models.User)** 객체는 인증 시스템의 핵심입니다. 그들은 일반적으로 사이트와 상호 작용하는 사람들을 대표하며 액세스 제한, 사용자 프로필 등록, 제작자와 콘텐츠 연결 등을 가능하게하는 데 사용됩니다. Django의 인증 프레임 워크에서 **['superusers'](https://docs.djangoproject.com/en/dev/ref/contrib/auth/#django.contrib.auth.models.User.is_superuser)**또는 admin **['staff'](https://docs.djangoproject.com/en/dev/ref/contrib/auth/#django.contrib.auth.models.User.is_staff)** 사용자는 특별한 속성이 설정된 사용자 객체일 뿐이며, 다른 사용자 클래스가 아닙니다.

Default User의 기본 속성은 다음과 같습니다.

- username
- password
- email
- first_name
- last_name

전체 참조 정보는 [전체 API 설명서](https://docs.djangoproject.com/en/dev/ref/contrib/auth/#django.contrib.auth.models.User)를 참조하십시오. 이후의 문서는 보다 작업 지향적입니다.

### Creating Users

### Creating superusers

### Changing passwords

### Authenticatin users



## Permissions and Authorization

Django는 간단한 권한 시스템과 함께 제공됩니다. 특정 사용자 및 사용자 그룹에 사용 권한을 할당하는 방법을 제공합니다. 

권한 시스템은 Django 관리자 사이트에서 사용하지만, 사용자 코드에서도 얼마든지 사용 할 수 있습니다.

Django 관리 사이트는 다음과 같이 권한 시스템을 사용합니다. 

- view 객체에 대한 액세스는 해당 유형의 객체에 대한 "view"또는 "change"권한이있는 사용자로 제한됩니다. 
- "add" form view에 접근해서 객체를 추가 할 수 있는 액세스 권한은 해당 객체 유형에 대한 "add"권한이있는 사용자로 제한됩니다. 
- change list view에서, "change" form에 접근해서 객체를 변경하는 액세스는 해당 유형의 객체에 대한 "change" 권한이있는 사용자로 제한됩니다. 
- 객체 삭제 권한은 해당 객체 유형에 대한 "delete"권한이있는 사용자로 제한됩니다.

권한은 객체 유형뿐만 아니라 특정 객체 인스턴스별로 설정할 수도 있습니다. ModelAdmin 클래스에서 제공하는 has_view_permission(), has_add_permission(), has_change_permission() 및 has_delete_permission() 메서드를 사용하면 동일한 유형의 서로 다른 객체 인스턴스에 대한 권한을 사용자 정의 할 수 있습니다.

User 객체는 두 개의 many-to-many 필드를 가집니다: groups와 user_permissions. User 객체는 다른 Django 모델과 같은 방법으로 관련 객체에 액세스 할 수 있습니다.

```python
myuser.groups.set([group_list])
myuser.groups.add(group, group, ...)
myuser.groups.remove(group, group, ...)
myuser.groups.clear()
myuser.user_permissions.set([permission_list])
myuser.user_permissions.add(permission, permission, ...)
myuser.user_permissions.remove(permission, permission, ...)
myuser.user_permissions.clear()
```

### Default permissions

django.contrib.auth가 INSTALLED_APPS 설정에 나열되면, 설치된 응용 프로그램 중 하나에 정의 된 각 Django 모델에 대해 4가지 기본 사용 권한 (추가, 변경, 삭제 및보기)이 생성 되도록 합니다.

이러한 권한은 manage.py migrate를 실행할 때 만들어집니다. django.contrib.auth를 INSTALLED_APPS에 추가 한 후 처음 마이그레이션을 실행하면 이전에 설치된 모든 모델과 그 당시 설치된 모든 새 모델에 대한 기본 사용 권한이 생성됩니다. 그 후에는 manage.py migrate (권한을 생성하는 함수가 post_migrate 신호에 연결됨)를 실행할 때마다 새 모델에 대한 기본 권한을 만듭니다.

app_label foo 및 Bar라는 모델이있는 응용 프로그램이 있다고 가정하면 기본 사용 권한을 테스트하려면 다음을 사용해야합니다.

- add: `user.has_perm('foo.add_bar')`
- change: `user.has_perm('foo.change_bar')`
- delete: `user.has_perm('foo.delete_bar')`
- view: `user.has_perm('foo.view_bar')`

[Permission](https://docs.djangoproject.com/en/dev/ref/contrib/auth/#django.contrib.auth.models.Permission) 모델은 거의 직접 액세스되지 않습니다.

### Groups

django.contrib.auth.models.Group 모델은 사용자를 분류하는 일반적인 방법으로 사용자에게 권한 또는 일부 다른 레이블을 적용 할 수 있습니다. 사용자는 몇 개든 상관없이 여러 그룹에 속할 수 있습니다.

그룹의 사용자에게는 해당 그룹에 부여 된 권한이 자동으로 부여됩니다. 예를 들어 그룹 **Site editors**에 can_edit_home_page 권한이 있으면 해당 그룹의 모든 사용자에게 해당 권한이 부여됩니다.

사용 권한 이외에도 그룹은 사용자에게 레이블 또는 확장 기능을 제공하기 위해 사용자를 분류하는 편리한 방법입니다. 예를 들어 '특수 사용자'그룹을 만들 수 있으며, 회원 전용 사이트에 대한 액세스 권한을 부여하거나 회원 전용 이메일 메시지를 보낼 수있는 코드를 작성할 수 있습니다.

### Progammatically creating permissions

모델의 메타 클래스 내에서 사용자 정의 권한을 정의 할 수 있지만 권한을 직접 작성할 수도 있습니다. 예를 들어 myapp에서 BlogPost 모델에 대한 can_publish 권한을 만들 수 있습니다.

```python
from myapp.models import BlogPost
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType

content_type = ContentType.objects.get_for_model(BlogPost)
permission = Permission.objects.create(
    codename='can_publish',
    name='Can Publish Posts',
    content_type=content_type,
)
```

생성한 권한은 user_permissions 속성을 통해 User에게 할당하거나 permissions 속성을 통해 Group에 할당 할 수 있습니다.

### Permission caching

[ModelBackend](https://docs.djangoproject.com/en/dev/ref/contrib/auth/#django.contrib.auth.backends.ModelBackend)는 권한 검사를 위해 처음으로 가져와야하는 사용자 개체에 대한 사용 권한을 캐시합니다. 권한은 일반적으로 추가 된 직후에 확인되지 않으므로 (예 : 관리자) 요청 - 응답주기에는 일반적으로 적합합니다. 예를 들어, 테스트 나 뷰에서 권한을 추가하고 즉시 확인하는 경우 가장 쉬운 방법은 데이터베이스에서 사용자를 다시 가져 오는 것입니다. 예 :

```python
from django.contrib.auth.models import Permission, User
from django.contrib.contenttypes.models import ContentType
from django.shortcuts import get_object_or_404

from myapp.models import BlogPost

def user_gains_perms(request, user_id):
    user = get_object_or_404(User, pk=user_id)
    # any permission check will cache the current set of permissions
    user.has_perm('myapp.change_blogpost')

    content_type = ContentType.objects.get_for_model(BlogPost)
    permission = Permission.objects.get(
        codename='change_blogpost',
        content_type=content_type,
    )
    user.user_permissions.add(permission)

    # Checking the cached permission set
    user.has_perm('myapp.change_blogpost')  # False

    # Request new instance of User
    # Be aware that user.refresh_from_db() won't clear the cache.
    user = get_object_or_404(User, pk=user_id)

    # Permission cache is repopulated from the database
    user.has_perm('myapp.change_blogpost')  # True

    ...
```

## Authentication in Web requests

Django는 세션과 미들웨어를 사용하여 인증 시스템을 request 객체에 연결합니다.

이들은 현재 사용자를 나타내는 모든 요청에 대해 request.user 속성을 제공합니다. 현재 사용자가 로그인하지 않은 경우이 속성은 AnonymousUser의 인스턴스로 설정되고, 그렇지 않으면 User의 인스턴스가됩니다.

is_authenticated를 사용하면 다음과 같이 구분할 수 있습니다.

```python
if request.user.is_authenticated:
    # Do something for authenticated users.
    ...
else:
    # Do something for anonymous users.
    ...

```

### How to log a user in

#### Selecting the authentication backend

사용자가 로그인하면 사용자의 ID와 인증에 사용 된 백엔드가 사용자의 세션에 저장됩니다. 이를 통해 동일한 인증 백엔드가 이후 요청시 사용자 세부 정보를 가져올 수 있습니다. 세션에 저장할 인증 백엔드는 다음과 같이 선택됩니다.

1. 제공되는 경우 선택적 **backend** 인수의 값을 사용하십시오.
2. user.backend 속성의 값을 사용하십시오 (있는 경우). 이렇게하면 authenticate() 및 login()을 쌍으로 지정(pairing)할 수 있습니다. authenticate()는 반환하는 사용자 객체에 user.backend 특성을 설정합니다.
3. 백엔드가 하나만 있는 경우 AUTHENTICATION_BACKENDS를 사용하십시오.
4. 그렇지 않으면 예외를 발생시킵니다.

### How to log a user out

### Limiting access to logged-in users

#### The raw way

#### The `login_required` decorator

#### The LoginRequired mixin

#### Limiting access to logged-in users that pass a test

특정 권한 또는 다른 테스트를 기반으로 액세스를 제한하려면 이전 섹션에서 설명한 것과 동일한 작업을 수행해야합니다.

간단한 방법은 뷰에서 request.user에 대한 테스트를 직접 실행하는 것입니다. 예를 들어, 이 보기는 사용자가 원하는 도메인에 전자 메일이 있는지 확인하고 그렇지 않은 경우 로그인 페이지로 리디렉션합니다.

```python
from django.shortcuts import redirect

def my_view(request):
    if not request.user.email.endswith('@example.com'):
        return redirect('/login/?next=%s' % request.path)
    # ...
```

##### `user_passes_test`(*test_func*, *login_url=None*, *redirect_field_name='next'*)[[source\]](https://docs.djangoproject.com/en/dev/_modules/django/contrib/auth/decorators/#user_passes_test)[¶](https://docs.djangoproject.com/en/dev/topics/auth/default/#django.contrib.auth.decorators.user_passes_test)

shortcut으로서 callable이 False를 반환 할 때 리디렉션을 수행하는 편리한 user_passes_test 데코레이터를 사용할 수 있습니다.

```python
from django.contrib.auth.decorators import user_passes_test

def email_check(user):
    return user.email.endswith('@example.com')

@user_passes_test(email_check)
def my_view(request):
    ...
```

user_passes_test()는 필수 인수를 취합니다:

- User 객체를 사용하고 사용자가 페이지를 볼 수 있으면 True를 반환하는 호출 가능 객체입니다.
- user_passes_test()는 User가 익명(anonymous)이 아닌지 자동으로 확인하지 않습니다.

##### `class UsersPassesTestMixin`

클래스 기반 뷰를 사용할 때 UserPassesTestMixin을 사용하여 이를 수행 할 수 있습니다.

`test_func()`

수행 할 테스트를 제공하려면 클래스의 test_func () 메소드를 대체해야합니다. 또한 [AccessMixin](https://docs.djangoproject.com/en/dev/topics/auth/default/#django.contrib.auth.mixins.AccessMixin)의 매개 변수를 설정하여 권한이없는 사용자의 처리를 사용자 정의 할 수 있습니다.

```python
from django.contrib.auth.mixins import UserPassesTestMixin

class MyView(UserPassesTestMixin, View):

    def test_func(self):
        return self.request.user.email.endswith('@example.com')
```

`get_test_func()`

또한 get_test_func() 메소드를 오버라이드하여 mixin이 test_func() 대신 다른 이름의 함수를 사용하도록 할 수있다.

>  Stacking `UserPassesTestMixin`
>
> UserPassesTestMixin이 구현된 방식으로 인해 상속 목록에 스택을 쌓을 수 없습니다. 다음은 작동하지 않습니다.
>
> ```python
> class TestMixin1(UserPassesTestMixin):
>     def test_func(self):
>         return self.request.user.email.endswith('@example.com')
> 
> class TestMixin2(UserPassesTestMixin):
>     def test_func(self):
>         return self.request.user.username.startswith('django')
> 
> class MyView(TestMixin1, TestMixin2, View):
>     ...
> ```
>
> TestMixin1이 super()를 호출하고 그 결과를 고려하면 TestMixin1은 더 이상 독립적으로 작동하지 않습니다.

#### The `permission_required` decorator

**`permission_required(perm, login_url=None, raise_exception=False)`**

사용자에게 특정 권한이 있는지 여부를 확인하는 것은 비교적 일반적인 작업입니다. 이런 이유로 Django는 그 케이스에 대한 바로 가기를 제공합니다 : permission_required() 데코레이터 :

```python
from django.contrib.auth.decorators import permission_required

@permission_required('polls.can_vote')
def my_view(request):
    ...

```

has_perm() 메소드와 마찬가지로 권한 이름은 **"\<app label\>.\<permission codename\>"**형식을 사용합니다 (polls 애플리케이션에서 모델에 대한 권한을 부여하려면 polls.can_vote).

데코레이터는 또한 반복 가능한 권한을 가질 수 있습니다.이 경우 view에 액세스하려면 사용자에게 모든 권한이 있어야합니다.

**permission\_required()**는 선택적 **login\_url** 매개 변수를 취합니다.

```python
from django.contrib.auth.decorators import permission_required

@permission_required('polls.can_vote', login_url='/loginpage/')
def my_view(request):
    ...

```

**[login\_required()](https://docs.djangoproject.com/en/dev/topics/auth/default/#django.contrib.auth.decorators.login_required)** 데코레이터에서와 같이 login_url의 기본값은 settings.LOGIN_URL입니다.

raise_exception 매개 변수가 제공되면 데코레이터는 PermissionDenied를 발생시켜 로그인 페이지로 리디렉션하지 않고 403 (HTTP 금지됨)보기를 표시합니다.

raise_exception을 사용하고 싶지만 사용자가 먼저 로그인 할 수있게하려면 login_required() 데코레이터를 추가하면됩니다.

```python
from django.contrib.auth.decorators import login_required, permission_required

@login_required
@permission_required('polls.can_vote', raise_exception=True)
def my_view(request):
    ...
```

이렇게하면 **[LoginView](https://docs.djangoproject.com/en/dev/topics/auth/default/#django.contrib.auth.views.LoginView)**의 **redirect\_authenticated\_user = True**이고 로그인 한 사용자에게 필요한 권한이 모두 없을 때 리디렉션 루프가 발생하지 않습니다.

#### The PermissionRequiredMixin mixin

클래스 기반 뷰에 권한 검사를 적용하려면 PermissionRequiredMixin :

**class PermissionRequireMixin**

이 mixin은 permission_required 데코레이터와 마찬가지로 뷰에 액세스하는 사용자에게 주어진 권한이 있는지 여부를 확인합니다. permission_required 매개 변수를 사용하여 사용 권한 (또는 사용 권한의 반복 가능)을 지정해야합니다.

```python
from django.contrib.auth.mixins import PermissionRequiredMixin

class MyView(PermissionRequiredMixin, View):
    permission_required = 'polls.can_vote'
    # Or multiple of permissions:
    permission_required = ('polls.can_open', 'polls.can_edit')
```

또한 AccessMixin의 매개 변수를 설정하여 권한이없는 사용자의 처리를 사용자 정의 할 수 있습니다.

다음 방법을 재정의 할 수도 있습니다.

**get\_permission\_required()**

mixin에 의해 사용되는 액세스권의 반복 가능한 것을 돌려줍니다. **permission\_required** 속성의 기본값이며, 필요한 경우 튜플로 변환됩니다.

**has\_permissions()**

현재 사용자에게 데코레이팅 된 view를 실행할 권한이 있는지 여부를 나타내는 bool값을 반환합니다. 기본적으로 이 메서드는 **get\_permission\_required()**에 의해 반환 된 사용 권한 목록과 함께 **has\_perms()**를 호출 한 결과를 반환합니다.

### Redirecting unauthorized requests in class-based-views

클래스 기반 뷰에서 액세스 제한을 쉽게 처리 할 수 있도록 AccessMixin을 사용하여 액세스가 거부 될 때보기의 동작을 구성 할 수 있습니다. 인증 된 사용자는 HTTP 403 Forbidden 응답으로 액세스가 거부됩니다. 익명 사용자는 raise_exception 속성에 따라 로그인 페이지로 리디렉션되거나 HTTP 403 Forbidden 응답을 표시합니다.

> **Changed in Django 2.1:**
>
> 이전 버전에서는 권한이없는 인증 된 사용자가 HTTP 403 Forbidden 응답을받는 대신 로그인 페이지로 리디렉션되었습니다 (루프가 발생 함).

##### class AccessMixin

**login\_url**

**get\_login\_url()**의 기본 반환 값입니다. 기본값은 None이며,이 경우 **get\_login\_url()**은 settings.LOGIN_URL로 돌아갑니다.

**permission\_denied\_message**

**get\_permission\_denied\_message()**의 기본 반환 값입니다. 기본값은 빈 문자열입니다.

...

...



#### Session invalidation on password change

AUTH_USER_MODEL이 AbstractBaseUser에서 상속 받거나 자체 get_session_auth_hash () 메소드를 구현하면 인증 된 세션에 이 함수가 반환한 해시가 포함됩니다. AbstractBaseUser의 경우 이것은 패스워드 필드의 HMAC입니다. Django는 각 request에 대한 세션의 해시가 request 중에 계산된 해시와 일치하는지 확인합니다. 이를 통해 사용자는 암호를 변경하여 모든 세션을 로그아웃 할 수 있습니다.

장고에는 기본적으로 탑재되어 있는 비밀번호 변경 view들이 있습니다. django.contrib.auth admin에 포함된 PasswordChangeView와 user\_change\_password view입니다. 

이들은 새 비밀번호 해시로 세션을 업데이트하므로 사용자가 자신의 비밀번호를 변경하더라도 자신을 로그 아웃하지 않습니다. 사용자 정의 암호 변경보기가 있고 유사한 동작을 원한다면 **update\_session\_auth\_hash()** 함수를 사용하십시오.

**update\_session\_auth\_hash(request, user)**

이 함수는 현재 request와 새 세션 해시가 파생 될 업데이트 된 사용자 개체를 가져 와서 세션 해시를 적절하게 업데이트합니다. 또한 도난당한 세션 쿠키가 무효화되도록 세션 키를 회전합니다.

Example usage:

```python
from django.contrib.auth import update_session_auth_hash

def password_change(request):
    if request.method == 'POST':
        form = PasswordChangeForm(user=request.user, data=request.POST)
        if form.is_valid():
            form.save()
            update_session_auth_hash(request, form.user)
    else:
        ...
```

> Note
>
> get\_session\_auth\_hash()는 SECRET_KEY를 기반으로하므로 새 secret을 사용하도록 사이트를 업데이트하면 기존 세션이 모두 무효화됩니다.

### Authentication Views

#### Using the views

#### All authentication views

### Helper functions

### Built-in forms

### Authentication data in templates

#### Users

#### Permissions

### Managing users in the admin

####Creating users

####Changin passwords 