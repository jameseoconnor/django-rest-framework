# django-rest-framework

Tutorial from [~~here~~](https://www.django-rest-framework.org/tutorial/1-serialization/), rewritten for my own understanding.

**Step 1 - Create Virtualenv + Initialise Django Project**

```
python3 -m venv django-rest-env
source django-rest-env/bin/activate

pip3 install django djangorestframework pygments
django-admin startproject tutorial
cd tutorial
```

_Side note_

```
Use this to remove any files that you want to remoive from git
git rm --cached .  -r
```

**Step 2 - Create Django App**

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

**Step 3 - Create Django Model(s)**

This tutorial involves creating a simple app that stores code snippets. So the first thing we always do is create our model.The LEXERS, LANGUAGE_CHOICES and STYLE_CHOICES are all just lists created from the pygments library to populate the choices in a CharField.

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

**Step 4 - Create Serializers**

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

**Step 5 - Test Using Python Shell**

To make it all work together we need four things:

```
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser
```

The Snippet model, the SnippetSerializer model and a renderer and parser from the rest_framework app.

Create a new code snippet:

```
snippet = Snippet(code='foo = "bar"\n')
snippet.save()
```

If we take this snippet, and pass it through our serializer, the data property of the returned object is converted to a Python primitive datatype.

```
serializer = SnippetSerializer(snippet)
serializer.data
```

Then if we take this Python primitive, and use JSONRenderer() from the rest_framework.renderers, we get our valid JSON output.

```
content = JSONRenderer().render(serializer.data)
```

If we want to deserialize from JSON to the object instance, we do the following.

1. Create a bytes stream to write the content to. This allows us to operate on the JSON as if it were a file, we write it to a stream and we can perform operations on it then. If you have a function that expects a file object to write to, then you can give it that in-memory buffer instead of a file.

2. Use the rest_framework JSONParser() class to parse out the stream to python primitive datatype

3. Create a new serializer from the SnippetSerializer class, pass the python primitive into it and call the save() method.

```
import io

stream = io.BytesIO(content)
data = JSONParser().parse(stream)
```

Serialize to JSON ===============================>

OBJECT INSTANCE <====> PYTHON PRIMITIVE <====> JSON

<============================ Deserialize from JSON

**Using ModelSerializer Vs Serializer**
There is a lot of duplication between our model definitions and our serializer definitions, so we can use the ModelSerializer to reduce this duplication.

Also we don't have to define a `create()` and an `update()` method for the serializer.

```
class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ['id', 'title', 'code', 'linenos', 'language', 'style']
```

**Step 6 - Writing Views Using Serializer**
So basically, what we have been doing with the python shell, we now have to replicate in the View. As we know, Django operates on the fundamental basis of HttpRequest and HttpResponse. Whena user requests a resource, the HttpRequest gets sent to the designated view and passed in as the first parameter. This is controlled in the urls.py file. It is the responsibility of the view to return an HttpResponse object.

Few things to note:

- We can use the django.shortcuts render() function to tie the request, the template and the context together in an HttpResponse object.
- We don't need to return a template or context if we are creating a REST API.
- JSONResponse is a subclass of HttpResponse, you don’t need to serialize the data before returning the response object.

The `snippets/views/py` file looks like this:

```
from django.http import HttpResponse, JsonResponse
from django.views.decorators.csrf import csrf_exempt
from rest_framework.parsers import JSONParser
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
```

We have the two response types from HttpResponse & JsonResponse, we have the csrf decorater which we use for dev to avoid csrf errors.
We also have the JSON parser from the rest_framework which we use to create a Python dict from the HtttpRequest object.
Then we have our Snippet class and SnippetSerializer class.

Code for retrieving all. creating snippets

```
@csrf_exempt
def snippet_list(request):
    """
    List all code snippets, or create a new snippet.
    """
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return JsonResponse(serializer.data, safe=False)

    elif request.method == 'POST':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data, status=201)
        return JsonResponse(serializer.errors, status=400)
```

Code for retrieving, updating or deleting specific snippets

```
@csrf_exempt
def snippet_detail(request, pk):
    """
    Retrieve, update or delete a code snippet.
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return JsonResponse(serializer.data)

    elif request.method == 'PUT':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(snippet, data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data)
        return JsonResponse(serializer.errors, status=400)

    elif request.method == 'DELETE':
        snippet.delete()
        return HttpResponse(status=204)
```

**Step 7 - Update urls.py**
Lastly, we have to update our `urls.py` file to route HttpRequest Objects to the correct views.

```
from django.urls import path
from snippets import views

urlpatterns = [
    path('snippets/', views.snippet_list),
    path('snippets/<int:pk>/', views.snippet_detail),
]
```

Also our main `urls.py` file:

```
from django.urls import path, include

urlpatterns = [
    path('', include('snippets.urls')),
]
```

**Step 7 -Test the solution**
Lastly, we can install `httpie` to test our REST API locally.

```
pip install httpie
python manage.py runserver
```

Open up a new terminal window:

```
http http://127.0.0.1:8000/snippets/
http http://127.0.0.1:8000/snippets/2/
```

And that's it! A functional REST API using Django and not much effort!
