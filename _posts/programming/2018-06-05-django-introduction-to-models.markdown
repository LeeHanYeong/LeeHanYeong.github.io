---
layout: post
title:  "(번역) Django 공식문서 - Introduction to models"
categories: ['Django', '번역']
---

- 원본 문서: [Introduction to models](https://docs.djangoproject.com/en/2.0/topics/db/models/)

공식문서에서 가장 중요하다고 생각하는 문서 중 하나인 **The model layer**섹션의 **Introduction to models**를 번역했습니다.

구글 번역기의 도움을 많이 받았으며, 오역이 있다면 댓글이나 메일로 알려주시면 감사하겠습니다.

---

# Models

모델은 데이터에 대한 정보를 나타내는 최종 소스입니다. 그것은 갖고있는 데이터의 필수 필드와 행동(함수)을 포함합니다.  
일반적으로, 각각의 모델은 데이터베이스의 테이블에 매핑됩니다.

기본 사항:  

* 각각의 모델은 `django.db.models.Model`의 서브클래스입니다.
* 모델의 각 속성은 데이터베이스의 필드를 나타냅니다.
* 이것들을 이용해서, 장고는 데이터베이스 액세스 API를 제공합니다. 참조: [쿼리만들기](https://docs.djangoproject.com/en/1.10/topics/db/queries/)


## Quick example

이 샘플 모델은 `first_name`과 `last_name`을 가진`Person`을 정의합니다.

```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
```

`first_name`과 `last_name`은 모델의 필드입니다. 각각의 필드는 클래스의 속성을 나타내며, 데이터베이스의 컬럼에 매핑됩니다.

위의 `Person`모델은 아래와 같은 데이터테이블을 만듭니다.

```sql
CREATE TABLE myapp_person (
    "id" serial NOT NULL PRIMARY KEY,
    "first_name" varchar(30) NOT NULL,
    "last_name" varchar(30) NOT NULL
);
```

몇 가지 기술적 정보:

* 테이블의 이름은 `myapp_person`으로 만들어지지만, 재정의 할 수 있습니다. 자세한내용: [테이블명](https://docs.djangoproject.com/en/1.10/ref/models/options/#table-names)
* `id`필드는 자동으로 추가되지만, 오버라이드(override)할 수 있습니다. 참조: [Automatic primary key fields](https://docs.djangoproject.com/en/1.10/topics/db/models/#automatic-primary-key-fields)
* 이 예제에서 쓰인 `CREATE TABLE` SQL은 `PostgreSQL` 문법이지만, 장고에서는 지정된 데이터베이스에 맞는 SQL을 사용합니다.

---

## Using models

모델을 정의하면, 장고에게 이 모델을 사용할 것임을 알려야 합니다. 설정 파일에서 `INSTALLED_APPS`설정에 `models.py`를 포함하고 있는 모듈의 이름을 추가해줍니다.

```python
INSTALLED_APPS = [
    #...
    'myapp',
    #...
]
```

새로운 애플리케이션을 `INSTALLED_APPS`에 추가했다면, `manage.py migrate`명령어를 실행해줍니다. 선택적으로는 `manage.py makemigrations`를 사용해서 먼저 마이그레이션을 만들어 줄 수 있습니다.

---

## Fields

모델에서 가장 중요한 부분이며, 반드시 필요한 부분입니다. 필드는 데이터베이스의 필드를 정의합니다. 또한, 필드는 클래스 속성으로 사용됩니다.  
clean, save, delete와 같은 [models API](https://docs.djangoproject.com/en/1.10/ref/models/instances/)와 중복되지 않도록 합니다.

예제:

```python
from django.db import models

class Musician(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    instrument = models.CharField(max_length=100)

class Album(models.Model):
    artist = models.ForeignKey(Musician, on_delete=models.CASCADE)
    name = models.CharField(max_length=100)
    release_date = models.DateField()
    num_stars = models.IntegerField()
```

### Field types

각각의 필드는 적절한 필드 클래스의 인스턴스여야 합니다.  
필드 클래스는 아래의 몇 가지 사항을 정의합니다.

* 데이터베이스 컬럼의 데이터 형
* form field를 렌더링 할 때 사용할 기본 HTML 위젯
* Django admin에서 자동으로 만들어지는 form의 검증 형태

장고는 매우 다양한 내장 필드 타입을 제공합니다. 전체 목록은 [model field reference](https://docs.djangoproject.com/en/1.10/ref/models/fields/#model-field-types)에서 찾을 수 있습니다.  
또한, 당신은 장고에서 제공하지 않는 자신만의 필드를 쉽게 만들 수 있습니다. [Writing custom model fields](https://docs.djangoproject.com/en/1.10/howto/custom-model-fields/)

---

### Field options

각 필드는 고유의 인수를 가집니다. (model field reference에 문서가 있습니다). 예를 들어, `CharField`는 `max_length`인수를 반드시 가져야 하며, 데이터베이스에서 VARCHAR필드의 사이즈를 지정합니다.

모든 필드에 사용 가능한 공통 인수도 있습니다. 이들은 모두 선택사항입니다. [​Common model field options](https://docs.djangoproject.com/en/1.10/ref/models/fields/#common-model-field-options)

**null**  
True일 경우, 장고는 빈 값을 **NULL**로 데이터베이스에 저장합니다. 기본값은 False입니다.

**blank**  
True일 경우, 필드는 빈 값을 허용합니다. 기본값은 False입니다.

**null**과 **blank**는 다릅니다.  
**null**은 데이터베이스에 NULL값이 들어가는 것을 허용하는 것이며, **blank**는 데이터베이스에 빈 문자열 값 ("")을 허용하는 것입니다.  
form validation은 **blank=True**일 경우 공백값을 허용합니다. 만약 **blank=False**라면, 해당 필드는 반드시 채워져야합니다.

**choices**  
반복가능한 (예를 들면, 리스트나 튜플) 튜플의 묶음을 선택목록으로 사용합니다. 이 인수가 주어지면, 기본 폼 위젯은 select box로 대체되어 선택값을 제한합니다.

choices는 다음과 같이 나타냅니다.

```python
YEAR_IN_SCHOOL_CHOICES = (
    ('FR', 'Freshman'),
    ('SO', 'Sophomore'),
    ('JR', 'Junior'),
    ('SR', 'Senior'),
    ('GR', 'Graduate'),
)
```

각 튜플의 첫 번째 요소는 데이터베이스에 저장되는 값이며, 두 번째 요소는 기본양식이나 위젯에 표시되는 값입니다.  
모델 인스턴스에서 표시되는 값을 액세스하기 위해서는 get_FOO_display() 함수를 사용합니다. 예제는 다음과 같습니다:

```python
from django.db import models

class Person(models.Model):
    SHIRT_SIZES = (
        ('S', 'Small'),
        ('M', 'Medium'),
        ('L', 'Large'),
    )
    name = models.CharField(max_length=60)
    shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)
```

```python
>>> p = Person(name="Fred Flintstone", shirt_size="L")
>>> p.save()
>>> p.shirt_size
'L'
>>> p.get_shirt_size_display()
'Large'
```

**default**  
필드에 기본값으로 설정됩니다.

**help_text**  
폼 위젯에서 추가적으로 보여줄 도움말 텍스트입니다. 폼을 사용하지 않아도, 문서화에 많은 도움이 됩니다.

**primary_key**  
True일 경우, 해당 필드는 모델의 primary key로 사용됩니다.

어떤 필드에도 `primary_key=True`를 설정하지 않으면, 장고는 자동으로 `IntegerField`를 생성해 primary key로 사용합니다. 그러므로 반드시 `primary_key=True`를 어떤 필드에 추가할 필요는 없습니다. 자세한 정보는 [Automatic primary key fields](https://docs.djangoproject.com/en/1.10/topics/db/models/#automatic-primary-key-fields)

primary key필드는 읽기전용입니다. 기존 개체의 primary key값을 변경한 후 저장하면, 이전 객체와는 별개의 새로운 객체가 생성됩니다.

**unique**  
True일 경우, 이 필드의 값은 테이블 전체에서 고유해야합니다.

이것들은 많이 사용되는 공통 필드 옵션들의 짧은 설명입니다. 전체 내용은 [common model field option reference](https://docs.djangoproject.com/en/1.10/ref/models/fields/#common-model-field-options)

---

### Automatic primary key fields

기본적으로, 장고는 각 모델에 다음 필드를 제공합니다.

```python
id = models.AutoField(primary_key=True)
```

이것은 auto-increment primary key입니다.

만약 임의의 primary key를 지정하고 싶다면, 필드 중 하나에 `primary_key=True`를 지정하면 됩니다. 장고는 당신이 primary key필드를 추가했을 경우, `id`컬럼을 추가하지 않습니다.

각각의 모델은 정확히 하나의 `primary_key=True`필드를 가져야 합니다.

---

### Verbose field names

`ForeignKey`, `ManyToManyField`, `OneToOneField`를 제외한 모든 필드에서, 자세한 필드명은 첫 번째 인수입니다. 만약 Verbose name이 주어지지 않을 경우, 장고는 자동으로 해당 필드의 이름을 사용해서 Verbose name을 만들어 사용합니다.

아래의 예에서, verbose name은 `person's first name`입니다.

```python
first_name = models.CharField("person's first name", max_length=30)
```

아래의 예에서는 verbose name은 `first name`입니다.

```python
​first_name = models.CharField(max_length=30)

```

`ForeignKey`, `ManyToManyField`, `OneToOneField`는 첫 번째 인자로 모델 클래스를 가져야 하므로, `verbose_name`인수를 사용합니다.

```python
poll = models.ForeignKey(
    Poll,
    on_delete=models.CASCADE,
    verbose_name="the related poll",
)
sites = models.ManyToManyField(Site, verbose_name="list of sites")
place = models.OneToOneField(
    Place,
    on_delete=models.CASCADE,
    verbose_name="related place",
)
```

첫 글자는 대문자로 지정하지 않습니다. 장고는 자동으로 첫 번째 글자를 대문자화합니다.

---

### Relationships

관계형 데이터베이스의 강력함은 테이블간의 관게에 있습니다. 장고는 데이터베이스의 관계 유형 중 가장 일반적인 3가지를 제공합니다: `many-to-one`, `many-to-many`, `one-to-one`

#### Many-to-one relationships

Many-to-one관계를 정의하기 위해, `django.db.models.ForeignKey`를 사용합니다. 다른 필드 타입과 비슷하게, 모델의 클래스속성으로 정의합니다.

`ForeignKey`는 관계를 정의할 모델 클래스를 인수로 가져야 합니다.

예를 들어, `Car`모델은 `Manufacturer`를 `ForeignKey`로 갖습니다. `Manufacturer`는 여러개의 `Car`를 가질 수 있지만, `Car`는 오직 하나의 `Manufacturer`만을 갖습니다.

```python
from django.db import models

class Manufacturer(models.Model):
    # ...
    pass

class Car(models.Model):
    manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
    # ...
```

또한 [recursive relationships](https://docs.djangoproject.com/en/1.10/ref/models/fields/#recursive-relationships)와 [relationships to models not yet defined](https://docs.djangoproject.com/en/1.10/ref/models/fields/#lazy-relationships)를 만들 수 있습니다. 자세한 사항은 [the model field reference](https://docs.djangoproject.com/en/1.10/ref/models/fields/#ref-foreignkey)를 참조합니다.

`ForeignKey`필드의 이름은 해당 모델의 lowercase를 추천합니다만, 필수는 아닙니다.

> **같이 보기**  
> ForeignKey필드의 추가 설명은 the model field reference에서 볼 수 있습니다.
> <https://docs.djangoproject.com/en/1.10/ref/models/fields/#foreign-key-arguments>
> 
> 역참조된(backwards-related) 객체들에 대한 참조는 Following relationships backward example에서 볼 수 있습니다.
> <https://docs.djangoproject.com/en/1.10/topics/db/queries/#backwards-related-objects>
> 
> 예제코드는 Many-to-one relationship model example에서 볼 수 있습니다.
> <https://docs.djangoproject.com/en/1.10/topics/db/examples/many_to_one/>

---

### Many-to-many relationships

many-to-many관계를 정의하기 위해서는, `ManyToManyField`를 사용합니다.  
`ManyToManyField`는 관계를 정의할 모델 클래스를 인수로 가져야 합니다.

예를 들어, `Pizza`모델은 여러 개의 `Topping`객체를 가질 수 있습니다. `Topping`은 여러개의 `Pizza`에 올라갈 수 있으며, `Pizza`역시 여러개의 `Topping`을 가질 수 있습니다. 이것은 다음과 같이 나타낼 수 있습니다.

```python
from django.db import models

class Topping(models.Model):
    # ...
    pass

class Pizza(models.Model):
    # ...
    toppings = models.ManyToManyField(Topping)
```

`ForeignKey`와 마찬가지로, 여기서도 recursive relationships와 relationships to models not yet defined를 만들 수 있습니다.

일반적으로 필드명은 관계된 모델 객체의 복수형을 추천하지만, 필수는 아닙니다.

어떤 모델이 `ManyToManyField`를 갖는지는 중요하지 않지만, 오직 관계되는 둘 중 하나의 모델에만 존재해야합니다.

일반적으로, `ManyToManyField`인스턴스는 form에서 수정할 객체에 가까워야 합니다. 위의 예제에서는 `toppings`가 `Pizza`에 있는 것이 `Topping`이 `pizzas`를 갖는 것보다는 자연스럽기 때문에, `Pizza`form에서 사용자는 `toppings`를 선택할 가능성이 높습니다.

> **같이 보기**  
> [Many-to-many relationship model example](https://docs.djangoproject.com/en/1.10/topics/db/examples/many_to_many/)에 전체 예제가 있습니다.

---

#### Extra fifelds on many-too-many relationships

피자와 토핑같은 간단한 many-to-many 관계를 만들 때, `ManyToManyField`는 필요로 하는 모든 것을 제공합니다. 하지만, 때때로 두 모델 사이의 관계와 데이터를 연결해야 할 수도 있습니다.

예를 들어, 음악가가 속한 음악 그룹을 트래킹(추적)하는 경우를 고려해봅니다. 사람과 그룹으로 멤버를 이루는 관계에서, `ManyToManyField`로 이 관계를 나타내고자 합니다. 하지만, 어떤 사람이 그룹으로 가입할 때 가입하는 날짜와 같은 세부사항들이 추가로 존재합니다.

이런 경우, 장고에서는 many-to-many관계를 관리하는 데 사용되는 모델을 지정할 수 있습니다. 그리고 중간 모델에 추가 필드를 넣을 수 있습니다. `ManyToManyField`의 `through` 인수에 중간 모델을 가리키도록 하여 연결할 수 있습니다. 음악가 예제에서는, 다음과 같이 나타냅니다.

```python
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=128)

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, through='Membership')

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Membership(models.Model):
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    date_joined = models.DateField()
    invite_reason = models.CharField(max_length=64)
```

중간 모델을 설정할 때, 명시적으로 many-to-many관계에 참여하는 모델들의 `ForeignKey`를 지정합니다. 이 명시적 선언은 두 모델이 관련되는 법을 정읳바니다.

중간 모델에는 몇 가지 제한 사항이 있습니다:

* 중간모델은 원본모델(위의 예에서 Group모델)에 대해 단 하나의 `ForeignKey`필드를 가져야 합니다. 여러개여도 안되며, 없어도 안됩니다. 아니면 반드시 `ManyToMany`필드에서 `through_fields`옵션으로 관계에 사용될 필드 이름을 지정해 주어야 합니다. 둘 중 하나가 아니면 Validation에러가 발생합니다. 타겟 모델(위 예제에서 `Person`)의 경우에도 동일합니다.
* 자기 자신에게 many-to-many관계를 가지는 모델의 경우에는 중간 모델에 동일한 모델에 대한 `ForeignKey`필드를 2개 선언할 수 있습니다. 3개 이상의 `ForeignKey`필드를 선언할 경우에는 앞에서 언급한 것과 같이 through_fields옵션을 설정해주어야 합니다.
* 자기자신에게 many-to-many관계를 가지고 중간모델을 직접 선언하는 경우에는 `ManyToMany`필드의 `symmetrical`옵션을 `False`로 설정해 주어야 합니다.

이제 `ManyToManyField`에서 중간 모델(이 경우, `Membership`)을 사용할 준비가 되었습니다. 중간 모델 인스턴스를 만들어봅시다:

```python
>>> ringo = Person.objects.create(name="Ringo Starr")
>>> paul = Person.objects.create(name="Paul McCartney")
>>> beatles = Group.objects.create(name="The Beatles")
>>> m1 = Membership(person=ringo, group=beatles,
...     date_joined=date(1962, 8, 16),
...     invite_reason="Needed a new drummer.")
>>> m1.save()
>>> beatles.members.all()
<QuerySet [<Person: Ringo Starr>]>
>>> ringo.group_set.all()
<QuerySet [<Group: The Beatles>]>
>>> m2 = Membership.objects.create(person=paul, group=beatles,
...     date_joined=date(1960, 8, 1),
...     invite_reason="Wanted to form a band.")
>>> beatles.members.all()
<QuerySet [<Person: Ringo Starr>, <Person: Paul McCartney>]>
```



일반적인 many-to-many필드와는 달리, **add()**, **create()**, **set()**(직접 할당) 명령어를 사용할 수 없습니다.

```python
>>> # THIS WILL NOT WORK
>>> beatles.members.add(john)
>>> # NEITHER WILL THIS
>>> beatles.members.create(name="George Harrison")
>>> # AND NEITHER WILL THIS
>>> beatles.members.set([john, paul, ringo, george])
```

​왜냐하면, `Person`과 `Group`관계를 설정할 때 중간 모델의 필드값들을 명시해주어야 하기 때문입니다. 즉, 위의 예와 같이 단순히 **add/create/set**하는 경우에는 중간 모델에서 person과 group필드값은 알 수 있지만, date_joined와 invite_reason필드값은 알 수 없기 때문입니다.

​​그러므로 중간모델을 직접 지정한 경우에는 중간 모델을 직접 생성하는 방법밖에는 없습니다.​
​

```python
>>> Membership.objects.create(person=ringo, group=beatles,
...     date_joined=date(1968, 9, 4),
...     invite_reason="You've been gone for a month and we miss you.")
>>> beatles.members.all()
>>> <QuerySet [<Person: Ringo Starr>, <Person: Paul McCartney>, <Person: Ringo Starr>]>
>>> # THIS WILL NOT WORK BECAUSE IT CANNOT TELL WHICH MEMBERSHIP TO REMOVE
>>> beatles.members.remove(ringo)
```

하지만, **clear()**함수는 사용가능합니다.

```python
>>> # Beatles have broken up
>>> beatles.members.clear()
>>> # Note that this deletes the intermediate model instances
>>> Membership.objects.all()
<QuerySet []>
```

관계를 생성할 때는 위와 같은 제약이 있지만, 쿼리시에는 일반적인 many-to-many관계와 동일하게 사용할 수 있습니다.  

```python
# Find all the groups with a member whose name starts with 'Paul'
>>> Group.objects.filter(members__name__startswith='Paul')
<QuerySet [<Group: The Beatles>]>
```

중간 모델을 사용하고 있기 때문에 아래와 같은 쿼리도 가능합니다.  
아래의 예제는 비틀즈 멤버중 1961년 1월 1일 이후에 합류한 멤버를 찾습니다.

```python
# Find all the members of the Beatles that joined after 1 Jan 1961
>>> Person.objects.filter(
...     group__name='The Beatles',
...     membership__date_joined__gt=date(1961,1,1))
<QuerySet [<Person: Ringo Starr]>
```

`Membership`모델 (중간모델)에 직접 쿼리할 수도 있습니다.

```python
>>> ringos_membership = Membership.objects.get(group=beatles, person=ringo)
>>> ringos_membership.date_joined
datetime.date(1962, 8, 16)
>>> ringos_membership.invite_reason
'Needed a new drummer.'
```

`Person`객체로부터 many-to-many역참조를 이용해도 위와 동일한 결과를 얻을 수 있습니다.

```python
>>> ringos_membership = ringo.membership_set.get(group=beatles)
>>> ringos_membership.date_joined
datetime.date(1962, 8, 16)
>>> ringos_membership.invite_reason
'Needed a new drummer.'
```

---

#### One-to-one relationships

one-to-one관계를 정의하려면, `OneToOneField`를 이용하면 됩니다. 다른 관계 필드와 마찬가지로 모델 클래스의 어트리뷰트로 선언하면 됩니다.

일대일 관계는 다른 모델을 확장하여 새로운 모델을 만드는 경우 유용하게 사용할 수 있습니다.

예를 들어, 가게(Places)정보가 담긴 데이터베이스를 구축한다고 합니다. 아마 데이터베이스에 주소, 전화번호 등의 정보가 들어갈 겁니다. 그런데, 맛집 데이터베이스를 추가적으로 구축할 경우, 새로 Restaurant모델을 만들 수도 있지만, 반복을 피하기 위해 Restaurant모델에 Place모델만 `OneToOneField`로 선언해 줄 수 있습니다.

`ForeignKeyField`와 마찬가지로 자기자신이나 아직 선언되지 않은 모델에 대해서도 관계를 가질 수 있습니다.

`OneToOneField`는 `parent_link`라는 옵션을 제공합니다. [관련설명](http://stackoverflow.com/questions/7269319/django-multi-table-inheritance-specify-custom-one-to-one-column-name)

`OneToOneField`클래스가 자동적으로 모델의 primary key가 되었던 적이 있습니다. 지금은 더 이상 그렇게 사용하지 않습니다. 물론, 직접 `primary_key=True`를 지정하여 primary key로 만들 수는 있습니다. 어쨌든, 결과적으로 하나의 모델이 여러개의 `OneToOneField`를 가질 수 있게 되었습니다.

---

### Models across files

다른 앱에 선언된 모델과 관계를 가질 수 있습니다. 그렇게 하려면, 다른 앱의 모델을 import해서 아래와 같이 관계 필드를 선언하면 됩니다.

```python
from django.db import models
from geography.models import ZipCode

class Restaurant(models.Model):
    # ...
    zip_code = models.ForeignKey(
        ZipCode,
        on_delete=models.SET_NULL,
        blank=True,
        null=True,
    )
```

---

### Field name restrictions

장고는 모델 필드명에 2가지 제약을 두고있습니다.

1] 파이썬 예약어는 필드명으로 사용할 수 없습니다. 이 경우, 파이썬 구문 에러가 발생합니다:

```python
class Example(models.Model):
    pass = models.IntegerField() # 'pass' is a reserved word!
```

2] 필드 이름에 밑줄 두개를 연속으로 사용할 수 없습니다. 이는 장고에서 특별한 문법으로 사용되기 때문입니다.

```python
class Example(models.Model):
    foo__bar = models.IntegerField() # 'foo__bar' has two underscores!
```

데이터베이스 컬럼명에 밑줄을 두 개 넣어야만 하는 상황이라면, `db_column`옵션을 사용해서 제약을 우회할 수 있습니다.

SQL 예약어의 경우(join, where, select)에는 필드 이름으로 허용됩니다. 장고에서 쿼리문을 만들 때, 모든 컬럼명과 테이블명은 이스케이프 처리하기 때문입니다. 실제 데이터베이스 엔진에 맞게 알아서 말이죠.

---

### Custom field types

장고에서 제공하는 필드 타입중 적절한 타입이 없거나, 특정 데이터베이스에서만 제공하는 특별한 타입을 사용하고 싶다면, 필드를 직접 만들어 사용할 수 있습니다. [Writing custom model fields](https://docs.djangoproject.com/en/1.10/howto/custom-model-fields/)

---

## Meta options

아래와 같이 모델 클래스 내부에 Meta라는 이름의 클래스를 선언해서 모델에 메타데이터를 추가할 수 있습니다.

```python
from django.db import models

class Ox(models.Model):
    horn_length = models.IntegerField()

    class Meta:
        ordering = ["horn_length"]
        verbose_name_plural = "oxen"
```

모델 메타데이터는 앞에서 보았던 필드의 옵션과 달리, 모델 단위의 옵션이라고 볼 수 있습니다. 예를 들어, 정렬 옵션(orering), 데이터베이스 테이블 이름(db_table), 또는 읽기 좋은 이름(verbose_name)이나 복수(verbose_name_plural)이름을 지정해 줄 수 있습니다.  
모델클래스에 Meta클래스를 반드시 선언해야 하는 것은 아니며, 또한 모든 옵션을 설정해야 하는 것도 아닙니다.

모든 메타데이터 옵션을 살펴보려면 [model option reference](https://docs.djangoproject.com/en/1.10/ref/models/options/)

---

## Model attributes

**objects**

모델 클래스에서 가장 중요한 속성은 `Manager`입니다. `Manager`객체는 모델 클래스를 기반으로 데이터베이스에 대한 쿼리 인터페이스를 제공하며, 데이터베이스 레코드를 모델 객체로 인스턴스화 하는데 사용됩니다. 특별히 `Manager`를 할당하지 않으면 장고는 기본 Manager를 클래스 속성으로 자동 할당합니다. 이 때, 속성의 이름이 **objects**입니다.

Manager는 모델 클래스를 통해 접근할 수 있으며, 모델 인스턴스(객체)를 통해서는 접근 할 수 없습니다.

---

## Model methods

모델 객체(row)단위의 기능을 구현하려면 모델 클래스에 메서드를 구현해주면 됩니다. 테이블단위의 기능은 Manager에 구현합니다.

이러한 규칙은 비즈니스 로직을 모델에서 관리하는데 있어 중요한 테크닉입니다.

예를 들어, 다음과 같이 커스텀 메서드를 추가할 수 있습니다.

```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    birth_date = models.DateField()

    def baby_boomer_status(self):
        "Returns the person's baby-boomer status."
        import datetime
        if self.birth_date < datetime.date(1945, 8, 1):
            return "Pre-boomer"
        elif self.birth_date < datetime.date(1965, 1, 1):
            return "Baby boomer"
        else:
            return "Post-boomer"

    def _get_full_name(self):
        "Returns the person's full name."
        return '%s %s' % (self.first_name, self.last_name)
    full_name = property(_get_full_name)
```

이 예제에서, 마지막 메서드는 [property](https://docs.djangoproject.com/en/1.10/glossary/#term-property)입니다.

> property는 메서드를 속성처럼 접근할 수 있도록 해줍니다

[model instance reference](https://docs.djangoproject.com/en/1.10/ref/models/instances/)에는 모델 클래스에 자동적으로 주어지는 메서드들이 나와 있습니다. 이러한 메서드를 여러분이 오버라이드해서 사용할 수 있는데, 자세한 내용은 다음 절에서 설명합니다.

**\_\_str\_\_()** (Python 3)  
모델 객체가 문자열로 표현되어야 하는 경우에 호출됩니다. admin이나 console에서 많이 쓰이게 됩니다.

기본 구현은 아무 도움이 되지 않는 문자열을 리턴하기 때문에, 모든 모델에 대해 오버라이드 해서 알맞게 구현해주는게 좋습니다.

**get\_absolute\_url()**  
이 메서드는 장고가 해당 모델 객체의 URL을 계산할 수 있도록 합니다. 장고는 이 메서드를 모델 객체를 URL로 표현하는 경우에 사용하며, admin사이트에서도 사용합니다.

모델 객체가 유일한 URL을 가지는 경우에는 이 메서드를 구현해주어야 합니다.


### Overriding predefined model methods

커스터마이징 할 데이터베이스 동작을 캡슐화하는 또 다른 모델 메서드 집합이 있습니다. 특히 `save()`및 `delete()`의 작업방식을 바꾸는 경구가 많습니다.

동작을 바꾸기위해 이 메서드들을 마음껏 오버라이드 할 수 있습니다.

내장된 메서드를 재정의하는 일반적인 사용 사례는 객체를 저장할 때마다 어떤 작업을 수행하기를 원할 때입니다.

받아들이는 매개변수에 대해서는 [save()](https://docs.djangoproject.com/en/1.11/ref/models/instances/#django.db.models.Model.save)문서를 참조하십시오.

```python
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def save(self, *args, **kwargs):
        do_something()
        super(Blog, self).save(*args, **kwargs) # Call the "real" save() method.
        do_something_else()
```

저장을 막을 수도 있습니다

```python
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def save(self, *args, **kwargs):
        if self.name == "Yoko Ono's blog":
            return # Yoko shall never have her own blog!
        else:
            super(Blog, self).save(*args, **kwargs) # Call the "real" save() method.
```

슈퍼 클래스 메서드를 호출하는 것을 기억하는 것이 중요합니다.
> super(Blog, self) .save (* args, ** kwargs) 를 말합니다. 

이는 객체가 데이터베이스에 저장되도록합니다.  
수퍼 클래스 메서드를 호출하는 것을 잊어 버리면 기본 동작히 실행되지 않으며 데이터베이스에 저장하지 않습니다.

또한 모델 메서드에 전달할 수있는 인수를 전달하는 것이 중요합니다. 이는 `*args`와 `**kwargs`두 변수가 수행합니다.  
Django는 수시로 내장 모델 메서드의 기능을 확장하여 새로운 인수를 추가합니다. 메서드 정의에서 `*args`, `**kwargs`를 사용하면 코드가 추가 될 때 해당 인수를 자동으로 지원한다는 보장을받습니다.

> **오버라이드된 모델 메서드는 `bulk operations`에서는 동작하지 않습니다.**
> 
> QuerySet을 사용하여 대량으로 객체를 삭제할 때 또는 계단식 삭제의 결과로 객체의 delete() 메서드가 반드시 호출되지 않을 수 있습니다. 사용자 정의 삭제 논리를 실행하려면 `pre_delete`와 `post_delete` 시그널을 사용할 수 있습니다.
> 
> 불행하게도 `bulk operations`에서는 `save()`메서드와 `pre_save`및 `post_save` 시그널이 호출되지 않기 때문에 객체를 대량으로 만들거나 업데이트 할 때 해결 방법이 없습니다.

### Executing custom SQL

또 다른 공통 패턴은 모델 메서드 및 모듈 수준 메서드에 사용자 지정 SQL 문을 작성하는 것입니다. 원시 SQL 사용에 대한 자세한 정보는 [using raw SQL](https://docs.djangoproject.com/en/2.0/topics/db/sql/) 문서를 참조하십시오.

---

## Model inheritance

`Django`의 모델 상속은 파이썬에서 일반적인 클래스 상속이 작동하는 방식과 거의 동일하게 작동하지만 반드시 따라야하는 기본 사항이 있습니다. 기본 클래스가 `django.db.models.Model`를 상속받아야 합니다.

부모 모델이 자체 데이터베이스 테이블을 가지는 모델이 될지, 또는 부모가 자식 모델에게 전달할 정보만을 가지고 있는지 여부만 결정하면됩니다.

`Django`에서는 세 가지 스타일의 상속을 제공합니다.

1. 흔히 부모 클래스를 사용하여 각 하위 모델에 대해 일일이 입력하지 않으려는 정보를 제공하는 경우입니다. 이 클래스는 따로 분리하여 사용하지 않으므로 추상 기본 클래스(Abstract base classes)를 사용합니다.
2. 기존 모델을 하위 클래스 화(다른 애플리케이션의 모델이어도 무관)하고, 각 모델이 자체 데이터베이스 테이블을 가지기를 원한다면 다중 테이블 상속(Multi table inheritance)이 필요합니다.
3. 마지막으로 모델 필드를 변경하지 않고 모델의 파이썬 수준 동작만 수정하려는 경우 `Proxy`모델을 사용할 수 있습니다.

### Abstract base classes

추상 기본 클래스는 몇 가지 공통된 정보를 여러 다른 모델에 넣으려 할 때 유용합니다. 기본 클래스를 작성하고 `Meta`class에  `abstract = True`를 넣습니다. 이 모델은 데이터베이스 테이블을 만드는 데 사용되지 않습니다. 대신 다른 모델의 기본 클래스로 사용될 때 해당 필드는 자식 클래스의 필드에 추가됩니다. 자식의 이름과 같은 이름(상속받은 클래스의 이름과 같은 이름의 필드)을 가진 추상 기본 클래스의 필드를 갖는 것은 오류이며, `Django`는 이에 대해 오류를 발생시킵니다.

예시:

```python
from django.db import models

class CommonInfo(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()

    class Meta:
        abstract = True

class Student(CommonInfo):
    home_group = models.CharField(max_length=5)
```

`Student`모델에는 `name`, `age`및 `home_group`의 세 가지 필드가 있습니다. `CommonInfo`모델은 `abstract base class`이기 때문에 일반 `Django` 모델로 사용할 수 없습니다. 이 모델은 데이터베이스 테이블을 생성하지 않으며 `Manager`를 가지지 않으므로 직접 인스턴스화하거나 저장할 수 없습니다.

많은 경우, 이 유형의 모델 상속이 일반적입니다. 그것은 파이썬 레벨에서 공통 정보를 제외시키는 방법을 제공하면서 데이터베이스 레벨에서 하위 모델 당 하나의 데이터베이스 테이블만 생성합니다.

#### Meta inheritance

추상 기본 클래스가 생성되면 Django는 기본 클래스에서 선언한 `Meta`내부 클래스를 속성으로 사용할 수 있게합니다. 자식 클래스가 자신의 메타 클래스를 선언하지 않으면 부모 클래스의 메타를 상속받습니다. 자식이 부모의 `Meta`클래스를 확장하려고하면 해당 클래스를 서브 클래스로 사용할 수 있습니다. 예 :

```python
from django.db import models

class CommonInfo(models.Model):
    # ...
    class Meta:
        abstract = True
        ordering = ['name']

class Student(CommonInfo):
    # ...
    class Meta(CommonInfo.Meta):
        db_table = 'student_info'
```

`Django`는 추상 기본 클래스의 `Meta`클래스를 조정합니다. `Meta`속성을 적용하기 전에 `abstract`속성 값을 `False`로 설정합니다. 즉, 추상 기본 클래스의 자식은 자동으로 추상 클래스가되지 않습니다. 물론 다른 추상 기본 클래스에서 상속받은 추상 기본 클래스를 만들 수 있습니다. 매번 `abstract = True`를 명시적으로 설정하는 것을 기억하면됩니다.

일부 속성은 추상 기본 클래스의 `Meta`클래스에 포함하는 것이 타당하지 않습니다. 예를 들어, `db_table`을 포함하는 것은 모든 자식 클래스(자신의 메타를 지정하지 않은 클래스)가 동일한 데이터베이스 테이블을 사용한다는 것을 의미합니다. 이는 대부분의 경우 원하지 않는 동작입니다.

#### Be careful with related_name and related\_query\_name

**ForeignKey**또는 **ManyToManyField**에서 **related_name**또는 **related_query_name**을 사용하는 경우 필드의 고유한 역 이름(**reverse name**)과 쿼리 이름(**query name**)을 항상 지정해야합니다. 이 필드들(**ForeignKey**, **ManyToManyField**)을 가진 추상 기본 클래스를 상속받은 경우 매번 해당 속성(**related_name**또는 **related\_query\_name**)에 대해 정확히 동일한 값이 사용되므로 일반적으로 문제가 발생합니다.

이 문제를 해결하려면 추상 기본 클래스에서 **related_name**또는 **related\_query\_name**을 사용할 때 값의 일부에 **'%(app\_label)s'**및 **'%(class)s'**를 포함해야 합니다.

- **'%(class)s'**는 필드가 사용되는 하위 클래스의 lower-cased이름으로 대체됩니다.
- **'%(app\_label)s'**는 하위 클래스가 포함 된 애플리케이션 이름의 lower-cased이름으로 바뀝니다. 설치된 각 응용 프로그램 이름은 고유해야하며 각 응용 프로그램 내의 모델 클래스 이름도 고유해야하므로 결과 이름이 달라집니다.

모델 예제 `common/models.py`:

```python
from django.db import models

class Base(models.Model):
    m2m = models.ManyToManyField(
        OtherModel,
        related_name="%(app_label)s_%(class)s_related",
        related_query_name="%(app_label)s_%(class)ss",
    )

    class Meta:
        abstract = True

class ChildA(Base):
    pass

class ChildB(Base):
    pass
```

다른 모델 예제 `rare/models.py`:

```python
from common.models import Base

class ChildB(Base):
    pass
```

**common.ChildA.m2m**필드의 **reverse name**은 **common\_childa\_related**이고, **reverse query name**은 **common\_childas**입니다. **common.ChildB.m2m**필드의 **reverse name**은 **common\_childb\_related**이고 **reverse query name**은 **common\_childbs**입니다.  
마지막으로, **rare.ChildB.m2m** 필드의 **reverse name**은 **rare\_childb\_related**이고 **reverse query name**은 **rare\_childbs**입니다. **'%(class)s'**및 **'%(app_label)s'**를 사용하여 관련 이름 또는 관련 검색어 이름을 작성하는 방법은 사용자에게 달렸지만, 사용하는 것을 잊을 경우 Django는 시스템 검사를 수행 할 때 오류를 발생시킵니다(또는 마이그레이션을 실행 할 때).

추상 기본 클래스의 필드에 **related_name**속성을 지정하지 않으면 상속받은 자식 클래스의 기본 **reverse name**은 필드를 직접 선언 한 경우와 마찬가지로 **'_set'**이 뒤에 오는 자식 클래스의 이름이됩니다. 예를 들어, 위 코드에서 **related_name**속성을 생략하면 **m2m** 필드의 **reverse name**은 **ChildA**의 경우 **childa_set**이되고 **ChildB**필드의 경우 **childb_set**이 됩니다.

---

### Multi-table inheritance

Django가 지원하는 모델 상속의 두 번째 유형은 계층 구조의 각 모델이 모두 각각 자신을 나타내는 모델일 때입니다. 각 모델은 자체 데이터베이스 테이블에 해당하며 개별적으로 쿼리하고 생성할 수 있습니다. 상속 관계는 자동으로 생성 된 **OneToOneField**를 통해 자식 모델과 각 부모 간의 링크를 만듭니다. 예:

```python
from django.db import models

class Place(models.Model):
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)

class Restaurant(Place):
    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)
```

**Place**의 모든 필드는 **Restaurant**에서 사용할 수 있지만 데이터는 다른 데이터베이스 테이블에 있습니다. 그래서 아래 두 명령 모두 가능합니다:

```python
>>> Place.objects.filter(name="Bob's Cafe")
>>> Restaurant.objects.filter(name="Bob's Cafe")
```

**Restaurant**이면서 **Place**가 있는 경우, 모델 이름의 소문자 버전을 사용하여 **Place**객체에서 **Restaurant**객체를 가져올 수 있습니다.

```python
>>> p = Place.objects.get(id=12)
# If p is a Restaurant object, this will give the child class:
>>> p.restaurant
<Restaurant: ...>
```

그러나 위 예제의 **p**가 **Restaurant**이 아닌 경우(즉, **Place**객체로 직접 작성되었거나 다른 클래스의 부모인 경우) **p.restaurant**를 참조하면 **Restaurant.DoesNotExist**예외가 발생합니다.

**Restaurant**에 자동으로 생성 된 **OneToOneField**는 다음과 같은 형태를 가집니다.

```python
place_ptr = models.OneToOneField(
    Place, on_delete=models.CASCADE,
    parent_link=True,
)
```

**Restaurant**에서 **parent_link=True**를 사용해 자신의 **OneToOneField**를 선언하여 해당 필드를 재정의 할 수 있습니다.

#### Meta and multi-table inheritance

다중 테이블 상속 상황에서 자식 클래스가 부모의 **Meta**클래스에서 상속받는 것은 의미가 없습니다. 모든 메타 옵션은 이미 상위 클래스에 적용되었고 다시 적용하면 모순된 행동만 발생합니다 (기본 클래스가 자체적으로 존재하지 않는 추상 기본 클래스의 경우와 대조적입니다).

따라서 자식 모델은 부모의 메타 클래스에 액세스 할 수 없습니다. 그러나 자식이 부모로부터 동작을 상속하는 몇 가지 제한된 경우가 있습니다. 자식이 **ordering**특성이나 **get_latest_by**특성을 지정하지 않으면 해당 특성을 부모로부터 상속합니다.

부모가 **ordering**되어 있고, 이를 해제하려면 명시적으로 사용을 중지 할 수 있습니다:

```python
class ChildModel(ParentModel):
    # ...
    class Meta:
        # Remove parent's ordering effect
        ordering = []
```

#### Inheritance and reverse relations

다중 테이블 상속은 암시적으로 **OneToOneField**를 사용하여 부모와 자식을 연결하기 때문에 위의 예와 같이 상위에서 하위로 이동할 수 있습니다. 그러나 이 경우  **related_name**의 값으로 **ForeignKey**및 **ManyToManyField**관계에 대한 기본 값을 사용합니다. 이러한 관계 유형들을 부모 모델의 하위 클래스에 배치하는 경우 해당 필드 각각에 **반드시** **related_name**속성을 지정해야합니다. 이를 잊어 버리면 Django는 유효성 검사 오류를 발생시킵니다.

예를 들어 위의 **Place**클래스에서 **ManyToManyField**를 사용하는 다른 하위 클래스를 만들어봅니다.

```python
class Supplier(Place):
    customers = models.ManyToManyField(Place)
```

결과는 다음과 같은 에러를 나타냅니다:

```python
Reverse query name for 'Supplier.customers' clashes with reverse query
name for 'Supplier.place_ptr'.

HINT: Add or change a related_name argument to the definition for
'Supplier.customers' or 'Supplier.place_ptr'.
```

**customers_name**필드에 **related_name**을 추가하면 **models.ManyToManyField(Place, related_name='provider')**에서 발생한 오류가 해결됩니다.

#### Specifying the parent link field

앞서 언급했듯이 Django는 자동적으로 자식 클래스를 임의의 추상이 아닌 부모 모델에 연결하는 **OneToOneField**를 만듭니다. 부모에게 다시 연결되는 속성의 이름을 제어하려는 경우 고유 한 **OneToOneField**를 만들고 **parent_link=True**로 설정하여 해당 필드가 부모 클래스에 대한 링크임을 나타낼 수 있습니다.

---

### Proxy models

다중 테이블 상속을 사용하면 모델의 각 하위 클래스에 대해 새 데이터베이스 테이블이 만들어집니다. 서브 클래스는 기본 클래스에없는 추가 데이터 필드를 저장할 장소가 필요하기 때문에 일반적으로 이것은 원하는 동작입니다. 그러나 때로는 모델의 파이썬에서의 동작만을 변경하고자 할 때가 있습니다. 예를 들어 기본 관리자를 변경하거나 새 메서드를 추가하는 경우가 이에 해당합니다.

프록시 모델 상속은 위와 같은 경우를 위한 것입니다: 원래 모델에 대한 **proxy**를 만듭니다. 프록시 모델의 인스턴스를 생성, 삭제 및 업데이트 할 수 있으며 원본(비 프록시) 모델을 사용하는 것처럼 모든 데이터가 저장됩니다. 차이점은 원본을 변경하지 않고 프록시의 기본 모델 순서(**ordering**) 또는 기본 관리자(**default manager**)와 같은 것을 변경할 수 있다는 것입니다.

프록시 모델은 일반 모델처럼 선언됩니다. Django에게 메타 클래스의 **proxy**속성을 **True**로 설정하여 Django에게 해당 모델이 프록시 모델임을 알려야 합니다.

예를 들어, **Person**모델에 메서드를 추가하려고한다고 가정할 경우, 다음과 같이 할 수 있습니다.

```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)

class MyPerson(Person):
    class Meta:
        proxy = True

    def do_something(self):
        # ...
        pass
```

**MyPerson**클래스는 상위 **Person**클래스와 동일한 데이터베이스 테이블에서 작동합니다. 특히 **Person**의 새로운 인스턴스는 **MyPerson**을 통해 액세스 할 수 있으며 그 반대의 경우도 가능합니다.

```python
>>> p = Person.objects.create(first_name="foobar")
>>> MyPerson.objects.get(first_name="foobar")
<MyPerson: foobar>
```

프록시 모델을 사용하여 모델에서 다른 기본 순서(**ordering**)를 정의 할 수도 있습니다. **Person**모델을 항상 **ordering**하고 싶지는 않지만, 프록시를 사용할 때 **last_name**속성으로 규칙적으로 **ordering**하고자 할 경우엔 이렇게 합니다:

```python
class OrderedPerson(Person):
    class Meta:
        ordering = ["last_name"]
        proxy = True
```

이제 일반적인 **Person**쿼리는 **ordering**되지 않으며, **OrderedPerson**쿼리는 **last_name**에 의해 **ordering**됩니다.

프록시 모델은 일반 모델과 동일한 방식으로 **Meta**속성을 상속합니다.

#### QuerySets still return the model that was requested

Django에서는 **Person**객체를 쿼리 할 때마다 **MyPerson**객체를 반환 할 방법이 없습니다. **Person**객체에 대한 queryset은 해당 유형의 객체를 반환합니다. 프록시 객체의 요점은 원래 **Person**을 사용하는 코드는 그것을 사용하고, 사용자 코드는 포함시킨 확장기능을 사용할 수 있다는 것입니다 (다른 코드는 그다지 의존하지 않음). 그것은 **Person**(또는 다른 어떤 것이든)모델을 항상 당신이 만든 모델로 대체하는 방법이 아닙니다.

#### Base class restrictions

프록시 모델은 정확히 하나의 비 추상적 모델 클래스를 상속해야합니다. 프록시 모델은 다른 데이터베이스 테이블의 행 사이에 연결을 제공하지 않으므로 여러개의 비 추상적 모델을 상속받을 수 없습니다. 프록시 모델은 모델 필드를 정의하지 않으면 추상 모델 클래스를 상속 받을 수 있습니다. 프록시 모델은, 공통의 비 추상 부모 클래스를 공유하는 임의의 수의 프록시 모델을 상속 받을  수 있습니다.

#### Proxy model managers

프록시 모델에 모델 관리자를 지정하지 않으면 모델 부모로부터 관리자를 상속받습니다. 프록시 모델에서 관리자를 정의하면 그것이 기본값이 되지만, 부모 클래스에 정의 된 관리자는 계속 사용할 수 있습니다.

위의 예를 계속해서 사용하며, **Person**모델을 쿼리 할 때 사용되는 기본 관리자를 다음과 같이 변경할 수 있습니다.

```python
from django.db import models

class NewManager(models.Manager):
    # ...
    pass

class MyPerson(Person):
    objects = NewManager()

    class Meta:
        proxy = True
```

기존 기본값을 바꾸지 않고 프록시에 새 관리자를 추가하려면 [custom manager](https://docs.djangoproject.com/en/2.0/topics/db/managers/#custom-managers-and-inheritance)문서에 설명 된 기술을 사용할 수 있습니다: 새 관리자를 포함하는 기본 클래스를 작성하고 primary base class다음으로 해당 관리자를 상속받습니다.

```python
# Create an abstract class for the new manager.
class ExtraManagers(models.Model):
    secondary = NewManager()

    class Meta:
        abstract = True

class MyPerson(Person, ExtraManagers):
    class Meta:
        proxy = True
```

아마도 이 작업을 자주 수행 할 필요는 없겠지만, 필요할 경우 이렇게 정의할 수 있습니다.


#### Differences between proxy inheritance and unmanaged models

프록시 모델 상속은 모델의 **Meta**클래스에서 관리되는 특성을 사용하여 관리되지 않는 모델(unmanaged model)을 만드는 것과 매우 비슷하게 보일 수 있습니다.

신중하게 **Meta.db_table**을 설정하면 관리되지 않는 모델을 만들어 기존 모델을 shadows처리하고 Python메서드를 추가 할 수 있습니다. 그러나 이 경우, 변경 작업을 수행 할 경우 두 복사본을 동기화 된 상태로 유지해야하므로 반복적으로 구조가 일치하지 않는 오류가 발생할 수 있습니다.

반면에 프록시 모델은 프록싱중인 모델과 정확히 동일하게 동작합니다. 부모 모델은 필드와 관리자를 직접 상속하므로 항상 부모 모델과 동기화됩니다.

일반적인 규칙은 다음과 같습니다:

1. 기존 모델이나 데이터베이스 테이블을 미러링하고 원래 데이터베이스 테이블 열을 모두 원하지 않는 경우 **Meta.managed = False**를 사용하십시오. 이 옵션은 일반적으로 Django가 제어하지 않는 데이터베이스 뷰와 테이블을 모델링 할 때 유용합니다.
2. 모델의 파이썬 전용 동작을 변경하려고하지만 원본과 동일한 필드를 모두 유지하려면 **Meta.proxy = True**를 사용하십시오. 이렇게하면 데이터를 저장할 때 프록시 모델이 원본 모델의 저장소 구조와 정확히 일치하도록 설정됩니다.

---

### Multiple inheritance

파이썬의 서브 클래싱과 마찬가지로, 장고 모델도 여러 부모 모델로부터 상속받을 수도 있습니다. 일반적인 파이썬 이름 해석 규칙(Name resolution rules)이 적용된다는 것을 명심하십시오. 특정 이름(예: **Meta**)이 나타나는 첫 번째 기본 클래스가 사용됩니다. 예를 들어 여러 부모가 메타 클래스를 포함하면 첫 번째 부모만 사용되며 다른 모든 부모는 무시됩니다.

일반적으로는 여러 부모로부터 상속하지 않아도됩니다. 이것이 유용한 주요 유스 케이스(use-case)는 "믹스 인 (mix-in)"클래스입니다. 믹스 인을 상속받은 모든 클래스에 특정 추가 필드나 메서드를 추가하는 것입니다. 상속 계층을 가능한 간단하고 직관적으로 유지하여 특정 정보가 어디에서 왔는지 알아 내려고 노력하지 않아도 되도록하십시오.

공통 **id** 기본 키 필드가있는 여러 모델을 상속하면 오류가 발생합니다. 다중 상속을 제대로 사용하려면 기본 모델에서 명시적인 **AutoField**를 사용합니다:

```python
class Article(models.Model):
    article_id = models.AutoField(primary_key=True)
    ...

class Book(models.Model):
    book_id = models.AutoField(primary_key=True)
    ...

class BookReview(Book, Article):
    pass
```

또는 공통 조상을 사용하여 **AutoField**를 유지합니다. 이렇게 하려면 자동으로 생성되고 하위에서 상속되는 필드 사이의 충돌을 피하기 위해 각 상위 모델에서 공통 조상으로 명시적 **OneToOneField**를 사용해야합니다.

```python
class Piece(models.Model):
    pass

class Article(Piece):
    article_piece = models.OneToOneField(Piece, on_delete=models.CASCADE, parent_link=True)
    ...

class Book(Piece):
    book_piece = models.OneToOneField(Piece, on_delete=models.CASCADE, parent_link=True)
    ...

class BookReview(Book, Article):
    pass
```

---

### Field name “hiding” is not permitted

일반적인 Python클래스 상속에서는 자식 클래스가 부모 클래스의 모든 특성을 재정의 할 수 있습니다. Django에서는 일반적으로 모델 필드에는 이것이 허용되지 않습니다. 비 추상적 모델 기본 클래스에 **author**라는 필드가있는 경우 해당 기본 클래스에서 상속하는 클래스에서 다른 모델 필드를 만들거나 **author**라는 속성을 정의 할 수 없습니다.

이 제한은 추상 모델에서 상속된 모델 필드에는 적용되지 않습니다. 이러한 필드는 다른 필드나 값으로 재정의 하거나 **field_name = None**을 설정하여 제거 할 수 있습니다.

> **Warning**
> 
> 모델 관리자는 추상 기본 클래스에서 상속됩니다. 상속 된 관리자가 참조하는 상속된 필드를 무시하면 미묘한 버그가 발생할 수 있습니다. [custom managers and model inheritance](https://docs.djangoproject.com/en/2.0/topics/db/managers/#custom-managers-and-inheritance)를 참조하십시오.

> **Note**
> 
> 일부 필드는 모델의 추가 속성을 정의합니다. **ForeignKey**는 필드 이름에 **\_id**가 추가 된 추가 속성과 외부 모델(foreign model)의 **related\_name**및 **related_query\_name**을 정의합니다.
>
> 이러한 추가 속성을 정의하는 필드가 변경되거나 제거되어 더 이상 추가 속성을 정의하지 않으면 이러한 추가 속성을 재정의 할 수 없습니다.

상위 모델의 필드를 재정의하면 새 인스턴스의 초기화(**Model.\_\_ init\_\_**에서 초기화 할 필드 지정) 및 직렬화와 같은 영역에서 어려움이 발생합니다. 이것들은 일반적인 파이썬 클래스 상속에서 똑같은 방식으로 처리할 필요가 없는 기능이므로 Django모델 상속과 파이썬 클래스 상속에서의 이러한 차이는 Django에서 제멋대로 바꾼 것이 아닙니다.

이 제한은 **Field**인스턴스인 속성에만 적용됩니다. 원하는 경우 일반 파이썬 속성은 재정의 할 수 있습니다. 또한 파이썬이 인식하는 속성의 이름에만 적용됩니다. 데이터베이스 열 이름을 수동으로 지정하는 경우 다중 테이블 상속에서는 자식 및 조상 모델에 같은 열 이름을 표시 할 수 있습니다(두 개의 서로 다른 데이터베이스 테이블에 열(column)들을 가집니다).

---

위 문단에서 

> 데이터베이스 열 이름을 수동으로 지정하는 경우 다중 테이블 상속에서는 자식 및 조상 모델에 같은 열 이름을 표시 할 수 있습니다  
> (두 개의 서로 다른 데이터베이스 테이블에 열(column)들을 가집니다)  
> **이 부분이 무슨 말인지 잘 이해가 안됩니다. 아시는 분 계시다면 댓글로 알려주시면 감사하겠습니다.**  
> **원본 문단은 아래와 같습니다.**

This restriction only applies to attributes which are Field instances. Normal Python attributes can be overridden if you wish. It also only applies to the name of the attribute as Python sees it: if you are manually specifying the database column name, you can have the same column name appearing in both a child and an ancestor model for multi-table inheritance (they are columns in two different database tables).

---

Django는 조상 모델에서 존재하는 모델 필드를 재정의하면 **FieldError**를 발생시킵니다.

---

## Organizing models in a package

**manage.py startapp** 명령은 **models.py**파일을 포함하는 애플리케이션(app) 구조를 만듭니다. 모델이 여러 개인 경우 별도의 파일로 구성하여 사용하는 것이 좋습니다.

그렇게하려면 **models**패키지를 만드십시오. **models.py**를 제거하고 **myapp/models/** 디렉토리를 만들고 **\_\_init\_\_.py**파일과 모델을 저장할 파일을 만듭니다. **\_\_init\_\_.py**파일에서 모델을 가져와야합니다.

예를 들어, **models**디렉토리에 **organic.py**와 **synthetic.py**가 있다면:

```python
# myapp/models/__init__.py
from .organic import Person
from .synthetic import Robot
```

**.models import \***를 사용하지 않고 명시적으로 각 모델을 가져오면 네임 스페이스가 복잡해지지 않아 코드를 읽기 쉽고 코드 분석 도구를 유용하게 유지할 수 있다는 이점이 있습니다.

> **See also**
> 
> [The Models Reference](https://docs.djangoproject.com/en/2.0/ref/models/)  
> 모델 필드, 관련 객체 및 **QuerySet**을 포함한 모든 모델 관련 API를 포함합니다.
