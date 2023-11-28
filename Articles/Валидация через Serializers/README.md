# Validators

## UniqueValidator

Для того чтобы продемонстрировать валидацию данных в serializers, нам нужно создать модель:

```python
# models.py
from django.db import models


class Film(models.Model):
    name = models.CharField(max_length=100, unique=True)
    author = models.CharField(max_length=100)
    published = models.DateField()
```

Также нужно создать serializers.py:

```python
from rest_framework import serializers
from rest_framework.validators import UniqueValidator
from .models import Film


class FilmSerializer(serializers.ModelSerializer):
    name = serializers.CharField(validators=[UniqueValidator(queryset=Film.objects.all())])

    class Meta:
        model = Film
        fields = ['name', 'author', 'published']
```

Здесь мы будем сразу проверять, чтобы в нашей базе данных не было повторяющихся полей name.

POST запрос:

```json
{
  "name": "Star Wars",
  "author": "George Lucas",
  "published": "1977-05-25"
}
```

Ответ(Успешный):

```json
{
  "name": "Star Wars",
  "author": "George Lucas",
  "published": "1977-05-25"
}
```

POST запрос:

```json
{
  "name": "Star Wars",
  "author": "Another Author",
  "published": "1980-05-17"
}
```

Ответ(Ошибка):

```json
{
  "error": "This field must be unique."
}
```

Также мы можем делать кастомные валидации данных. Например, мне хочется, чтобы в именах фильмов не было слово “cow”,
тогда мы можем добавить функцию, которая будет проверять это:

```python
    def validate_name(self, value):
    if "cow" in value.lower():
        raise serializers.ValidationError("Film name cannot contain 'cow'")
    return value
```

POST запрос:

```json
{
  "name": "Cowboy Adventure",
  "author": "John Doe",
  "published": "2000-01-01"
}
```

Ответ(Ошибка):

```python
{
    "error": "Film name cannot contain 'cow'"
}
```

## UniqueTogetherValidator

Для дальнейших демонстраций, нам потребуется модифицировать нашу модель:

```python
from django.db import models


class Film(models.Model):
    name = models.CharField(max_length=100, unique=True)
    author = models.CharField(max_length=100)
    published = models.DateField()
    genre = models.CharField(max_length=50)

    class Meta:
        unique_together = ('author', 'genre')
```

Также преображаем нам serializers.py:

```python
from rest_framework.validators import UniqueTogetherValidator


class FilmSerializer(serializers.ModelSerializer):
    class Meta:
        model = Film
        fields = ['name', 'author', 'published', 'genre']
        validators = [
            UniqueTogetherValidator(
                queryset=Film.objects.all(),
                fields=['author', 'genre']
            )
        ]
```

POST запрос:

```json
{
  "name": "Dunkirk",
  "author": "Christopher Nolan",
  "published": "2017-07-21",
  "genre": "War"
}
```

Ответ(Успешный):

```json
{
  "name": "Dunkirk",
  "author": "Christopher Nolan",
  "published": "2017-07-21",
  "genre": "War"
}
```

POST запрос:

```json
{
  "name": "Another Film",
  "author": "Christopher Nolan",
  "published": "2018-09-22",
  "genre": "War"
}
```

Ответ(Ошибка):

```json
{
  "error": "The field's author, genre must make a unique set."
}
```

## UniqueForYearValidator

Если мы хотим проверять на уникальность год, то можно это сделать с помощью UniqueForYearValidator:

```python
from rest_framework import serializers
from rest_framework.validators import UniqueForYearValidator
from .models import Film


class FilmSerializer(serializers.ModelSerializer):
    class Meta:
        model = Film
        fields = ['name', 'author', 'published', 'genre']
        validators = [
            UniqueForYearValidator(
                queryset=Film.objects.all(),
                field='name',
                date_field='published'
            )
        ]
```

POST запрос:

```json
{
  "name": "Interstellar",
  "author": "Christopher Nolan",
  "published": "2014-11-07",
  "genre": "Sci-Fi"
}
```

Ответ(Успешный):

```json
{
  "name": "Interstellar",
  "author": "Christopher Nolan",
  "published": "2014-11-07",
  "genre": "Sci-Fi"
}
```

POST запрос:

```json
{
  "name": "Interstellar",
  "author": "Another Director",
  "published": "2014-12-01",
  "genre": "Adventure"
}

```

Ответ(Ошибка):

```json
{
  "error": "This field must be unique for the year 2014."
}
```
