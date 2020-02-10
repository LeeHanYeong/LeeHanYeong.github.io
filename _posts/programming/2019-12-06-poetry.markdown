---
layout: post
title:  'Django admin에서 모델의 property에 short_description 설정하기'
categories: ['Django', 'Python']
---

- [Python공식문서 (property)](https://docs.python.org/ko/3.7/library/functions.html#property)
- [Django공식문서 (ModelAdmin.readonly_fields)](https://docs.djangoproject.com/en/dev/ref/contrib/admin/#django.contrib.admin.ModelAdmin.readonly_fields)
  - 메서드를 사용해 label을 설정하는 방법에 대해 설명되어 있습니다
- [Django공식문서 (ModelAdmin.list_display)](https://docs.djangoproject.com/en/dev/ref/contrib/admin/#django.contrib.admin.ModelAdmin.list_display)
  - property에 short_description을 적용하는 방법이 설명되어 있습니다

Django의 내장 패키지인 admin에서는, Model의 필드가 아닌 메서드를 admin field에 출력하는 기능을 제공합니다. 출력되는 내용의 label은 기본적으로 해당 메서드의 이름을 따르며, 이를 커스터마이징 할 때는 해당 메서드에 `short_description`이라는 속성을 할당합니다.

Model에 property를 설정한 경우, admin에서 해당 property속성을 출력하는 데 에는 문제가 없으나, property에 직접 `short_description`속성을 할당할 수 없습니다. 이번 포스팅에서는 이를 해결한 사례를 설명합니다.

> property에 관한 기초, Django admin의 short_description에 대한 설명은 생략합니다. 상단의 공식문서 링크들을 참고해주세요.

## property.fget

[Python공식문서 (property)](https://docs.python.org/ko/3.7/library/functions.html#property) 에서, 반환 된 property객체는 생성자 인자에 해당하는 `fget`, `fset`및 `fdel` 속성을 가지며, 클래스 속성으로 접근 가능하다는 설명이 있습니다.

```python
# C.x는 property객체
>>> C.x
<property at 0x106702c78>
```

```python
# C.x.fget은 property의 getter함수
>>> C.x.fget
<function __main__.C.x(self)>
```

클래스 속성으로 갖고 있는 `fget` 속성은 메서드(함수)이며, 인스턴스 메서드이기 때문에 클래스에서 호출하면 에러가 발생합니다.

```python
# 클래스에서 property의 getter함수를 호출한 경우
>>> C.x.fget()
TypeError: x() missing 1 required positional argument: 'self'
```

여기에서 C의 property인 `x`의 `getter`역할을 하는 함수는 `C.x.fget` 입니다. 이 `getter`함수에 속성을 적용한다면, 아래와 같은 코드를 사용합니다.

```python
class C:
    def __init__(self):
        self._x = None

    @property
    def x(self):
        """I'm the 'x' property."""
        return self._x
    
    # x property의 getter함수에 속성 설정
    x.fget.description = "I'm the 'x' getter description"
```

```python
# property의 getter함수의 description속성
>>> C.x.fget.description
"I'm the 'x' getter description"
```

`x.fget`속성은 오직 클래스에서만 접근 가능하며, C의 인스턴스에서는 `x` 속성에 접근 시 함수대신 함수의 return값이 반환되기 때문에 사용할 수 없습니다.

```python
# x속성에 접근 시 getter함수에서 리턴하는 객체가 반환
>>> c = C()
>>> c.x = "C instance's attribute x"
>>> c.x
"C instance's attribute x"

# 리턴된 객체는 'str'이기 때문에 getter함수의 속성에는 접근 불가능
>>> c.x.description
AttributeError: 'str' object has no attribute 'description'
```

여기까지 보면 해당 속성을 언제 사용할 지 애매한데, Django의 admin애플리케이션 코드에 이 속성을 활용하는 부분이 있습니다.



## django.contrib.admin.utils (line. 368)

```python
def label_for_field(name, model, model_admin=None, return_attr=False):
    """
    Returns a sensible label for a field name. The name can be a callable,
    property (but not created with @property decorator) or the name of an
    object's attribute, as well as a genuine fields. If return_attr is
    True, the resolved attribute (which could be a callable) is also returned.
    This will be None if (and only if) the name refers to a field.
    """
    ...
    
    try:
        field = _get_non_gfk_field(model._meta, name)
        try:
            label = field.verbose_name
        except AttributeError:
            # field is likely a ForeignObjectRel
            label = field.related_model._meta.verbose_name
    except FieldDoesNotExist:
        if hasattr(attr, "short_description"):
            label = attr.short_description
        elif (isinstance(attr, property) and
              hasattr(attr, "fget") and
              hasattr(attr.fget, "short_description")):
            label = attr.fget.short_description
        elif callable(attr):
            if attr.__name__ == "<lambda>":
                label = "--"
            else:
                label = pretty_name(attr.__name__)
        else:
            label = pretty_name(name)
```

위는 admin에서 field의 label을 얻어내는 함수입니다. 사용하려 하는 field가 출력하려는 Model 인스턴스에 존재하지 않는 경우, 아래 순서대로 label을 추출해냅니다.

- `<Model instance>.<field name>.short_description`
- `<Model instance>.<field name(property)>.fget.short_description`
- callable하나 lambda(익명함수)인 경우, label을 할당하지 않음 ('--'를 사용, 함수명이 없기때문으로 유추)
- callable한 함수인 경우, 함수의 이름을 사용

여기서 2번째 경우인 인스턴스의 속성이 존재하며 property인 경우, 해당 property객체의 `fget.short_description`을 사용합니다.



## Django admin example

위 코드에 의해, Django admin에서 이 속성을 아래와 같이 활용할 수 있습니다.

**`models.py`**

```python
class Contract(models.Model):
    _contract_date_number = models.PositiveSmallIntegerField(default=0)
    
    @property
    def contract_number(self):
        return '{date}-{number}'.format(
            date=self.created_on.strftime('%Y%m%d'),
            number=self._contract_date_number,
        )

    contract_number.fget.short_description = '계약등록번호'
```



**`admin.py`**

```python
class ContractAdmin(admin.ModelAdmin):
    list_display = ('contract_number',)
    readonly_fields = ('contract_number',)
	fieldsets = (
        ('계약서 정보', {
            'fields': ('contract_number',)
        }),
    )
```

출력

![admin]({{ site.url }}/images/django-admin-property.png)



## 마치며

property.fget으로 short_description을 설정하는 방법은 Django문서에 나와있지 않습니다. 이 부분에 대해 [pull request](<https://github.com/django/django/pull/11191>)를 보내놓은 상태이며, 적용된다면 이 글에 추가 예정입니다.

