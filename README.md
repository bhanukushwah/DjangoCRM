# DjangoCRM


## Installing PIP
PIP is a Python package manager which's used to install Python packages from Python Package Index which is more advanced than easy_install the default Python package manager that's installed by default when you install Python.

You should use PIP instaed of easy_install whenever you can but for installing PIP itself you should use easy_install. So let's first install PIP:

Open your terminal and enter:
```
$ sudo easy_install pip
```
You can now install Django on your system using pip


```
$ sudo pip install django
```
While you can do this to install Django, globally on your system, it's strongly not recommend. Instead you need to use a virtual environement to install packages.

Creating a MySQL Database
We'll be using a MySQL database. In your terminal invoke the mysql client using the following command:
```
$ mysql -u root -p
```
Enter your MySQL password and hit Enter.

Next, run the following SQL statement to create a database:
```
mysql> create database crmdb;
```

Creating a Virtual Environment
Let's start our tutorial by creating a virtual environment. Open a new terminal, navigate to a working folder and run the following command:
```
$ cd ~/demos
$ python3 -m venv .env
```
Next, activate the virtual environment using the following command:
```
$ source .env/bin/activate
```

Installing Django and Django REST Framework
Now, that you have created and activated your virtual environment, you can install your Python packages using pip. In your terminal where you have activated the virtual environment, run the following commands to install the necessary packages:
```
$ pip install django
$ pip install djangorestframework
```
You will also need to install the MySQL client for Python using pip:
```
$ pip install mysqlclient
```
Creating a Django Project
Now, let's proceed to creating our django project. In your terminal, run the following command:
```
$ django-admin startproject simplecrm
```
This command will take care of creating a bunch of necessary files for the project.

Executing the tree command in the root of our created project will show us the files that were created.

    .
    ├── simplecrm
    │   ├── __init__.py
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    └── manage.py

__init__ is the Python way to mark the containing folder as a Python package which means a Django project is a Python package.

settings.py is the project configuration file. You can use this file to specify every configuration option of your project such as the installed apps, site language and database options etc.

urls.py is a special Django file which maps all your web app urls to the views.

wsgi.py is necessary for starting a wsgi application server.

manage.py is another Django utility to manage the project including creating database and starting the local development server.

These are the basic files that you will find in every Django project. Now the next step is to set up and create the database.

Next, open the settings.py file and update the database setting to point to our crmdb database:
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', 
        'NAME': 'crmdb',
        'USER': 'root',
        'PASSWORD': 'YOUR_DB_PASSWORD',
        'HOST': 'localhost',   
        'PORT': '3306',
    }    
}
```
Next, add rest_framework to the INSTALLED_APPS array:
```
INSTALLED_APPS = [
    # [...]
    'rest_framework'
]
```
Finally, migrate the database using the following commands:
```
$ cd simplecrm
$ python manage.py migrate
```

You will be able to access your database from the 127.0.0.1:8000 address.

Create an Admin User
Let's create an admin user using the following command:
```
$ python manage.py createsuperuser
```
Creating a Django Application
Next, let's create a Django application for encapsulating our core CRM functionality. In your terminal, run the following command:
```
$ python manage.py startapp crmapp
```
Next, you need to add it in the settings.py file:
```
INSTALLED_APPS = [
    # ...
    'rest_framework',
    'crmapp'
]
```
Creating the Database Models
Let's now proceed to create the database models for our application. We are going to create the following models:

Contact
Account
Activity
ContactStatus
ContactSource
ActivityStatus
We have three main models which are Contact, Account and Activity. The last three models are simply lookup tables (They can be replaced by an enum).

Open the crmapp/models.py file and the following code:
```
from django.db import models
from django.contrib.auth.models import User

INDCHOICES = (
    ('FINANCE', 'FINANCE'),
    ('HEALTHCARE', 'HEALTHCARE'),
    ('INSURANCE', 'INSURANCE'),
    ('LEGAL', 'LEGAL'),
    ('MANUFACTURING', 'MANUFACTURING'),
    ('PUBLISHING', 'PUBLISHING'),
    ('REAL ESTATE', 'REAL ESTATE'),
    ('SOFTWARE', 'SOFTWARE'),
)

class Account(models.Model):
    name = models.CharField("Name of Account", "Name", max_length=64)
    email = models.EmailField(blank = True, null = True)
    phone = models.CharField(max_length=20, blank = True, null = True)
    industry = models.CharField("Industry Type", max_length=255, choices=INDCHOICES, blank=True, null=True)
    website = models.URLField("Website", blank=True, null=True)
    description = models.TextField(blank=True, null=True)
    createdBy = models.ForeignKey(User, related_name='account_created_by', on_delete=models.CASCADE)
    createdAt = models.DateTimeField("Created At", auto_now_add=True)
    isActive = models.BooleanField(default=False)

    def __str__(self):
        return self.name

class ContactSource(models.Model):
    status = models.CharField("Contact Source", max_length=20)

    def __str__(self):
        return self.status

class ContactStatus(models.Model):
    status = models.CharField("Contact Status", max_length=20)

    def __str__(self):
        return self.status

class Contact(models.Model):
    first_name = models.CharField("First name", max_length=255, blank = True, null = True)
    last_name = models.CharField("Last name", max_length=255, blank = True, null = True)
    account = models.ForeignKey(Account, related_name='lead_account_contacts', on_delete=models.CASCADE, blank=True, null=True)
    email = models.EmailField()
    phone = models.CharField(max_length=20, blank = True, null = True)
    address = models.TextField(blank=True, null=True)
    description = models.TextField(blank=True, null=True)
    createdBy = models.ForeignKey(User, related_name='contact_created_by', on_delete=models.CASCADE)
    createdAt = models.DateTimeField("Created At", auto_now_add=True)
    isActive = models.BooleanField(default=False)

    def __str__(self):
        return self.first_name

class ActivityStatus(models.Model):
    status = models.CharField("Activity Status", max_length=20)

    def __str__(self):
        return self.status

class Activity(models.Model):
    description = models.TextField(blank=True, null=True)
    createdAt = models.DateTimeField("Created At", auto_now_add=True)
    contact = models.ForeignKey(Contact, on_delete=models.CASCADE, blank=True, null=True)

    def __str__(self):
        return self.description
```

## Creating Model Serializers
After creating models we need to create the serializers. In the crmapp folder create a serializers.py file:
```
$ cd crmapp
$ touch serializers.py
```
Next, open the file and add the following imports:
```
from rest_framework import serializers

from .models import Account, Activity, ActivityStatus, Contact, ContactSource, ContactStatus
Next, add a serializer class for each model:

class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = "__all__"

class ActivitySerializer(serializers.ModelSerializer):
    class Meta:
        model = Activity
        fields = "__all__"

class ActivityStatusSerializer(serializers.ModelSerializer):
    class Meta:
        model = ActivityStatus
        fields = "__all__"

class ContactSerializer(serializers.ModelSerializer):
    class Meta:
        model = Contact
        fields = "__all__"

class ContactSourceSerializer(serializers.ModelSerializer):
    class Meta:
        model = ContactSource
        fields = "__all__"

class ContactStatusSerializer(serializers.ModelSerializer):
    class Meta:
        model = ContactStatus
        fields = "__all__"
        
```
## Creating API Views
After creating the model serializers, let's now create the API views. Open the crmapp/views.py file and add the following imports:
```
from rest_framework import generics
from .models import Account, Activity, ActivityStatus, Contact, ContactSource, ContactStatus
from .serializers import AccountSerializer, ActivitySerializer, ActivityStatusSerializer, ContactSerializer, ContactSourceSerializer, ContactStatusSerializer
```
Next, add the following views:
```
from rest_framework import generics
from .models import Account, Activity, ActivityStatus, Contact, ContactSource, ContactStatus
from .serializers import AccountSerializer, ActivitySerializer, ActivityStatusSerializer, ContactSerializer, ContactSourceSerializer, ContactStatusSerializer

class AccountAPIView(generics.ListCreateAPIView):
    queryset = Account.objects.all()
    serializer_class = AccountSerializer

class ActivityAPIView(generics.ListCreateAPIView):
    queryset = Activity.objects.all()
    serializer_class = ActivitySerializer

class ActivityStatusAPIView(generics.ListCreateAPIView):
    queryset = ActivityStatus.objects.all()
    serializer_class = ActivitySerializer

class ContactAPIView(generics.ListCreateAPIView):
    queryset = Contact.objects.all()
    serializer_class = ContactSerializer

class ContactStatusAPIView(generics.ListCreateAPIView):
    queryset = ContactStatus.objects.all()
    serializer_class = ContactSerializer

class ContactSourceAPIView(generics.ListCreateAPIView):
    queryset = ContactSource.objects.all()
    serializer_class = ContactSourceSerializer
```
    
After creating these models, you need to create migrations using the following command:
```
$ python manage.py makemigrations
```
Next, you need to migrate your database using the following command:
```
$ python manage.py migrate
```

## Creating API URLs
Let's now create the API URLs to access our API views. Open the urls.py file and add the following imports:
```
from django.contrib import admin
from django.urls import path
from crmapp import views
```
Next, add the following content:
```
urlpatterns = [
    path('admin/', admin.site.urls),
    path(r'accounts', views.AccountAPIView.as_view(), name='account-list'),
    path(r'contacts', views.ContactAPIView.as_view(), name='contact-list'),
    path(r'activities', views.ActivityAPIView.as_view(), name='activity-list'),
    path(r'activitystatuses', views.ActivityStatusAPIView.as_view(), name='activity-status-list'),
    path(r'contactsources', views.ContactSourceAPIView.as_view(), name='contact-source-list'),
    path(r'contactstatuses', views.ContactStatusAPIView.as_view(), name='contact-status-list')
]
```
## Enabling CORS
For development purposes, we'll need to enable CORS (Cross Origin Resource Sharing) in our Django application.

So start by installing django-cors-headers using pip
```
$ pip install django-cors-headers
```
Next, you need to add it to your project settings.py file:
```
INSTALLED_APPS = (
    ## [...]
    'corsheaders'
)
```
Next, you need to add corsheaders.middleware.CorsMiddleware middleware to the middleware classes in settings.py
```
MIDDLEWARE = (
    'corsheaders.middleware.CorsMiddleware',
    # [...]
)
```
You can then, either enable CORS for all domains by adding the following setting:
```
CORS_ORIGIN_ALLOW_ALL = True
```
You can find more configuration options from the docs.

Starting the local development server
Django has a local development server that can be used while developing your project. It's a simple and primitive server which's suitable only for development not for production.

To start the local server for your project, you can simply issue the following command inside your project root directory:

```
$ python manage.py runserver
```
Next navigate to the http://localhost:8000/ address with a web browser.

You should see a web page with a message:

It worked!
