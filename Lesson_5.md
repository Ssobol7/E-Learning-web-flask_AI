![Alt text](flask_app/img/flask-python.png)



# Lesson 5: 

# 19. Структура и эскиз приложения Flask

До этого момента все приложение хранилось в одном файле main2.py. Это нормально для маленьких программ, но когда масштабы растут, ими становится сложно управлять. Если разбить крупный файл на несколько, код в каждом из них становится читабельнее и предсказуемее.

У Flask нет никаких ограничений в плане структурирования приложений. Тем не менее существуют советы (гайдлайны) о том, как делать их модульными.

Мы будем использовать следующую структуру приложения:

```
/app_dir
    /app
	__init__.py
	/static
	/templates
	views.py
    config.py
    runner.py

```

Ниже описание каждого файла и папки:

* **/app_dir** - Корневая папка проекта

* **/app** - Пакет Python с файлами представления, шаблонами и статическими файлами

* **__init__.py** -	Этот файл сообщает Python, что папка app — пакет Python

* **/static** -	Папка со статичными файлами проекта

* **/templates** - Папка с шаблонами

* **views.py** - Маршруты и функции представления

* **config.py** - Настройки приложения

* **runner.py** - Точка входа приложения


Оставшаяся часть урока будет посвящена преобразованию проекта к такой структуре. 




&nbsp;


&nbsp;

### Настройки на основе классов

**Проект по созданию ПО обычно работает в трех разных средах:**

1. Разработка
2. Тестирование
3. Рабочий режим

При развитии проекта понадобится определить разные параметры для разных сред. Некоторые также будут оставаться неизменными вне зависимости от среды. Внедрить такую систему конфигурации можно с помощью классов.

Начать стоит с определения настроек по умолчанию в базовом классе и только потом — создавать классы для отдельных сред, которые будут наследовать параметры из базового. Специальные классы могут перезаписывать или дополнять настройки, необходимые для конкретной среды.

**Начнем с создания config.py**

* Создадим файл **config.py** внутри папки `/flask_app` и добавим следующий код:

```
tConfig(BaseConfig):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('DEVELOPMENT_DATABASE_URI') or \
        'mysql+pymysql://root:pass@localhost/flask_app_db'


class TestingConfig(BaseConfig):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('TESTING_DATABASE_URI') or \
			      'mysql+pymysql://root:pass@localhost/flask_app_db'


class ProductionConfig(BaseConfig):
    DEBUG = False
    SQLALCHEMY_DATABASE_URI = os.environ.get('PRODUCTION_DATABASE_URI') or \
	'mysql+pymysql://root:pass@localhost/flask_app_db'

```

Стоит обратить внимание, что в этом коде значения некоторых настроек впервые берутся у переменных среды. Также здесь есть некоторые значения по умолчанию, если таковые для сред не будут указаны. Этот метод особенно полезен, когда имеется конфиденциальная информацию, и ее не хочется вписывать напрямую в приложение.

* Считывать настройки из класса будет метод `from_object()`:

```
app.config.from_object('config.Create')

```

### Создание пакета приложения

В папке `/flask_app` нужно создать новую папку под названием `/app` и переместить все файлы и папки в нее (за исключением `env` и `migrations`, а также созданного только что файла `config.py`). 

Внутри папки `/app` нужно создать файл `__init__.py` со следующим кодом:

```
import os, config
from flask import Flask
from flask_migrate import Migrate, MigrateCommand
from flask_mail import Mail, Message
from flask_sqlalchemy import SQLAlchemy
from flask_script import Manager, Command, Shell
from flask_login import LoginManager


# создание экземпляра приложения
app = Flask(__name__)
app.config.from_object(os.environ.get('FLASK_ENV') or 'config.DevelopementConfig')

# инициализирует расширения
db = SQLAlchemy(app)
mail = Mail(app)
migrate = Migrate(app, db)
login_manager = LoginManager(app)
login_manager.login_view = 'login'

# import views
from . import views
# from . import forum_views
# from . import admin_views


```

**__init__.py** создает экземпляр приложения и запускает расширения. Если переменная среды `FLASK_ENV` не задана, приложение запустится в режиме отладки (то есть, `app.debug = True`). Чтобы перевести приложение в рабочий режим, нужно установить для переменной среды `FLASK_ENV` значение `config.ProductionConfig`.

После запуска расширений инструкция `import` на **21** строке импортирует все функции представления. Это нужно, что подключить экземпляр приложение к этим функциям, иначе **Flask** не будет о них знать.

Необходимо переименовать файл **main2.py** на **views.py** и обновить его так, чтобы он содержал только маршруты и функции представления. 

> Это полный код обновленного файла **views.py**:

```
from app import app
from flask import render_template, request, redirect, url_for, flash, make_response, session
from flask_login import login_required, login_user,current_user, logout_user
from .models import User, Post, Category, Feedback, db
from .forms import ContactForm, LoginForm
from .utils import send_mail


@app.route('/')
def index():
    return render_template('index.html', name='Jerry')


@app.route('/user/<int:user_id>/')
def user_profile(user_id):
    return "Profile page of user #{}".format(user_id)


@app.route('/books/<genre>/')
def books(genre):
    return "All Books in {} category".format(genre)


@app.route('/login/', methods=['post', 'get'])
def login():
    if current_user.is_authenticated:
	return redirect(url_for('admin'))
    form = LoginForm()
    if form.validate_on_submit():
	user = db.session.query(User).filter(User.username == form.username.data).first()
	if user and user.check_password(form.password.data):
	    login_user(user, remember=form.remember.data)
	     return redirect(url_for('admin'))
	flash("Invalid username/password", 'error')
	return redirect(url_for('login'))
    return render_template('login.html', form=form)


@app.route('/logout/')
@login_required
def logout():
    logout_user()
    flash("You have been logged out.")
    return redirect(url_for('login'))


@app.route('/contact/', methods=['get', 'post'])
def contact():
    form = ContactForm()
    if form.validate_on_submit():
	name = form.name.data
	email = form.email.data
	message = form.message.data
	
	# здесь логика БД 
	feedback = Feedback(name=name, email=email, message=message)
	db.session.add(feedback)
	db.session.commit()

	send_mail("New Feedback", app.config['MAIL_DEFAULT_SENDER'], 'mail/feedback.html',
name=name, email=email)
	
	flash("Message Received", "success")
	return redirect(url_for('contact'))

    return render_template('contact.html', form=form)


@app.route('/cookie/')
def cookie():
    if not request.cookies.get('foo'):
	res = make_response("Setting a cookie")
	res.set_cookie('foo', 'bar', max_age=60*60*24*365*2)
    else:
	res = make_response("Value of cookie foo is {}".format(request.cookies.get('foo')))
    return res


@app.route('/delete-cookie/')
def delete_cookie():
    res = make_response("Cookie Removed")
    res.set_cookie('foo', 'bar', max_age=0)
    return res


@app.route('/article', methods=['POST', 'GET'])
def article():
    if request.method == 'POST':
	res = make_response("")
	res.set_cookie("font", request.form.get('font'), 60*60*24*15)
	res.headers['location'] = url_for('article')
	return res, 302

    return render_template('article.html')


@app.route('/visits-counter/')
def visits():
    if 'visits' in session:
	session['visits'] = session.get('visits') + 1
    else:
	session['visits'] = 1
    return "Total visits: {}".format(session.get('visits'))


@app.route('/delete-visits/')
def delete_visits():
    session.pop('visits', None)  # удаление посещений
    return 'Visits deleted'


@app.route('/session/')
def updating_session():
    res = str(session.items())

    cart_item = {'pineapples': '10', 'apples': '20', 'mangoes': '30'}
    if 'cart_item' in session:
	session['cart_item']['pineapples'] = '100'
	session.modified = True
    else:
	session['cart_item'] = cart_item
return res


@app.route('/admin/')
@login_required
def admin():
     return render_template('admin.html')

```

**views.py** содержит не только функции представления. 

Сюда перемещен код моделей, классов форм и другие функции для соответствующих файлов:

> models.py


```
from app import db, login_manager
from datetime import datetime
from flask_login import (LoginManager, UserMixin, login_required,
			  login_user, current_user, logout_user)
from werkzeug.security import generate_password_hash, check_password_hash

class Category(db.Model):
    __tablename__ = 'categories'
    id = db.Column(db.Integer(), primary_key=True)
    name = db.Column(db.String(255), nullable=False, unique=True)
    slug = db.Column(db.String(255), nullable=False, unique=True)
    created_on = db.Column(db.DateTime(), default=datetime.utcnow)
    posts = db.relationship('Post', backref='category', cascade='all,delete-orphan')

    def __repr__(self):
	return "<{}:{}>".format(self.id, self.name)


post_tags = db.Table('post_tags',
    db.Column('post_id', db.Integer, db.ForeignKey('posts.id')),
    db.Column('tag_id', db.Integer, db.ForeignKey('tags.id'))
)


class Post(db.Model):
    __tablename__ = 'posts'
    id = db.Column(db.Integer(), primary_key=True)
    title = db.Column(db.String(255), nullable=False)
    slug = db.Column(db.String(255), nullable=False)
    content = db.Column(db.Text(), nullable=False)
    created_on = db.Column(db.DateTime(), default=datetime.utcnow)
    updated_on = db.Column(db.DateTime(), default=datetime.utcnow, onudate=datetime.utcnow)
    category_id = db.Column(db.Integer(), db.ForeignKey('categories.id'))

    def __repr__(self):
	return "<{}:{}>".format(self.id, self.title[:10])


class Tag(db.Model):
    __tablename__ = 'tags'
    id = db.Column(db.Integer(), primary_key=True)
    name = db.Column(db.String(255), nullable=False)
    slug = db.Column(db.String(255), nullable=False)
    created_on = db.Column(db.DateTime(), default=datetime.utcnow)
    posts = db.relationship('Post', secondary=post_tags, backref='tags')
    def __repr__(self):
	return "<{}:{}>".format(self.id, self.name)


class Feedback(db.Model):
    __tablename__ = 'feedbacks'
    id = db.Column(db.Integer(), primary_key=True)
    name = db.Column(db.String(1000), nullable=False)
    email = db.Column(db.String(100), nullable=False)
    message = db.Column(db.Text(), nullable=False)
    created_on = db.Column(db.DateTime(), default=datetime.utcnow)

    def __repr__(self):
	return "<{}:{}>".format(self.id, self.name)


class Employee(db.Model):
    __tablename__ = 'employees'
    id = db.Column(db.Integer(), primary_key=True)
    name = db.Column(db.String(255), nullable=False)
    designation = db.Column(db.String(255), nullable=False)
    doj = db.Column(db.Date(), nullable=False)


@login_manager.user_loader
def load_user(user_id):
    return db.session.query(User).get(user_id)


class User(db.Model, UserMixin):
    __tablename__ = 'users'
    id = db.Column(db.Integer(), primary_key=True)
    name = db.Column(db.String(100))
    username = db.Column(db.String(50), nullable=False, unique=True)
    email = db.Column(db.String(100), nullable=False, unique=True)
    password_hash = db.Column(db.String(100), nullable=False)
    created_on = db.Column(db.DateTime(), default=datetime.utcnow)
    updated_on = db.Column(db.DateTime(), default=datetime.utcnow, onupdate=datetime.utcnow)

    def __repr__(self):
	return "<{}:{}>".format(self.id, self.username)

    def set_password(self, password):
	self.password_hash = generate_password_hash(password)

    def check_password(self, password):
	return check_password_hash(self.password_hash, password)
forms.py

from flask_wtf import FlaskForm
from wtforms import Form, ValidationError
from wtforms import StringField, SubmitField, TextAreaField, BooleanField
from wtforms.validators import DataRequired, Email


class ContactForm(FlaskForm):
    name = StringField("Name: ", validators=[DataRequired()])
    email = StringField("Email: ", validators=[Email()])
    message = TextAreaField("Message", validators=[DataRequired()])
    submit = SubmitField()


class LoginForm(FlaskForm):
    username = StringField("Username", validators=[DataRequired()])
    password = StringField("Password", validators=[DataRequired()])
    remember = BooleanField("Remember Me")
    submit = SubmitField()

```

&nbsp;

> utils.py

```
from . import mail, db
from flask import render_template
from threading import Thread
from app import app
from flask_mail import Message


def async_send_mail(app, msg):
    with app.app_context():
	mail.send(msg)


def send_mail(subject, recipient, template, **kwargs):
    msg = Message(subject, sender=app.config['MAIL_DEFAULT_SENDER'],  recipients=[recipient])
    msg.html = render_template(template, **kwargs)
    thrd = Thread(target=async_send_mail, args=[app,  msg])
    thrd.start()
    return thrd

```

> Наконец, для запуска приложения нужно добавить следующий код в файл **runner.py**:

```
import os
from app import app, db
from app.models import User, Post, Tag, Category, Employee, Feedback
from flask_script import Manager, Shell
from flask_migrate import MigrateCommand

manager = Manager(app)

# эти переменные доступны внутри оболочки без явного импорта
def make_shell_context():
    return dict(app=app, db=db, User=User, Post=Post, Tag=Tag,  Category=Category, Employee=Employee, Feedback=Feedback)

manager.add_command('shell', Shell(make_context=make_shell_context))
manager.add_command('db', MigrateCommand)

if __name__ == '__main__':
    manager.run()

```
**runner.py** — это точка входа проекта.

В первую очередь файл создает экземпляр объекта `Manager()`. Затем он определяет функцию `make_shell_context()`. Объекты, которые вернет `make_shell_context()`, будут доступны в оболочке без импорта соответствующих инструкций. Наконец, метод `run()` экземпляра `Manager` будет вызван для запуска сервера.

&nbsp;

&nbsp;

### Порядок выполнения - разъяснения всего процесса

В этом уроке было создано немало файлов, и достаточно легко запутаться в том, за что отвечает каждый из них, а также в порядке, в котором они запускаются. Этот раздел создан для разъяснения всего процесса!

Все начинается с исполнения файла **runner.py**. 

Вторая строка в файле **runner.py** импортирует `app` и `db` из пакета `app`. Когда интерпретатор Python доходит до этой строки, управление программой передается файлу **__init__.py**, который в этот момент начинает исполняться. 

На **7** строке **__init__.py** импортирует модуль `config`, который передает управление **config.py**. Когда исполнение **config.py** завершается, управление снова возвращается к **__init__.py**. 

На **21** строке **__init__.py** импортирует модуль `views`, который передает управление **views.py**. 

Первая строка **views.py** снова импортирует экземпляр приложения `app` из пакета `app`. Экземпляр приложения `app` уже в памяти, поэтому снова он не будет импортирован. 

На строках **4, 5** и **6** **views.py** импортирует модели, формы и функцию `send_mail` и временно передает управление соответствующим файлам. Когда исполнение **views.py** завершается, управление программой возвращается к **__init__.py**. Это завершает исполнение **__init__.py**. Управление возвращается к **runner.py** и начинается исполнения инструкции на строке **3**.

Третья строка runner.py импортирует классы, определенные в модуле models.py. Поскольку модели уже доступны в файле **views.py**, файл **models.py** не будет исполняться.

Поскольку **runner.py** работает как основной модуль, условие на **17** строке выполнено, и `manager.run()` запускает приложение.

&nbsp;

&nbsp;

### Запуск проекта

Теперь можно запускать проект. В терминале для запуска сервера нужно ввести следующую команду.

```
(env) ssob@rh:~/flask_app$ python runner.py runserver
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 391-587-440
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

```
Если переменная среды `FLASK_ENV` не установлена, предыдущая команда запустит приложение в режиме отладки. 

Если зайти на https://127.0.0.1:5000/, откроется домашняя страница со следующим содержанием:

```
Name: Sunny

```
Также необходимо проверить остальные страницы приложения, чтобы убедиться, что все работает.

Приложение теперь является гибким. Оно может получить совсем иной набор настроек с помощью всего лишь одной переменной среды.

Предположим, нужно перевести приложение в рабочий режим. Для нужно всего лишь создать переменную среды `FLASK_ENV` со значением `config`.ProductionConfig.

* В терминале в **Linux** или **macOS** нужно вести следующую команду для создания переменной среды `FLASK_ENV`:
```
(env) ssob@rh:~/flask_app$ export FLASK_ENV=config.ProductionConfig

```

Пользователи **Windows** могут использовать следующую команду:

```
(env) C:\Users\ssob\flask_app>set FLASK_ENV=config.ProductionConfig

```

* Снова запустим приложение:

```
(env) ssob@rh:~/flask_app$ python runner.py runserver
* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

```

Теперь приложение работает в рабочем режиме. Если сейчас Python вызовет исключения, то вместо трассировки стека отобразится ***ошибка 500***.

Поскольку сейчас все еще этап разработки, переменную среды `FLASK_ENV` следует удалить. Она будет удалена автоматически при закрытии терминала. 

Чтобы сделать это вручную, нужно в терминале **Linux** ввести следующую команду:

```
(env) ssob@rh:~/flask_app$ unset FLASK_ENV

```

Пользователи **Windows** могут использовать следующую команду:
```
(env) C:\Users\ssob\flask_app>set FLASK_ENV=

```
Проект теперь в лучшей форме. Его элементы организованы более логично. Использованный здесь подход подойдет для маленьких и средних по масштабу проектов. 

Тем не менее у **Flask** есть еще несколько козырей для тех, кто хочет быть еще продуктивнее.


&nbsp;
---
&nbsp;

## Blueprints (чертежы/схемы/планы/эскизы)

**Эскизы** — еще один способ организации приложений. Они предполагают разделение на уровне представления. 

Как и приложение Flask, эскиз может иметь собственные функции представления, шаблоны и статические файлы. Для них даже можно выбрать собственные URI. Предположим, ведется работа над блогом и административной панелью. Чертеж для блога будет включать функцию представления, шаблоны и статические файлы, необходимые только блогу. В то же время эскиз административной панели будет содержать файлы, которые нужны ему. Их можно использовать в виде модулей или пакетов.

Пришло время добавить ***эскиз*** к проекту.


### Создание ***эскиза***

* Сначала нужно создать папку main в папке `flask_app/app` и переместить туда **views.py** и **forms.py**. 

* Внутри папки `main` необходимо создать файл **__init__.py** со следующим кодом:

```
from flask import Blueprint

main = Blueprint('main', __name__)

from . import views

```

Здесь создается объект ***эскиза*** с помощью класса `Blueprint`. 

Конструктор `Blueprint()` принимает два аргумента: **имя эскиза** и **имя пакета**, где он расположен; для большинства приложений достаточно будет передать `__name__`.

По умолчанию функции представления в ***эскизе*** будут искать шаблоны и статические файлы в папках приложения `templates` и `static`.

Изменить это можно, задав местоположение шаблонов и статических файлов при создании объекта `Blueprint`:

```
main = Blueprint('main', __name__, 
		template_folder='templates_dir',
		static_folder='static_dir')

```

В этом случае **Flask** будет искать шаблоны и статические файлы в папках `templates_dir` и `static_dir`, которые находятся в папке ***эскиза***.

Путь шаблона, добавленный в ***эскизе***, имеет более *низкий приоритет* по сравнению с папкой шаблонов приложения. Это значит, что если есть два шаблона с одинаковыми именами в папках `templates_dir` и `templates`, **Flask** использует шаблон из папки `templates`.

***Вот некоторые вещи, которые важно помнить, когда речь заходит о эскизах:***

 1. При использовании эскизов маршруты определяется с помощью декоратора `route`, а не экземпляра приложения `(app)`.
 2. Для создания URL при использовании эскизов нужно в качестве префикса указать название эскиза, а через оператор точки (`.`) — конечную точку. Это необходимо для создания **URL** и в Python, и в шаблонах. 

Например:

```
url_for("main.index")
```

Этот код вернет URL маршрута index эскиза main.

* Можно не указывать название эскиза, если работа ведется в том же эскизе, для которого создается URL.

Например:
```
url_for(".index")
```
Этот код вернет **URL** маршрута `index` для эскиза `main` в том случае, если код редактируется в функции представления или шаблоне эскиза `main`.


&nbsp;


* Чтобы приспособить изменения, нужно обновить инструкции `import`, вызовы `url_for()` и маршруты во **views.py**. 

Это обновленная версия файла **views.py**:

```
from app import app, db
from . import main
from flask import Flask, request, render_template, redirect, url_for, flash, make_response, session
from flask_login import login_required, login_user, current_user, logout_user
from app.models import User, Post, Category, Feedback, db
from .forms import ContactForm, LoginForm
from app.utils import send_mail


@main.route('/')
def index():
    return render_template('index.html', name='Jerry')


@main.route('/user/<int:user_id>/')
def user_profile(user_id):
    return "Profile page of user #{}".format(user_id)


@main.route('/books/<genre>/')
def books(genre):
    return "All Books in {} category".format(genre)


@main.route('/login/', methods=['post', 'get'])
def login():
    if current_user.is_authenticated:
	return redirect(url_for('.admin'))
    form = LoginForm()
    if form.validate_on_submit():
	user = db.session.query(User).filter(User.username == form.username.data).first()
	if user and user.check_password(form.password.data):
	    login_user(user, remember=form.remember.data)
	    return redirect(url_for('.admin'))

	flash("Invalid username/password", 'error')
	return redirect(url_for('.login'))
    return render_template('login.html',  form=form)


@main.route('/logout/')
@login_required
def logout():
    logout_user()
    flash("You have been logged out.")
    return redirect(url_for('.login'))


@main.route('/contact/', methods=['get', 'post'])
def contact():
    form = ContactForm()
    if form.validate_on_submit():
	name = form.name.data
	email = form.email.data
	message = form.message.data
	print(name)
	print(email)
	print(message)

	# здесь логика БД 
	feedback = Feedback(name=name, email=email, message=message)
	db.session.add(feedback)
	db.session.commit()

	send_mail("New Feedback", app.config['MAIL_DEFAULT_SENDER'], 'mail/feedback.html',
		  name=name, email=email)

	print("\nData received. Now redirecting ...")
	flash("Message Received", "success")
	return redirect(url_for('.contact'))

    return render_template('contact.html',  form=form)


@main.route('/cookie/')
def cookie():
    if not request.cookies.get('foo'):
	res = make_response("Setting a cookie")
	res.set_cookie('foo', 'bar', max_age=60*60*24*365*2)
    else:
	res = make_response("Value of cookie foo is {}".format(request.cookies.get('foo')))
    return res


@main.route('/delete-cookie/')
def delete_cookie():
    res = make_response("Cookie Removed")
    res.set_cookie('foo', 'bar', max_age=0)
    return res


@main.route('/article/', methods=['POST', 'GET'])
def article():
    if request.method == 'POST':
	print(request.form)
	res = make_response("")
	res.set_cookie("font", request.form.get('font'), 60*60*24*15)
	res.headers['location'] = url_for('.article')
	return res, 302

    return render_template('article.html')


@main.route('/visits-counter/')
def visits():
    if 'visits' in session:
	session['visits'] = session.get('visits') + 1  # чтение и обновление данных сессии
    else:
	session['visits'] = 1  # настройка данных сессии
    return "Total visits: {}".format(session.get('visits'))


@main.route('/delete-visits/')
def delete_visits():
    session.pop('visits', None)  # удаление посещений
    return 'Visits deleted'


@main.route('/session/')
def updating_session():
    res = str(session.items())
    
    cart_item = {'pineapples': '10', 'apples': '20', 'mangoes': '30'}
    if 'cart_item' in session:
	session['cart_item']['pineapples'] = '100'
	session.modified = True
    else:
	session['cart_item'] = cart_item
    return res


@main.route('/admin/')
@login_required
def admin():
    return render_template('admin.html')

```

Стоит обратить внимание, что в файле **views.py** URL создаются без определения названия ***эскиза***, потому что работа ведется в этом же ***эскизе***.

Также нужно следующим образом обновить вызов `url_for()` в **admin.html**:

```
#...
<p><a href="{{ url_for('.logout') }}">Logout</a></p>
#...

```
Функции представления во **views.py** теперь ассоциируются с эскизом `main`. 

* Дальше нужно зарегистрировать ***эскизы*** в приложении **Flask**. 

  Необходимо открыть `app/__init__.py` и изменить его следующим образом:

```
#...
# создать экземпляр приложения
app = Flask(__name__)
app.config.from_object(os.environ.get('FLASK_ENV') or 'config.DevelopementConfig')

# инициализирует расширения
db = SQLAlchemy(app)
mail = Mail(app)
migrate = Migrate(app,  db)
login_manager = LoginManager(app)
login_manager.login_view = 'main.login'

# регистрация blueprints
from .main import main as main_blueprint
app.register_blueprint(main_blueprint)

#from .admin import main as admin_blueprint
#app.register_blueprint(admin_blueprint)

```
Метод `register_blueprint()` экземпляра приложения используется для регистрации эскиза. 

Можно зарегистрировать несколько эскизов, вызвав `register_bluebrint()` для каждого. Важно обратить внимание, что на **11** строке `login_manager.login_view` присваивается `main.login`. В этом случае важно указать, о каком эскизе идет речь, иначе Flask не сможет понять, на какой эскиз ссылается код.


&nbsp;

***Сейчас структура приложения выглядит так:***

```
├── flask_app/
├── app/
│  ├── __init__.py
│  ├── main/
│  │  ├── forms.py
│  │  ├── __init__.py
│  │  └── views.py
│  ├── models.py
│  ├── static/
│  │  └── style.css
│  ├── templates/
│  │  ├── admin.html
│  │  ├── article.html
│  │  ├── contact.html
│  │  ├── index.html
│  │  ├── login.html
│  │  └── mail/
│  │  └── feedback.html
│  └── utils.py
├── migrations/
│  ├── alembic.ini
│  ├── env.py
│  ├── README
│  ├── script.py.mako
│  └── versions/
│  ├── 0f0002bf91cc_adding_users_table.py
│  ├── 6e059688f04e_adding_employees_table.py
├── runner.py
├── config.py
├── env/
```

&nbsp;
---
&nbsp;


## Фабрика приложения

В приложении уже используются пакеты и эскизы (`blueprints`). Его можно улучшать и дальше, передав функцию создания экземпляров приложения Фабрике приложения. Это всего лишь функция, создающая объект.

Что это даст:

Упростит процесс тестирования, потому что можно будет создать экземпляр приложения с разными настройками
Можно будет запускать несколько экземпляров одного приложения в одном и том же процессе. Это удобно, когда используется балансировка нагрузки трафика между разными серверами.

* Для внедрения фабрики приложения нужно обновить `app/__init__.py`:

```
from flask import Flask
from flask_migrate import Migrate, MigrateCommand
from flask_mail import Mail, Message
from flask_sqlalchemy import SQLAlchemy
from flask_script import Manager, Command, Shell
from flask_login import LoginManager
import os, config

db = SQLAlchemy()
mail = Mail()
migrate = Migrate()
login_manager = LoginManager()
login_manager.login_view = 'main.login'

# Фабрика приложения
def create_app(config):
    
    # создание экземпляра приложения
    app = Flask(__name__)
    app.config.from_object(config)

    db.init_app(app)
    mail.init_app(app)
    migrate.init_app(app,  db)
    login_manager.init_app(app)

    from .main import main as main_blueprint
    app.register_blueprint(main_blueprint)

    #from .admin import main as admin_blueprint
    #app.register_blueprint(admin_blueprint)
    
    return app

```

еперь за создание экземпляров приложения ответственна функция create_app. Она принимает один аргумент config и возвращает экземпляр приложения.

Фабрика приложений разделяет процесс создания экземпляров расширений и их настройки. Создание экземпляров происходит до того, как create_app() вызывается, а настройка происходит внутри функции create_app() с помощью метода init_app().

Дальше нужно обновить runner.py для фабрики приложения:

```
import os
from app import  db,  create_app
from app.models import User, Post, Tag, Category, Employee, Feedback
from flask_script import Manager, Shell
from flask_migrate import MigrateCommand

app = create_app(os.getenv('FLASK_ENV') or 'config.DevelopementConfig')
manager = Manager(app)

def make_shell_context():
    return dict(app=app, db=db, User=User, Post=Post, Tag=Tag, Category=Category,
		Employee=Employee, Feedback=Feedback)

manager.add_command('shell', Shell(make_context=make_shell_context))
manager.add_command('db', MigrateCommand)

if __name__ == '__main__':
    manager.run()

```

Стоит заметить, что при использовании фабрик приложения пропадает доступ к экземпляру приложения в эскизе во время импорта. Для получения доступа к экземплярам в эскизе нужно использовать прокси current_app из пакета flask. Необходимо обновить проект для использования переменной current_app:

```
from app import db
from . import main
from flask import (render_template, request, redirect, url_for, flash,
		   make_response, session, current_app)
from flask_login import login_required, login_user, current_user, logout_user
from app.models import User, Feedback
from app.utils import send_mail
from .forms import ContactForm, LoginForm


@main.route('/')
def index():
    return render_template('index.html', name='Jerry')


@main.route('/user/<int:user_id>/')
def user_profile(user_id):
    return "Profile page of user #{}".format(user_id)


@main.route('/books/<genre>/')
def books(genre):
    return "All Books in {} category".format(genre)


@main.route('/login/', methods=['post', 'get'])
def login():
    if current_user.is_authenticated:
	return redirect(url_for('.admin'))
    form = LoginForm()
    if form.validate_on_submit():
	user = db.session.query(User).filter(User.username == form.username.data).first()
	if user and user.check_password(form.password.data):
	    login_user(user, remember=form.remember.data)
	    return redirect(url_for('.admin'))

	flash("Invalid username/password", 'error')
	return redirect(url_for('.login'))
    return render_template('login.html', form=form)


@main.route('/logout/')
@login_required
def logout():
    logout_user()
    flash("You have been logged out.")
    return redirect(url_for('.login'))


@main.route('/contact/', methods=['get', 'post'])
def contact():
    form = ContactForm()
    if form.validate_on_submit():
	name = form.name.data
	email = form.email.data
	message = form.message.data

	# логика БД здесь
	feedback = Feedback(name=name, email=email, message=message)
	db.session.add(feedback)
	db.session.commit()

	send_mail("New Feedback", current_app.config['MAIL_DEFAULT_SENDER'], 'mail/feedback.html',
		  name=name, email=email)
	
	flash("Message Received", "success")
	return redirect(url_for('.contact'))

    return render_template('contact.html', form=form)


@main.route('/cookie/')
def cookie():
    if not request.cookies.get('foo'):
	res = make_response("Setting a cookie")
	res.set_cookie('foo', 'bar', max_age=60*60*24*365*2)
    else:
	res = make_response("Value of cookie foo is {}".format(request.cookies.get('foo')))
    return res


@main.route('/delete-cookie/')
def delete_cookie():
    res = make_response("Cookie Removed")
    res.set_cookie('foo', 'bar', max_age=0)
    return res


@main.route('/article', methods=['POST', 'GET'])
def article():
    if request.method == 'POST':
	res = make_response("")
	res.set_cookie("font", request.form.get('font'), 60*60*24*15)
	res.headers['location'] = url_for('.article')
	return res, 302

    return render_template('article.html')


@main.route('/visits-counter/')
def visits():
    if 'visits' in session:
	session['visits'] = session.get('visits') + 1
    else:
	session['visits'] = 1
    return "Total visits: {}".format(session.get('visits'))


@main.route('/delete-visits/')
def delete_visits():
    session.pop('visits', None)  # удаление посещений
    return 'Visits deleted'


@main.route('/session/')
def updating_session():
    res = str(session.items())
    
    cart_item = {'pineapples': '10', 'apples': '20', 'mangoes': '30'}
    if 'cart_item' in session:
	session['cart_item']['pineapples'] = '100'
	session.modified = True
    else:
	session['cart_item'] = cart_item

    return res


@main.route('/admin/')
@login_required
def admin():
    return render_template('admin.html')

```

> utils.py

```
from . import mail, db
from flask import render_template, current_app
from threading import Thread
from flask_mail import Message


def async_send_mail(app, msg):
    with app.app_context():
	mail.send(msg)


def send_mail(subject, recipient, template, **kwargs):
    msg = Message(subject, sender=current_app.config['MAIL_DEFAULT_SENDER'], recipients=[recipient])
    msg.html = render_template(template, **kwargs)
    thr = Thread(target=async_send_mail, args=[current_app._get_current_object(), msg])
    thr.start()
    return thr

```

>В этих уроках речь шла о многих вещах, которые дают необходимые знания о Flask, 
его составляющих и том, как они сочетаются между собой.


&nbsp;


&nbsp;

---


© 2023 S.Sobolewski
