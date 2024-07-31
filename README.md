# CRUD
### 1. 프로젝트 생성
- 폴더 생성 후 들어가서 code로 열기
```bash
django-admin startproject board .
```

### 2. 가상환경 생성
```bash
python -m venv venv
```

### 3. 가상환경 활성화
```bash
source venv/Scripts/activate
```

### 4. 가상환경 내부에 django 설치
```bash
pip install django
```

### 5. 앱생성
```bash
django-admin startapp articles
```

### 6. git init 관리
- git init 후 구글에 gitignore.io 검색
- python, VisualStudioCode, windows, macOS, Django 생성
- code로 돌아와 .gitignore 파일 만들고 생성된 코드 붙여넣기

### 7. 서버 실행(서버 종료는 ctrl+c)
- 실행후 나오는 주소 (예를들어 http://127.0.0.1:8000/ 에 ctrl+클릭하면 이동됨.)
```bash
python manage.py runserver
```

### 8. templates 폴더 생성.
- 4번에서 생성한 `articles` 폴더에 마우스 우클릭 하고 `templates` 폴더 생성.
- html 파일 관리 장소


### 9. app 등록
- `board` > `settings.py` INSTALLED_APPS 에 'articles', 등록
```python
INSTALLED_APPS = [
    ...
    'articles',
]
```

### 10. 공통 base.html구조 작성
- articles 안에 urls.py 파일 만들기
    - 이정표를 2개로 만들어서 상위, 하위 urls로 관리할거임
    - 장점: 우리가 사용하는 앱이 1개밖에없는데(articles) 이게 여러개로 쌓이면 유지보수하기 힘듬. 그래서 같은 모델을 관리하는 urls 들을 1개로 묶어두는거임.
    - 어제 수업 생각하면 상위 urls에 path를 7개 썻는데 오늘은 그걸 분류해줄거임
    - 지금까지 상위 urls와 views.py의 결로가 달라서 urls.py 에 `from articles import views.py` 해줬는데 지금은 하위 urls.py를 views.py와 같은 경로에 만들어 줬기때문에 코드가 달라짐(`from . import views`)

- `article` > `templates` 에 index.html 파일 만들기
```html
<!--articles/templates/index.html-->
{% extends 'base.html' %}

{% block body %}
    <h1>여기는 index입니다.</h1>
{% endblock %}
```
- `article`에 `urls.py` 만들기
```python
# articles/urls.py
from . import views

urlpatterns = [
    path('', views.index),
]
```
- `article` > `views.py` 함수 지정
```python
def index(request):
    return render(request, 'index.html')
```
- 상위경로의 templates 만들기
- BOARD 위치에 templates 폴더 만들면 됨
- 상위경로 `templates` 에 `base.html` 파일 만들기
```html
<body>
    <h1>여기는 base입니다.</h1>

    {% block body %}
    {% endblock %}
</body>
```

- settings.py 에 TEMPLATES 문 확인
- 근데 장고는 최상단 templates 찾지못하기에 설정해줘야함
- BASE_DIR/ 'templates': BASE_DIR은 우리가 처음에 만든 최상위 폴더임. 즉, 최상위 폴더의 `'templates'` 도 찾을 수 있게 만드는 코드
```python
# TEMPLATES 코드 수정
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'], # DIRS 대괄호 안에 BASE_DIR/ / 'templates' 추가
        'APP_DIRS': True,
        'OPTIONS': {
            ...
        ...
    ...
]
```

- 홈 화면 경로 만들기
- urls.py 는 어떠한 기능으로써 동작하게 끔 만드는 곳.
- `articles` > `urls.py` 에서 동작하는 것도 출력하게끔 경로 지정.
```python
# board > urls.py
from django.urls import path, include # include 추가

urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', include('articles.urls')) # 추가
]
```

### 11. 모델링
- 오늘 `articles` > `models.py` 에 만들 모델은 `Article()`
- 장고 코드 안에 `class Model` 가 이미 구현되어 있음.
- `articles/models.py` class 추가
```python
class Article(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True) # 작성 시간
```

### 12. admin에 Article 모델 추가
- /admin 관리자 페이지에 우리가 관리할 모델인 article을 등록하는 과정
- 안하면 article 모델이 등록이안됨.
```python
# articles/admin.py
from .models import Article # 추가

admin.site.register(Article) # 추가
```
- SQL 환경과 python 환경 연결(서버실행 종료 후)
- `python manage.py makemigrations`
- `python manage.py migrate`

- 관리자 아이디 비번 만들기(서버실행 종료 후)
- `python manage.py createsuperuser`
- 아이디: admin, 비번:
- /admin


### 13. Read(All)기능구현 / bootstrap적용
- `articles/templates/index.html` 코드 수정
```html
{% extends 'base.html' %}

{% block body %}
    <!--    <h1>여기는 index입니다.</h1> 변경  -->
    <table class="table">
        <thead>
            <tr>
                <th>title</th>
                <th>created_at</th>
                <th>link</th>
            </tr>
        </thead>
        <tbody>
            {% for article in articles %}
                <tr>
                    <td>{{article.title}}</td>
                    <td>{{article.created_at}}</td>
                    <td><a href="">link</a></td>
                </tr>
            {% endfor %}
        </tbody>
    </table>
{% endblock %}
```
- `articles` > `views.py` 에 index 함수 수정
- models.py 의 코드를 불러오는 코드
```python
from .models import Article

def index(request):
    # return render(request, 'index.html') 변경
    articles = Article.objects.all()

    context = {
        'articles': articles,
    }

    return render(request, 'index.html', context)
```
- `templates/base.html` 코드 수정
- `base.html` 에 </head> 끝나기 전에
- <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous"> 추가
- </body> 끝나기 전에
- <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-YvpcrYf0tY3lHB60NNkmXc5s9fDVZLESaAA55NDzOxhy9GkcIdslK1eN7N6jIeHz" crossorigin="anonymous"></script> 추가

- <h1> 뺴고 그 자리에 코드 추가
```html
<nav class="nav">
        <a class="nav-link" href="/articles/">Home</a>
        <a class="nav-link" href="#">Link</a>
        <a class="nav-link" href="#">Link</a>
        <a class="nav-link disabled" aria-disabled="true">Disabled</a>
    </nav>

 <!--{% block body %}
    {% endblock %}   변경 -->
    <div class="container">
        {% block body %}
        {% endblock %}
    </div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-YvpcrYf0tY3lHB60NNkmXc5s9fDVZLESaAA55NDzOxhy9GkcIdslK1eN7N6jIeHz" crossorigin="anonymous"></script>
```

### 14. settings.py 수정
- 한국시간으로 조정하기
- `board` > `settings.py` 에 107번 줄부터 수정
```python
# LANGUAGE_CODE = 'en-us'
LANGUAGE_CODE = 'ko-kr'

# TIME_ZONE = 'UTC'
TIME_ZONE = 'Asia/seoul'
```

### 15. Read(1)기능구현
- `articles/templates` 에 `detail.html` 파일 생성
```html
{% extends 'base.html' %}

{% block body %}
<div class="card">
    <div class="card-header">
        {{article.title}}
    </div>
    <div class="card-body">
        {{article.content}}
    </div>
    <div class="card-footer">
        {{article.created_at}}
    </div>
</div>
{% endblock %}
```
- `articles/templates/index.html` 코드 수정
```html
<tr>
                    <td>{{article.title}}</td>
                    <td>{{article.created_at}}</td>
                    <!--    <td><a href="">link</a></td> 변경   -->
                    <td><a href="{% url 'articles:detail' id=article.id %}">link</a></td>
                </tr>
            {% endfor %}
```
- `articles/urls.py`코드 수정
```python
app_name = 'articles' # 추가

urlpatterns = [
    # path('', views.index), 변경
    path('', views.index, name='index'), # 추가
    path('<int:id>/', views.detail, name='detail'), # 추가
```
- `articles/views.py` 함수 추가
```python
def detail(request, id):
    article = Article.objects.get(id=id)

    context = {
        'article': article,
    }

    return render(request, 'detail.html', context)
```
- `templates/base.html` 코드 수정
```html
</head>
<body>
    <nav class="nav">
        <!--       <a class="nav-link" href="/articles/">Home</a> 변경     -->
        <a class="nav-link" href="{% url 'articles:index' %}">Home</a> <!-- 변경 -->
        <a class="nav-link" href="#">Link</a>
        <a class="nav-link" href="#">Link</a>
        <a class="nav-link disabled" aria-disabled="true">Disabled</a>
```

### 16. Create 기능구현
- `articles/templates`에 `new.html` 생성
```html
{% extends 'base.html' %}

{% block body %}
<form action="{% url 'articles:create' %}" method="POST">
    {% csrf_token %}
    <div class="mb-3">
        <label for="title" class="form-label">Title</label>
        <input type="text" id="title" class="form-control" name="title">
    </div>
    <div class="mb-3">
        <label for="content" class="form-label">Content</label>
        <textarea id="content" class="form-control" rows="10" name="content"></textarea>
    </div>

    <button type="submit" class="btn btn-primary">submit</button>
</form>
{% endblock %}
```
- `articles/urls.py` 에 새로만들 `create` 기능과 이를 위한 새 페이지 기능인 `new` 경로 만들기
```python
urlpatterns = [
    ...
    path('new/', views.new, name='new'), # 추가
    path('create/', views.create, name='create'), # 추가

]
```
- `articles/views.py` 에 `redirect` import 하고 `new` 와 `create` 함수 만들기
```python
from django.shortcuts import render, redirect # redirect 추가

def new(request):
    return render(request, 'new.html')


def create(request):
    title = request.POST.get('title')
    content = request.POST.get('content')

    article = Article()
    article.title = title
    article.content = content
    article.save()

    return redirect('articles:detail', id=article.id)
```
- `templates/base.html` 수정하기
```html
<body>
    <nav class="nav">
        <a class="nav-link" href="{% url 'articles:index' %}">Home</a>
        <!-- <a class="nav-link" href="#">Link</a>  아래 코드로 수정 -->
        <a class="nav-link" href="{% url 'articles:new' %}">Create</a>
        <a class="nav-link" href="#">Link</a>
        <a class="nav-link disabled" aria-disabled="true">Disabled</a>
    </nav>
```

### 17. delete 기능구현
- `articles/templates/detail.html` 에 delete 버튼 추가를 위한 수정
```html
{{article.created_at}}
    </div>
</div>
<a href="{% url 'articles:delete' id=article.id %}" class="btn btn-danger">delete</a>
{% endblock %}
```
- `articles/urls.py` 경로 지정
```python
urlpatterns = [
    ...
    path('<int:id>/delete/', views.delete, name='delete'), # 추가

]
```
- `articles/views.py` 함수 생성
```python
def delete(request, id):
    article = Article.objects.get(id=id)
    article.delete()

    return redirect('articles:index')
```


# 18. update 기능구현




# Delete 기능 만들기
- detaril.html 에서 만들기

# 수정
- detaril.html 수정버튼 만들기


### POST
- 