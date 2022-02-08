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

This tutorial involves creating a simple app that stores code snippets. So the first thing we always do is create our model.The LEXERS,  LANGUAGE_CHOICES and STYLE_CHOICES are all just lists created from the pygments library to populate the choices in a CharField. 

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

__Step 4 - Create Serializers__:

The serializers effectively help with converting to and from the Django inbuilt Querysets, which are harder to work with when building out a REST API. So these take the Querysets and output JSON. 

The rest_framework app has a module called serializer which is basically like a Django Form. 

Import it: 
```
from rest_framework import serializers
```

Then define a serializer class and derive it from the parent Serializer class: 
```
class SnippetSerializer(serializers.Serializer):
```

Define the fields that we want to serialize/ deserialize, much in the same way we do for model definition: 
```
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)
    code = serializers.CharField(style={'base_template': 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')
```

Define a method to create a new item via the serializer. This takes in a dict of validated data and gets unpacked to fill out the key-value fields in the Snippet.objects.create() method.  
```
    def create(self, validated_data):
        """
        Create and return a new `Snippet` instance, given the validated data.
        """
        return Snippet.objects.create(**validated_data)
```

And another one to update an existing object. This takes the object and the new data as two parameters. 
```
    def update(self, instance, validated_data):
        """
        Update and return an existing `Snippet` instance, given the validated data.
        """
        instance.title = validated_data.get('title', instance.title)
        instance.code = validated_data.get('code', instance.code)
        instance.linenos = validated_data.get('linenos', instance.linenos)
        instance.language = validated_data.get('language', instance.language)
        instance.style = validated_data.get('style', instance.style)
        instance.save()
        return instance
```
