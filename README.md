
# Django로 로그인 기능 구현하기

Django로 웹사이트를 개발할 때 로그인 기능을 구현하는 방법은 다음과 같습니다.

## 1. 프로젝트 및 앱 설정
먼저 Django 프로젝트와 앱을 생성합니다.

```bash
django-admin startproject myproject
cd myproject
python manage.py startapp accounts
```

`accounts`는 사용자 인증을 담당할 앱입니다.

## 2. 앱 등록
생성한 앱을 `settings.py` 파일의 `INSTALLED_APPS`에 추가합니다.

```python
INSTALLED_APPS = [
    ...
    'accounts',
]
```

## 3. URL 설정
프로젝트의 `urls.py` 파일에 앱의 URL을 포함시킵니다.

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/', include('accounts.urls')),
]
```

## 4. 템플릿 디렉토리 설정
`settings.py` 파일에서 템플릿 디렉토리를 지정합니다.

```python
TEMPLATES = [
    {
        ...
        'DIRS': [BASE_DIR / 'templates'],
        ...
    },
]
```

## 5. 로그인 뷰 작성
Django의 내장 인증 뷰를 사용할 수 있지만, 커스텀 뷰를 만들 수도 있습니다.

`accounts/views.py`

```python
from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login
from django.contrib import messages

def login_view(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
        user = authenticate(request, username=username, password=password)
        if user is not None:
            login(request, user)
            return redirect('home')  # 로그인 후 이동할 페이지
        else:
            messages.error(request, '아이디 또는 비밀번호가 올바르지 않습니다.')
    return render(request, 'accounts/login.html')
```

## 6. URL 매핑
`accounts/urls.py` 파일을 생성하고 URL 패턴을 정의합니다.

```python
from django.urls import path
from . import views

urlpatterns = [
    path('login/', views.login_view, name='login'),
]
```

## 7. 로그인 템플릿 작성
`templates/accounts/login.html`

```html
<!DOCTYPE html>
<html>
<head>
    <title>로그인</title>
</head>
<body>
    <h2>로그인</h2>
    {% if messages %}
        {% for message in messages %}
            <p style="color: red;">{{ message }}</p>
        {% endfor %}
    {% endif %}
    <form method="post">
        {% csrf_token %}
        <label for="username">아이디:</label>
        <input type="text" name="username" id="username" required><br>
        <label for="password">비밀번호:</label>
        <input type="password" name="password" id="password" required><br>
        <button type="submit">로그인</button>
    </form>
</body>
</html>
```

## 8. 로그인 필요 페이지 설정
로그인이 필요한 뷰에 `login_required` 데코레이터를 사용합니다.

```python
from django.contrib.auth.decorators import login_required

@login_required
def dashboard(request):
    return render(request, 'accounts/dashboard.html')
```

## 9. 로그아웃 기능 추가
`accounts/views.py`

```python
from django.contrib.auth import logout

def logout_view(request):
    logout(request)
    return redirect('home')
```

`accounts/urls.py`

```python
urlpatterns = [
    ...
    path('logout/', views.logout_view, name='logout'),
]
```

## 10. 회원가입 기능 (선택 사항)
회원가입을 구현하려면 Django의 `UserCreationForm`을 활용할 수 있습니다.

`accounts/views.py`

```python
from django.contrib.auth.forms import UserCreationForm

def register(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('login')  # 회원가입 후 이동할 페이지
    else:
        form = UserCreationForm()
    return render(request, 'accounts/register.html', {'form': form})
```

`accounts/urls.py`

```python
urlpatterns = [
    ...
    path('register/', views.register, name='register'),
]
```

`templates/accounts/register.html`

```html
<!DOCTYPE html>
<html>
<head>
    <title>회원가입</title>
</head>
<body>
    <h2>회원가입</h2>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">가입하기</button>
    </form>
</body>
</html>
```

## 추가 팁
- **내장 인증 시스템 활용**: Django는 강력한 인증 시스템을 제공합니다. 가능하면 내장 뷰와 폼을 활용하여 개발 시간을 절약하세요.
- **리다이렉트 설정**: 로그인하지 않은 사용자가 접근할 수 없는 페이지에 접근할 경우 로그인 페이지로 리다이렉트됩니다.
- **메시지 프레임워크 사용**: 사용자에게 유용한 메시지를 전달하기 위해 Django의 메시지 프레임워크를 사용하세요.

## 참고 문서
- [Django 공식 문서 - 인증](https://docs.djangoproject.com/en/stable/topics/auth/)
- [Django 공식 문서 - 내장 인증 뷰](https://docs.djangoproject.com/en/stable/topics/auth/default/)
