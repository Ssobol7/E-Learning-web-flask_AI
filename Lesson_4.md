![Alt text](flask_app/img/flask-python.png)



# Lesson 4: 

# 18. Аутентификация во Flask

**Аутентификация** — один из самых важных элементов веб-приложений. Этот процесс предотвращает попадание неавторизованных пользователей на непредназначенные для них страницы. Собственную систему аутентификации можно создать с помощью куки и хэширования паролей. Такой миниатюрный проект станет отличной проверкой полученных навыков.

Как можно было догадаться, уже существует расширение, которое может значительно облегчить жизнь. Flask-Login — это расширение, позволяющее легко интегрировать систему аутентификации в приложение Flask. Установить его можно с помощью следующей команды:

```
(env) ssob@rh:~/flask_app$  pip install flask-login

```

### Создание модели пользователя

Сейчас информация о пользователях, которые являются администраторами или редакторами сайта, нигде не хранится. 

* **Первая задача** — создать модель **User** для хранения пользовательских данных. 
  Откроем **main2.py**, чтобы добавить модель **User** после модели **Employee**:

```
#..
class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer(), primary_key=True)
    name = db.Column(db.String(100))
    username = db.Column(db.String(50), nullable=False, unique=True)
    email = db.Column(db.String(100), nullable=False, unique=True)
    password_hash = db.Column(db.String(100), nullable=False)
    created_on = db.Column(db.DateTime(), default=datetime.utcnow)
    updated_on = db.Column(db.DateTime(), default=datetime.utcnow,  onupdate=datetime.utcnow)

    def __repr__(self):
	return "<{}:{}>".format(self.id, self.username)
#...

```

* Для обновления базы данных нужно создать новую миграцию. В терминале для создания нового скрипта миграции необходимо ввести следующую команду:

```
(env) ssob@rh:~/flask_app$ python main2.py db migrate -m "Adding users table"

```

* Запустить миграцию необходимо с помощью команды `upgrade`:

```
(env) ssob@rh:~/flask_app$ python main2.py db upgrade

INFO [alembic.runtime.migration] Context impl MySQLImpl.
INFO [alembic.runtime.migration] Will assume non-transactional DDL.
INFO [alembic.runtime.migration] Running upgrade 6e059688f04e -> 0f0002bf91cc,
Adding users table

(env) ssob@rh:~/flask_app$

```

Это создаст таблицу ***users*** в базе данных.



###  Хэширование паролей

Пароли никогда не должны храниться в виде чистого текста в базе данных. Если так делать, злоумышленник, способный взломать базу данных, получит возможность узнать и пароли, и электронные адреса. Известно, что люди используют один и тот же пароль для разных сайтов, а это значит, что одна комбинация откроет злоумышленнику доступ к остальным аккаунтам пользователей.

Вместо хранения паролей прямо в базе данных, нужно сохранять их хэши. 

**Хэш** — это строка символов, которые смотрятся так, будто бы были подобраны случайно:

```
pbkdf2:sha256:50000$Otfe3YgZ$4fc9f1d2de2b6beb0b888278f21a8c0777e8ff980016e043f3eacea9f48f6dea

```

Хэш создается с помощью *односторонней функции хэширования*. Она принимает длину переменной и возвращает вывод фиксированной длины, которую мы и называем хэшем. Безопасным хэш делает тот факт, что его нельзя использовать для получения изначальной строки (поэтому функция и называется односторонней). Тем не менее для одного ввода односторонняя функция хэширования будет возвращать один и тот же результат.

Вот процессы, которые задействованы при создании хэша пароля:

Когда пользователь передает пароль (на этапе регистрации), необходимо его хэшировать и сохранить хэш в базу данных. Когда пользователь будет снова авторизоваться, функция повторно создаст хэш и сравнит его с тем, что хранится в базе данных. Если они совпадают, пользователь получит доступ к аккаунту. В противном случае, возникнет ошибка.

**Flask** поставляется с пакетом **Werkzeug**, в котором есть две вспомогательные функции для хэширования паролей.

* Метод `generate_password_hash(password)` - Принимает пароль и возвращает хэш. По умолчанию использует одностороннюю функцию **pbkdf2** для создания хэша.

* Метод `check_password_hash(password_hash, password)` - Принимает хэш и пароль в чистом виде, затем сравнивает **password** и **password_hash**. Если они одинаковые, возвращает **True**.

**Следующий код демонстрирует, как работать с этими функциями:**

```
>>>
>>> from werkzeug.security import generate_password_hash, check_password_hash
>>>
>>> hash = generate_password_hash("secret password")
>>>
>>> hash
'pbkdf2:sha256:50000$zB51O5L3$8a43788bc902bca96e01a1eea95a650d9d5320753a2fbd16bea984215cdf97ee'
>>>
>>> check_password_hash(hash, "secret password")
True
>>>
>>> check_password_hash(hash, "pass")
False
>>>

```
Стоит обратить внимание, что когда `check_password_hash()`` вызывается с правильными паролем **(“secret password”)**, возвращается **True**, а если с неправильными — **False**.


* Дальше нужно обновить модель **User**, и добавить в нее хэширование паролей:

```
#...
from werkzeug.security import generate_password_hash,  check_password_hash
#...

#...
class User(db.Model):
    #...
    updated_on = db.Column(db.DateTime(), default=datetime.utcnow, onupdate=datetime.utcnow)

    def __repr__(self):
	return "<{}:{}>".format(self.id, self.username)

    def set_password(self, password):
	self.password_hash = generate_password_hash(password)

    def check_password(self,  password):
	return check_password_hash(self.password_hash, password)
    #...
```

* Создадим пользователей, чтобы проверить хэширование паролей.
```
(env) ssob@rh:~/flask_app$ python main2.py shell
>>>
>>> from main2 import db, User
>>>
>>> u1 = User(username='spike', email='spike@example.com')
>>> u1.set_password("spike")
>>>
>>> u2 = User(username='tyke', email='tyke@example.com')
>>> u2.set_password("tyke")
>>>
>>> db.session.add_all([u1, u2])
>>> db.session.commit()
>>>
>>> u1, u2
(<1:spike>, <2:tyke>)
>>>
>>>
>>> u1.check_password("pass")
False
>>> u1.check_password("spike")
True
>>>
>>> u2.check_password("foo")
False
>>> u2.check_password("tyke")
True
>>>
>>>
```
Вывод демонстрирует, что все работает как нужно, и в базе данных теперь есть **два** пользователя.



### Интеграция Flask-Login

Для запуска **Flask-Login** нужно импортировать класс `LoginManager` из пакета `flask_login` и создать новый экземпляр `LoginManager`:

```
from werkzeug.security import generate_password_hash, check_password_hash
from flask_login import LoginManager

app = Flask(__name__)
app.debug = True
app.config['SECRET_KEY'] = 'a really really really really long secret key'
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:pass@localhost/flask_app_db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['MAIL_SERVER'] = 'smtp.googlemail.com'
app.config['MAIL_PORT'] = 587
app.config['MAIL_USE_TLS'] = True
app.config['MAIL_USERNAME'] = 'youmail@gmail.com'
app.config['MAIL_DEFAULT_SENDER'] = 'youmail@gmail.com'
app.config['MAIL_PASSWORD'] = 'password'

manager = Manager(app)
manager.add_command('db', MigrateCommand)
db = SQLAlchemy(app)
migrate = Migrate(app,  db)
mail = Mail(app)
login_manager = LoginManager(app)
#...

```
Для проверки пользователей **Flask-Login** требует добавления нескольких методов в класс **User**. 

Эти методы и их описание перечислены ниже:
                  
* `is_authenticated()` - Возвращает **True**, если пользователь проверен (то есть, зашел с корректным паролем). В противном случае — **False**.

* `is_active()` - Возвращает **True**, если действие аккаунта не приостановлено.

* `is_anonymous()` - Возвращает **True** для неавторизованных пользователей.

* `get_id()` - Возвращает уникальный идентификатор объекта **User**.


**Flask-Login** предлагает реализацию этих методов по умолчанию с помощью класса `UserMixin`. Так, вместо определения их вручную, можно настроить их наследование из класса `UserMixin`. 

Откроем **main2.py**, чтобы изменить заголовок модели **User**:

```
#...
from flask_login import LoginManager, UserMixin

#...
class User(db.Model, UserMixin):
    __tablename__ = 'users'
#...

```

* Осталось только добавить обратный вызов `user_loader`. Соответствующий метод можно добавить над моделью **User**:

```
#...
@login_manager.user_loader
def load_user(user_id):
    return db.session.query(User).get(user_id)
#...

```

Функция, принимающая в качестве аргумента декоратор `user_loader`, будет вызываться с каждым запросом к серверу. Она загружает пользователя из идентификатора пользователя в куки сессии. **Flask-Login** делает загруженного пользователя доступным с помощью прокси `current_user`. Для использования `current_user` его нужно импортировать из пакета `flask_login`. Он ведет себя как глобальная переменная и доступен как в функциях представления, так и в шаблонах. В любой момент времени `current_user` ссылается либо на вошедшего в систему, либо на анонимного пользователя. Различать их можно с помощью атрибута `is_authenticated` прокси `current_user`. Для анонимных пользователей `is_authenticated` вернет **False**. В противном случае — **True**.



#### Ограничение доступа к просмотру

Пока что на сайте нет никакой административной панели. В этом уроке она будет представлена обычной страницей. Чтобы не допустить неавторизованных пользователей к защищенным страница у **Flask-Login** есть декоратор `login_required`. 

Добавим следующий код в файле **main2.py** сразу за функцией представления `updating_session()`:

```
#...
from flask_login import LoginManager, UserMixin, login_required
#...
@app.route('/admin/')
@login_required
def admin():
    return render_template('admin.html')
#...

```

Декоратор `login_required` гарантирует, что функция представления `admin()` вызовется только в том случае, если пользователь авторизован. По умолчанию, если анонимный пользователь попытается зайти на защищенную страницу, он получит ошибку **401 «Не авторизован»**.

Необходимо запустить сервер и зайти на https://localhost:5000/login, чтобы проверить, как это работает.

Откроется такая страница:

![](img/stranica-s-oshibkoj-401.png)

Вместо того чтобы показывать пользователю **ошибку 401**, лучше перенаправить его на страницу авторизации. 

Чтобы сделать это, нужно передать атрибуту `login_view` экземпляра `LoginManager` значение функции представления `login()`:

```
#...
migrate = Migrate(app, db)
mail = Mail(app)
login_manager = LoginManager(app)
login_manager.login_view = 'login'

class  Faker(Command):
    'A command to add fake data to the tables'
#...

```

* Сейчас функция `login()` определена следующим образом (но ее нужно будет поменять):

```
#...
@app.route('/login/', methods=['post',  'get'])
def login():
    message = ''
    if request.method == 'POST':
	print(request.form)
	username = request.form.get('username')
	password = request.form.get('password')

	if username == 'root' and password == 'pass':
	    message = "Correct username and password"
	else:
	    message = "Wrong username or password"
    
    return render_template('login.html', message=message)
#...

```

Если теперь зайти на https://localhost:5000/admin/, произойдет перенаправление на страницу авторизации:


![](img/stranica-avtorizacii-vo-flask.png)

**Flask-Login** также настраивает всплывающее сообщение, когда пользователя перенаправляют на страницу авторизации, но сейчас никакого сообщения нет, потому что шаблон авторизации (**template/login.html**) не отображает никаких сообщений. Нужно открыть **login.html** и добавить следующий код перед тегом `<form>`:

```
#...
    {% endif %}

    {% for category, message in  get_flashed_messages(with_categories=true) %}
	<spam class="{{ category }}">{{ message }}</spam>
    {% endfor %}

    <form action="" method="post">
#...

```

Если снова зайти на https://localhost:5000/admin/, на странице отобразится сообщение.

![](img/oshibka-avtorizacii-vo-flask.png)


Чтобы изменить содержание сообщения, нужно передать новый текст атрибуту `login_message` экземпляра `LoginManager``.

Заодно почему бы не создать шаблон для функции представления `admin()`. 

Создадим новый шаблон **admin.html** со следующим кодом:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<h2>Logged in User details</h2>

<ul>
    <li>Username: {{ current_user.username }}</li>
    <li>Email: {{ current_user.email }}</li>
    <li>Created on: {{ current_user.created_on }}</li>
    <li>Updated on: {{ current_user.updated_on }}</li>
</ul>

</body>
</html>

```
Здесь используется переменная current_user для отображения подробностей о авторизованном пользователе.


### Создание формы авторизации

Перед авторизацией нужно создать форму. 

В ней будет три поля: ***имя пользователя***, ***пароль*** и ***запомнить меня***. 

Откроем **forms.py**, чтобы добавить класс `LoginForm` под классом `ContactForm`:

```
#...
from wtforms import StringField, SubmitField, TextAreaField,  BooleanField, PasswordField
#...
#...
class LoginForm(FlaskForm):
    username = StringField("Username", validators=[DataRequired()])
    password = PasswordField("Password", validators=[DataRequired()])
    remember = BooleanField("Remember Me")
    submit = SubmitField()

```


### Авторизация пользователей

Для авторизации пользователя Flask-Login предоставляет функцию `login_user()`. Она принимает объект пользователя. В случае успеха возвращает **True** и устанавливает сессию. В противном случае — **False**. 

По умолчанию сессия, установленная `login_user()`, заканчивается при закрытии браузера. Чтобы позволить пользователям оставаться авторизованными на дольше, нужно передать `remember=True` функции `login_user()` при авторизации пользователя. 

Откроем **main2.py**, чтобы изменить функцию представления `login()`:

```
#...
from forms import ContactForm, LoginForm
#...
from flask_login import LoginManager, UserMixin, login_required, login_user, current_user

#...
@app.route('/login/', methods=['post', 'get'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
	user = db.session.query(User).filter(User.username == form.username.data).first()
	if user and user.check_password(form.password.data):
	    login_user(user, remember=form.remember.data)
	    return redirect(url_for('admin'))

	flash("Invalid username/password", 'error')
	return redirect(url_for('login'))
    return render_template('login.html', form=form)
#...

```
Дальше нужно обновить **login.html**, чтобы использовать класс `LoginForm()`. 

Нужно добавить в файл следующие изменения:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
</head>
<body>

    {% for category, message in  get_flashed_messages(with_categories=true) %}
	<spam class="{{ category }}">{{ message }}</spam>
    {% endfor %}

    <form action="" method="post">
	{{ form.csrf_token }}
	<p>
	    {{ form.username.label() }}
	    {{ form.username() }}
	    {% if form.username.errors %}
		{% for error in form.username.errors %}
		    {{ error }}
		{% endfor %}
	    {% endif %}
	</p>
	<p>
	    {{ form.password.label() }}
	    {{ form.password() }}
	    {% if form.password.errors %}
		{% for error in form.password.errors %}
		    {{ error }}
		{% endfor %}
	    {% endif %}
	</p>
	<p>
	    {{ form.remember.label() }}
	    {{ form.remember() }}
	</p>
	<p>
	    {{ form.submit() }}
	</p>
    </form>
    
</body>
</html>

```

Теперь можно авторизоваться. 

Если зайти https://localhost:5000/admin, произойдет перенаправление на страницу авторизации:

![](img/perenapravlenie-na-stranicu-avtorizacii.png)


Необходимо ввести корректное имя пользователя и пароль и нажать **sumbit**. Произойдет перенаправление на страницу администратора, которая должна выглядеть следующим образом:

![](img/perenapravlenie-na-stranicu-administratora.png)


Если не кликнуть **“Remember Me”** при авторизации, после закрытия браузера сайт выйдет из аккаунта. Если кликнуть, то логин останется.

Если ввести неправильные имя пользователя или пароль, произойдет перенаправление на страницу авторизации со всплывающим сообщением:

![](img/oshibka-avtorizacii-vo-flask-2.png)



### Завершение сеансов пользователей (выход из аккаунтов)

Функция `logout_user()` во **Flask-Login** завершает сеанс пользователя, удаляя его идентификатор из сессии. 

В файле **main2.py** нужно добавить следующий код под функцией представления `login()`:

```
#...
from flask_login import LoginManager, UserMixin,  login_required, login_user, current_user, logout_user
#...
@app.route('/logout/')
@login_required
def logout():
    logout_user()
    flash("You have been logged out.")
    return redirect(url_for('login'))
#...

```

Далее необходимо обновить шаблон **admin.html**, чтобы добавить ссылку на маршрут `logout`:

```
#...
<ul>
    <li>Username: {{ current_user.username }}</li>
    <li>Email: {{ current_user.email }}</li>
    <li>Created on: {{ current_user.created_on }}</li>
    <li>Updated on: {{ current_user.updated_on }}</li>
</ul>

<p><a href="{{ url_for('logout') }}">Logout</a></p>

</body>
</html>

```

Если сейчас зайти на https://localhost:5000/admin/ (будучи авторизованным), то в нижней части страницы должны быть ссылка для выхода из аккаунта:

![](img/logout_link_in_admin_page.png)


Если ее нажать, произойдет перенаправление на страницу авторизации:


![](img/vyhod-iz-akkaunta.png)


### Финальные штрихи

Есть одна маленькая проблема со страницей авторизации. Сейчас если авторизованный пользователь зайдет на https://localhost:5000/login/, то он снова увидит страницу авторизации. Нет смысла в демонстрации формы авторизованному пользователю. 

Для разрешения этой проблемы нужно добавить следующие изменения в функцию представления `login()`:

```
#...
@app.route('/login/', methods=['post', 'get'])
def login():
    if current_user.is_authenticated:
	return redirect(url_for('admin'))
    form = LoginForm()
    if form.validate_on_submit():
#...

```

После этих изменений если авторизованный пользователь зайдет на страницу авторизации, он будет перенаправлен на страницу администратора.


&nbsp;


&nbsp;

---
© 2023 S.Sobolewski
