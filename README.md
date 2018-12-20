# create-signup-page-using-django

<h1 align="center">
  <br>
  <img alt="Signup Page Using Django" title="Signup" src="https://github.com/saktikanta/create-signup-page-using-django/blob/master/signup.GIF" width="700"></a>
  <br>
  Signup Page Using Django
  <br>
</h1>

### Install Django on winsows
```cmd
pip install django
```

### Create project using django
```cmd
django-admin startproject myproject
```

### Create a supper user/admin user for the webserver before starting the server
```cmd
python manage.py createsuperuser
```
#### It will prompt you for user id followed by password and confirmation password.
### Start the Django server
```cmd
cd myproject

python manage.py runserver
```
#### The above will start the webserver and the default port will be 8000. To start the webserver on different port use below command
```cmd
python manage.py runserver 127.0.0.1:5081
```
or
```cmd
python manage.py runserver 5081
```

### For help use the below command
```cmd
python manage.py help
```

### Tree view of files and folders
#### To create the signup page you need to have below file structure. You can downloa the project from [here](https://github.com/saktikanta/create-signup-page-using-django/tree/master/myproject2).
```powershell
│   db.sqlite3
│   manage.py
│
└───myproject2
    │   settings.py
    │   urls.py
    │   wsgi.py
    │   __init__.py
    │
    ├───core
    │       forms.py
    │       tokens.py
    │       views.py
    │
    └───templates
            base.html
            home.html
            login.html
            logout.html
            signup.html
```
### In forms.py file
```python
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User


class SignUpForm(UserCreationForm):
    email = forms.EmailField(max_length=254, help_text='Required. Inform a valid email address.')

    class Meta:
        model = User
        fields = ('username', 'email', 'password1', 'password2', )
```
### In tokens.py file
```python
from django.contrib.auth.tokens import PasswordResetTokenGenerator
from django.utils import six

class AccountActivationTokenGenerator(PasswordResetTokenGenerator):
    def _make_hash_value(self, user, timestamp):
        return (
            six.text_type(user.pk) + six.text_type(timestamp) +
            six.text_type(user.profile.email_confirmed)
        )

account_activation_token = AccountActivationTokenGenerator()
```

### In views.py file
```python
from django.contrib.auth import login
from django.contrib.auth.decorators import login_required
from django.contrib.auth.models import User
from django.contrib.sites.shortcuts import get_current_site
from django.shortcuts import render, redirect
from django.utils.encoding import force_bytes, force_text
from django.utils.http import urlsafe_base64_encode, urlsafe_base64_decode
from django.template.loader import render_to_string

from myproject2.core.forms import SignUpForm
from myproject2.core.tokens import account_activation_token


@login_required
def home(request):
    return render(request, 'home.html')


def signup(request):
    if request.method == 'POST':
        form = SignUpForm(request.POST)
        if form.is_valid():
            user = form.save(commit=False)
            user.is_active = False
            user.save()

            current_site = get_current_site(request)
            subject = 'Activate Your MySite Account'
            message = render_to_string('account_activation_email.html', {
                'user': user,
                'domain': current_site.domain,
                'uid': urlsafe_base64_encode(force_bytes(user.pk)),
                'token': account_activation_token.make_token(user),
            })
            user.email_user(subject, message)

            return redirect('account_activation_sent')
    else:
        form = SignUpForm()
    return render(request, 'signup.html', {'form': form})


def account_activation_sent(request):
    return render(request, 'account_activation_sent.html')


def activate(request, uidb64, token):
    try:
        uid = force_text(urlsafe_base64_decode(uidb64))
        user = User.objects.get(pk=uid)
    except (TypeError, ValueError, OverflowError, User.DoesNotExist):
        user = None

    if user is not None and account_activation_token.check_token(user, token):
        user.is_active = True
        user.profile.email_confirmed = True
        user.save()
        login(request, user)
        return redirect('home')
    else:
        return render(request, 'account_activation_invalid.html')
```

### in base.html file
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>{% block title %}Simple is Better Than Complex{% endblock %}</title>
  </head>
  <body>
    <header>
      <h1>My Site</h1>
      {% if user.is_authenticated %}
        <a href="{% url 'logout' %}">logout</a>
      {% else %}
        <a href="{% url 'login' %}">login</a> / <a href="{% url 'signup' %}">signup</a>
      {% endif %}
      <hr>
    </header>
    <main>
      {% block content %}
      {% endblock %}
    </main>
  </body>
</html>
```

### in home.html file
```html
{% extends 'base.html' %}

{% block content %}
  <h2>Welcome, {{ user.username }}!</h2>
{% endblock %}
```

### in login.html file
```html
{% extends 'base.html' %}

{% block content %}
  <h2>Log in to My Site</h2>
  {% if form.errors %}
    <p style="color: red">Your username and password didn't match. Please try again.</p>
  {% endif %}
  <form method="post">
    {% csrf_token %}
    <input type="hidden" name="next" value="{{ next }}" />
    {% for field in form %}
      <p>
        {{ field.label_tag }}<br>
        {{ field }}<br>
        {% for error in field.errors %}
          <p style="color: red">{{ error }}</p>
        {% endfor %}
        {% if field.help_text %}
          <p><small style="color: grey">{{ field.help_text }}</small></p>
        {% endif %}
      </p>
    {% endfor %}
    <button type="submit">Log in</button>
    <a href="{% url 'signup' %}">New to My Site? Sign up</a>
  </form>
{% endblock %}
```

### in login.html file
```html
{% extends 'base.html' %}

{% block content %}
  <h2>Log in to My Site</h2>
  {% if form.errors %}
    <p style="color: red">Your username and password didn't match. Please try again.</p>
  {% endif %}
  <form method="post">
    {% csrf_token %}
    <input type="hidden" name="next" value="{{ next }}" />
    {% for field in form %}
      <p>
        {{ field.label_tag }}<br>
        {{ field }}<br>
        {% for error in field.errors %}
          <p style="color: red">{{ error }}</p>
        {% endfor %}
        {% if field.help_text %}
          <p><small style="color: grey">{{ field.help_text }}</small></p>
        {% endif %}
      </p>
    {% endfor %}
    <button type="submit">Log in</button>
    <a href="{% url 'signup' %}">New to My Site? Sign up</a>
  </form>
{% endblock %}
```

### in logout.html file
```html
{% extends 'base.html' %}

{% block content %}
  <h2>Log in to My Site</h2>
  {% if form.errors %}
    <p style="color: red">Your username and password didn't match. Please try again.</p>
  {% endif %}
  <form method="post">
    {% csrf_token %}
    <input type="hidden" name="next" value="{{ next }}" />
    {% for field in form %}
      <p>
        {{ field.label_tag }}<br>
        {{ field }}<br>
        {% for error in field.errors %}
          <p style="color: red">{{ error }}</p>
        {% endfor %}
        {% if field.help_text %}
          <p><small style="color: grey">{{ field.help_text }}</small></p>
        {% endif %}
      </p>
    {% endfor %}
    <button type="submit">Log in</button>
    <a href="{% url 'signup' %}">New to My Site? Sign up</a>
  </form>
{% endblock %}
```

### sin signup.html page
```html
{% extends 'base.html' %}

{% block content %}
  <h2>Sign up</h2>
  <form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Sign up</button>
  </form>
{% endblock %}
```
### in urls.py file
```python
from django.contrib import admin
from django.urls import path
from django.conf.urls import url
from myproject2.core import views as core_views
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('admin/', admin.site.urls),
    url(r'^$', core_views.home, name='home'),
    url(r'^login/$', auth_views.LoginView.as_view(template_name='login.html'), name='login'),
    url(r'^logout/$', auth_views.LogoutView.as_view(next_page='login'), name='logout'),
    url(r'^signup/$', core_views.signup, name='signup'),
    url(r'^account_activation_sent/$', core_views.account_activation_sent, name='account_activation_sent'),
    url(r'^activate/(?P<uidb64>[0-9A-Za-z_\-]+)/(?P<token>[0-9A-Za-z]{1,13}-[0-9A-Za-z]{1,20})/$', core_views.activate, name='activate'),
]
```

### wsgi.py file
```python
import os

from django.core.wsgi import get_wsgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject2.settings')

application = get_wsgi_application()
```

### in settings.py file
```python
import os

# Build paths inside the project like this: os.path.join(BASE_DIR, ...)
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))


# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/2.1/howto/deployment/checklist/

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = '_0)!d66w*9^=tmrwsuod(g%jz+h5nl()x02w4v84jbw94)5j%r'

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

ALLOWED_HOSTS = []


# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'myproject2.core',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'myproject2.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        # 'DIRS': [],
        'DIRS': [os.path.join(BASE_DIR, 'myproject2/templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'myproject2.wsgi.application'


# Database
# https://docs.djangoproject.com/en/2.1/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}


# Password validation
# https://docs.djangoproject.com/en/2.1/ref/settings/#auth-password-validators

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]


# Internationalization
# https://docs.djangoproject.com/en/2.1/topics/i18n/

LANGUAGE_CODE = 'en-us'

TIME_ZONE = 'UTC'

USE_I18N = True

USE_L10N = True

USE_TZ = True


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/2.1/howto/static-files/

STATIC_URL = '/static/'

LOGIN_URL = 'login'
LOGOUT_URL = 'logout'
LOGIN_REDIRECT_URL = 'home'
```

#### Once you have all these files are in its place then, run the django server on port let say 5081 and then check the url http://127.0.0.1:5081/signup/.
