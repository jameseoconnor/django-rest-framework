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

__Step 2 - Create Django App__

This tutorial involves creating a simple app that stores code snippets. So the first thing we always do id 
