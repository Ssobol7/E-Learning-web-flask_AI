![Alt text](flask_app/img/flask-python.png)



# Lesson 2:

# 14. Создание баз данных во Flask


В этом уроке речь пойдет о взаимодействии с базой данных. 

Мы рассмотрим Реляционные базы данных:


**Реляционные базы данных** используются в веб-приложениях и хранят данные в *таблицах и колонках*, используя внешний ключ для создания связи между несколькими таблицами.

Реляционные базы данных также поддерживают транзакции. Это значит, что можно исполнить набор *SQL-операторов*, которые должны быть **атомарными (atomic)**.

Под **атомарными (atomic)** подразумеваются все операторы, которые исполняются по принципу «все или ничего».

Реляционные базы данных доказали свою надежность и безопасность во многих отраслях.

### **SQLAlchemy** и **Flask-SQLAchemy**

**SQLAlchemy** – это **фреймворк** для работы с реляционными базами данных в Python. Он был создан Майком Байером в 2005 году. **SQLAlchemy** поддерживает следующие базы данных: **MySQL**, **PostgreSQL**, **Oracle**, **MS-SQL**, **SQLite** и другие.

**SQLAchemy** поставляется с мощным **ORM** (Object-Relational Mapping) это технология объектно-реляционного отображения, которая позволяет работать с разными базами данных с помощью объектно-ориентированного кода, а не сырого **SQL** (языка структурированных запросов). Конечно, это не обязывает использовать только ORM. В любой момент можно задействовать возможности **SQL**.

**Flask-SQLAlchemy** – это расширение, которое интегрирует **SQLAlchemy** во фреймворк **Flask**. Он также предлагает дополнительные методы, благодаря которым работать с **SQLAlchemy** становится немного проще. 

Установить **Flask-SQLAlchemy** вместе с дополнительными модулями можно с помощью следующей команды:

```
(env) gvido@vm:~/flask_app$  pip install flask-sqlalchemy
```

Для использования **Flask-SQLAlchemy** нужно импортировать класс **SQLAlchemy** из пакета **flask_sqlalchemy** и создать экземпляр объекта **SQLAlchemy**, передав ему экземпляр приложения. 

Откроем файл **main2.py**, чтобы изменить код следующим образом:
```
#...
from forms import ContactForm
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.debug = True
app.config['SECRET_KEY'] = 'a really really really really long secret key'

manager = Manager(app)
db = SQLAlchemy(app)

class Faker(Command):
#...
```

Экземпляр **db** объекта **SQLAlchemy** предоставляет доступ к функциям **SQLAlchemy**.

Дальше нужно сообщить **SQLAlchemy** местоположение базы данных в виде **URL**. Формат **URL** базы данных следующий:

```
dialect+driver://username:password@host:port/database
```

**dialect** ссылается на имя базы данных, такое как **mysql**, **mssql**, **postgresql** и так далее.

**driver** ссылается на **DBAPI**, который он использует, чтобы соединяться с базой данных. По умолчанию **SQLAlchemy** работает только с **SQLite** без дополнительных драйверов. Чтобы работать с другими базами данных, нужно установить конкретный драйвер для базы данных, совместимый с **DBAPI**.

#### Что такое DBAPI?

**DBAPI** – это всего лишь стандарт, определяющий **Python API** Specification v2.0 для доступа к базам данных от разных производителей.

*Следующая таблица содержит некоторые базы данных и драйвера для них, совместимые с* **DBAPI:**

|База данных  |  Драйвер DBAPI  |
|:------------|:----------------|
|MySQL        |	pymysql         |
|PostgreSQL	  | psycopg2        |
|MS-SQL	      | pyodbc          |
|Oracle	      | cx_Oracle       |

**Username** и **password** указываются только при необходимости. Если указаны, они будут использоваться для авторизации в базе данных.

**host** — местоположение сервера базы данных.

**port** — порт сервера базы данных.

**database** — имя базы данных.


**Вот некоторые примеры URL баз данных для самых популярных типов:**

```
    # URL базы данных для MySQL с использованием драйвера pymysql
    'mysql+pymysql://root:pass@localhost/my_db'

    # URL базы данных для PostgreSQL с использованием psycopg2
    'postgresql+psycopg2://root:pass@localhost/my_db'

    # URL базы данных для MS-SQL с использованием драйвера pyodbc
    'mssql+pyodbc://root:pass@localhost/my_db'

    # URL базы данных для Oracle с использованием драйвера cx_Oracle
    'oracle+cx_oracle://root:pass@localhost/my_db'
```

Формат **URL** базы данных для **SQLite** слегка отличается. Поскольку **SQLite** – это база данных, *основанная на файле*, и она не требует имени пользователя и пароля, в **URL** базы данных указывается только путь к файлу базы.
```
    # Для Linux или MacOS мы используем 4 слеша
    sqlite:////absolute/path/to/my_db.db

    # Для Windows мы используем 3 слеша
    sqlite:///c:/absolute/path/to/mysql.db
```

* **Flask-SQLAlchemy** использует конфигурационный ключ **SQLALCHEMY_DATABASE_URL** для определения **URL** базы данных. 

Откроем **main2.py**, чтобы добавить **SQLALCHEMY_DATABASE_URI**:

```
#...
app = Flask(__name__)
app.debug = True
app.config['SECRET_KEY'] = 'a really really really really long secret key'
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:pass@localhost/flask_app_db'

manager = Manager(app)
db = SQLAlchemy(app)
#...
```

В этом курсе будет использоваться база данных **MySQL**. 

*Поэтому прежде чем переходить к следующему разделу, нужно убедиться, что MySQL работает на компьютере.*

### Создание моделей

**Модель** — это класс в Python, который представляет собой таблицу базы данных. Ее атрибуты сопоставляются со столбцами таблицы. Класс модели наследуется из **db.Mobel** и определяет колонки как экземпляры класса **db.Column**. 

Откроем **main2.py**, чтобы добавить следующий класс перед функцией представления **updating_session()**:

```
#...
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

#...

class Post(db.Model):
    __tablename__ = 'posts'
    id = db.Column(db.Integer(), primary_key=True)
    title = db.Column(db.String(255), nullable=False)
    slug = db.Column(db.String(255), nullable=False)
    content = db.Column(db.Text(), nullable=False)
    created_on = db.Column(db.DateTime(), default=datetime.utcnow)
    updated_on = db.Column(db.DateTime(), default=datetime.utcnow, onupdate=datetime.utcnow)

    def __repr__(self):
	return "<{}:{}>".format(self.id,  self.title[:10])
```

Здесь создается модель **Post** с 5-тью переменными класса. Каждая переменная класса, кроме **\__tablename\__**, — это экземпляр класса **db.Column**.

**\__tablename\__** — это специальная переменная класса, используемая для определения имени таблицы базы данных. По умолчанию **SQLAlchemy** не следует соглашению о создании имен во множественном числе, поэтому *название таблицы здесь — это название модели*. Если на хочется опираться на такое поведение, следует использовать переменную **\__tablename\__**, чтобы **явно** указать название таблицы.

Первый аргумент конструктора **db.Column()** — это тип колонки, которая создается. SQLAlchemy предлагает большое количество типов колонок, а если их недостаточно, то можно создать свои. 

**Следующая таблица описывает основные типы колонок в SQLAlchemy и их соответствующие типы в Python и SQL:**

|SQLAlchemy  |	Python	| SQL |
|------------|----------|-----|
|BigInteger|	int|	BIGINT|
|Boolean |	bool	|BOOLEAN или SMALLINT |
|Date|	datetime.date |	DATE |
|DateTime	| datetime.date	|DATETIME |
|Integer	|int	| INTEGER |
| Float	| float	| FLOAT или REAL|
| Numeric	| decimal.Decimal	| NUMERIC  |
|Text	|  str	| TEXT |


Также можно задать дополнительные ограничения для колонки, передав их в виде аргументов-ключевых слов *конструктору* **db**.Column. 

**Следующая таблица включает некоторые широко используемые ограничения:**

| Ограничение	|  Описание  |
|---------------|------------|
| nullable	    | Когда значение равно *False*, делает колонку обязательной, значение по умолчанию — *True*|
|default	|Создает значение по умолчанию для колонки|
|index	|Логический атрибут, если *True*, то создает индексированную колонку|
|onupdate	|Создает значение по умолчанию для колонки при обновлении записи|
|primary_key	|Логический атрибут, если *True*, отмечает колонку основным ключом таблицы|
|unique	|Логический атрибут, если *True*, то каждая колонка должна быть уникальной|


В строках **16-17** был определен метод **\__repr\__()**. Он не является необходим, но если есть, то создает строчное представление объекта.

Можно было заметить, что значениями по умолчанию для **created_on** и **updated_on** выбрано название метода **(datetime.utcnow)**, а не его вызов **(datetime.utcnow())**. Так сделано, потому что при исполнении кода вызывать метод **datetime.utcnow()** нет необходимости. Вместо этого его стоит вызывать, когда запись добавляется или обновляется.


### Определение отношений (связей)


В прошлом разделе была создана модель **Post** с парой полей. На практике классы моделей существуют сами по себе. Большую часть времени они связаны с другими моделями различными типами отношений: **один-к-одному**, **один-ко-многим**, **многие-ко-многим**.

Стоит дальше поработать над аналогией блога. Обычно, пост в блоге относится к одной категории и имеет один или несколько тегов. Другими словами, есть отношение **один-к-одному** между категорией и постом и отношение **многие-ко-многим** между постом и тегом.

Откроем main2.py, чтобы добавить модели **Category** и **Tag**:

```
#...
def updating_session():
    #...
    return res

class Category(db.Model):
    __tablename__ = 'categories'
    id = db.Column(db.Integer(), primary_key=True)
    name = db.Column(db.String(255), nullable=False)
    slug = db.Column(db.String(255), nullable=False)
    created_on = db.Column(db.DateTime(), default=datetime.utcnow)

    def __repr__(self):
	return "<{}:{}>".format(id, self.name)

class Posts(db.Model):
    # ...

class  Tag(db.Model):
    __tablename__ = 'tags'
    id = db.Column(db.Integer(), primary_key=True)
    name = db.Column(db.String(255), nullable=False)
    slug = db.Column(db.String(255), nullable=False)
    created_on  =  db.Column(db.DateTime(), default=datetime.utcnow)

    def __repr__(self):
	return "<{}:{}>".format(id, self.name)
#...
```


#### Отношение один-ко-многим

Для создания отношения **один-ко-многим** нужно разместить внешний ключ в дочерней таблице. *Это самый распространенный тип отношений*. 

Для создания отношения **один-ко-многим** в **SQLAlchemy** нужно выполнить следующие шаги:

1. Создать новый экземпляр **db.Column** с помощью ограничения **db.ForeignKey** в дочернем классе.

2. Определить новое свойство с помощью инструкции **db.relationship** в родительском классе. Это свойство будет использоваться для получения доступа к связанным объектам.

Откроем **main2.py**, чтобы изменить модели **Post** и **Catеgory**:

```
#...
class Category(db.Model):
    # ...
    created_on = db.Column(db.DateTime(), default=datetime.utcnow)
    posts = db.relationship('Post', backref='category')

class Post(db.Model):
    # ...
    updated_on = db.Column(db.DateTime(), default=datetime.utcnow, onupdate=datetime.utcnow)
    category_id = db.Column(db.Integer(), db.ForeignKey('categories.id'))
#...
```

Здесь для модели **Post** в **Category** были добавлены два новых атрибута: **posts** и **category_id**.

**db.ForeignKey()** принимает имя столбца, внешний ключ которого используется. Здесь значение **categories.id** передается исключению **db.ForeignKey()**. Это значит, что атрибут **category_id** у **Post** может принимать значение только у колонки **id** таблицы **categories**.

Далее в модели **Catagory** имеется атрибут **posts**, определенный инструкцией **db.relationship()**. 

**db.relationship()** используется для добавления двунаправленной связи. Другими словами, она добавляет атрибут классу модели для доступа к связанным объектам. Простыми словами, она принимает как минимум один позиционный аргумент, который является именем класса на другой стороне отношений:

```
class Category(db.Model):
    # ...
    posts = db.relationship('Post')
```

Например, если есть объект **Category** (назовём - **c**), тогда доступ ко всем постам можно получить с помощью **c.posts**. А что, если нужно получить данные с другой стороны, то есть, получить категорию у объекта поста? Для этого используется **backref**.

Наш код:

```
posts = db.relationship('Post', backref='category')
```

добавляет атрибут **category** объекту **Post**. Это значит, что если есть объект **Post** (назовём - **p**), тогда доступ к категории можно получать с помощью **p.category**.

Атрибуты **category** и **posts** у объектов **Post** и **Category** существуют только для удобства. Они не являются реальными колонками в таблице.

Стоит отметить, что в отличие от атрибута, представленного внешним ключом (который должен быть определен на стороне *«много»* в отношениях), **db.relationship()** можно определять с любой стороны.


#### Отношение один-к-одному

Создание отношения **один-к-одному** в SQLAlchemy** – это почти то же самое, что и отношение один-ко-многим. Единственное отличие — то, что инструкции **db.relationship()** передается дополнительный аргумент **uselist=False**.

**Пример:**

```
class  Employee(db.Model):
    __tablename__ = 'employees'
    id = db.Column(db.Integer(), primary_key=True)
    name = db.Column(db.String(255), nullable=False)
    designation = db.Column(db.String(255), nullable=False)
    doj = db.Column(db.Date(), nullable=False)
    dl = db.relationship('DriverLicense', backref='employee', uselist=False)

class DriverLicense(db.Model):
    __tablename__ = 'driverlicense'
    id = db.Column(db.Integer(), primary_key=True)
    license_number = db.Column(db.String(255), nullable=False)
    renewed_on = db.Column(db.Date(), nullable=False)
    expiry_date = db.Column(db.Date(), nullable=False)
    employee_id = db.Column(db.Integer(), db.ForeignKey('employees.id'))  # Foreign key
```

**Примечание:** в этих класса предполагается, что у сотрудника **(employee)** не может быть большого одного водительского удостоверения **(driver license)**. Поэтому отношения между сотрудником и правами — **один-к-одному**.

С объектом **Employee** можно использовать **e.dl**, чтоб вернуть объект **DriverLicense**. Если не передать инструкции **db.relationship()** значение **uselist=False**, тогда между **Employee** и **DriverLicense** будет установлено отношение **один-ко-многим**, и **e.dl** вернет список объектов **DriverLicense**, вместо одного объекта. При этом аргумент **uselist=False** не повлияет на атрибут **employee** объекта **DriverLicense**. Как и обычно, он вернет один объект.


#### Отношение многие-ко-многим

Отношение **многие-ко-многим** требует дополнительной ассоциативной таблицы. В качестве примера можно взять блог.

Пост в блоге обычно имеет один или несколько тегов. Аналогичным образом один тег может ассоциироваться с одним или несколькими постами. Так образовывается отношение между **posts** и **tags**. Недостаточно добавить внешний ключ, ссылающийся на **id*** постов, потому что у тега может быть один или несколько постов.

В качестве решения нужно создать новую таблицу ассоциаций, определив **2** внешних ключа, ссылающихся на колонки **post.id** и **tag.id**:

![Alt text](flask_app/img/otnoshenie-mnogie-ko-mnogim.png)

Как видно на изображении, отношение **многие-ко-многим** между постом и тегом создается с помощью двух отношений **один-к-одному**. Первое такое отношение установлено между таблицами **posts** и **post_tags**, второе — между **tags** и **post_tags**. 

Продемонстрируем на примере, как создать отношение **многие-ко-многим** в **SQLAlchemy**. 

Откроем файл **main2.py**, чтобы добавить следующий код:

```
# ...
class Category(db.Model):
    # ...
    def __repr__(self):
	return "<{}:{}>".format(id, self.name)
	
post_tags = db.Table('post_tags',
    db.Column('post_id', db.Integer, db.ForeignKey('posts.id')),
    db.Column('tag_id', db.Integer, db.ForeignKey('tags.id'))
)

class Post(db.Model):
    # ...
    
class Tag(db.Model):
    # ...
    created_on = db.Column(db.DateTime(), default=datetime.utcnow)
    posts = db.relationship('Post', secondary=post_tags, backref='tags')
#...
```

На строках **7-10** таблица ассоциаций определяется в виде объекта **db.Table()**. 

Первый аргумент таблицы **db.Table()** — имя таблицы, а **дополнительные аргументы** — это колонки, представленные экземплярами **db.Column()**. 

Синтаксис для создания таблицы ассоциаций может показаться странным, если сравнивать с процессом создания класса модели. Это потому что таблица ассоциаций создается с помощью **SQLAlchemy Core** – еще одного элемента **SQLAlchemy**.

Дальше нужно сообщить классу модели о таблице ассоциаций, которая будет использоваться. За это отвечает *аргумент-ключевое слово* **secondary**. На **18** строке **db.relationship()** вызывается с аргументом **secondary**, значение которого — **post_tags**. Хотя отношение было определено в модели **Tag**, его можно так же просто определить в модели **Post**.

Если есть, например, объект p класса **Post**, тогда доступ ко всем его тегам можно получить с помощью **p.tags**. С помощью объекта класса **Tag (t)**, доступ к постам можно получить командой **t.posts**.

Пришло время создать базу данных и таблицы.


#### Создание таблиц

Чтобы выполнить все шаги урока, нужно убедиться, что база **MySQL** установлена на компьютер.

> Стоит напомнить, что по умолчанию **SQLAlchemy** работает только с базой данных **SQLite**. Для работы с другими базами данных нужно установить драйвер, совместимый с **DBAPI**. Для использования **MySQ**L подойдет драйвер **pymysql**.

```
(env) ssob@rh:~/flask_app$ pip install pymysql
```

После этого необходимо авторизоваться на сервере **MySQL** и создать базу данных **flask_app_db** с помощью следующей команды:

```
(env) ssob@rh:~/flask_app$ mysql -u root -p
mysql>
mysql> CREATE DATABASE flask_app_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
Query OK, 1 row affected (0.26 sec)
mysql> \q
Bye
(env) ssob@rh:~/flask_app$
```
Эта команда создает базу данных **flask_app_db** с полной поддержкой **Unicode**.

Для создания необходимых таблицы нужно запустить метод **create_all()** объекта **SQLAlchemy — db**. 

Далее нужно запустить оболочку **Python** и выполнить следующую команду:

```
(env) ssob@rh:~/flask_app$ python main2.py shell
>>>
>>> from main2 import db
>>>
>>> db.create_all()
>>>
```

Метод **create_all()** создает таблицы только в том случае, если их нет в базе данных. Поэтому запускать его можно несколько раз. Также этот метод не берет во внимание изменения моделей при создании таблиц. Это значит, что если запустить метод **create_all()** после изменения его метода, когда таблица уже создана, то он не поменяет схему таблицы. Чтобы сделать это, нужно воспользоваться инструментом переноса **Alembic**. О том, как переносить базы данных с помощью **Alembic**, будет рассказано в отдельном уроке  *«Перенос базы данных с помощью Alembic»*.

Чтобы посмотреть созданные таблицы, нужно авторизоваться на сервере MySQL и выполнить следующую команду:

```
mysql>
mysql> use flask_app_db
Database changed
mysql>
mysql> show tables;

+------------------------+
| Tables_in_flask_app_db |
+------------------------+
|  categories 		 |
|  post_tags 		 |
|  posts  		 |
|  tags 		 |
+------------------------+
4 rows in set (0.02 sec)

mysql>
```

#### HeidiSQL

***HeidiSQL** - это инструмент администрирования базы данных, еще один способ посмотреть таблицы. Это кроссплатформенное ПО с открытым исходным кодом для управления базами данных **MySQL**, **MS-SQL** и **PostgreSQL**. Оно позволяет просматривать и редактировать данные, смотреть схему, менять таблицу и делать многое другое без единой строчки **SQL**. 

Отсюда можно скачать: [HeidiSQL](https://www.heidisql.com/download.php)

Установив **HeidiSQL** поверх базы данных **flask_app_db**, можно получить приблизительно следующий список таблиц:

![Alt text](flask_app/img/upravleniya-bazami-dannyh-mysql-ms-sql-i-postgresql.png)


База данных **flask_app_db** имеет **4 таблицы**. Таблицы с названиями **categories**, **posts** и **tags** созданы прямо из *моделей*, а **post_tags** — это таблица *ассоциаций*, которая представляет собой отношение **многие-ко-многим** между моделями **Post** и **Tag**.

Класс **SQLAlchemy** также определяет метод **drop_all()**, который используется для удаления всех таблиц в базе данных. Стоит помнить, что метод **drop_all()** не учитывает, есть ли данные в таблице или нет. Он удаляет все данные, поэтому использовать его нужно с умом.

Все таблицы на месте. Пора добавить в них какие-то данные.


---

# 15. Основы **ORM** SQLAlchemy


### Добавление данных

Чтобы создать новую запись с данными с помощью **SQLAlchemy**, нужно выполнить следующие шаги:

1. Создать объект
2. Добавить объект в сессию
3. Загрузить (сделать коммит) сессию


В **SAQLAlchemy** взаимодействие с базой данных происходит с помощью сессии. К счастью, ее не нужно создавать вручную. Это делает **Flask-SQLAlchemy**. Доступ к объекту сессии можно получить с помощью **db.session**. Это объект сессии, которые отвечает за подключение к базе данных. Он же отвечает за процесс транзакции. По умолчанию транзакция запускается и остается открытой до тех пор, пока выполняются коммиты и откаты.

Запустим оболочку Python для создания некоторых объектов модели:

```
(env) ssob@rh:~/flask_app$ python main2.py shell
>>>
>>> from main2 import db, Post, Tag, Category
>>>
>>>
>>>  c1 = Category(name='Python',  slug='python')
>>>  c2 = Category(name='Java',  slug='java')
>>>
```

Были созданы два объекта **Category**. Получить доступ к их атрибутам можно с помощью оператора точки **(.)**:

```
>>>
>>> c1.name, c1.slug
('Python', 'python')
>>>
>>> c2.name, c2.slug
('Java', 'java')
>>>
```

Дальше необходимо добавить объекты в сессию:

```
>>>
>>> db.session.add(c1)
>>> db.session.add(c2)
>>>
```

Добавление объектов не записывает их в базу данных , этот процесс лишь готовит их для сохранения при следующей коммите. 

Удостовериться в этом можно, проверив первичный ключ объектов:

```
>>>
>>> print(c1.id)
None
>>>
>>> print(c2.id)
None
>>>
```

Значение атрибута **id** обоих объектов — **None**. Это значит, что объекты не сохранены в базе данных.

Вместо добавления по одному объекту в сессию каждый раз, можно использовать метод **add_all()**. Метод **add_all()** принимает список объектов, которые нужно добавить в сессию:

```
>>>
>>> db.session.add_all([c1, c1])
>>>
```

Если попытаться добавить объект в сессию несколько раз, ошибок не возникнет. В любой момент можно посмотреть все объекты сессии с помощью **db.session.new**:

```
>>>
>>> db.session.new
IdentitySet([<None:Python>, <None:java>])
>>>
```

Наконец, для сохранения объектов в базе данных нужно вызвать метод commit():

```
>>>
>>> db.session.commit()
>>>
```

Если обратиться к атрибуту **id** объекта **Category** сейчас, то он вернет *первичный ключ*, а не **None**:

```
>>>
>>> print(c1.id)
1
>>>
>>> print(c2.id)
2
>>>
```

На этом этапе таблица **categories** в **HeidiSQL** должна выглядеть примерно так:

![Alt text](flask_app/img/tablica-categories-v-heidisql.png)

Новые категории пока не связаны с постами. Поэтому **c1.posts** и **c2.posts** вернут пустой список:

```
>>>
>>> c1.posts
[]
>>>
>>> c2.posts
[]
>>>
```

Стоит попробовать создать несколько постов:

```
>>>
>>> p1 = Post(title='Post 1', slug='post-1',  content='Post 1', category=c1)
>>> p2 = Post(title='Post 2', slug='post-2',  content='Post 2', category=c1)
>>> p3 = Post(title='Post 3', slug='post-3',  content='Post 3', category=c2)
>>>
```

Вместо того чтобы передавать категорию при создании объекта **Post**, можно выполнить следующую команду:

```
>>>
>>> p1.category = c1
>>>
```

Дальше нужно добавить объекты в сессию и сделать коммит:

```
>>>
>>> db.session.add_all([p1,  p2,  p3])
>>> db.session.commit()
>>>
```

Если сейчас попробовать получить доступ к атрибуту posts объекта **Category**, то он вернет *не-пустой список*:

```
>>>
>>> c1.posts
[<1:Post 1>, <2:Post 2>]
>>>
>>> c2.posts
[<3:Post 3>]
>>>
```

С другой стороны отношения, можно получить доступ к объекту **Category**, к которому относится пост, с помощью атрибута **category** у объекта **Post**:

```
>>>
>>> p1.category
<1:Python>
>>>
>>> p2.category
<1:Python>
>>>
>>> p3.category
<2:Java>
>>>
```

Стоит напомнить, что все это возможно благодаря инструкции **relationship()** в модели **Category**. 

Сейчас в базе данных есть три поста, но ни один из них не связан с тегами:

```
>>>
>>> p1.tags, p2.tags, p3.tags
([], [], [])
>>>
```

Пришло время создать теги. Это можно сделать в оболочке следующим образом:

```
>>>
>>> t1 = Tag(name="refactoring", slug="refactoring")
>>> t2 = Tag(name="snippet", slug="snippet")
>>> t3 = Tag(name="analytics", slug="analytics")
>>>
>>> db.session.add_all([t1, t2, t3])
>>> db.session.commit()
>>>
```

Этот код создает три объекта тегов и делает их коммит в базу данных. Посты все еще не привязаны к тегам. Вот как можно связать объект **Post** с объектом **Tag.**

```
>>>
>>> p1.tags.append(t1)
>>> p1.tags.extend([t2, t3])
>>> p2.tags.append(t2)
>>> p3.tags.append(t3)
>>>
>>> db.session.add_all([p1, p2, p3])
>>>
>>> db.session.commit()
>>>
```

Этот коммит добавляет следующие пять записей в таблицу **post_tags**:

![Alt text](flask_app/img/pyat-zapisej-v-tablice-post_tags.png)


Посты теперь связаны с одним или большим количеством тегов:

```
>>>
>>> p1.tags
[<1:refactoring>, <2:snippet>, <3:analytics>]
>>>
>>> p2.tags
[<2:snippet>]
>>>
>>> p3.tags
[<3:analytics>]
>>>
```

С другой стороны можно получить доступ к постам, которые относятся к конкретному тегу:

```
>>>
>>> t1.posts
[<1:Post 1>]
>>>
>>> t2.posts
[<1:Post 1>, <2:Post 2>]
>>>
>>> t3.posts
[<1:Post 1>, <3:Post 3>]
>>>
>>>
```

Важно отметить, что вместо изначального коммита объектов **Tag** и последующей их связи с объектами **Post**, все это можно сделать и таким способом:

```
>>>
>>> t1 = Tag(name="refactoring", slug="refactoring")
>>> t2 = Tag(name="snippet", slug="snippet")
>>> t3 = Tag(name="analytics", slug="analytics")
>>>
>>> p1.tags.append(t1)
>>> p1.tags.extend([t2, t3])
>>> p2.tags.append(t2)
>>> p3.tags.append(t3)
>>>
>>> db.session.add(p1)
>>> db.session.add(p2)
>>> db.session.add(p3)
>>>
>>> db.session.commit()
>>>
```

Важно обратить внимание, что на строках **11-13** в сессию добавляются только объекты **Post**. Объекты **Tag** и **Post** связаны отношением **многие-ко-многим**. В результате, добавление объекта **Post** в сессию влечет за собой добавление связанных с ним объектов **Tag**. Но даже если сейчас вручную добавить объекты *Tag** в сессию, ошибки не будет.


### Обновление данных


Для обновления объекта нужно всего лишь передать его атрибуту новое значение, добавить объект в сессию и сделать коммит:

```
>>>
>>> p1.content # начальное значение
'Post 1'
>>>
>>> p1.content = "This is content for post 1"  # задаем новое значение
>>> db.session.add(p1)
>>>
>>> db.session.commit()
>>>
>>> p1.content  # обновленное значение
'This is content for post 1'
>>>
```

### Удаление данных


Для удаления объекта нужно использовать метод **delete()** объекта сессии. Он принимает объект и отмечает, что тот подлежит удалению при следующем коммите.

Создадим новый временный тег seo и свяжем его с постами **p1** и **p2**:

```
>>>
>>> tmp = Tag(name='seo', slug='seo')  # создание временного объекта Tag
>>>
>>> p1.tags.append(tmp)
>>> p2.tags.append(tmp)
>>>
>>> db.session.add_all([p1, p2])
>>> db.session.commit()
>>>
```

Этот коммит добавляет всего **3** строки: одну в таблицу **table** и еще две — в таблицу **post_tags**. В базе данных эти три строки выглядят следующим образом:

![Alt text](flask_app/img/vremennyj-teg-seo.png)

![Alt text](flask_app/img/vremennyj-teg-seo-2.png)

Теперь нужно удалить тег **seo**:

```
>>>
>>> db.session.delete(tmp)
>>> db.session.commit()
>>>
```

Этот коммит удаляет все три строки, добавленные в предыдущем шаге. Тем не менее он не удаляет пост, с которым тег был связан.

По умолчанию при удалении объекта в родительской таблице (например **categories**) значение внешнего ключа объекта, который с ним связан в дочерней таблице (например **posts**) становится **NULL**. Следующий код демонстрирует это поведение на примере создания нового объекта категории и объекта поста, который с ней связан, и дальнейшим удалением объекта категории:

```
>>>
>>> c4 = Category(name='css', slug='css')
>>> p4 = Post(title='Post 4', slug='post-4', content='Post 4', category=c4)
>>>
>>> db.session.add(c4)
>>>
>>> db.session.new
IdentitySet([<None:css>, <None:Post 4>])
>>>
>>> db.session.commit()
>>>
```

Этот коммит добавляет две строки. Одну в таблицу **categories**, и еще одну — в таблицу **posts**.

![Alt text](flask_app/img/sozdanie-i-udalenie-svyazannyh-obektov.png)

![Alt text](flask_app/img/sozdanie-i-udalenie-svyazannyh-obektov-5.png)

Теперь нужно посмотреть, что происходит при удалении объекта **Category**:

```
>>>
>>> db.session.delete(c4)
>>> db.session.commit()
>>>
```

Этот коммит удаляет категорию css из таблицы **categories** и устанавливает значение внешнего ключа (**category_id**) для поста, который с ней связан, на **NULL**:

![Alt text](flask_app/img/tablica-categories-v-heidisql.png)

![Alt text](flask_app/img/sozdanie-i-udalenie-svyazannyh-obektov-4.png)

В некоторых случаях может возникнуть необходимость удалить все дочерние записи при том, что родительские записи уже удалены. Это можно сделать, передав **cascade=’all,delete-orphan’** инструкции **db.relationship()**. 

Откроем **main2.py**, чтобы изменить инструкцию **db.relationship()** в модели **Catagory**:

```
#...
class Category(db.Model):
    #...
    posts = db.relationship('Post', backref='category', cascade='all,delete-orphan')
#...
```

С этого момента удаление категории повлечет за собой удаление постов, которые с ней связаны. Чтобы это начало работать, нужно перезапустить оболочку. Далее импортируем нужные объекты и создаем категорию вместе с постом:

```
(env) ssob@rh:~/flask_app$ python main2.py shell
>>>
>>> from main2 import db, Post, Tag, Category
>>>
>>>  c5 = Category(name='css', slug='css')
>>>  p5 = Post(title='Post 5', slug='post-5', content='Post 5', category=c5)
>>>
>>> db.session.add(c5)
>>> db.session.commit()
>>>
```

Вот как база данных выглядит после этого коммита:

![Alt text](flask_app/img/sozdanie-i-udalenie-svyazannyh-obektov-6.png)

![Alt text](flask_app/img/sozdanie-i-udalenie-svyazannyh-obektov-2.png)

Удалим категорию:

```
>>>
>>> db.session.delete(c5)
>>> db.session.commit()
>>>
```

После этого коммита база данных выглядит вот так:

![Alt text](flask_app/img/tablica-categories-v-heidisql.png)

![Alt text](flask_app/img/sozdanie-i-udalenie-svyazannyh-obektov-4.png)


### Запрос данных

Чтобы выполнить запрос к базе данных, используется метод **query()** объекта **session**. Метод **query()** возвращает объект **flask_sqlalchemy.BaseQuery**, который является расширением оригинального объекта **sqlalchemy.orm.query.Query**. 

Объект **flask_sqlalchemy.BaseQuery** представляет собой оператор **SELECT**, который будет использоваться для осуществления запросов к базе данных. 

*В этой таблице перечислены основные методы класса* **flask_sqlalchemy.BaseQuery**:

|Метод	| Описание |
|-------|----------|
|all() |	Возвращает результат запроса (представленный flask_sqlalchemy.BaseQuery) в виде списка|
|count()	|Возвращает количество записей в запросе|
|first()	| Возвращает первый результат запроса или None, если в нем нет строк|
|first_or_404() |	Возвращает первый результат запроса или ошибку 404, если в нем нет строк |
|get(pk)	|Возвращает объект, который соответствует данному первичному ключу или None, если объект не найден|
| get_or_404(pk)|	Возвращает объект, который соответствует данному первичному ключу или ошибку 404, если объект не найден|
|filter(*criterion) |	Возвращает новый экземпляр flask_sqlalchemy.BaseQuery с оператором WHERE |
|limit(limit)	| Возвращает новый экземпляр flask_sqlalchemy.BaseQuery с оператором LIMIT |
|offset(offset)	| Возвращает новый экземпляр flask_sqlalchemy.BaseQuery с оператором OFFSET |
|order_by(*criterion)	| Возвращает новый экземпляр flask_sqlalchemy.BaseQuery с оператором OFFSET |
|join() |	Возвращает новый экземпляр flask_sqlalchemy.BaseQuery после создания SQL JOIN |


### Метод **all()**

В своей простейшей форме метод **query()** принимает в качестве аргументов один или больше классов модели или колонки. 

Следующий код вернет все записи из таблицы **posts**:

```
>>>
>>> db.session.query(Post).all()
[<1:Post 1>, <2:Post 2>, <3:Post 3>, <4:Post 4>]
>>>
```

Похожим образом следующий код вернет все записи из таблиц **categories** и **tags**:

```
>>>
>>> db.session.query(Category).all()
[<1:Python>, <2:Java>]
>>>
>>>
>>> db.session.query(Tag).all()
[<1:refactoring>, <2:snippet>, <3:analytics>]
>>>
```

Чтобы получить чистый **SQL**, использованный для запроса к базе данных, нужно просто вывести объект **flask_sqlalchemy.BaseQuery**:

```
>>>
>>> print(db.session.query(Post))
SELECT
    posts.id  AS  posts_id,
    posts.title AS  posts_title,
    posts.slug AS  posts_slug,
    posts.content AS  posts_content,
    posts.created_on AS  posts_created_on,
    posts.u  pdated_on AS  posts_updated_on,
    posts.category_id AS  posts_category_id
FROM
    posts
>>>
```

В предыдущих примерах данные возвращались со всех колонок таблицы. Это можно поменять, передав методу q**uery()** названия колонок:

```
>>>
>>> db.session.query(Post.id,  Post.title).all()
[(1, 'Post 1'), (2, 'Post 2'), (3, 'Post 3'), (4, 'Post 4')]
>>>
```


#### Метод count()

Метод **count()** возвращает количество результатов в запросе.

```
>>>
>>> db.session.query(Post).count()  # получить общее количество записей в таблице Post
4
>>> db.session.query(Category).count()  # получить общее количество записей в таблице Category
2
>>> db.session.query(Tag).count()  # получить общее количество записей в таблице Tag
3
>>>
```



#### Метод first()

Метод **first()** вернет только первый запрос из запроса или **None**, если в запросе нет результатов.

```
>>>
>>> db.session.query(Post).first()
<1:Post 1>
>>>
>>> db.session.query(Category).first()
<1:Python>
>>>
>>> db.session.query(Tag).first()
<1:refactoring>
>>>
```


#### Метод get()

Метод **get()** вернет экземпляр объекта с соответствующим первичным ключом или **None**, если такой объект не был найден.

```
>>>
>>> db.session.query(Post).get(2)
<2:Post 2>
>>>
>>> db.session.query(Category).get(1)
<1:Python>
>>>
>>> print(db.session.query(Category).get(10))  # ничего не найдено по первичному ключу 10
None
>>>
```


#### Метод get_or_404()

То же самое, что и метод **get()**, но вместо **None** вернет ошибку **404**, если объект не найден.

```
>>>
>>> db.session.query(Post).get_or_404(1)
<1:Post 1>
>>>
>>>
>>> db.session.query(Post).get_or_404(100)
Traceback (most recent call last):
...
werkzeug.exceptions.NotFound: 404  Not  Found: The requested URL was not found on the server. If  you entered the URL manually please check your spelling and try again.
>>>
```

#### Метод filter()

Метод **filter()** позволяет отсортировать результатов с помощью оператора **WHERE**, примененного к запросу. Он принимает колонку, оператор или значение. 

**Пример:**

```
>>>
>>> db.session.query(Post).filter(Post.title == 'Post 1').all()
[<1:Post 1>]
>>>
```

Запрос вернет все посты с заголовком **"Post 1"**. SQL-эквивалент запроса следующий:

```
>>>
>>> print(db.session.query(Post).filter(Post.title == 'Post 1'))
SELECT
    posts.id AS posts_id,
    posts.title AS posts_title,
    posts.slug AS posts_slug,
    posts.content AS posts_content,
    posts.created_on AS posts_created_on,
    posts.u  pdated_on AS posts_updated_on,
    posts.category_id AS posts_category_id
FROM
    posts
WHERE
    posts.title = % (title_1) s
>>>
>>>
```
Строка **% (title_1)** s в условии **WHERE** — это заполнитель. На ее месте будет реальное значение при выполнении запроса.

Методу **filter()** можно передать несколько значений и они будут объединены оператором **AND** в **SQL**. 

**Пример:**

```
>>>
>>> db.session.query(Post).filter(Post.id >= 1, Post.id <= 2).all()
[<1:Post 1>, <2:Post 2>]
>>>
>>>
```

Этот запрос вернет все посты, первичный ключ которых больше **1**, но меньше **2**. SQL-эквивалент:

```
>>>
>>> print(db.session.query(Post).filter(Post.id >= 1, Post.id <= 2))
SELECT
    posts.id AS posts_id,
    posts.title AS posts_title,
    posts.slug AS posts_slug,
    posts.content AS posts_content,
    posts.created_on AS posts_created_on,
    posts.u pdated_on AS posts_updated_on,
    posts.category_id AS posts_category_id
FROM
    posts
WHERE
    posts.id >= % (id_1) s
AND posts.id <= % (id_2) s
>>>
```

#### Метод first_or_404()

Делает то же самое, что и метод **first()**, но вместо **None** возвращает ошибку **404**, если запрос без результата:

```
>>>
>>> db.session.query(Post).filter(Post.id > 1).first_or_404()
<2:Post 2>
>>>
>>> db.session.query(Post).filter(Post.id > 10).first_or_404().all()
Traceback (most recent call last):
...
werkzeug.exceptions.NotFound: 404 Not Found: The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.
>>>
```

#### Метод limit()

Метод **limit()** добавляет оператор **LIMIT** к запросу. Он принимает количество строк, которые нужно вернуть с запросом:

```
>>>
>>> db.session.query(Post).limit(2).all()
[<1:Post 1>, <2:Post 2>]
>>>
>>> db.session.query(Post).filter(Post.id >= 2).limit(1).all()
[<2:Post 2>]
>>>
```

**SQL-эквивалент:**

```
>>>
>>> print(db.session.query(Post).limit(2))
SELECT
posts.id AS posts_id,
posts.title AS posts_title,
posts.slug AS posts_slug,
posts.content AS posts_content,
posts.created_on AS posts_created_on,
posts.u  pdated_on AS posts_updated_on,
posts.category_id AS posts_category_id
FROM
    posts
LIMIT % (param_1) s
>>>
>>>
>>> print(db.session.query(Post).filter(Post.id >= 2).limit(1))
SELECT
    posts.id AS posts_id,
    posts.title AS posts_title,
    posts.slug AS posts_slug,
    posts.content AS posts_content,
    posts.created_on AS posts_created_on,
    posts.u  pdated_on AS posts_updated_on,
    posts.category_id AS posts_category_id
FROM
    posts
WHERE
    posts.id >= % (id_1) s
LIMIT % (param_1) s
>>>
>>>
```

#### Метод offset()

Метод **offset()** добавляет условие **OFFSET** в запрос. В качестве аргумента он принимает смещение. Часто используется вместе с **limit()**:

```
>>>
>>> db.session.query(Post).filter(Post.id > 1).limit(3).offset(1).all()
[<3:Post 3>, <4:Post 4>]
>>>
```

**SQL-эквивалент:**

```
>>>
>>> print(db.session.query(Post).filter(Post.id > 1).limit(3).offset(1))
SELECT
    posts.id AS posts_id,
    posts.title AS posts_title,
    posts.slug AS posts_slug,
    posts.content AS posts_content,
    posts.created_on AS posts_created_on,
    posts.u  pdated_on AS posts_updated_on,
    posts.category_id AS posts_category_id
FROM
    posts
WHERE
    posts.id > % (id_1) s
LIMIT % (param_1) s, % (param_2) s
>>>
```

Строки **% (param_1) s** и **% (param_2)** — заполнители для смещения и ограничения вывода, соответственно.


#### Метод order_by()

Метод **order_by()** используется, чтобы упорядочить результат, добавив к запросу оператор **ORDER BY**. Он принимает количество колонок, для которых нужно установить порядок. По умолчанию сортирует в порядке возрастания:

```
>>>
>>> db.session.query(Tag).all()
[<1:refactoring>, <2:snippet>, <3:analytics>]
>>>
>>> db.session.query(Tag).order_by(Tag.name).all()
[<3:analytics>, <1:refactoring>, <2:snippet>]
>>>
```

Для сортировки по убыванию нужно использовать функцию **db.desc():**

```
>>>
>>> db.session.query(Tag).order_by(db.desc(Tag.name)).all()
[<2:snippet>, <1:refactoring>, <3:analytics>]
>>>
```

#### Метод join()

Метод **join()** используется для создания **JOIN** в **SQL**. Он принимает имя таблицы, для которой нужно создать **JOIN**:

```
>>>
>>> db.session.query(Post).join(Category).all()
[<1:Post 1>, <2:Post 2>, <3:Post 3>]
>>>
```

**SQL-эквивалент:**

```
>>>
>>> print(db.session.query(Post).join(Category))
SELECT
    posts.id  AS posts_id,
    posts.title AS posts_title,
    posts.slug AS posts_slug,
    posts.content AS posts_content,
    posts.created_on AS posts_created_on,
    posts.u  pdated_on AS posts_updated_on,
    posts.category_id AS posts_category_id
FROM
    posts
```

Метод **join()** широко используется, чтобы получить данные из одной или большего количества таблиц одним запросом. 

**Пример:**

```
>>>
>>> db.session.query(Post.title,  Category.name).join(Category).all()
[('Post 1', 'Python'),  ('Post 2', 'Python'), ('Post 3', 'Java')]
>>>
```

Можно создать **JOIN** для большее чем двух таблиц с помощью цепочки методов **join()**:

```
db.session.query(Table1).join(Table2).join(Table3).join(Table4).all()
```

Закончить урок можно завершением контактной формы.

Стоит напомнить, что в уроке *«Работа с формами во Flask»* была создана контактная форма для получения обратной связи от пользователей. Пока что функция представления **contact()** не сохраняет отправленные данные. Она только выводит их в консоли. Для сохранения полученной информации сначала нужно создать новую таблицу.

Откроем **main2.py**, чтобы добавить модель **Feedback** следом за моделью **Tag**:

```
#...
class Feedback(db.Model):
    __tablename__ = 'feedbacks'
    id = db.Column(db.Integer(), primary_key=True)
    name = db.Column(db.String(1000), nullable=False)
    email = db.Column(db.String(100), nullable=False)
    message = db.Column(db.Text(), nullable=False)
    created_on = db.Column(db.DateTime(), default=datetime.utcnow)

    def __repr__(self):
	return "<{}:{}>".format(self.id, self.name)
#...
```

Дальше нужно перезапустить оболочку **Python** и вызвать метод **create_all()** объекта **db** для создания таблицы **feedbacks**:

```
(env) ssob@rh:~/flask_app$ python main2.py shell
>>>
>>> from main2 import db
>>>
>>> db.create_all()
>>>
```

Также нужно изменить функция представления **contact()**:

```
#...
@app.route('/contact/', methods=['get', 'post'])
def contact():
    form = ContactForm()
    if form.validate_on_submit():
	name = form.name.data
	email = form.email.data
	message = form.message.data
	print(name)
	print(Post)
	print(email)
	print(message)

	# здесь логика базы данных
	feedback = Feedback(name=name, email=email, message=message)
	db.session.add(feedback)
	db.session.commit()

	print("\nData received. Now redirecting ...")
	flash("Message Received", "success")
	return redirect(url_for('contact'))
    
    return render_template('contact.html', form=form)
#...
```


Запустим сервер и зайдем на [https://127.0.0.1:5000/contact/](https://127.0.0.1:5000/contact/), чтобы заполнить и отправить форму:

![Alt text](flask_app/img/otpravka-formy-vo-flask-s-sohraneniem-v-bazu-dannyh.png)


Отправленная запись в **HeidiSQL** будет выглядеть следующим образом:

![Alt text](flask_app/img/zapis-soobsheniya-v-baze-dannyh.png)

---


© 2023 S.Sobolewski
