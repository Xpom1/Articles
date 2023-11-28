Для начала нам нужно создать Models:
```python
#models.py
class Post(models.Model):
   header = models.CharField(max_length=255)
   post_text = models.CharField(max_length=1024)
   author = models.CharField(max_length=255, null=True)
```

После чего нам понадобится сам Serializer:
```python
#serializer.py
class PostSerializer(serializers.Serializer):
   header = serializers.CharField(max_length=255)
   post_text = serializers.CharField(max_length=1024)
   author = serializers.CharField(max_length=255)
```

Далее создадим Views:
```python
#views.py
class PostAPIView(APIView):
   serializer = PostSerializer
```

И наконец добавим ссылку, по которой мы будем передавать данные в наш сериализатор:
```python
#settings.py
urlpatterns = [
   path('test/', PostAPIView.as_view()),
   path('test/<int:pk>/', PostAPIView.as_view()),
]
```

Какими методами мы можем пользоваться в serializer?
- Create
- Read
- Update
- Delete?

У нас есть полный надо CRUD методов, для реализации наших нужд.

## Create:
```python
#views.py
def post(self, request):
   # Записываем сериализованные данные в поле
   serializer = PostSerializer(data=request.data)
   # Проверяем их на валидацию, если она не пройдет,
   # то с помощью поднятого флага raise_exception
   # код выдаст конкретную ошибку, которую нам не нужно будет обрабатывать.
   # В данный момент программа зайдет в метод create в serializer.py и выполнит функцию. 
   # Так же проверив данные на валидность, сопоставив их с заранее обозначимимыи полями в serializer.py
   serializer.is_valid(raise_exception=True)
   serializer.save()


return Response({'post': serializer.data})
```

```python
#serializer.py
def create(self, validated_data):
   return Post.objects.create(**validated_data)
```

После того как функция create вернет значение в post запрос, мы сохраняем все изменения и возвращаем ответ.


Проверяем что мы накодили.

Отправляем POST запрос на на URL http://127.0.0.1:8000/test/ с вот таким вот телом:
```json
{
    "header": "Test",
    "post_text": "blablabla",
    "author":"xpom"
}
```

В ответе мы получаем следующее:
```json
{
    "post": {
        "header": "Test",
        "post_text": "blablabla",
        "author": "xpom"
    }
}
```

Также в бд создавалась нужная нам запись:

| id |    header     | post_text | author    |
|:--:|:-----------:|:----:|-----|
| 1  | Test | blablabla  | xpom    |

Функция создания работает корректно.


## Update:
```python
#views.py
def put(self, request, *args, **kwargs):
   pk = kwargs.get("pk", None)
   if not pk:
       return Response({"error": "Method PUT not allowed"})
   try:
       # Поиск объекта
       instance = Post.objects.get(pk=pk)
   except:
       # Выводим ошибку, если не получилось найти объект
       return Response({"error": "Object does not exists"})


   # Тут кроме принимаемого тела, мы передаем еще и объект, с которым будет производиться сравнение.
   serializer = PostSerializer(data=request.data, instance=instance)
   serializer.is_valid(raise_exception=True)
   serializer.save()


   return Response({"put": serializer.data})
```
```python
#serializer.py
def update(self, instance, validated_data):
   instance.header = validated_data.get("header", instance.header)
   instance.post_text = validated_data.get("post_text", instance.post_text)
   instance.author = validated_data.get("author", instance.author)
   instance.save()
   return instance

```

Пару слов про обновление полей. Мы производим сравнение всех полей с нашим instance объектом, если вы хотите сделать обновление только определенного элемента, то это нужно делать через partial update.

Теперь мы можем отправить PUT запрос на http://127.0.0.1:8000/test/6/, с таким телом, для обновлниея данных:
```json
{
    "header": "Test",
    "post_text": "Тут был другой текст",
    "author":"xpom"
}
```

В ответе мы получим:
```json
{
    "put": {
        "header": "Test",
        "post_text": "Тут был другой текст",
        "author": "xpom"
    }
}
```


## Get:

Для отображения данных, нам понадобится только модифицировать файл views.py:
```python
def get(self, request, *args, **kwargs):
   pk = kwargs.get("pk", None)
   if not pk:
       # Возвращаем весь список объектов, которые есть у нас в DB
       return Response(PostSerializer(Post.objects.all(), many=True).data)
   try:
       # Пытаемся найти нужный нам объект и вывести только его
       return Response(PostSerializer(Post.objects.get(pk=pk)).data)
   except:
       # Выводим ошибку, если не получилось найти объект
       return Response({"error": "Object does not exists"})
```


При GET запросе на http://127.0.0.1:8000/test/, мы получим список всех, лежащий в DB, постов:
```json
[
    {
        "header": "Test",
        "post_text": "Тут был другой текст",
        "author": "xpom"
    }
]
```

Если мы сделаем GET запрос на http://127.0.0.1:8000/test/1/, то получим только нужный нам объект, а не весь список:
```json
{
    "header": "Test",
    "post_text": "Тут был другой текст",
    "author": "xpom"
}
```

## Delete:
```python
#views.py
def delete(self, request, *args, **kwargs):
   pk = kwargs.get("pk", None)
   if not pk:
       return Response({"error": "Method DELETE not allowed"})
   try:
       # Поиск объекта и его удаление
       Post.objects.get(pk=pk).delete()
   except:
       # Выводим ошибку, если не получилось найти объект
       return Response({"error": "Object does not exists"})


   return Response({"delete": "delete post " + str(pk)})
```

При отправке DELETE метода на http://127.0.0.1:8000/test/1/, мы получим следующее(Если был такой Id):
```json
{"delete": "delete post 1"}
```