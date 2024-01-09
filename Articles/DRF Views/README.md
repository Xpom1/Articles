# DRF Views

Views делятся на два вида:

APIView и ViewSet

Для начала нам понадобится модель:

```python
from django.db import models

class Film(models.Model):
    title = models.CharField(max_length=100)
    director = models.CharField(max_length=100)
    release_year = models.IntegerField()
    description = models.TextField()
```

### APIView

Этот обработчик, может работать с:
- get
- post
- put
- patch
- delete

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from .models import Film
from .serializers import FilmSerializer

class FilmAPIView(APIView):
    def get(self, request):
        films = Film.objects.all()
        serializer = FilmSerializer(films, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = FilmSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors)
```

### ViewSet

Это абстракция над APIView, которая предоставляет действия в виде методов:
- list: только для чтения, возвращает несколько ресурсов (get). Вернет список объектов.
- retrieve: только для чтения, один ресурс (get, но будет ожидать id(pk) в URL). Вернет один объект.
- create: создает новый объект (post)
- update/partial_update: редактирует объект (put/patch)
- destroy: удаляет объект (delete)

```python
from rest_framework import viewsets
from .models import Film
from .serializers import FilmSerializer
from rest_framework.response import Response

class FilmViewSet(viewsets.ViewSet):
    def list(self, request):
        queryset = Film.objects.all()
        serializer = FilmSerializer(queryset, many=True)
        return Response(serializer.data)

    def create(self, request):
        serializer = FilmSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors)

    def retrieve(self, request, pk=None):
        queryset = Film.objects.all()
        film = get_object_or_404(queryset, pk=pk)
        serializer = FilmSerializer(film)
        return Response(serializer.data)

    def update(self, request, pk=None):
        film = get_object_or_404(Film, pk=pk)
        serializer = FilmSerializer(film, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors)

    def destroy(self, request, pk=None):
        film = get_object_or_404(Film, pk=pk)
        film.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### GenericAPIView

GenericAPIView: для APIView - Добавляет обычно требуемое поведение для стандартных списковых и подробных представлений. Дает вам некоторые атрибуты, такие как serializer_class, также дает pagination_class, filter_backend и т.д

```python
from rest_framework.generics import GenericAPIView
from rest_framework.mixins import ListModelMixin, CreateModelMixin
from .models import Film
from .serializers import FilmSerializer

class FilmListCreateAPIView(GenericAPIView, ListModelMixin, CreateModelMixin):
    queryset = Film.objects.all()
    serializer_class = FilmSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```

### GenericViewSet

Generic ViewSet: Существует множество GenericViewSet, наиболее распространенным из которых является ModelViewSet. Они наследуются от GenericAPIView и имеют полную реализацию всех действий: list, retrieve, destroy, updated и т.д. Конечно, вы также можете выбрать некоторые из них, ознакомившись с документами.

```python
from rest_framework import viewsets
from .models import Film
from .serializers import FilmSerializer

class FilmModelViewSet(viewsets.ModelViewSet):
    queryset = Film.objects.all()
    serializer_class = FilmSerializer
```

Для большей углубленности в материал, вы можете посетить [Classy Django REST Framework](https://www.cdrf.co/) и [Views - Django REST framework](https://www.django-rest-framework.org/api-guide/views/) 
