# Django ORM Lookup
## Ссылки

- [PostgreSQL specific model fields](https://github.com/Xpom1/Articles/tree/main/Articles/Django%20ORM%20lookups#postgresql-specific-model-fields)
- [HStoreField](https://github.com/Xpom1/Articles/tree/main/Articles/Django%20ORM%20lookups#hstorefield)
- [F() и Q()](https://github.com/Xpom1/Articles/tree/main/Articles/Django%20ORM%20lookups#f-%D0%B8-q)
---
Всех приветствую, сегодня мы будем обсуждать Django ORM Lookup.
У нас есть 2 заполненные таблицы, также показаны их модели, чтобы было проще.

```python
class CommonInfo(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()

    class Meta:
        abstract = True

    def __str__(self):
        return f'{self.name}'


class Student(CommonInfo):
    home_group = models.ForeignKey('GroupInfo', on_delete=models.CASCADE, related_name='ginfo')


class GroupInfo(models.Model):
    info = models.CharField()
    body = models.PositiveIntegerField()
```

- CommonInfo + Student

| id |   name   | age | home_group_id |
|:--:|:--------:|:---:|:-------------:|
| 1  |  Антон   | 19  |       1       |
| 2  |  Валера  | 39  |       2       |
| 3  | Геннадий | 24  |       2       |
| 4  | Вероника | 20  |       3       |
| 5  |  Прохар  | 18  |       3       |

- GroupInfo

| id |    info     | body |
|:--:|:-----------:|:----:|
| 1  | Корпус МИФИ | 314  |
| 2  | Корпус МАИ  | 512  |
| 3  | Корпус МАДИ | 784  |

---

### Exact

exact - это лукап по-умолчанию. Запрос вида “.filter(name='var')” не отличается от “.filter(name__exact='var')”. Но это
только для PostgreSQL.

```python
>> > Student.objects.filter(home_group_id__exact=2)
< QuerySet[ < Student: Валера >, < Student: Геннадий >] >

>> > Student.objects.filter(home_group_id=2)
< QuerySet[ < Student: Валера >, < Student: Геннадий >] >
```

### Iexact

iexact - Лукап который помогает искать нужно слово, не задумываясь о его регистре.

```python
>> > Student.objects.filter(name__iexact='антоН')
< QuerySet[ < Student: Антон >] >
```

### Contains

contains - Ищет точное совпадение в тексте.

```python
>> > GroupInfo.objects.filter(info__contains='МАДИ')
< QuerySet[ < GroupInfo: Корпус
МАДИ >] >
```
icontains - Работает так же как contains, но ему не важен регистр.
```python
>> > GroupInfo.objects.filter(info__icontains='мади')
< QuerySet[ < GroupInfo: Корпус
МАДИ >] >
```

### gt, gte, lt, lte

gt - больше

```python
>> > Student.objects.filter(home_group_id__gt=2)
< QuerySet[ < Student: Вероника >, < Student: Прохар >] >
```

gte - больше или равно

lt - меньше

lte - меньше или равно

### Isnull

isnull - Проверяет есть ли значение у поля, если есть то возвращает те поля, у которых что-то заполнено.

```python
>> > Student.objects.filter(home_group_id__isnull=True)
< QuerySet[] >
>> > Student.objects.filter(home_group_id__isnull=False)
<
QuerySet[ < Student: Антон >, < Student: Валера >, < Student: Вероника >, < Student: Прохар >, < Student: Геннадий >] >
```

### In

in - Ищет те поля, у которых значения входят в заданный список.

```python
>> > Student.objects.filter(home_group_id__in=(1, 3))
< QuerySet[ < Student: Антон >, < Student: Вероника >, < Student: Прохар >] >
```

### Range

range - Ищет те поля, у которых значения входят в заданный диапазон.

```python
>> > Student.objects.filter(age__range=(20, 100))
< QuerySet[ < Student: Валера >, < Student: Вероника >, < Student: Геннадий >] >
```

### Date

```python
>> > Entry.objects.filter(pub_date__date=datetime.date(2005, 1, 1))
>> > Entry.objects.filter(pub_date__date__gt=datetime.date(2005, 1, 1))
```

---
С помощью лукапов можно делать сложные запросы к DB
Запрос который выдает пользователей в чьих группах есть упоминание “МАИ”

```python
>> > Student.objects.filter(home_group__info__icontains='маи')
< QuerySet[ < Student: Валера >, < Student: Геннадий >] >
```

# PostgreSQL specific model fields

```python
from django.contrib.postgres.fields import ArrayField


class Pizza(models.Model):
    name = models.CharField()
    ingredients = ArrayField(models.CharField(), blank=True)

    def __str__(self):
        return self.name
```

### ArrayField

ArrayField - это тип поля, в которое можно записать список со значениями.
```python
Pizza.objects.create(name='Маргарита', ingredients=['Томатный соус', 'моцарелла', 'орегано'])
Pizza.objects.create(name='Карбонара', ingredients=['Томатный соус', 'моцарелла', 'пармезан', 'яйца', 'бекон'])
Pizza.objects.create(name='Крудо', ingredients=['Томатный соус', 'пармская ветчина'])
```

```python
Pizza.objects.filter(ingredients__contains=['Томатный соус', 'моцарелла'])
<QuerySet [<Pizza: Маргарита>, <Pizza: Карбонара>]>

Pizza.objects.filter(ingredients__contained_by=['Томатный соус', 'орегано', 'моцарелла'])
<QuerySet [<Pizza: Маргарита>]>
```

Есть лукап contains, он проверяет, что переданное множество, является подмножеством у значений в БД, а есть contained_by, это обратная операция, она проверяет, что значения в БД, являются подмножеством у переданных значений

```python
Post.objects.filter(tags__contained_by=["thoughts", "django"])
первая запись — ["thoughts", "django"] является подмножеством ["thoughts", "django"]
вторая запись — ["thoughts"] является подмножеством ["thoughts", "django"]
третья запись — ["tutorial", "django"] НЕ является подмножеством ["thoughts", "django"]

Post.objects.filter(tags__contained_by=["thoughts", "django", "tutorial"])
первая запись — ["thoughts", "django"] является подмножеством ["thoughts", "django", "tutorial"]
вторая запись — ["thoughts"] является подмножеством ["thoughts", "django", "tutorial"]
третья запись — ["tutorial", "django"] является подмножеством ["thoughts", "django", "tutorial"]
```

Чтобы найти ингредиенты не смотря на их регистр нужно будет делать двойную фильтрацию, подобно этой:

```python
Pizza.objects.filter(ingredients__icontains='томатный соус').filter(ingredients__icontains='Моцарелла')
<QuerySet [<Pizza: Маргарита>, <Pizza: Карбонара>]>
```

Тк если делать такой запрос, то мы ничего не получим. 

```python
Pizza.objects.filter(ingredients__icontains=['томатный соус', 'моцарелла'])
<QuerySet []>
```

Причина заключается в обработке данного запроса самой SQL, у нее нет возможности найти LIKE IN. Поскольку наш запрос выглядит как 

```mysql
... WHERE "pizza"."ingredients"::text LIKE %['томатный соус', 'моцарелла']%
``` 

Помимо двойной фильтрации эту проблему можно решить с помощью [regex](https://docs.djangoproject.com/en/4.2/ref/models/querysets/#iregex), [SearchQuery](https://docs.djangoproject.com/en/4.2/ref/contrib/postgres/search/) или [Trigram](https://docs.djangoproject.com/en/4.2/ref/contrib/postgres/search/#trigram-similarity).

### Overlap
overlap - метод выводит объекты, у которых есть хотя бы 1 совпадение.
```python
Pizza.objects.filter(ingredients__overlap=['пармезан', 'орегано', ])
<QuerySet [<Pizza: Маргарита>, <Pizza: Карбонара>]>
```

## HStoreField

Если мы можем полем сделать список, скорее всего мы можем сделать полем и словарь, так как же это сделать? 

Для начала нам надо правильно настроить все.

Чтобы использовать словарь, вам надо:
Добавить 'django.contrib.postgres' в ваше INSTALLED_APPS.
[Настройка hstore расширения](https://docs.djangoproject.com/en/4.2/ref/contrib/postgres/operations/#create-postgresql-extensions) в PostgreSQL

Вы увидите ошибку, в которой будет сказано, что невозможно добавить словарь, если вы пропустите первичную настройку. 

Теперь перейдем к практике, вот такая модель у нас есть.

```python
class Dog(models.Model):
    name = models.CharField(max_length=200)
    data = HStoreField()

    def __str__(self):
        return self.name
```

Для начала нам надо создать объекты.

```python
Dog.objects.create(name="Rufus", data={"breed": "labrador"})
Dog.objects.create(name="Meg", data={"breed": "collie"})
```

Теперь мы можем с помощью лукапов искать нужные нам данные:

```python
Dog.objects.filter(data__breed="collie")
<QuerySet [<Dog: Meg>]>

>>> from django.db.models import F
>>> rufus = Dog.objects.annotate(breed=F("data__breed"))[0]
>>> rufus.breed
'labrador'
```

Так же мы можем искать и полное совпадение:

```python
>>> Dog.objects.create(name="Rufus", data={"breed": "labrador", "owner": "Bob"})
>>> Dog.objects.create(name="Meg", data={"breed": "collie", "owner": "Bob"})
>>> Dog.objects.create(name="Fred", data={})

>>> Dog.objects.filter(data__contains={"owner": "Bob"})
<QuerySet [<Dog: Rufus>, <Dog: Meg>]>
```

Если вам надо найти определенный ключ, то сделать это можно так:

```python
>>> Dog.objects.create(name="Rufus", data={"breed": "labrador"})
>>> Dog.objects.create(name="Meg", data={"breed": "collie", "owner": "Bob"})

>>> Dog.objects.filter(data__has_key="owner")
<QuerySet [<Dog: Meg>]>


Dog.objects.filter(data__values__icontains="coLLie")
<QuerySet [<Dog: Meg>]>
Dog.objects.filter(data__values__contains=["collie"])
<QuerySet [<Dog: Meg>]>
```

# F() и Q()

### F()

F() - Экземпляры F() действуют как ссылка на поле модели в запросе. Затем эти ссылки можно использовать в фильтрах запросов для сравнения значений двух разных полей в одном экземпляре модели.

```python
>>> from django.db.models import F
>>> Entry.objects.filter(number_of_comments__gt=F("number_of_pingbacks"))


>>> Entry.objects.filter(number_of_comments__gt=F("number_of_pingbacks") * 2)


>>> Entry.objects.filter(authors__name=F("blog__name"))
```

### Q()

Сложный поиск с использованием Q объектов
Запросы с аргументом ключевого слова – в filter() и т.д. – это “И”, написанные вместе. Если вам нужно выполнять более сложные запросы (например, запросы с операторами OR), вы можете использовать Q objects.

Объект Q (django.db.models.Q) - это объект, используемый для инкапсуляции набора аргументов ключевого слова. Эти аргументы ключевого слова указаны так же, как в разделе “Поиск по полю” выше.

Например, этот объект инкапсулирует один подобный запрос:

```python
from django.db.models import Q

Student.objects.filter(Q(home_group_id=2), Q(age__gte=24))
<QuerySet [<Student: Валера>, <Student: Геннадий>]>
```
Этот запрос эквивалентен 

```mysql
WHERE home_group_id = 2 AND age >= 24
```

```python
Student.objects.filter(Q(home_group_id=2) | Q(age__gte=20))
<QuerySet [<Student: Валера>, <Student: Вероника>, <Student: Геннадий>]>
```

Этот запрос эквивалентен 
```mysql
WHERE home_group_id = 2 OR age >= 20
```



