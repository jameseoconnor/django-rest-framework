# django-rest-framework

Tutorial from [~~here~~](
https://www.django-rest-framework.org/tutorial/1-serialization/), rewritten for my own understanding. 



__Step 1 - Create Virtualenv + Initialise Django Project__
```
python3 -m venv django-rest-env
source django-rest-env/bin/activate

pip3 install django djangorestframework pygments
django-admin startproject tutorial
cd tutorial
```

*Side note*
```
Use this to remove any files that you want to remoive from git 
git rm --cached .  -r 
```

__Step 2 - Create Django App__

```
python manage.py startapp snippets
```
As per usual, we have to add this app to "INSTALLED_APPS" in the project settings file. We also add the 'rest_framework' app as well at this stage. 

```
INSTALLED_APPS = [
    ...
    'rest_framework',
    'snippets',
]
```

__Step 3 - Create Django Model(s)__

This tutorial involves creating a simple app that stores code snippets. So the first thing we always do is create our model. 

```
from pyexpat import model
from django.db import models

from pygments.lexers import get_all_lexers
from pygments.styles import get_all_styles

LEXERS = [item for item in get_all_lexers() if item[1]]
LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
STYLE_CHOICES = sorted([(item, item) for item in get_all_styles()])


# Create your models here.
class Snippet: 
    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

    class Meta:
        ordering = ['created']

```

and then we make those migrations and migrate 

```
python manage.py makemigrations snippets
python manage.py migrate snippets
```
