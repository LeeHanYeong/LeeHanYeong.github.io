---
layout: post
title:  "정적 파일을 기본값으로 갖는 ImageField구현"
categories: ['Django', 'Python']
---

장고에서 `FileField`와 `ImageField`는 실제 파일이 아닌 파일이 저장될 상대경로 문자열을 저장합니다. 이 때, 파일이 주어지지 않았을 경우를 대비해 `default`속성에 지정한 값은 동적으로 해당 경로를 찾는것이 아니라 해당 값을 필드에 문자열로 저장합니다.

`ImageField`를 사용할 때, 파일이 없는 경우 장고 프로젝트의 정적파일을 서빙하는 법에 대해 알아봅니다.

## ImageFieldFile 재정의

### 참고문서

- [FileField and FieldFile](https://docs.djangoproject.com/en/2.0/ref/models/fields/#filefield-and-fieldfile)
- [Finder Module](https://docs.djangoproject.com/en/2.0/ref/contrib/staticfiles/#finders-module)
- [STATICFILES_STORAGE](https://docs.djangoproject.com/en/2.0/ref/settings/#staticfiles-storage)
- [staticfiles_storage.url](https://docs.djangoproject.com/en/2.0/ref/files/storage/#django.core.files.storage.Storage.url)

`FieldFile`과 `ImageFieldFile`은 각각 모델 인스턴스의 `FileField`와 `ImageField`속성에 접근 시 파일에 액세스 하기 위한 프록시 역할을 합니다. 지금은 템플릿에서 필드의 `url`속성에 대해 접근하기 때문에 해당 속성을 재정의 해줍니다.

**`fields.py`**
```python
DEFAULT_IMAGE_PATH = 'django_fields/no_image.png'


class DefaultStaticImageFieldFile(ImageFieldFile):
    @property
    def url(self):
        try:
            # 파일이 존재한다면  부모의 url속성을 리턴합니다.
            return super().url
        except ValueError:
            # 파일이 없다면 ValueError가 발생합니다.
            from django.contrib.staticfiles.storage import staticfiles_storage
            from django.contrib.staticfiles import finders
            # staticfiles app의 finders를 사용해 FieldFile에 연결된 field의 
            # static_image_path값에 해당하는 파일이 있는지 검사합니다.
            if finders.find(self.field.static_image_path):
                # 존재할경우 staticfiles_storage의 url메서드를 사용해
                # 정적파일에 액세스 할 수 있는 URL을 반환합니다.
                return staticfiles_storage.url(self.field.static_image_path)
            # static_image_path값에 해당하는 경로에 파일이 없다면
            # 'django_fields/no_image.png'경로의 파일을 사용합니다.
            return staticfiles_storage.url(DEFAULT_IMAGE_PATH)
```

## ImageField 재정의

그리고 위에서 정의한 `DefaultStaticImageFieldFile`을 클래스 속성 `attr_class`로 사용하는 `ImageField`클래스를 상속을 이용해 정의해줍니다.

**`fields.py`**
```python
class DefaultStaticImageField(ImageField):
    # field에 접근 시 프록시로 사용할 필드파일 클래스를 지정합니다.
    attr_class = DefaultStaticImageFieldFile

    def __init__(self, *args, **kwargs):
        # 필드의 속성중 'default_image_path'키로 주어진 값을 가져와 인스턴스의 static_image_path값으로 할당합니다.
        # 이 때, 속성이 지정되어 있지 않으면 settings모듈의 'DEFAULT_IMAGE_PATH'속성의 값을 가져오며, 
        # 이것도 정의되어 있지 않다면 현재 모듈의 'DEFAULT_IMAGE_PATH'상수의 값을 사용합니다.
        self.static_image_path = kwargs.pop(
            'default_image_path',
            getattr(settings, 'DEFAULT_IMAGE_PATH', DEFAULT_IMAGE_PATH))
        # 추가 정의된 키워드인수의 값을 pop()으로 제거한 뒤, 부모의 초기화 메서드를 실행합니다.
        super().__init__(*args, **kwargs)
```

모듈의 최상단에 있는 `DEFAULT_IMAGE_PATH`속성의 값이 정적파일에 포함될 수 있도록 프로젝트의 `STATICFILES_DIRS`에 포함되는 경로에 `django_fields/no_image.png`파일을 추가해주어야 합니다.

## 사용법

위와 같이 커스텀 필드를 만든 후, 아래와 같이 사용합니다.

```python
class AnyModel(models.Model):
    # 필드의 속성에 사용할 정적 경로를 입력해주는 경우
    image1 = DefaultStaticImageField(blank=True, default_image_path='images/no_image.png')
    
    # settings.py에 정의된 DEFAULT_IMAGE_PATH 또는 fields모듈 최상단의 값을 사용
    image2 = DefaultStaticImageField(blank=True)
```

이 코드는 재사용하기 위해 [PyPI - django-default-imagefield](https://pypi.python.org/pypi/django-default-imagefield)와 [GitHub - Django-Default-ImageField](https://github.com/LeeHanYeong/Django-Default-ImageField) 에 업로드 되어 있습니다.

**`Terminal`**
```
pip install django-default-imagefield
```

위 명령어로 패키지 설치 후 `from django_fields import DefaultStaticImageField`문으로 필드를 불러와 사용할 수 있으며, 자세한 사용법은 [README](https://github.com/LeeHanYeong/Django-Default-ImageField/blob/master/README.rst)에 작성되어 있습니다.