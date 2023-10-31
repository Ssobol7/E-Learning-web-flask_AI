![Alt text](flask_app/img/flask-python.png)



# Lesson 3: 

# 16. Миграции базы данных с помощью Alembic


**Миграция базы данных** — это что-то похожее на систему контроля версий для баз данных. Стоит напомнить, что метод **create_all()** в **SQLAlchemy** лишь создает недостающие таблицы из моделей. Когда таблица уже создана, он не меняет ее схему, основываясь на изменениях в модели.

**Alembic** — это инструмент для миграции базы данных, используемый в SQLAlchemy. 

При разработке приложения распространена практика изменения схемы таблицы. Здесь и приходит на помощью **Alembic**. Он, как и другие подобные инструменты, позволяет менять схему базы данных при развитии приложения. Он также следит за изменениями самой базы, так что можно двигаться туда и обратно. Если не использовать **Alembic**, то за всеми изменениями придется следить вручную и менять схему с помощью **Alter.**

**Flask-Migrate** — это расширение, которое интегрирует Alembic в приложение Flask. Установить его можно с помощью следующей команды:

```
(env) ssob@rh:~/flask_app$  pip install flask-migrate
```

&nbsp;

Для интеграции **Flask-Migrate** с приложением нужно импортировать классы `Migrate` и `MigrateCommand` из пакета `flask_package`, а также создать экземпляр класса `Migrate`, передав экземпляр приложения (app) и объект SQLAlchemy (db):
```
#...
from flask_migrate import Migrate, MigrateCommand

app = Flask(__name__)

app.debug = True
app.config['SECRET_KEY'] = 'a really really really really long secret key'
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:pass@localhost/flask_app_db'
manager = Manager(app)
db = SQLAlchemy(app)
migrate = Migrate(app,  db)
manager.add_command('db', MigrateCommand)
#...
```
&nbsp;


Класс **MigrateCommand** определяет некоторые команды миграции базы данных, доступные во **Flask-Script**. 

На 12 строке эти команды выводятся с помощью аргумента командной строки **db**. 

Чтобы посмотреть созданные команды, нужно вернуться обратно в терминал и ввести следующую команду:

```
(env) ssob@rh:~/flask_app$ python main2.py

positional arguments:
  {db,faker,foo,shell,runserver}
    db			Perform database migrations
    faker		A command to add fake data to the tables
    foo 		Just a simple command
    shell 		Runs a Python shell inside Flask application context.
    runserver 		Runs the Flask development server i.e. app.run()
    
optional arguments:
  -?, --help		show this help message and exit

(env) ssob@rh:~/flask_app$

```

Так, можно видеть, что новая команда **db** используется для миграций базы данных. 

Чтобы посмотреть полный список подкоманд для **dv**, нужно ввести следующее:

```
(env) ssob@rh:~/flask_app$ python main2.py db -?

Perform database migrations

positional arguments:
  {init,revision,migrate,edit,merge,upgrade,downgrade,show,history,heads,branche 
s,current,stamp}
    init 		Creates a new migration repository
    revision 		Create a new revision file.
    migrate 		Alias for 'revision --autogenerate'
    edit 		Edit current revision.
    merge 		Merge two revisions together. Creates a new migration
			file
    upgrade 		Upgrade to a later version
    downgrade 		Revert to a previous version
    show 		Show the revision denoted by the given symbol.
    history 		List changeset scripts in chronological order.
    heads 		Show current available heads in the script directory
    branches 		Show current branch points
    current 		Display the current revision for each database.
    stamp 		'stamp' the revision table with the given revision;
			don't run any migrations

optional arguments:
  -?, --help 		show this help message and exit

```

Это реальные команды, которые будут использоваться для миграций базы данных.

Перед тем как **Alembic** начнет отслеживать изменения, нужно установить репозиторий миграции. 

&nbsp;


**Репозиторий миграции** — это всего лишь папка, которая содержит настройки **Alembic** и скрипты миграции. 

* Для создания репозитория нужно исполнить команду **init**:


```
(env) ssob@rh:~/flask_app$ python main2.py db init

Creating directory /home/ssob/flask_app/migrations ... done
Creating directory /home/ssob/flask_app/migrations/versions  ...  done
Generating /home/ssob/flask_app/migrations/README ... done
Generating /home/ssob/flask_app/migrations/env.py ... done
Generating /home/ssob/flask_app/migrations/alembic.ini ... done
Generating  /home/ssob/flask_app/migrations/script.py.mako ... done

Please edit configuration/connection/logging settings in
'/home/ssob/flask_app/migrations/alembic.ini' before proceeding.

(env) ssob@rh:~/flask_app$

```

* Эта команда создаст папку **/migrations** внутри папки **/flask_app**. 


&nbsp;

**Структура папки */migrations* следующая:**

```
/migrations
|
├── alembic.ini
├── env.py
├── README
├── script.py.mako
└── /version

```


***Краткое описание каждой папки и файла:***

* **alembic.ini** — файл с настройки Alembic.

* **env.py** — файл Python, который запускается каждый раз, когда вызывается Alembic. Он соединяется с базой данных, запускает транзакцию и вызывает движок миграции.

* **README** — файл README.

* **script.py.mako** — файл шаблона Mako, который будет использоваться для создания скриптов миграции.

* **/version** — папка для хранения скриптов миграции.

&nbsp;

&nbsp;


### Создание скрипта миграции

*Alembic* хранит миграции базы данных в скриптах миграции, которые представляют собой обычные файлы Python. 

Скрипт миграции определяет две функции: **upgrade()** и **downgrade()**. 

**upgrade()** — применить изменения к базе данных

**downgrade()** — откатить изменения обратно

Когда миграция применяется, вызывается функция **upgrade()**, при возврате обратно — **downgrade()**.


**Alembic** предлагает два варианта создания миграций:

1. Вручную с помощью команды **revision**.

2. Автоматически с помощью команды **migrate**.


&nbsp;


> 1. **Ручная миграция**

**Ручная** или **пустая** миграция создает скрипт миграции с пустыми функциями **upgrade()** и **downgrade()**. 

**Задача** — заполнить их с помощью инструкций **Alembic**, которые и будет применять изменения к базе данных. 

Ручная миграция используется тогда, когда нужен полный контроль над процессом миграции. 

Для создания **пустой** миграции нужно ввести следующую команду:


```
(env) ssob@rh:~/flask_app$ python main2.py db revision -m  "Initial migration"
```

Эта команда создаст новый скрипт миграции в папке **/migrations/version**. 

Название файла должно быть в формате **someid_initial_migration.py**. 

Файл должен выглядеть вот так:

```
"""Initial migration

Revision ID: 945fc7313080
Revises:
Create Date: 2019-06-03 14:39:27.854291

"""
from alembic import op
import sqlalchemy as sa


# идентификаторы изменений, используемые Alembic.
revision = '945fc7313080'
down_revision = None
branch_labels = None
depends_on = None


def upgrade():
    pass
    

def downgrade():
    pass

```

Он начинается с закомментированной части, которая содержит сообщение, заданное с помощью метки -m, **ID**-изменения и времени, когда файл был создан. 

Следующая важная часть — *идентификаторы изменения*. Каждый скрипт миграции получает уникальный **ID**- изменения, который хранится в переменной **revision**. На следующей строке есть переменная **down_revision** со значением **None**. 

**Alembic** использует переменную **down_revision**, чтобы определить, какую миграцию запускать и в каком порядке.

Переменная **down_revision** указывает на идентификатор изменения родительской миграции. В этом случае его значение — **None**, потому что это только первый скрипт миграции. В конце файла есть пустые функции **upgrade()** и **downgrade()**.

* Теперь нужно отредактировать файл миграции, чтобы добавить операции создания и удаления таблицы для функций **upgrade()** и **downgrade()**, соответственно.

В функции **upgrade()** используется инструкция **create_table()** Alembic. Инструкция **create_table()** использует оператор **CREATE TABLE.**

В функции **downgrade()** инструкция **drop_table()** задействует оператор **DROP TABLE**.

При первом запуске миграции будет создана таблица **users**, а при откате — эта же миграция удалит таблицу **users**.


* Теперь можно выполнить первую миграцию. Для этого нужно ввести следующую команду:

```
(env) ssob@rh:~/flask_app$ python main2.py db upgrade
```

Эта команда исполнит функцию **upgrade()** скрипта миграции. Команда **db upgrade** вернет базу данных к последней миграции. Стоит заметить, что **db upgrade** не только запускает последнюю миграции, но все, которые еще не были запущены. Это значит, что если миграций было создано несколько, то **db upgrade** запустит их все вместе в порядке создания.

* Вместо запуска последней миграции можно также передать идентификатор изменения нужной миграции. В таком случае **db upgrade** остановится после запуска конкретной миграции и не будет выполнять последующие.

```
(env) ssob@rh:~/flask_app$ python main2.py db upgrade 945fc7313080
```

Поскольку миграция запускается первый раз, **Alembic** также создаст таблицу **alembic_version**. Она состоит из одной колонки **version_num**, которая хранит идентификатор изменения последней запущенной миграции. Именно так **Alembic** знает текущее состояние миграции, и откуда ее нужно исполнять.

Сейчас таблица **alembic_version** выглядит вот так:

![Alt text](flask_app/img/tablica-alembic_version.png)

* Определить последнюю примененную миграцию можно с помощью команды **db current**. Она вернет идентификатор изменения последней миграции,если таковой не было, то ничего не вернется:

```
(env) ssob@rh:~/flask_app$  python main2.py  db current

INFO [alembic.runtime.migration] Context impl MySQLImpl.
INFO [alembic.runtime.migration] Will assume non-transactional DDL.
945fc7313080 (head)

(env) ssob@rh:~/flask_app$

```

Вывод показывает, что текущая миграция — **945fc7313080**. Также нужно обратить внимание на строку **(head)** после идентификатора изменения, которая указывает на то, что **945fc7313080** — последняя миграция.

* Создадим еще одну пустую миграцию с помощью команды db revision:

```
(env) ssob@rh:~/flask_app$ python main2.py db revision -m  "Second migration"
```

* Дальше нужно снова запустить команду **db current.** 
  В этот раз идентификатор изменения будет отображаться без строки **(head)**, потому что миграция **945fc7313080** — не последняя:

```
(env) ssob@rh:~/flask_app$  python main2.py db current

INFO [alembic.runtime.migration] Context impl MySQLImpl.
INFO [alembic.runtime.migration] Will assume non-transactional DDL.
945fc7313080

(env) ssob@rh:~/flask_app$

```

* Чтобы посмотреть полный список миграций (запущенных и нет), нужно использовать команду **db history.** Она вернет список миграций в обратном хронологическом порядке (последняя миграция будет отображаться первой):

```
(env) ssob@rh:~/flask_app$ python main2.py db history

945fc7313080 -> b0c1f3d3617c (head), Second migration
<base> -> 945fc7313080, Initial migration

(env) ssob@rh:~/flask_app$

```

Вывод показывает, что **945fc7313080** — первая миграция, а следом за ней идет **b0c1f3d3617** — последняя миграция. Как и обычно, **(head)** указывает на последнюю миграцию.

* Таблица **users** был создана исключительно в целях тестирования. Вернуть базу данных к исходному состоянию, которое было до исполнения команды **db upgrade**, можно с помощью отката миграции. 

Чтобы откатиться к последней миграции, используется команда **db downgrade**:

```
(env) ssob@rh:~/flask_app$  python main2.py db downgrade

INFO [alembic.runtime.migration]  Context impl MySQLImpl.
INFO [alembic.runtime.migration]  Will assume non-transactional DDL.
INFO [alembic.runtime.migration]  Running downgrade  945fc7313080  -> , Initial mi
gration

(env) ssob@rh:~/flask_app$

```

Она выполнит метод **downgrade()** миграции **945fc7313080**, которая удалит таблицу **users** из базы данных. Как и в случае с командой **db upgrade**, можно передать идентификатор изменения миграции, к которому нужно откатиться. 

Например: 

 - Чтобы откатиться к миграции **645fc5113912**, нужно использовать следующую команду:

```
(env) ssob@rh:~/flask_app$ python main2.py db downgrade  645fc5113912

```

 - Чтобы вернуть все принятые миграции, нужно использовать следующую команду:

```
(env) ssob@rh:~/flask_app$ python main2.py db downgrade base

```

* Сейчас к базе данных не применено ни единой миграции. Убедиться в этом можно, запустив команду db current:

```
(env) ssob@rh:~/flask_app$ python main2.py db current

INFO [alembic.runtime.migration] Context impl MySQLImpl.
INFO [alembic.runtime.migration] Will assume non-transactional DDL.

(env) ssob@rh:~/flask_app$

```

Как видно, вывод не возвращает идентификатор изменения. Стоит обратить внимание, что откат миграции лишь отменяет изменения базы данных, но не удаляет сам скрипт миграции. 

В результате команда **db history** покажет два скрипта миграции:

```
(env) ssob@rh:~/flask_app$ python main2.py db history

945fc7313080  ->  b0c1f3d3617c (head), Second migration
<base> -> 945fc7313080, Initial migration

(env) ssob@rh:~/flask_app$

```

* Что будет, если сейчас запустить команду **db upgrade**?

Команда **db upgrade** в первую очередь запустит миграцию **945fc7313080**, а следом за ней — **b0c1f3d3617**.

База данных снова в изначальном состоянии, а поскольку изменения в скриптах миграции не требуются, их можно удалить.


&nbsp;


> 2. **Автоматическая миграция**

    * Примечание: перед тем как двигаться дальше, нужно убедиться, что миграции из прошлого раздела удалены.

Автоматическая миграция создает код для функций **upgrade()** и **downgrade()** после сравнения моделей с текущей версией базы данных.

Для создания автоматической миграции используется команда `migrate`, которая по сути повторяет то, что делает `revision --autogenerate``. 

В терминале нужно ввести команду `migrate`. Важно обратить внимание, что на последней строчке вывода написано *”No changes in schema detected.”*. Это значит, что модели синхронизированы с базой данных.

* Откроем **main2.py**, чтобы добавить модель **Employee** после модели **Feedback**:

```
#...
class Employee(db.Model):
    __tablename__ = 'employees'
    id = db.Column(db.Integer(), primary_key=True)
    name = db.Column(db.String(255), nullable=False)
    designation = db.Column(db.String(255), nullable=False)
    doj = db.Column(db.Date(), nullable=False)
#...

```

Дальше нужно снова запустить команду `db migrate`. В этот раз **Alembic** определит, что была добавлена новая таблица **employees** и создаст скрипт миграции с функциями для последующего создания и удаления таблицы **employees**.

```
(env) ssob@rh:~/flask_app$ python main2.py db migrate -m  "Adding employees table"
```

Скрипт миграции, созданный с помощью предыдущей команды, должен выглядеть вот так:

```
"""Adding employees table

Revision ID: 6e059688f04e
Revises:
Create Date: 2019-06-03 16:01:28.030320

"""
from alembic import op
import sqlalchemy as sa


# идентификаторы изменений, используемые Alembic.
revision = '6e059688f04e'
down_revision = None
branch_labels = None
depends_on  = None


def upgrade():
    # ### автоматически генерируемые команды Alembic - пожалуйста, настройте! ###
    op.create_table('employees',
    sa.Column('id', sa.Integer(), nullable=False),
    sa.Column('name', sa.String(length=255), nullable=False),
    sa.Column('designation', sa.String(length=255),  nullable=False),
    sa.Column('doj', sa.Date(), nullable=False),
    sa.PrimaryKeyConstraint('id')
)
    # ### конец команд Alembic ###


def downgrade():
    # ### автоматически генерируемые команды Alembic - пожалуйста, настройте! ###
    op.drop_table('employees')
    # ### конец команд Alembic ###

```

Ничего нового здесь нет. Функция **upgrade()** использует инструкцию **create_table** для создания таблицы, а функция **downgrade()** — инструкцию **drop_table** для ее удаления.

Запустим миграцию с помощью команды `db upgrade`:

```
(env) ssob@rh:~/flask_app$ python main2.py db upgrade

INFO [alembic.runtime.migration] Context impl MySQLImpl.
INFO [alembic.runtime.migration] Will assume non-transactional DDL.
INFO [alembic.runtime.migration] Running upgrade  ->  6e059688f04e, Adding emplo
yees table

(env) ssob@rh:~/flask_app$

```
Это добавит таблицу ***employees*** в базу данных.


&nbsp;


3. **Ограничения автоматической миграции**

Автоматическая миграция не идеальна. Она не определяет все возможные изменения.

**Операции, которые Alembic умеет выявлять:**

* Добавление и удаление таблиц
* Добавление и удаление колонок
* Изменения во внешних ключах
* Изменения в типах колонок
* Изменения в индексах и использованных уникальных ограничениях

**Изменения, которые Alembic не определяет:**

* Имени таблицы
* Имени колонки
* Ограничения с отдельно указанными именами

Для создания скриптов миграции для операций, которые **Alembic** не умеет выявлять, нужно создать пустой скрипт миграции и заполнить функции **upgrade()** и **downgrade()** соответствующим образом.


&nbsp;

&nbsp;

---

# 17. Отправка email во Flask

Веб-приложения отправляют электронные письма постоянно, и в этом уроке речь пойдет о том, как добавить инструмент для отправки email в приложение Flask.

В стандартной библиотеке Python есть модуль smtplib, который можно использовать для отправки сообщений. Хотя сам модуль smtplib не является сложным, он все равно требует кое-какой работы. Для облегчения процесса работы с ним было создано расширение Flask-Mail. Flask-Mail построен на основе модуля Python smtplib и предоставляет простой интерфейс для отправки электронных писем. Он также предоставляет возможности по массовой рассылке и прикрепленным к сообщениям файлам. Установить Flask-Mail можно с помощью следующей команды:

```
(env) ssob@rh:~/flask_app$ pip install flask-mail

```

* Чтобы запустить расширение, нужно импортировать класс **Mail** из пакета **flask_mail** и создать экземпляр класса **Mail**:

```
#...
from flask_mail import Mail, Message

app = Flask(__name__)
app.debug = True
app.config['SECRET_KEY'] = 'a really really really really long secret key'
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:pass@localhost/flask_app_db'

manager = Manager(app)
manager.add_command('db', MigrateCommand)
db = SQLAlchemy(app)
migrate = Migrate(app,  db)
mail = Mail(app)
#...

```

* Дальше нужно указать некоторые параметры настройки, чтобы **Flask-Mail** знал, к какому *SMTP-серверу* подключаться. 

  Для этого в файл **main2.py** нужно добавить следующий код:

```
#...
app.config['SECRET_KEY'] = 'a really really really really long secret key'
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:pass@localhost/flask_app_db'
app.config['MAIL_SERVER'] = 'smtp.googlemail.com'
app.config['MAIL_PORT'] = 587
app.config['MAIL_USE_TLS'] = True
app.config['MAIL_USERNAME'] = 'test@gmail.com'  # введите свой адрес электронной почты здесь
app.config['MAIL_DEFAULT_SENDER'] = 'test@gmail.com'  # и здесь
app.config['MAIL_PASSWORD'] = 'password'  # введите пароль

manager = Manager(app)
manager.add_command('db', MigrateCommand)
db = SQLAlchemy(app)
mail = Mail(app)
#...

```

В данном случае используется *SMTP-сервер Google*. Стоит отметить, что *Gmail* позволяет отправлять только **100-150** сообщений в день. Если этого недостаточно, то стоит обратить внимание на альтернативы: **SendGrid** или **MailChimp**.

Вместо того чтобы напрямую указывать **email** и пароль в приложении, как это было сделано ранее, лучше хранить их в переменных среды. В таком случае, если почта или пароль поменяются, не будет необходимости обновлять код. О том, как это сделать, будет рассказано в следующих уроках.


&nbsp;

---

### Основы Flask-Mail
&nbsp;

* Для составления электронного письма, нужно создать экземпляр класса Message:

```
msg = Message("Subject", sender="sender@example.com",  recipients=['recipient_1@example.com'])

```

* Если при настройке параметров конфигурации `MAIL_DEFAULT_SENDER` был указан, то при создании экземпляра **Message** передавать значение **sender** не обязательно:

```
msg = Message("Subject", recipients=['recipient@example.com'])

```

* Для указания тела письма необходимо использовать атрибут **body** экземпляра **Message**:

```
msg.body = "Mail body"

```

* Если оно состоит из HTML, передавать его следует атрибуту html:

```
msg.html = "<p>Mail body</p>"

```

* Наконец, отправить сообщение можно, передав экземпляр `Message` метод **send()** экземпляра **Mail**:

```
mail.send(msg)

```

Пришло время проверить настройки, отправив email с помощью командной строки.


&nbsp;


### Отправка тестового сообщения
&nbsp;

Откроем терминал, чтобы ввести следующие команды:

```
(env) ssob@rh:~/flask_app$ python main2.py shell

>>>
>>> from  main2 import  mail,  Message
>>> # введите свою почту
>>> msg = Message("Subject", recipients=["you@mail.com"])
>>> msg.html = "<h2>Email Heading</h2>\n<p>Email Body</p>"
>>>
>>> mail.send(msg)
>>>

```

* Если операция прошла успешно, то на почту должно прийти следующее сообщение с темой **“Subject”**:
```
Email Heading
Email Body

```

Стоит заметить, что отправка через SMTP-сервер Gmail не сработает, если не отключить двухфакторную аутентификацию и не разрешить небезопасным приложениям получать доступ к аккаунту.


&nbsp;


### Интеграция email в приложение
&nbsp;

Сейчас когда пользователь отправляет обратную связь, она сохраняется в базу данных, сам пользователь получает уведомление о том, что его сообщение было отправлено, и на этом все. Но в идеале приложение должно уведомлять администраторов о полученной обратной связи. Это можно сделать. 

Откроем **main2.py**, чтобы изменить функцию представления **contact()** так, чтобы она отправляла сообщения:

```
#...
@app.route('/contact/', methods=['get', 'post'])
def contact():
    #...
	db.session.commit()
	
	msg = Message("Feedback", recipients=[app.config['MAIL_USERNAME']])
	msg.body = "You have received a new feedback from {} <{}>.".format(name, email)
	mail.send(msg)

	print("\nData received. Now redirecting ...")
    #...

```

Дальше нужно запустить сервер и зайти на https://localhost:5000/contact/. Заполним и отправим **форму**. 

Если все прошло успешно, должен прийти email.


Можно было обратить внимание на задержку между отправкой обратной связи и появлением уведомления о том, что она была отправлена успешно. Проблема в том, что метод **mail.send()** блокирует исполнение функции представления на несколько секунд. В результате, код с перенаправлением страницы не будет исполнен до тех пор, пока не вернется метод **mail.send()**. Решить это можно с помощью потоков (**threads**).

Также прямо сейчас можно слегка изменить код отправки сообщений. На данный момент если email потребуется отправить в любом другом месте кода, нужно будет копировать и вставлять те самые строки. Но можно сохранить несколько строк, заключив логику отправки сообщений в функцию.

Откроем **main2.py**, чтобы добавить следующий код перед **index**:

```
#...
from threading import Thread
#...
def shell_context():
    import os, sys
    return dict(app=app, os=os, sys=sys)

manager.add_command("shell",  Shell(make_context=shell_context))

def async_send_mail(app, msg):
    with app.app_context():
	mail.send(msg)


def send_mail(subject, recipient, template, **kwargs):
    msg = Message(subject,      sender=app.config['MAIL_DEFAULT_SENDER'],  recipients=[recipient])
    msg.html = render_template(template,  **kwargs)
    thr = Thread(target=async_send_mail,  args=[app,  msg])
    thr.start()
    return thr

@app.route('/')
def index():
    return render_template('index.html', name='Sunny')
#...

```

Было сделано несколько изменений. Функция `send_mail()` теперь включает в себя всю логику отправки email. Она принимает тему письма, получателя и шаблон сообщения. Ей также можно передать дополнительные аргументы в виде аргументов-ключевых слов. Почему именно так? Дополнительные аргументы представляют собой данные, которые нужно передать шаблону. На **17** строке рендерится шаблон, а его результат передается атрибуту **msg.html**. На строке **18** создается объект **Thread**. Это делается с помощью передачи названия функции и аргументов функции, с которыми она должна быть вызвана. Следующая строка запускает потоки. Когда поток запускается, вызывается **async_send_mail()**. Теперь самое интересное. Впервые в коде работа происходит вне приложения (то есть, вне функции представления) в новом потоке. **with app.app_context()**: создает контекст приложения, а **mail.send()** отправляет email.

Дальше нужно создать шаблон для сообщения обратной связи. В папке **templates** необходимо создать папку **mail**. Она будет хранить шаблоны для электронных писем. Внутри папки необходимо создать шаблон **feedback.html** со следующим кодом:

```
<p>You have received a new feedback from {{ name }} &lt;{{ email }}&gt; </p>

```

Теперь нужно изменить функцию представления `contact()``, чтобы использовать функцию **send_mail()**

После этого нужно снова зайти на https://localhost:5000/contact, заполнить форму и отправить ее. В этот раз задержки не будет.


&nbsp;

&nbsp;

---


© 2023 S.Sobolewski
