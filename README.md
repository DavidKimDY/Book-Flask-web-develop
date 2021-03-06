# Book-Flask-web-develop
# Flask

# 1. Feature

- Flask는 마이크로 프레임워크다.
- WSGI 서브시스템은 Werkzeung에, 템플릿은 jinja2에 의존한다.

# 2. Application Structure

### Application Instance

- 애플리케이션 인스턴스를 생성해야 한다.
- 웹 서버는 클라이언트로부터 request를 수신하는데 이를 Flask의 어플리케이션 인스턴스에서 처리하는데 이때 WSGI 라는 protocol을 사용한다.

### Route and View Function

- Route : URL과 URL를 처리하는 함수의 관련성을 Route라고 한다.  URL을 함수에 매핑하는 기능
- View Function : URL을 처리하는 함수를 View Function이라고 한다.
- Flask에서는 @app.route() decorator로 라우트를 정의해 줄 수 있다.
- 동적 컴포넌트는 @add.route('/user/<id>') 와 같이 지정해 줄 수 있다
- 동적 컴포넌트는 타입을 정의해줄 수 있다. @add.route('/user/<int: id>') int, float, path를 지원한다.
- 동적 컴포넌트의 입력 타입이 다를 경우 Not Found를 리턴한다.

### Server startup

- application instance는 run 메소드로 웹 서버를 실행한다.
- 서버를 실행하고 나면 서버는 루프를 진행한다.
- 플라스크에서 제공하는 웹 서버는 제품 생산을 목적으로 하지 않는다.

### Request context

- Flask가 client 에서 request를 수신하면 이를 처리하기 위해 view function에서는 여러개의 오브젝트를 생성해야 한다.
- requests object는 HTTP requests를 캡슐화 한다.
- view funtion이 request object에 액세스할 수 있는 확실한 방법은 인수로서 받는 것인데 이는 여분의 인수를 갖도록 요구한다. request object 만이 view function이 access 해야하는 오브젝트가 아니기 때문에 인수로 받는 것에는 무리가 있다. view function이 다수의 오브젝트를 인수로 받는걸 피하기 위해 Flask는 컨텍스트를 사용하여 임시적으로 오브젝트를 글로벌하게 액세스하도록 한다.

    ```python
    from flask import request

    @app.route('/')
    def index():
    	user_agent = request.headers.get('User-Agent')
    	return f'<p>your brower is {user_agent} </p>'
    ```

- [python context](https://www.geeksforgeeks.org/context-manager-in-python/)
- 다음과 같은 context 들이 있다 17.p

    → current_app : 활성화된 app을 위한 app  instance

    → g : request 마다 생성되는 임시 스토리지 오브젝트

    → request : HTTP request context를 캡슐화 하는 오브젝트

    → session : 사용자 세션

### Request Dispatch

- 클라이언트에서 request를 수신하면 그것을 서비스하기 위한 view function을 URL을 통해 찾는다. 이때 URL map에서 이를 찾는다. 파이썬 파일이름이 [practice.py](http://practice.py) 라고 한다면 다음과 같이 찾을 수 있다.

    ```python
    $ python
    >>> from practice import app
    >>> app.url_map
    Map([<Rule '/' (OPTIONS, HEAD, GET) -> index>,
     <Rule '/static/<filename>' (OPTIONS, HEAD, GET) -> static>,
     <Rule '/user/<name>' (OPTIONS, HEAD, GET) -> user>])
    ```

### Request Hooks

- request를 처리하기 전후에 코드를 동작시켜야 하는 경우가 있다. DB connection, 사용자 인증 같은 경우다.
- Flask는 request hooks를 데코레이터 형태로 실행하도록 한고 다음과 같은 4개의 종류가 있다

    → before_first_request

    → before_request

    → after_request

    → tesrdown_request

- Request Hooks function과 View function 사이에 데이터를 공유하기 위해 g context 전역 변수를 사용한다.

### Response

- Flask는 200을 기본 상태 코드로 응답한다.
- 200 이외의 상태코드를 리턴할때는 아래와 같이 return에 추가해주면 된다.

    ```python
    @app.route('/')
    def index():
    	return '<h1>Bad Requests</h1>', 400
    ```

- Response object를 return해줄수도 있다. 이때는 응답 설정이 가능하다는 장점이 있다.
- 아래코드는 쿠키를 설정하고 Response object를 return 하는 예이다.

```python
from flask import make_response

@app.route('/')
def index():
	response = make_response('<h1>This document carries a cookie </h1>')
	response.set_cookie('answer', '42')
	return response
```

### Redirect

- 응답의 한 종류이다.
- page document를 포함하지 않으며 새로운 page를 로드하는 새로운 URL을 브라우저에 전달한다.
- Flask는 redirect 함수를 제공한다

    ```python
    from flask import redirect

    @app.route('/')
    def index():
    	return redirect('http://www.example.com')
    ```

- 응답코드는 302 이다.

### Flask extension

- Flask-Script

    → 커맨드 라인 파서를 추가하는 확장

    → Flask의 startup 설정 옵션을 커맨드 라인에서 조절하게 해준다.즉, app.run() 호출의 인자를 커맨드 라인에서 조절가능하게 해준다.

    → 근데 최근에는 Flask 자체에 built-in CLI 기능을 제공한다고 한다.

    [Flask-Script - Flask-Script 0.4.0 documentation](https://flask-script.readthedocs.io/en/latest/)

# 3. Template

- 플라스크 뷰 함수는 완전히 독립된 두 개의 기능이 마치 하나의 기능처럼 보이는데 이는 유지 보수에 어려움을 줄 수 있다.
- 뷰 함수는 정확히 request에 대한 response를 생성하는 것이다.
- 일반적으로 request는 application의 상태를 변경한다

    사용자가 계정을 생성한다 
    → 사용자가 id와 pw를 입력하고 전송버튼을 누른다 
    → 서버에서는 데이터를 포함하는 사용자의 request를 받는다 
    → 플라스크는 request에 대한 동작을 처리하는 뷰 함수에 이 request를 dispatch 한다 
    → 뷰 함수는 데이터 베이스에 연결하여 사용자를 추가한다. 
    → 그에대한 응답을 사용자에게 보낸다.

    - 이 두 가지의 형태를 **비지니스 로직** 과 **프레젠테이션 로직** 으로 부른다.
    - 두가지 로직이 섞여 있으면 코드의 유지 보수는 어려워진다.
- 프레젠테이션 로직을 **템플릿** 으로 이동시키는 것은 application 유지 보수에 도움을 준다.
- 템플릿은 응답 텍스트를 포함하고 있는 파일이다.
- 템플릿은 request 내용에서 인식 가능한 동적 파트에 대한 변수들을 포함하고 있다.
- 변수들을 실제 값으로 바꾸는 프로세스와 최종 응답 문자열을 리턴하는 프로세스를 **렌더링** 이라고 한다
- 템플릿을 렌더링하는 작업을 위해 Flask는 **Jinja2** 라는 강려크한 템플릿 엔진을 사용한다

### Jinja2 Template Engine

- Jinja2 Tamplate는 response text를 include하는 file이다.
- 보통 플라스크에서는 application folder 안에 위치하는 tamplates 서브폴더에서 템플릿을 검색한다.
- 플라스크에서 Template을 rendering 하기 위해서는 render_template 함수를 사용한다.

```python
from flask import Flask, render_template
```

- 템플릿에서 사용되는 {{name}} 은 변수를 의마한다.
- Jinja2 는 어떤 타입의 변수라도 인식한다.
- 변수는 filter를 사용해서 수정할 수 있으며 다음과 같이 사용할 수 있다

    ```python
    Hello, {{name|capitalize}}
    ```

- Jinja2는 기본적으로 모든 변수를 보안 목적으로 **이스케이프**한다.
- Jinja2는 If문, for문, macro(like function)문 등을 템플릿에서 사용할 수 있게 해준다. 뿐만 아니라 import, include, 상속등 다양한 기능을 제공한다.

### Flask-Bootstrap과 Tweeter-Bootstrap의 통합

- Bootstrap은 트위터에서 제공하는 오픈소스 프레임워크다.
- Bootstrap은 모든 웹 브라우저와 호환되는 깔끔하고 매력적인 웹 페이지를 생성할 수 있도록하는 사용자 인터페이스를 제공한다.
- Bootstrap은 client 측의 프레임워크다.
- 서버에서는 부트스트랩의 CSS와 JavaScript 파일을 참조하는 HTML 응답을 제공해야 한다.
- 또한 서버는 HTML, CSS, JavaScript 코드를 통해서 원하는 컴포넌트들을 인스턴스화 한다.
- 이런 작업을은 템플릿에서 이루어지기에 적합하다.
- 부트스트랩과 APP을 통합하는 가장 확실한 방법은 필요한 모든 변경 사항들을 템플릿에 만들어 두는 것이다.
- flask-bootstrap 을 설치하자
- 플라스크의 확장은 보통 App instance가 생성될 때 초기화된다.

⚠️ `import flask.ext` 는 더이상 지원하지 않는다

- Flask-Bootstrap이 한번 초기화되면 모든 부트스트랩 파일을 포함하는 베이스 템플릿을 애플리케이션에서 사용할 수 있다.
- Jinja2 extends 는 템플릿 상속을 구현한다.
- Flask-Bootstrap의 베이스 템플릿은 모든 부트스트램 CSS, JavaScript를 포함하는 스켈레톤 웹 페이지를 제공한다.
- 베이스 템플릿은 파생된 템플릿이 오버라이드할 수 있도록 블록을 정의한다.
- 블록은 `{% block %}` and `{% endblock %}` 으로 정의한다.

### 링크

- 내비게이션 바와 같이 서로 다른 페이지들을 연결하는 것을 링크라고 부른다
- 동적 라우트를 위해 템플릿에서 URL을 구성하는 것은 의존성이 발생하거나 라우트 재구성으로 링크가 깨질 수 있다.
- Flask는 `url_for()` 헬퍼 함수를 제공한다.
- 이 함수는 URL 맵에 저장된 정보를 이용한다.
- 이 함수는 하나의 뷰 함수 이름 (혹은 app.add_url_route( )와 함께 정의된 라우트를 위한 endpoint 이름) 갖고 URL을 리턴한다.
- 사용법은 `href = {{ url_for('user', name= 'David') }}`

### 정적 파일

- Flask는 기본적으로 application의 root 폴더에 있는 static이라고 하는 서브디렉토리에서 정적 파일을 찾는다.

### Flask-Moment를 이용한 날짜와 시간 지역화

- 웹 어플리케이션에서 날짜와 시간을 처리하는 것은 사용자가 전 세계에서 다른 시간대를 사용하기 때문에 간단한 문제가 아니다.
- 서버는 UTC를 사용하고 웹 브라우저는 전송받은 시간 유닛을 자신의 로컬 시간으로 변경하고 렌더링한다.

# 4. Web Form

- Flask-WTF 확장은 웹 폼을 사용하여 훨씬 멋진 경험을 하도록 도와준다

### 크로스-사이트 리퀘스트 위조(CSRF) 보호

- Flask-WTF는 크로스 사이트 리퀘스트 위조 (CSRF) 공격으로부터 모든 폼을 보호한다.
- CSRF 공격은 악의적 웹사이트에서 희생자가 로그인한 다른 웹사이트로 리퀘스트를 전송할 때 일어난다.
- CSRF 보호를 위해 Flask-WTF는 암호화 키를 설정하기 위한 app이 필요하다.
- Flask-WTF는 이 키를 사용하여 암호화된 토큰을 생성하고 이 토큰은 폼 데이터와 함께 리퀘스트를 인증한다.
- 암호 설정 방법은 다음과 같다

    ```python
    app = Flask(__name__)
    app.config['SECRET_KEY'] = 'secret key'
    ```

- `app.config` 는 설정 면수를 저장하기 위해 일반적으로 사용하는 공간이다.
- 보안성 향상을 위해 보안 키는 환경변수에 저장한다.

### Form class

- Flask-WTF은  Form 클래스로부터 상속한 클래스에 의해 표현된다.
- 각 필드의 오브젝트는 하나 이상의 검증자가 붙어 있다. 검증자는 사용자의 입력값이 올바른지 확인한다.
- WTFrom에서는 HTML 필드를 와 검증자를 지원한다.
- HTML 필드는 텍스트 필드, 패스워드 필드, True or False값 필드 등등이 있다.
- 검증자는 이메일 주소검증, IPv4 검증, 입력값 문자열 길이 검증 등등이 있다.

### Form의 HTML Rendering

- 폼 필드는 호출이 가능하다.
- 폼 필드가 호출되면 템플릿은 HTML로 랜더링 된다.
- Flask-Bootstrap은 전체 Flask-WTF 폼을 랜더링하기 위해 부트스트랩의 미리 정의된 폼 스타일을 사용할 수 있게 헬퍼 함수를 제공하는데 한 번의 호출로 이루어지고 이전 폼이 랜더링 된다.

    ```python
    {% import "bootstrap/wtf.html" as wtf %}
    {{ wtf.quick_form(form) }}
    ```

    ### View Function에서의 Form 처리

    - `app.route` 데코레이터에 추가된 `methods` 인수는 플라스크에서 URL 맵의 GET과 POST 리퀘스트를 위한 핸들러로 뷰 함수를 등록하도록 한다.
    - 서브미션은 되도록이면 POST 리퀘스트로 처리한다. GET에서도 할 수 있지만 모든 데이터가 URL에 포함되어 날아가서 보안상의 문제가 발생할 수 있다.

    ### 리다이렉트와 사용자세션

    ![no-rediction](https://github.com/DavidKimDY/Book-Flask-web-develop/blob/main/static/no_redirection.png)

    - POST 방식으로 요청하고서 새로고침을 눌렀을때 종종 일어나는 반응이다. 즉, 새로고침은 폼 서브미션을 두 번 하게되는 문제를 일으킨다.
    - 따라서 웹 app이 브라우저에 의해 전송된 마지막 요청으로 POST 요청을 남기지 않는 것이 좋은 습관이다.
    - 이 습관은 정상적인 응답 대신에 리다이렉트와 함께 POST 요청에 대해 응답하는 것이다. 이를 **Post/Redirect/Get pattern** 이다.
    - 합리적이고 정당한 일들에 인내하고 불합리하고 불의한 일에 참지 않는 것이 바르게 인생을 사는 방법이다.
    - 위 방법에는 문제점이 있다. 리다이렉션하는 순간 Form data는 사라진다. 따라서 app은 사용자의 Form data를 저장해두어야 한다.
    - app은 사용자 세션에 데이터를 저장하여 사용자의 데이터를 기억해낸다.
    - 기본적으로 사용자 세션은 클라이언트 측 쿠키에 저장된다. 이는 설정되어 있는 SECRET_KEY로 암호화되어 있다. 쿠키의 콘텐츠가 조작되면 서명이 무효화되고 결국 세션 자체가 무효화된다.
    - 세션을 사용하면 로컬 변수였던 name은 session['name'] 으로 저장되었고 이후 리퀘스트에서도 사용가능하다.
	     
# 5. DataBase

### SQL DataBase

- Structured Query Language
- 관계형 모델이라고 한다.
- 테이블에 데이터를 저장한다.
- Column의 수는 테이블마다 고정되어 있다.
- Row 수는 늘어난다.
- 기본키(primary key)와 외래키(foreign key)를 설정할 수 있다.
- 기본키는 해당 행의 식별자이고 외래키는 이를 참조한다.

### NoSQL DataBase

- 테이블 대신 컬랙션(collection)
- 레코드 대신 도큐먼트(document) 를 사용한다.
- 역정규화(denormalization)라고 하는 오퍼레이션을 사용하며 데이터 중복을 방지한다.

### ORM

- 객체와 관계형 데이터베이스의 데이터를 자동으로 매핑(연결)해주는 것을 말한다.
- 객체 지향 프로그래밍은 클래스를 사용하고, 관계형 데이터베이스는 테이블을 사용한다.
- 객체 모델과 관계형 모델 간에 불일치가 존재한다.
- ORM을 통해 객체 간의 관계를 바탕으로 SQL을 자동으로 생성하여 불일치를 해결한다.
	     
# E-mail

## Flask-Mail

- Flask-Mail은 SMTP (Simple Mail Tranfer Protocol) 서버와 연결하여 이메일을 서버에 전달함으로 이메일을 발송한다.
- 다른 설정이 없다면 Flask-Mail은 포트 25번을 통해 **[localhost](http://localhost)** 와 연결한다.
- 이때는 인증없이 이메일을 발송한다.
- 외부 SMTP 서버를 연결하는 것이 더 편할 수 있다. -> google의 smtp를 이용한다
- 외부 SMTP 서버를 연결하는 것이 더 편할 수 있다.
- 다음은 구글의 Gmail 계정을 통해 이메일을 전송하는 방법이다.

    ```python
    app.config['MAIL_SERVER'] = 'smtp.googlemail.com'
    app.config['MAIL_PORT'] = 587
    app.config['MAIL_USE_TLS'] = True
    app.config['MAIL_USERNAME'] = os.environ.get('MAIL_USERNAME')
    app.config['MAIL_PASSWORD'] = os.environ.get('MAIL_PASSWORD') 
    ```

    - 개인정보는 절대로 스크립트에 입력하지 말고 위와 같이 환경변수로 사용해야 한다.
    - 환경변수를 설정하는 방법은 다음과 같다

        ```bash
        export MAIL_USERNAME=구글아이디
        export MAIL_PASSWORD=비밀번호
        ```

        > ⚠️  만약 실습에 사용하는 구글아이디가 2-step 인증이라면 `smtplib.SMTPAuthenticationError: (535, b'5.7.8 Username and Password not accepted. Learn more at\n5.7.8 [https://support.google.com/mail/?p=BadCredentials](https://support.google.com/mail/?p=BadCredentials)`  이라는 Error가 뜰 것이다. 그때는 Google Account에 Less secure app access 에서 `Allow Less Security App`  을 `On` 으로 설정해주어야 한다.	    

### Application에서 Email 기능

- 이메일 전송 기능을 함수로 추상화 하는 것을 추천한다.
- 이 함수는 Jinja2 템플릿으로 이메일 본문을 렌더링한다.
    - code에서는 이메일 템플릿을 mail directory에 모아두었음을 알 수 있다.

### Code 설명

[Book-Flask-web-develop/email.py at main · DavidKimDY/Book-Flask-web-develop](https://github.com/DavidKimDY/Book-Flask-web-develop/blob/main/code/email.py)

### 비동기 이메일 전송

- 위의 코드를 실행해보면 mail을 보내는 동안 web browser가 멈추는 모습을 볼 수 있다.
- 리퀘스트 핸들링을 하는 동안 불필요한 지연을 피하기 위해, 이메일 전송 함수는 백그라운드 스레드로 처리한다.

```python
def send_async_email(app, msg):
    with app.app_context():
        mail.send(msg)

def send_email(to, subject, template, **kwargs):
    msg = Message(app.config['FLASKY_MAIL_SUBJECT_PREFIX'] + ' ' + subject,
                  sender=app.config['FLASKY_MAIL_SENDER'], recipients=[to])
    msg.body = render_template(template + '.txt', **kwargs)
    msg.html = render_template(template + '.html', **kwargs)
    thr = Thread(target=send_async_email, args=[app, msg])
    thr.start()
    return thr
```

- 많은 플라스크 확장은 활성화된 어플리케이션과 [리퀘스트 컨텍스트](https://www.notion.so/Flask-d499f6e36cb1414e97abac05f1092e29)가 있다는 가정하에 동작한다.
- Flask-Mail은 send() 함수에서 [current_app](https://www.notion.so/Flask-d499f6e36cb1414e97abac05f1092e29)을 사용하여 애플리케이션 컨텍스트가 활성화될 것을 요구한다. 하지만 send() 함수가 다른 스레드에서 실행되면 애플리케이션 컨텍스트가 인위적으로 app.app_context()를 생성해야 한다.
- 대용량 이메일 전송 작업과 같은 경우에는 이메일 전송을 하나의 작업으로 만드는 것이 이메일을 전송할 때마다 새로운 스레드를 시작하는 것보다 더 낫다.
- 예를 들면 send_async_email() 함수의 실행은 Celery 태스크 큐에 전송하는 것이다.

# 대규모 어플리케이션 구조

- 어플리케이션이 커질수록 하나의 스크립트 소스 파일에서 작업하는 것은 비효율적이다.
- 디른 프레임워크와 달리 플라스크는 특정 구조를 요구하지 않는다.
- 따라서 프로젝트 구조를 구성하는 것은 개발자의 몫이다.

## 프로젝트 구조

```python
flasky/
├── LICENSE
├── README.md
├── app
│   ├── __init__.py
│   ├── email.py
│   ├── main
│   │   ├── __init__.py
│   │   ├── errors.py
│   │   ├── forms.py
│   │   └── views.py
│   ├── models.py
│   ├── static
│   │   └── favicon.ico
│   └── templates
│       ├── 404.html
│       ├── 500.html
│       ├── base.html
│       ├── index.html
│       └── mail
│           ├── new_user.html
│           └── new_user.txt
├── config.py
├── flasky.py
├── migrations
│   ├── README
│   ├── alembic.ini
│   ├── env.py
│   ├── script.py.mako
│   └── versions
│       └── 38c4e85512a9_initial_migration.py
├── requirements.txt
└── tests
    ├── __init__.py
    └── test_basics.py
```

## 설정 옵션

- 어플리케이션은 종종 여러 설정값이 필요하다.
- 개발, 테스트, 제품화 과정 중에 서로 다른 데이터베이스를 사용해야 하는 경우가 대표적이다.

```python
import os
basedir = os.path.abspath(os.path.dirname(__file__))

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'hard to guess string'
    MAIL_SERVER = os.environ.get('MAIL_SERVER', 'smtp.googlemail.com')
    MAIL_PORT = int(os.environ.get('MAIL_PORT', '587'))
    MAIL_USE_TLS = os.environ.get('MAIL_USE_TLS', 'true').lower() in \
        ['true', 'on', '1']
    MAIL_USERNAME = os.environ.get('MAIL_USERNAME')
    MAIL_PASSWORD = os.environ.get('MAIL_PASSWORD')
    FLASKY_MAIL_SUBJECT_PREFIX = '[Flasky]'
    FLASKY_MAIL_SENDER = 'Flasky Admin <flasky@example.com>'
    FLASKY_ADMIN = os.environ.get('FLASKY_ADMIN')
    SQLALCHEMY_TRACK_MODIFICATIONS = False

    @staticmethod
    def init_app(app):
        pass

class DevelopmentConfig(Config):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('DEV_DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'data-dev.sqlite')

class TestingConfig(Config):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('TEST_DATABASE_URL') or \
        'sqlite://'

class ProductionConfig(Config):
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'data.sqlite')

config = {
    'development': DevelopmentConfig,
    'testing': TestingConfig,
    'production': ProductionConfig,

    'default': DevelopmentConfig
}
```

- Config class는  모든 설정에 사용되는 공통값들을 포함하고 있다.
- 좀 더 호환성있고 안정한 설정을 하려면 환경 변수에서 옵션으로 임포트해야 한다.
    - 예를 들면 `SECRET_KEY = os.environ.get('SECRET_KEY') or 'hard to guess string'`
- `SQLALCHEMY_DATABASE_URI` 변수는 각 설정에 따라 다른 값으로 설정되어 있다. 서로 다른 DB 환경에서 동작할 수 있게 해준다.
- `init_app()` 클래스 메소드는 설정에 따른 초기화 작업을 수행해준다.

## 어플리케이션 패키지

```bash
app
├── __init__.py
├── email.py
├── main
│   ├── __init__.py
│   ├── errors.py
│   ├── forms.py
│   └── views.py
├── models.py
├── static
│   └── favicon.ico
└── templates
    ├── 404.html
    ├── 500.html
    ├── base.html
    ├── index.html
    └── mail
        ├── new_user.html
        └── new_user.txt
```

- app 패키지에는 어플리케이션의 코드, 템플릿, 정적파일, 데이터베이스 모델, 이메일 작업 코드 등이 존재한다.

### 블루프린트

[블루프린트를 가진 모듈화된 어플리케이션 - Flask 0.11-dev documentation](https://flask-docs-kr.readthedocs.io/ko/latest/blueprints.html)

- 대규모 어플리케이션 구조를 지원하기 위한 Flask의 기능
- 블루프린트는 대형 어플리케이션이 동작하는 방식을 단순화한다
- 블루프린트의 기본 개념은 어플리케이션에 블루프린트이 등록될 때 실행할 동작을 기록한다는 것이다.
- 플라스크는 요청을 보내고 하나의 끝점에서 다른 곳으로 URL을 생성할 때 뷰 함수와 블루프린트의 연관을 맺는다.

### 어플리케이션 팩토리

- 일반적인 패턴은 청사진을 임포트할 때 어플리케이션 객체를 생성하는 것이다.
- 하지만 이 객체의 생성을 함수로 옮긴다면, 나중에 이 객체에 대한 복수 개의 인스턴스를 생성할 수 있다.
	     
# 사용자 인증

## 패스워드 보안

- 패스워드를 저장할 때는 해싱(hashing)을 사용한다
- 즉, 패스워드를 있는 그래도 DB에 저장하는 것이아닌 해싱을 통해 변환된 문자열을 저장하는 방법이다.
- 예를 들어보자.

    Password : "applebanana" → [Hash function] → DB : "AE21-00FF-E112"

    [로그인 과정]

    1. 사용자가 "applebanana"로 password를 입력
    2. hash function을 통해서 변환된 값을 획득
    3. 변환된 값을 가지고 DB에 저장된 값과 비교

    [보안성]

    - "applebanana" → hash →  "AE21-00FF-E112"  (Possible)
    - "AE21-00FF-E112" → hash → "applebanana"   (Impossible)
    - 즉, 변환된 값을 가지고 변환되기 전 값을 찾는 것은 불가능하다.

### Werzeung [월-적]

- Werkzeug [월적] 은 WSGI web application library이다.
- 월적의 보안 모듈을 사용하면 보안 패스워드 해싱을 편하게 구현할 수 있다.

    `generate_password_hash(password, method=pbkdf2:sha1, salt_lenth=8)`

    → 이 함수는 plain-text인 password를 받아서 password hash를 return한다. 

    → method, length는 그대로 두고 사용해도 된다.

    `check_password_hash(hash, password)`

    → DB에서 추출한 password hash 와 사용자가 입력한 password를 비교한다. 

    → True가 반환되면 패스워드가 일치한다는 뜻이다.

### 인증 블루프린트 생성

- 사용자 인증과 관련된 라우트는 auth 블루프린트에 추가할 수 있다.
- 애플리케이션 기능의 다른 집합을 위해 다른 블루프린트를 사용하는 것은 코드를 구조적으로 보기 좋게 관리하는 방법이다.
1. 같은 이름의 패키지에서 호스트한다.

    `app/auth/__init__.py` 에서 auth 블루프린트를 생성한다

    ```python
    from flask import Blueprint

    auth = Blueprint('auth', __name__)

    from . import views
    ```

2. [views.py](http://views.py) 에서 auth를 호출한다.

    `app/auth/views.py`

    ```python
    from flask import render_template
    from . import auth

    @auth.route('/login')
    def login():
    	return render_template('auth/login.html')
    ```

    언뜻보면 순환 구조처럼 보이지만 `__init__.py` 는 `auth` 패키지가 임포트 될때 한번 실행된다.

    그리고 `[views.py](http://views.py)` 가 실행되면서 `__init__.py` 에서 선언한 auth blueprint를 불러와서 사용한다.

    `render_template('auth/login.html')` 에서 괄호 안의 경로는 `app/templates/auth/login.html` 이다. `render_tamplate()` 함수는 먼저 어플리케이션용으로 설정된 템플릿 풀더를 우선 검색한다.

3. 블루프린트를 부착한다.

    `app/__inti__.py`

    ```python
    def create_app(config_name):
    	# ...
    	from .auth import auth as auth_blueprint
    	app.register_blueprint(auth_blueprint, url_prefix='/auth')

    	return app
    ```

- `url_prefix` 인수는 옵션이다.
- `url_prefix` 인수 옵션을 사용하면 해당 블루프린트에 정의된 모든 라우트는 주어진 접두어를 사용하여 등록된다
- 따라서 `/login` 라우트는 `/auth/login` 로 등록되었고 [http://localhost:500/auth/login](http://localhost:500/auth/login) 에서 접속이 가능해 진다.
