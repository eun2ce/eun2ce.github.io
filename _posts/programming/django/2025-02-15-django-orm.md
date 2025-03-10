---
title: "[django] sql 을 이해하고 django ORM 사용하기"
date: 2025-02-15 20:03:00 +0900
categories: [ "programming", "django" ]
tags: [ "community", "django", "python", "programming", "prefetch_related", "select_related" ]
pin: false
math: false
mermaid: false
---

django ORM 사용할 때 query 개수를 줄이는 방법 중 select_related, prefetch_related 에 대해 알아봅니다.

select_related, prefetch_related 는 하나의 queryset 을 가져올 때, 미리 related objects 까지 불러오는 함수입니다.

# django rest framework 를 위한 json 직렬화

## ModelSerializer 를 통한 json 직렬화

ModelSerializer 를 통해 JsonRenderer 에서 변환 가능한 형태로 데이터를 변환합니다.  
Serializer 는 장고의 Form 과 유사하며 Form 은 **html**을 생성하고 Serializer 는 **json 문자열**을 생성한다는 차이가 있습니다.

## view 에서의 json 응답

json 포맷으로 직렬화 된 문자열은 view 에서 클라이언트로 넘격주어야 합니다. 넘기는 방식에는 두 가지가 있습니다.

* json.dumps 를 통해 직렬화한 문자열을 HttpResponse 를 통해 응답
* json.dumps 가 내장되어있는 JsonResponse 를 통해 응답

이때 JsonResponse 는 장고의 DjangoJSONEncoder 를 내장하는데, 이는 QuerySet 에 대한 직렬화를 지원하지 않습니다.
즉, 커스텀하여 사용해야 합니다.

# Serializers 에 대해

## Serializer 사용하기

[django-rest-framework serializers](https://www.django-rest-framework.org/api-guide/serializers/)

각 과정을 잘 알아두는 것이 좋습니다.

### Serializing objects (GET)

객체를 가져와 데이터로 뿌려줍니다.

```python
serializer = CommentSerializer(comment)
seriallizer.data
```

## Deserializing objects (POST, PUT, PATCH)

데이터를 받아서 DB에 저장할 수 있도록합니다.

1) 데이터 유효성 검사
2) python dictionary 형태로 변환

```python
serializer = CommentSerializer(data=data)
serializer.is_valid()
# True
serializer.validated_data
# {'content': 'foo bar', 'email': 'leila@example.com', 'created': datetime.datetime(2012, 08, 22, 16, 20, 09, 822243)}
```

## Saving instances

Deserializing 된 object 를 DB 에 저장 합니다.

```python
comment = serializer.save()
# .save() will create a new instance.
serializer = CommentSerializer(data=data)
serializer.save()

# .save() will update the existing `comment` instance.
serializer = CommentSerializer(comment, data=data)
serializer.save()
```

### passing additional attributes to .save()

saving instances 시 추가로 전달하고 싶은 정보가 있는 경우 아래와 같이 함께 정보를 전달할 수 있습니다.

```python
serializer.save(owner=request.user)
```

### overriding .save() directly

객체를 생성 할 필요 없이 단순히 보내기만 하는 로직이 필요하다면 .save()를 오버라이드 해서 로직을 수정해 줄 필요가 있습니다.

```python
class ContactForm(serializers.Serializer):
    email = serializers.EmailField()
    message = serializers.CharField()

    def save(self):
        email = self.validated_data['email']
        message = self.validated_data['message']
        send_email(from=email, message=message)
```

## Serializer Fields

[https://www.django-rest-framework.org/api-guide/fields/](https://www.django-rest-framework.org/api-guide/fields/)

구체적인 필드 종류는 공식문서를 사용하지만, 주로 사용하는 serializer 의 속성은 잘 이해하고 있어야 합니다.

### Core Argument

| 속성명                | 기본값        | 설명                                                        |
|--------------------|------------|-----------------------------------------------------------|
| **read_only**      | `False`    | 읽기 전용 필드로 설정 (클라이언트에서 수정 불가)                              |
| **write_only**     | `False`    | 쓰기 전용 필드로 설정 (클라이언트에서 읽을 수 없음)                            |
| **required**       | `True`     | 필수 입력 필드 (기본적으로 값이 반드시 필요)                                |
| **allow_null**     | `False`    | `null` 값을 허용 (`None` 저장 가능)                               |
| **source**         | 모델 필드명과 동일 | 모델 필드명을 다르게 매핑할 때 사용 (ex: `source="user.username"`)       |
| **validators**     | -          | 사용자 정의 검증 로직을 추가 (ex: `validators=[MyCustomValidator()]`) |
| **error_messages** | -          | 커스텀 오류 메시지 지정 (ex: `{ "required": "필수 입력 항목입니다." }`)      |

### 간단한 예제

```python
from rest_framework import serializers
from myapp.models import UserProfile

class UserProfileSerializer(serializers.ModelSerializer):
    username = serializers.CharField(read_only=True)  # 읽기 전용
    email = serializers.EmailField(required=True)  # 필수 입력
    phone = serializers.CharField(write_only=True)  # 쓰기 전용
    age = serializers.IntegerField(allow_null=True)  # null 허용
    full_name = serializers.CharField(source="user.get_full_name")  # 다른 필드 매핑
    email = serializers.EmailField(
        validators=[my_custom_validator],  # 사용자 정의 검증기 추가
        error_messages={"required": "이메일은 반드시 입력해야 합니다!"}  # 오류 메시지 지정
    )

    class Meta:
        model = UserProfile
        fields = ["username", "email", "phone", "age", "full_name"]
```

# Function based CRUD

rest 프레임워크는 api 뷰를 만드는데 사용할 수 있는 2가지 wrapper 를 제공합니다

* 함수형을 위한 @api_view
* 클래스형을 위한 APIView

wrapper 에서는 뷰에서 request 인스턴스를 수신하고, 해당 메서드를 인자로 전달하여 해당 메서드에 맞는 로직이 실행될 수 있도록 합니다.
즉, @api_view 는 클래스형 뷰의 as_view()처럼 여러가지 유형의 메서드를 함수형에서도 처리할 수 있도록 돕는다고 생각하면 편합니다.

## endpoint 구성 (urls.py)

엔드포인트는 주로 pk 정보가 필요없는 list, create 를 수행하는 뷰와 pk 정보가 필요한 detail, update, delete 를 수행하는 view 두가지 정도가 될 수 있습니다.

# ModelSerializer

앞서 보았던 Serializer 는 각 필드를 하나씩 정의해주어야 했습니다. 그러한 번거로움을 해결하기 위해 나온 것이 ModelSerializer 입니다.
ModelSerializer 는 크게 아래와 같은 기능 세가지를 제공합니다.

* (의존하는 모델에 기반하여) Serializer 를 만들어 줍니다.
* Serializer 를 위한 Validator 를 제공합니다. e.g.)unique_together_validators
* create, update 함수를 제공하기때문에 별도로 만들 필요가 없습니다.

## 사용법

### class Meta 작성

```python
class JournalistSerializer(serializers.ModelSerializer):
    class Meta:
        model = Journalist # 모델명
        fields = '__all__' # all외에도 exclude, 직접명시('id','email')
        # read_only_field = ['id']
```

### serializer 로 정의해야 하는 필드

* 추가하고자 하는 필드가 있는 경우, `serializer.SerializerMethodField()`로 정의
* ForeginKey 로 연결된 필드가 있는 경우
  * default 는 ForeginKey 로 연결 된 필드의 pk 를 반환
  * 만약 해당 데이터의 String 이나 Hyperlink 를 가져오고 싶다면 별도로 지정

```python
class ArticleSerializer(serializers.ModelSerializer):

    time_since_publication = serializers.SerializerMethodField()

    class Meta:
        model = Article
        fields = "__all__" # Article 모델의 모든 필드 시리얼라이즈
    
    def get_time_since_publication(self, object):
        publication_date = object.publication_date
        now = datetime.now(timezone.utc)   # 에러 해결. timezone 임포트한 뒤 추가
        time_delta = timesince(publication_date, now)
        return time_delta
```

## Nested RelationShips

앞서 언급했던 것 처럼 ForeignKey 필드를 사용하고자 할 때, 기본적으로 아무 설정이 없다면 참조하고 있는 pk 값을 가져옵니다.
만약 pk 값 이외의 다른 값을 가져오고 싶다면 아래와 같은 메서드를 이용할 수 있습니다.

### String of related object (StringRelatedField)

* FK 로 연결 된 모델의 `__str__` 메서드에서 정의한 string 을 리턴
* 인자로 many=True, read_only=True 등을 가집니다

```python
class ArticleSerializer(serializers.ModelSerializer):
    ...
    author = serializers.StringRelatedField() 
```

> 자세한 사항은 [이 곳](https://www.django-rest-framework.org/api-guide/serializers/#customizing-field-mappings)을 참고해주세요.
{: .prompt-info}

# GenericAPIView 와 Mixins

* drf 에서는 GenericAPIView 에 list, create 등 다양한 믹스인 클래스를 결합해 APIView 를 구현할 수 있습니다.
* GenericAPIView 는 CRUD 에서 **공통적으로 사용되는 속성**을 제공하고, Mixin 은 **CRUD 중 특정 기능을 수행하는 메서드**를 제공합니다.
* GenericAPIView 와 Mixins 으로 정확한 기능 구현이 어려운 경우, 커스터마이징 하여 사용할 수 있습니다.

## GenericAPIView

```python
from rest_framework import generics
from rest_framework import mixins

from ebooks.models import Ebook
from ebooks.api.serializers import EbookSerializer

class EbookListCreateAPIView(mixins.ListModelMixin,
                             mixins.CreateModelMixin,
                             generics.GenericAPIView):

    queryset = Ebook.objects.all()
    serializer_class = EbookSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```

## basic settings

다음 속성들을 통해 view 를 컨트롤 합니다.

* queryset: view 에서 객체를 반환하는 데 사용해야하는 쿼리셋
  * 반드시
    1) queryset 속성을 설정하거나
    2) get_queryset() 메서드로 override 하여 사용해야 합니다
* serializer_class: 입력된 값을 validate 하거나 deserialize 하거나, 출력값을 serialize 할 때 사용하는 serializer 클래스입니다.
  * 일반적으로
    1) 이 속성을 설정하거나
    2) get_serializer_class() 메서드로 override 하여 사용해야 합니다.
* lookup_field: 개별 모델 인스턴스의 object 조회를 수행할 때 사용해야 하는 모델 필드 입니다.
  * 기본 값은 `pk` 하이퍼링크 된 api 에 custom 값을 사용해야 하는 경우 api views 와 serializer 클래스가 lookup field 를 설정해야 합니다.

## pagination

### pagination_class

리스트 결과들을 페이지네이션 할 때 사용합니다.
default 는 DEFAULT_PAGINATION_CLASS(`rest_framework.pagination.PageNumberPagination` 모듈안에) 세팅으로 결정됩니다.

## filtering

### filter_backends

> A list of filter backend classes that should be used for filtering the queryset. Defaults to the same value as the
> DEFAULT_FILTER_BACKENDS setting.

## 주요 mixins

django 에서 mixins 를 사용하지 않는다면 모든 기능을 view 에 직접, 반복적으로 구현해야 합니다.
하지만 api 를 작업할 때 목록을 보여주거나, CRUD 등은 항상 사용되고 반복적인 일이 일어납니다.

따라서 이런 반복적인 기능을 하나의 Mixin 클래스로 제공한다면 반복적인 일은 줄어들고 가독성, 생산성을 높여줄 수 있습니다.

단, mixin 클래스에 존재하는 메서드나 속성을 상속받는 클래스에서 사용할 경우 mixin 클래스의 메서드가 오버라이딩되어 의도치 않게 작동할 수 있으니 주의해야 합니다.

* ListModelMixin
  * queryset 을 listing 하는 메서드
  * `.list(request, *args, **kargs)` 메서드 호출로 사용
* CreateModelMixin
  * 모델 인스턴스를 생성하고 저장하는 mixin
* RetrieveModelMixin
  * 존재하는 모델 인스턴스를 리턴
* UpdateModelMixin
  * 모델 인스턴스를 수정하여 저장
* DestroyModelMixin
  * 모델 인스턴스를 삭제

> 더 많은 내용을 알고싶다면 [이 곳](https://www.django-rest-framework.org/api-guide/generic-views/#mixins)을 참고하세요. 
{: .prompt-info }

> Qna  
> view > APIView > GenericAPIView 는 무슨 차이가 있는가 ?
> * view: django 기본 view 로 json 처리 기능이 없고 html 을 반환해줍니다.
> * APIView: DRF 의 기본 view 로 request, response 객체 사용이 가능하고 인증/권한/스크롤링을 지원합니다.
> * GenericAPIView: APIView 의 확장으로 queryset, serializer_class get_object() get_queryset() 등과 같은 헬퍼 메서드를 지원합니다.
