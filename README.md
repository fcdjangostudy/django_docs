# Django Models
https://docs.djangoproject.com/en/1.11/topics/db/models/

## 질문리스트
* One-to-one 모델과 상속의 차이
* One-to-one 모델에서 parent_link ? 
* model manager가 뭔지? 어떤 개념인지? object?
* 중간 메서드는 관계되어 있는 두 모델과는 아예 다른 새 테이블로 보는 것이 맞는지?



## 데이터베이스, 테이블, 필드, 컬럼, 로우, 어트리뷰트
```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
```

위의예제에서....

| 데이터베이스|테이블|Person이라는 클래스
----|----|----
|필드|컬럼|first-name, last_name
|어트리뷰트|로우|이제 다른곳에서 할당될 인스턴스들...


## 개별 내용
### ForeignKey
class ForeignKey(othermodel, on_delete, **options)

#### Arguments
ForeignKey.on_delete

>* CASCADE : ForeignKey가 참조하는 테이블의 어튜리뷰트가 삭제되었을 때 자동으로 같이 삭제되는 옵션
>* PROTECT: ForeignKey가 참조하는 테이블의 어트리뷰트가 삭제되어도 남아있도록 하는 옵션


### null 과 blank
#### null
>null = True, 장고에서는 null은 기본적으로 False이나 이를 True로 바꾸면 빈값을 허용한다. 값이 들어가는데 아무것도 아닌 값이 들어간다.

#### blank
>blank = True, 이는 위의 null과 다르다. 이도 기본적으로는 False이나 이를 True로 바꾸면 아무것도 입력하지 않을 수 있다. 이는 유효성 검사와 관련이 있다.

### primary_key = True 인 경우
사용자가 임의로 프라이머리 키를 지정하였는데, 거기에 새로운 값을 지정하면 아예 새롭게 복사된 어트리뷰트가 생성된다.(기존 프라이머리키는 변경이 불가능 하다)


## 관계

### Many-to-one

한쪽만 여러관계를 가지는 경우

일대다 관계를 정의하려면 django.db.models.ForeignKey 클래스를 이용하여 필드를 선언하면 된다.

```python
from django.db import models

class Manufacturer(models.Model):
    # ...
    pass

class Car(models.Model):
    manufacturer = models.ForeignKey(Manufacturer)
    # ...
    
```

제조사는 여러개의 자동차를 가지지만, 자동차는 단한개의 제조사만 가지는 경우이다.

재귀적인 관계도 가능(자기 자신을 ForeignKey로 가리킴.


### Many-to-many 관계

다대다 관계를 선언할때는 ManyToManyField를 사용.  

```python
from django.db import models

class Topping(models.Model):
    # ...
    pass

class Pizza(models.Model):
    # ...
    toppings = models.ManyToManyField(Topping)
```
여러개의 피자가 여러개의 토핑을 가지는 경우이다. 

위의 toppings처럼 이름은 복수로 지어주는 것이 좋고, 두 모델 클래스 중 하나만 선언하면 된다. 왠만하면 하위요소라고 생각되는 곳에 선언하는 편이 좋다.

#### inerediate model(Extra fields on many-to-many relationships)

두 모델간의 관계가 추가적인 데이터를 가지는 경우

```python
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=128)

    def __str__(self):              # 호출시 나타남
        return self.name

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, through='Membership')

    def __str__(self):              # 호출시 나타남
        return self.name

class Membership(models.Model):
    person = models.ForeignKey(Person)
    group = models.ForeignKey(Group)
    date_joined = models.DateField()
    invite_reason = models.CharField(max_length=64)
```

Person과 Group에는 없는 추가적인 데이터를 처리하고 싶을 때 중간 모델을 사용한다. 위와 같이 한쪽에 ForeignKey를 넣을 때 옵션으로 throgh를 설정하고 중간모델 클래스에 두개의 ForeignKey를 넣는다.

위의 경우는 사람은 여러 그룹에 가입할 수 있고, 각 그룹은 여러명의 사람들을 회원 그런으로 갖는 경우인데, 그룹에 가입한 날짜를 저장하고 싶을 때 저렇게 설계한다.

#### 사용
중간모델은 직접할당이 불가하다. (add, create, remove 등)

```python
# THIS WILL NOT WORK
beatles.members.add(john)
# NEITHER WILL THIS
beatles.members.create(name="George Harrison")
# AND NEITHER WILL THIS
beatles.members = [john, paul, ringo, george]
```

그러나 clear()메서드는 사용가능

```python
>>> # Beatles have broken up
>>> beatles.members.clear()
>>> # Note that this deletes the intermediate model instances
>>> Membership.objects.all()
[]
```

### One-to-one 관계
일대일 관계를 정의하려면, OneToOneField를 이용
전형적으로는 상속하는 개념과 비슷하다.

```python
from django.db import models

class Place(models.Model):
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)

    def __str__(self):              # 
        return "%s the place" % self.name

class Restaurant(models.Model):
    place = models.OneToOneField(
        Place,
        on_delete=models.CASCADE,
        primary_key=True,
    )
    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)

    def __str__(self):              # 
        return "%s the restaurant" % self.place.name

class Waiter(models.Model):
    restaurant = models.ForeignKey(Restaurant, on_delete=models.CASCADE)
    name = models.CharField(max_length=50)

    def __str__(self):              # 
        return "%s the waiter at %s" % (self.name, self.restaurant)
```
장소는 장소명과 주소를 가지는데, 이러한 속성을 식당도 같이 가지고자 할 경우 일대일 관계로 연결할 수 있다.

옵션으로 parent_link를 가질 수 있다. 이 옵션이 True이면 ???

### 다른 모델 파일들 import하기 

```python
from django.db import models
from geography.models import ZipCode

class Restaurant(models.Model):
    # ...
    zip_code = models.ForeignKey(ZipCode)
```
다른 앱에 선언된 모델과 관계를 가질 수 있다.

### 모델의 field name 제약
1. 파이썬 예약어는 사용 불가 (SQL예약어는 사용가능 eg.join, where, select 등)

2. 밑줄 두개는 연속으로 사용 불가


## 모델단위(class단위) 옵션
### Meta option


### 모델 기본 어트리뷰트
Manager라는 기본 어트리뷰트가 있는데, 이름이 object이다.
 Manager 객체는 모델 클래스 선언을 기반으로 실제 데이터베이스에 대한 쿼리 인터페이스를 제공하며, 데이터베이스 레코드를 모델 객체로 인스턴스화 하는데 사용된다.
 
 이 외에도
 ~~_set : ~~는 모델명, 관계된 모델들을 엑세스 할 수 있도록 해줌.

### 모델 클래스 메서드
레코드(row)단위로 기능을 구현할 때 구현, 

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
```

"테이블 단위"의 기능의 경우에는 Manager에 구현

### Overriding(기본 메서드 커스터마이징)

기본적으로 제공되는 메서드 커스터마이징 할 일이 많단다. 예를들어 save()나 delete()같은 것들......

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
여기서 핵심은 super()로 부모 메서드 호출해서 진짜 save를 불러와 주는 거다.


## 상속(inheritance)

### Abstract class
부모클래스는 실제 테이블을 만들지 않고, 자식만 만들어 관리하는 경우
부모클래스는 테이블을 만들지도 않고, Maneger도 가지지도 않으며, 인스턴스화도 되지 않는다. 오로지 상속을 통해서만 발현된다.

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
결과적으로 Student 모델은 3개의 필드(name, age, home_group)를 가지게 된다.

#### 부모의 Meta class 상속
부모가 메타클래스가 있는 경우

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
#### 여러개를 상속했을 때 기본 어트리뷰트 역참조 중복 해결

```python
# common/models.py

from django.db import models

class Base(models.Model):
    m2m = models.ManyToManyField(OtherModel, related_name="%(app_label)s_%(class)s_related")

    class Meta:
        abstract = True

class ChildA(Base):
    pass

class ChildB(Base):
    pass
```

```python
# rare/models.py:

from common.models import Base

class ChildB(Base):
    pass
```
위와 같이 같은 클래스명으로 중복 상속 되었을 경우 기본 어트리뷰트의 중복이 발생한다. 예를들면 childB_set()등과 같은.....
그러나 위와 같이 선언해주면
 common_childa_related, common_childb_related
 rare_childb_related
 로 각기 다른 이름의 기본 어트리뷰트들이 생성이 되어 중복을 피할 수 있다.
 

### Multi-table 상속
Multi-table 상속은 부모 모델이든 자식 모델이든 모두 각자의 데이터베이스 테이블을 가진다. 즉, ***공통부분의 데이터는 부모모델의 테이블에 저장***하고, 자식모델의 데이터는 자식모델의 테이블에 저장되며, 자식 모델은 부모 모델에 대한 링크를 가진다. 이때, 링크는 내부적으로 OneToOneField를 사용.

```python
from django.db import models

class Place(models.Model):
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)

class Restaurant(Place):
    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)


>>> p = Place.objects.create(name="Bob's Cafe", address="Bob's Street")
>>> p.restaurant  # Restaurant.DoesNowExist 예외 발생
```
부모 자식간에 서로 독립적이다. 자식만 부모 유전자 가지고 간다.

#### Meta and multi-table inheritance
부모의 메타속성은 multi-table 상속에서는 기본적으로 상속되지 않는다.

이와같은 이유로, multi-table 상속의 경우 기본적으로 자식모델은 부모모델의 Meta 클래스를 상속받지 않는다. 단, 자식모델에 ordering, get_latest_by 옵션이 지정되지 않은 경우에 한해, 부모 모델의 해당 설정값을 상속받는다

### Proxy 상속
multi-table 상속을 사용하는 경우, 각각의 자식 클래스들 마다 테이블이 생성된다. 부모 모델이 가지는 데이터 이외에 추가적으로 자식 모델이 가지는 데이터를 저장하기 위한 당연한 동작입니다. 하지만 가끔은 부모 모델의 메서드(파이썬 코드)만 재정의 한다던가 새로 추가하고 싶을 수도 있습니다. 즉, 테이블을 가지는 모델을 상속받되, 자식모델은 테이블을 만들필요가 없는 경우에 해당됩니다.

이런 경우에 proxy 모델 상속을 사용합니다. 이 경우 proxy 모델(자식모델)을 이용해서 모델객체를 만들고, 수정하고 삭제할 수도 있습니다. 물론 이러한 동작은 원본(부모)모델의 테이블 상에서 이루어 집니다. 

proxy 모델이 원본모델과 다른 점은 기본 정렬값과 같은 설정값을 원하는 데로 변경할 수 있다는 점입니다. 물론 원본 모델의 설정값은 변경되지 않습니다.

Proxy 모델 선언은 일반 모델들과 동일합니다. 다만, Meta 클래스에 proxy=True 로 선언해주면 됩니다

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
MyPerson 클래스는 부모 클래스인 Person 클래스의 테이블을 사용합니다. 그렇기 때문에, 아래와 같이 Person 모델로 레코드를 추가하더라도 MyPerson 클래스로 조회가 가능합니다.

```python
>>> p = Person.objects.create(first_name="foobar")
>>> MyPerson.objects.get(first_name="foobar")
<MyPerson: foobar>
```
그러나 proxy 모델에서 변경한 사항은 부모 모델에 영향을 끼치지 않는다.

### Proxy model managers
Proxy 모델에 manager를 지정하지 않는 경우, 부모 모델의 manager를 상속받습니다. 직접 지정한 경우, 지정한 manager가 디폴트로 사용되며, 부모클래스의 manager도 사용할 수 있다.

### Multiple inheritance


### Field name “hiding” is not permitted
필드 어트리뷰트는 오버라이딩이 허용되지 않는다. 파이썬 어트리뷰트는 원한다면 오버라이드 가능하다.


















