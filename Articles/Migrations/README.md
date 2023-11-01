# Migrations

Миграции — это способ Django распространить изменения, которые вы вносите в свои модели (добавление поля, удаление модели и т. д.), в схему вашей базы данных. Они спроектированы так, чтобы быть в основном автоматическими, но вам нужно знать, когда выполнять миграцию, когда ее запускать, а также с типичными проблемами, с которыми вы можете столкнуться.

К миграции надо относиться, как к контролю версий вашего проекта, только для базы данных. 
Существует несколько команд, которые вы будете использовать для взаимодействия с миграциями и обработкой схемы базы данных в Django:
- migrate, который отвечает за применение и отмену миграции.
- makemigrations, который отвечает за создание новых миграций на основе изменений, внесенных вами в ваши модели.
- sqlmigrate, который отображает инструкции SQL для миграции.
- showmigrations, в котором перечислены миграции проекта и их статус.

Миграции поддерживаются на всех бэкэндах, с которыми поставляется Django, а также на любых сторонних бэкэндах, если они запрограммированы на поддержку изменения схемы.

Наиболее популярные базы данных, с которыми работает Django:
- [PostgreSQL](https://docs.djangoproject.com/en/4.2/topics/migrations/#postgresql)
- [MySQL](https://docs.djangoproject.com/en/4.2/topics/migrations/#mysql) 
- [SQLite](https://docs.djangoproject.com/en/4.2/topics/migrations/#sqlite) 

Как проходит сам процесс

Вы изменяете свою модель, далее в терминале прописываете команду:
```python
python manage.py makemigrations
```
И джанго все сделает за вас, если какие-то поля будут отсутствовать или появятся новые, то все изменения сделаются автоматически.
```python
Migrations for 'books':
books/migrations/0003_auto.py:
  - Alter field author on book
```
После применения миграции зафиксируйте миграцию и изменение моделей в вашей системе контроля версий как одну фиксацию — таким образом, когда другие разработчики (или ваши рабочие серверы) проверят код, они получат оба изменения в ваших моделях.

Если вы хотите дать миграции(ям) значимое имя вместо сгенерированного, вы можете использовать опцию: makemigrations --name

```python
$ python manage.py makemigrations --name changed_my_model your_app_label
```
Файлы миграций.

Они отсортированы в папке migrations. 
Базовая миграция выглядит вот так:
```python
from django.db import migrations, models

class Migration(migrations.Migration):
    dependencies = [("migrations", "0001_initial")]

    operations = [
        migrations.DeleteModel("Tribble"),
        migrations.AddField("Author", "rating", models.IntegerField(default=0)),
    ]
```

При миграции программа будет ожидать 2 параметра:
dependencies, список миграций, от которых зависит эта.
operations, список Operation Классов, определяющих, что делает эта миграция.

Операции являются ключевыми; они представляют собой набор декларативных инструкций, которые сообщают Django, какие изменения в схеме необходимо внести.


Обратная миграция

Миграцию можно отменить, передав номер предыдущей миграции. Например, чтобы отменить миграцию books.0003:
```python
...\> py manage.py migrate books 0002
Operations to perform:
  Target specific migration: 0002_auto, from books
Running migrations:
  Rendering model states... DONE
  Unapplying books.0003_auto... OK
```
Точно так же вы можете отменить все миграции, используя имя zero:
```python
...\> py manage.py migrate books zero
Operations to perform:
  Unapply all migrations: books
Running migrations:
  Rendering model states... DONE
  Unapplying books.0002_auto... OK
  Unapplying books.0001_initial... OK
```

