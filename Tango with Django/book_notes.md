# Notes on Tango with Django

## [Intro](http://www.tangowithdjango.com/book17/chapters/overview.html)

#### Common Design Specifications
1. N-Tier Architecture & Technologies Used
2. Wireframes
3. Pages & URL-Mappings
4. Entity-Relationship Diagram

## [Set-up](http://www.tangowithdjango.com/book17/chapters/requirements.html)

######1. Know the Terminal 
- Be familiar with [Unix](http://www.ee.surrey.ac.uk/Teaching/Unix/unixintro.html)
- Know the core commands

######2. Install Software
- Install python
- Set-up the PYTHONPATH
- Use setuptools & pip
- Install django
- Install pillow
- Install other packages
    
######3. Set-up the Development Environment
- Set-up a virtual environment
- Create a code repository

## Creating a Django project
Create the project folder, `tango_with_django_project`.
```
$ django-admin.py startproject tango_with_django_project
```
Run the project in your local server.
```
python manage.py runserver
```
## Creating a Django Application
Create the app folder for the `rango` app.
```
python manage.py startapp rango
```
Include the newly created app in the project's settings.
```
# settings.py

INSTALLED_APPS = (
    ...
    'rango`,
)
```

## Creating a View
The `views.py` is where you can store a series of functions that take a clients’s requests and return responses.
```python
# rango/views.py

from django.http import HttpResponse

def index(request):
    return HttpResponse("Rango says hey there world!")
```

## Mapping URLs
The contents of `urls.py` will allow you to map URLs for your application (e.g. http://www.tangowithdjango.com/rango/) to specific views.
```python
# rango/urls.py

from django.conf.urls import patterns, url
from rango import views

urlpatterns = patterns('',
        url(r'^$', views.index, name='index'))
```
Join rango app's urls to your project’s master `urls.py` file.
```python
# tango_with_django_project/urls.py

urlpatterns = patterns('',
    ...
    url(r'^rango/', include('rango.urls')),
)
```
## Adding Templates

Set the templates directory path in your project’s `settings.py` file.

```python
# settings.py

TEMPLATE_PATH = os.path.join(BASE_DIR, 'templates')

TEMPLATE_DIRS = (
     TEMPLATE_PATH,
)
```

Create a `templates` directory in your project folder wherein you save the following template. You may use Django template variables (e.g. `{{ variable_name }}`) in your template.

```
# templates/rango/index.html
<!DOCTYPE html>

{% load static %}

<html>
	<head>
		<title>Rango</title>
	</head>
	<body>
		<h1>Rango says...</h1>
		hello world! <strong>{{ boldmessage }}</strong><br />
	</body>
</html>
```

Find or create a new view within an application’s `views.py` file. Add your view-specific logic to the view.

```python
# rango/views.py

def index(request):
	# Construct a dictionary object which you can pass to the template engine 
	# as part of the template’s context.
    context_dict = {'boldmessage': "I am bold font from the context"}

    # Make use of the `render()` helper function to generate the rendered response. 
    # Ensure you reference the request, then the template file, followed by the context dictionary!
    return render(request, 'rango/index.html', context_dict)
```

Map the view to a URL by modifying your project’s `urls.py` file - and the application-specific `urls.py` file, if any.

## Using Static Media
Set the static directory path in your project’s `settings.py` file.
```python
# settings.py

STATIC_PATH = os.path.join(BASE_DIR, 'static')

STATIC_URL = '/static/'

STATICFILES_DIRS = (
    STATIC_PATH,
)
```

Create a `static` directory in your project folder wherein you save your static files, the directory you specified in your project’s STATICFILES_DIRS tuple within settings.py.
```
static/
    images/
         rango.jpg
```

Add a reference to the static media file to a template. For example, an image would be inserted into an HTML page through the use of the <img /> tag. Remember to use the {% load static %} and {% static "filename" %} commands within the template to access the static files.
```python
# index.html

{% load static %}

...
<img src="{% static "images/rango.jpg" %}" alt="Picture of Rango" /><br />
...
```

## Database Setup

With a new Django project, you should first tell Django about the database you intend to use. 

```python
# settings.py

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

## Working with Models
A **model** is a Python object that describes your data model/table. 

###### Adding models

First, create the new models in your application.
```python
# rango/models.py

class Category(models.Model):
    name = models.CharField(max_length=128, unique=True)

    def __unicode__(self):  #For Python 2, use __str__ on Python 3
        return self.name

class Page(models.Model):
    category = models.ForeignKey(Category)
    title = models.CharField(max_length=128)
    url = models.URLField()
    views = models.IntegerField(default=0)

    def __unicode__(self):      #For Python 2, use __str__ on Python 3
        return self.title
```

*Model/Database Table Relations:*
- `ForeignKey`, a field type that allows us to create a **one-to-many** relationship.
- `OneToOneField`, a field type that allows us to define a strict **one-to-one** relationship.
- `ManyToManyField`, a field type which allows us to define a **many-to-many** relationship.

###### Admin Registration

Update `admin.py` to include and register your new model(s).

```python
from django.contrib import admin
from rango.models import Category, Page

admin.site.register(Category)
admin.site.register(Page)
```
###### Database Migration

Then perform the migration `$ python manage.py makemigrations`.

Apply the changes `$ python manage.py migrate`. This will create the necessary infrastructure within the database for your new model(s).

Create/Edit your population script for your new models, if any.

In which case you have to delete your database, run the `migrate` command, then `createsuperuser` command, followed by the `sqlmigrate` commands for each app, then you can populate the database.

## Creating Data Driven Pages
###### 5 Steps

1. Import the models you wish to use into your application’s `views.py` file.  Within `views.py`, query the model to get the data you want to present.

	```
	# views.py
	from rango.models import Category
	
	def category(request, category_name_slug):
	    context_dict = {}
	    try:
	        category = Category.objects.get(slug=category_name_slug)
	        context_dict['category_name'] = category.name
	        pages = Page.objects.filter(category=category)
	        context_dict['pages'] = pages
	        context_dict['category'] = category
	        context_dict['category_name_slug'] = category_name_slug
	    except Category.DoesNotExist:
	        pass
	    return render(request, 'rango/category.html', context_dict)
	```

2. Pass the results from your model into the template’s context.

3. Setup your template to present the data to the user.

	```
	<!-- index.html -->
	
	{% if category %}
	    {% if pages %}
	    <ul>
	        {% for page in pages %}
	        <li><a href="{{ page.url }}">{{ page.title }}</a></li>
	        {% endfor %}
	    </ul>
	    {% else %}
	        <strong>No pages currently in category.</strong>
	    {% endif %}
	{% else %}
	    The specified category {{ category_name }} does not exist!
	{% endif %}
	```

4. Map a URL to your view.


## Working with Forms

1. Create a `forms.py` file within your Django application’s directory to store form-related classes. Create a `ModelForm` class for each model that you wish to represent as a form. Customise the forms as you desire.
	
	```python
	# rango/forms.py
	from django import forms
	from rango.models import Category
	
	class CategoryForm(forms.ModelForm):
	    name = forms.CharField(max_length=128, help_text="Please enter the category name.")
	    views = forms.IntegerField(widget=forms.HiddenInput(), initial=0)
	    likes = forms.IntegerField(widget=forms.HiddenInput(), initial=0)
	    slug = forms.CharField(widget=forms.HiddenInput(), required=False)
	
	    class Meta:
	        model = Category
	        fields = ('name',)
	```
	
2. Create or update a view to handle the form - including displaying the form, saving form data, and flagging up errors.

	```python
	# rango/views.py
	
	from rango.forms import CategoryForm

	def add_category(request):
	    if request.method == 'POST':
	        form = CategoryForm(request.POST)
	        if form.is_valid():
	            form.save(commit=True)
	            return index(request)
	        else:
	            print form.errors
	    else:
	        form = CategoryForm()
	    return render(request, 'rango/add_category.html', {'form': form})
	```
	
3. Create or update a template to display the form.

	```
	<!-- templates/rango/add_category.html -->
	
	<!DOCTYPE html>
	<html>
		<head>
			<title>Rango</title>
		</head>
		<body>
			<h1>Add a Category</h1>
			<form id='category_form' method='post' action='/rango/add_category/'>
				{% csrf_token %}
				{% for hidden in form.hidden_fields %}
					{{ hidden }}
				{% endfor %}
				{% for field in form.visible_fields %}
					{{ field.errors }}
					{{ field.help_text }}
					{{ field}}
				{% endfor %}
				<input type='submit' name='submit' value='Create Category' />
			</form>
		</body>
	</html>
	```

4. Add a urlpattern to map to the new view.

	```python
	# rango/urls.py
	
	urlpatterns = patterns('',
	    ...
	    url(r'^add_category/$', views.add_category, name='add_category'),
	    ...
	    )
	
	```

## Authenticating Users


#### Adding Log-in
1. Create a login in view to handle user credentials

	```python
	# views.py
	
	def user_login(request):
	    if request.method == 'POST':
	        username = request.POST.get('username')
	        password = request.POST.get('password')
	        user = authenticate(username=username, password=password)
	        if user:
	            if user.is_active:
	                login(request, user)
	                return HttpResponseRedirect('/rango')
	            else:
	                return HttpResponse('Your Rango account is disabled.')
	        else:
	            print 'Invalid login details: {0}, {1}'.format(username, password)
	            return HttpResponse('Invalid login details supplied.')
	    else:
	        return render(request, 'rango/login.html', {})
	 ```

2. Create a login template to display the login form.
3. Map the login view to a url.
4. Provide a link to login from the index page.

#### Restricting Access with a Decorator

1. Place the **decorator** directly above the function signature, and put a @ before naming the decorator. Python will execute the decorator before executing the code of your function/method.

	```python
	# rango/views.py
	
	from django.contrib.auth.decorators import login_required
	
	@login_required
	def restricted(request):
	    return HttpResponse("Since you're logged in, you can see this text!")
	```
	
2. Define the `LOGIN_URL` with the URL you’d like to redirect users to that are NOT logged in.

	```
	# settings.py
	
	LOGIN_URL = '/rango/login/'
	```

#### Logging Out

1. Add the a view called user_logout() with the following code:

	```python
	# views.py
	
	@login_required
	def user_logout(request):
	    logout(request)
	    return HttpResponseRedirect('/rango/')
	```

2. Map the login view to a url.

3. When a user is authenticated and logged in, he or she can see the 'Restricted Page' and 'Logout' links. If he or she isn’t logged in, 'Register Here' and 'Login' are presented. 

	```
	<!-- index.html -->
	
	{% if user.is_authenticated %}
		<a href="/rango/restricted/">Restricted Page</a><br />
		<a href="/rango/logout/">Logout</a><br />
	{% else %}
		<a href="/rango/register/">Register Here</a><br />
		<a href="/rango/login/">Login</a><br />
	{% endif %}
	```

## Template Inheritance

1. Identify the re-occurring parts of each page that are repeated across your application (i.e. header bar, sidebar, footer, content pane).

2. In a base template, i.e. `base.html`, provide the skeleton structure of a standard page along with any common content, and then define a number of blocks which are subject to change depending on which page the user is viewing.
	
	```
	<!DOCTYPE html>
	
	<html>
	    <head lang="en">
	        <meta charset="UTF-8">
	        <title>Rango - {% block title %}How to Tango with Django!{% endblock %}</title>
	    </head>
	    <body>
	        {% block body_block %}{% endblock %}
	    </body>
	</html>
	
	```

3. Create specific templates - all of which inherit from the base template using `extends` - and specify the contents of each block.

	```
	{% extends 'base.html' %}
	
	{% load staticfiles %}
	
	{% block title %}{{ category_name }}{% endblock %}
	
	{% block body_block %}
	
	    <h1>{{ category_name }}</h1>
	
	{% endblock %}
	```

## Cookies & Sessions

- **Cookies** can keep information in the user's browser until deleted, but they may be blocked.
- **Sessions** are not reliant on the user allowing a cookie. They are not stored in the browser, and work instead like a token allowing access and passing information while the user has their browser open.

#### Client-side cookies
1. Check if the cookie you want exists. The `request.COOKIES.has_key('<cookie_name>')` function returns a boolean value indicating whether a cookie <cookie_name> exists on the client’s computer or not.
2. If the cookie exists, you can then retrieve its value with `request.COOKIES[<cookie_name>]`. Cookies are all returned as strings.
3. If the cookie doesn’t exist, or you wish to update the cookie, pass the value you wish to save to the **response** you generate. Call `response.set_cookie('<cookie_name>', value)`.

#### Session-based cookies
This is a more secure alternative to client-side cookies.

1. Make sure that `MIDDLEWARE_CLASSES` in `settings.py` contains `‘django.contrib.sessions.middleware.SessionMiddleware’`.
2. Configure your session backend `SESSION_ENGINE`.
3. Check to see if the cookie exists via `requests.sessions.get('<cookie_name>')`.
4. Update or set the cookie via the session dictionary, `requests.session['<cookie_name>']`.

## Using 3rd party packages

1. Install the package with pip.

	```$ pip install django-registration-redux```
	
2. Include the package in `settings.py`.

	```
	INSTALLED_APPS = (
		...
		'registration', # add in the registration package
	)
	```
	
3. Update the `urlpatterns` so it includes a reference to the registration package.

	```
	urlpatterns = patterns('',
		...
		(r'^accounts/', include('registration.backends.simple.urls')),
	)
	```
	
4. Read the docs of the 3rd party package to use the rest of its functions.

## Bootstrap for Templates

1. Choose from [Bootstrap's sample layouts](http://getbootstrap.com/getting-started/#examples).
2. Copy and paste the page source of the chosen layout to your own template.
3. Replaced all references of `../../` to be `http://getbootstrap.com`.
4. Remeber to place the reference to the css of the chosen layout after the boostrap css reference.

	```
	<link href="http://getbootstrap.com/examples/dashboard/dashboard.css" rel="stylesheet">
	```
5. Add classes to the template elements for styling. Refer to the [Bootstrap Documentation](http://getbootstrap.com/css/).


## Using Template Tags

**Template Tags** are included in the template used for requesting data, instead of passing data from the views to the context_dict.

1. Create a `templatetags` directory in your app directory. Create 2 files- `__init__.py` and another one containing your template tag code, e.g. `rango_extras.py`.

	```python
	# rango/templatetags/rango_extras.py
	
	from django import template
	from rango.models import Category
	
	register = template.Library()
	
	@register.inclusion_tag('rango/cats.html')
	def get_category_list(cat=None):
	    return {'cats': Category.objects.all(), 'actual_cat': cat}
	```
    	
2. Create a template to be associated with your template tag.

	```
	# templates/rango/cats.html
	
	{% if cats %}
	    <ul class="nav nav-sidebar">
	    {% for c in cats %}
	        {% if c == actual_cat %}<li class='active'>{% else%}<li>{% endif %}
	            <a href="{% url 'category'  c.slug %}">{{ c.name }}</a></li>
	    {% endfor %}
	{% else %}
	    <li> <strong >There are no category present.</strong></li>
	    </ul>
	{% endif %}
	```
	
3. Access the template tag with `get_category_list`, and include any parameters if any, e.g. `category`, to get the actual category.

	```
	# base.html
	
	{% load rango_extras %}
	
	{% get_category_list category %}
	```

## Using APIs
#### Adding Search with Bing API
`run_query()` requests results from Bing’s servers, which are returned in JSON format, parsed into a Python object dictionary, and rendered as part of a template within our application.

```python
import json
import urllib, urllib2

from keys import BING_API_KEY

def run_query(search_terms):

	# Prepares for connecting to Bing by preparing the URL that we will be requesting.
	root_url = 'https://api.datamarket.azure.com/Bing/Search/'
	source = 'Web'
	results_per_page = 10
	offset = 0
	query = "'{0}'".format(search_terms)
	query = urllib.quote(query)
	search_url = "{0}{1}?$format=json&$top={2}&$skip={3}&Query={4}".format(
	        root_url,
	        source,
	        results_per_page,
	        offset,
	        query)
	        
	# Prepares authentication
	username = ''
	password_mgr = urllib2.HTTPPasswordMgrWithDefaultRealm()
	password_mgr.add_password(None, search_url, username, BING_API_KEY)
	
	results = []
	
	# Connect to the Bing API
	try:
		handler = urllib2.HTTPBasicAuthHandler(password_mgr)
		opener = urllib2.build_opener(handler)
		urllib2.install_opener(opener)
		
		# The results from the server are read and saved as a string.
        response = urllib2.urlopen(search_url).read()

        # The string is then parsed into a Python dictionary object.
        json_response = json.loads(response)
        print json_response

        # We loop through each of the returned results, populating a results dictionary. 
        for result in json_response['d']['results']:
            results.append({
                'title': result['Title'],
                'link': result['Url'],
                'summary': result['Description']})
    
    except urllib2.URLError, e:
        print "Error when querying bing API: ", e

    return results
```

## jQuery in Django

1. Add the jquery library and your own jquery codes in the static folder.

	```
	tango_with_django_project /
		static /
		 	js /
		 		jquery-1.11.1.js
		 		rango-jquery.js
	```
	
2. Include a reference to the static files on your base template.

	```
	{% load staticfiles %}
	
	<script src="{% static "js/jquery-1.11.1.js" %}"></script>
	<script src="{% static "js/rango-jquery.js" %}"></script>
	```

3. Once the document is ready i.e. the complete page is loaded, then the anonymous function denoted by function(){ } will be executed.

	```javascript
	// rango-jquery.js
	
	$(document).ready(function() {
	        // JQuery code to be added in here.
	});
	```

## AJAX in Django with jQuery
Instead of reloading the full page, only part of the page or the data in the page is reloaded.

In the following sample code, we add a “Like” button to let users “like” a particular category. The no. of likes are incremented automatically, and the like button is hidden when clicked.

1. Put the necessary `id` or `class` of the HTML element, together with any data you will need e.g. `data-catid`.

	```
	<button id="likes" data-catid="{{category.id}}" class="btn btn-primary" type="button">
	```
	
2. Create a view which will examine the request, perform the necessary database manipulation, and return the data for display.

	```python
	# views.py
	
	def like_category(request):
	
	    cat_id = None
	    if request.method == 'GET':
	        cat_id = request.GET['category_id']
	
	    likes = 0
	    if cat_id:
	        cat = Category.objects.get(id=int(cat_id))
	        if cat:
	            likes = cat.likes + 1
	            cat.likes =  likes
	            cat.save()
	
	    return HttpResponse(likes)
	```

3. Add some JQuery code to perform an AJAX GET request to call the view, and send data to the template, among other things.

	```javascript
	$('#likes').click(function(){
	    var catid;
	    catid = $(this).attr("data-catid");
	    $.get('/rango/like_category/', {category_id: catid}, function(data){
	               $('#like_count').html(data);
	               $('#likes').hide();
	    });
	});
	```
	
4. Map a URL to the view.


## Automated Testing

##### Running Tests
This will run through the tests associated with the rango application.

```
$ python manage.py test rango
```

When you run tests, a temporary database is constructed, which your tests can populate, and perform operations on.

##### Testing the models

We have to inherit from `TestCase`. The naming over the method in the class also follows a convention, all tests start with `test_` and they also contain some type of assertion, which is the test. 

Here we are check if the values are equal, with the assertEqual method, but other types of assertions are also possible. 

```python
# rango/tests.py

from django.test import TestCase
from rango.models import Category

class CategoryMethodtests(TestCase):

	def test_ensure_views_are_positive(self):
		"""
		This should result to True for categories where views are zero or positive.
		"""
		cat = Category(name='test', views=-1, likes=0)
		cat.save()
		self.assertEqual((cat.views >= 0), True)
```

Run `$ python manage.py test rango` to see if the test passes. Initially, this test fails. We will need to update the model, to ensure that this requirement is fulfilled. Check again if the test passes.

##### Testing the views

Django tests views with a mock client, that internally makes a call to a django view via the url. In the test, you have access to the response (including the html) and the context dictionary.

The following test checks that when the index page loads, it displays the message that There are no categories present, when the Category model is empty.

```python
# rango/tests.py

from django.core.urlresolvers import reverse


class IndexViewTests(TestCase):

    def test_index_view_with_no_categories(self):
        """
        If no questions exist, an appropriate message should be displayed.
        """
        response = self.client.get(reverse('index'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "There are no categories present.")
        self.assertQuerysetEqual(response.context['categories'], [])
```

The Django TestCase has access to a client object, which can make requests. It uses the helper function reverse to look up the url of the index page. Then, it tries to get that page, where the response is stored. The test then checks a number of things: 
- Did the page load ok? 
- Does the response, i.e. the html contain the phrase “There are no categories present.”
- Does the context dictionary contain an empty categories list?

##### Coverage Testing

Code `coverage` measures how much of your code base has been tested, and how much of your code has been put through its paces via tests. 

```
coverage run --source='.' manage.py test rango
```

This will run through all the tests and collect the coverage data for the rango application.