## Интеграция Celery и Redis в Django.

Celery с Redis в Django представляет собой мощное сочетание для быстрой обработки задач. Это руководство объединяет концептуальное понимание и практическую настройку, чтобы вы могли эффективно использовать эти технологии в своих Django-проектах.

### Что такое Celery и Redis?

- Celery - это асинхронная очередь задач, используемая для параллельной обработки в фоновом режиме.
- Redis служит как брокер сообщений для Celery, управляя очередями задач.

### Установка и Настройка

#### Установка Celery:

```shell
pip install celery
```

#### Установка Redis:
MacOS:
```shell
brew install redis
```

Debian/Ubuntu:
```shell
sudo apt-get install redis-server
```

#### Конфигурация Celery в Django:

В корне вашего Django-проекта создайте файл celery.py и добавьте следующий код:

```python
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'your_project.settings')

app = Celery('your_project')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()
```

Настройка Redis как брокера для Celery:

В settings.py вашего Django-проекта, укажите Redis в качестве брокера:

```python
CELERY_BROKER_URL = 'redis://localhost:6379/0'
CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'
```

#### Создание и Вызов Задач Celery
Определение Задачи:

В tasks.py вашего приложения, определите задачу Celery:

```python
from celery import shared_task

@shared_task
def my_task(x, y):
    x + y
```

Вызов Задачи:

Вызовите задачу из Django-вьюх или моделей:

```python
from .tasks import my_task
my_task.delay(params)
```

#### Мониторинг и Управление

Flower:

Установите Flower для мониторинга задач Celery:

```shell
pip install flower
```

Запустите Flower:

```shell
flower -A your_project --port=5555
```

#### Примеры Использования

- Асинхронная отправка электронной почты:
Создайте задачу Celery для отправки электронной почты, чтобы не блокировать основной поток выполнения.
- Обработка больших данных:
Используйте Celery для обработки больших объемов данных, таких как импорт или экспорт данных.


