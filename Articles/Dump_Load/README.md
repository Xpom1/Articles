# Как в Django сделать миграции данных?

Можно воспользоваться Fixtures, Dump/Load, Мануальное создание

### Fixtures

Fixtures это наборы данных, которые Django может прочитать и загрузить в свою базу данных. Fixtures также можно использовать или создавать для хранения существующих данных.

Поддерживаемые форматы и структуры данных
В настоящее время Django поддерживает три формата: JSON, XML, YAML

Django ожидает, что фикстуры (fixtures) будут следовать определенному шаблону. Любой другой шаблон приведет к ошибке. Для JSON шаблон будет следующим:



```json
[
  {
    "model": "myapp.person",
    "pk": 1,
    "fields": {
      "first_name": "John",
      "last_name": "Lennon"
    }
  },
  {
    "model": "myapp.person",
    "pk": 2,
    "fields": {
      "first_name": "Paul",
      "last_name": "McCartney"
    }
  }
]

// Взято из официальной документации
```

В полях мы должны указать модель, для которой эти значения предназначены, pk - под каким уникальным ключем должна быть запись в данной базе и сами полня.

Эту же информацию можно поместить в YAML, и это будет выглядеть следующим образом:

```yaml
- model: myapp.person
  pk: 1
  fields:
    first_name: John
    last_name: Lennon
- model: myapp.person
  pk: 2
  fields:
    first_name: Paul
    last_name: McCartney
```

### Расположение fixture
Django может найти фикстуры в проекте тремя способами:

- Область действии приложении: по умолчанию Django ищет директорию fixtures внутри приложения. 
- Область действии проекта: мы также можем хранить все наши фикстуры на уровне проекта. Для этого нам потребуется создать каталог под названием fixtures на корневом уровне проекта. Затем нам нужно добавить настройки FIXTURE_DIRS в наш файл settings.py, чтобы указать места, где хранятся фикстуры. 
- Командная строка: мы также можем указать Django выполнить поиск в определенном месте или файле, добавив параметр в команду, которую мы будем запускать для загрузки данных. Например:
    ```python
    python manage.py loaddata location/to/your/data.json
    ```



### Загрузка данных
loaddata используется для загрузки данных в базу данных. 
```python
# укажите расположение файла, имя и расширение
python manage.py loaddata location/to/the/file/data.json

# укажите имя файла и расширение
python manage.py loaddata data.json

# укажите только имя файла
python manage.py loaddata data
```

В первой команде мы указали путь, имя файла и расширение. Django будет искать этот файл во всех директориях fixture, которые мы определили ранее. Итак, если у нас есть каталог fixtures в корне нашего проекта, с помощью этой команды Django будет искать структуру директориях (расположение/в/файл/) и имя файла внутри этой структуры с соответствующим расширением. 

Во второй команде нет пути; поэтому Django будет искать имя файла и расширение в любом из указанных мест. 

В третьей команде Django будет искать любой файл с именем файла в указанных местах. В этом случае расширения не имеют значения. Отметим, что если мы укажем расширение фикстуры, Django сначала вызовет конкретный сериализатор для десериализации данных. Если мы не укажем расширение, Django сначала будет искать файл и вызывать сериализатор на основе расширения найденного файла.

Есть параметры, которые мы можем добавить к нашей команде loaddata:
```text
--database <db_name>: указывает базу данных, в которую будут загружены данные. По умолчанию будет использоваться база данных указанная в файле settings.py.

--app <app_name>: указывает приложению, где искать фикстуры

--format <format_name>: указывает формат сериализации (json/xml/yaml)

--exclude <file_name>: указывает любой файл, который следует исключить из загрузки
```

Исходные данные для всего проекта
Если нам нужно заполнить данные из нескольких фикстур, повторное выполнение команды неэффективно. Мы можем справиться с этим, запустив команду, которая загружает все фикстуры, используя символы.



Команда будет выглядеть так, как показано ниже, когда она загружает все файлы json в директорию fixtures:
```python
python manage.py loaddata fixtures/*.json
```



Создание резервных копий и восстановление из командной строки

Утилита pg_dump - встроенный инструмент для создания резервных копий в PostgreSQL. Пример использования: # pg_dump zabbix > /tmp/zabbix.dump.

Утилита pg_dumpall - реализует резервное копирование всего экземпляра базы данных. Пример использования: # pg_dumpall > /tmp/instance.bak.

Утилита pg_restore - позволяет восстанавливать данные из резервных копий. Пример использования: # pg_restore -d zabbix /tmp/zabbix.bak.

Мануальное создание DB с помощью ORM в Django:
Можно посмотреть [тут](https://github.com/Xpom1/Articles/blob/main/Articles/Django_models/README.md)