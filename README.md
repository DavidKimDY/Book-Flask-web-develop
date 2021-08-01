# Book-Flask-web-develop
# Flask

# Feature

- 마이크로 프레임워크다.
- WSGI 서브시스템음 Werkzeung에 템플릿은 jinja2에 의존한다.

# Application Structure

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

# Template

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

# Web Form

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
