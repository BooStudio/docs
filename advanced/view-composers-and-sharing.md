# View Composers and Sharing

## View Composers and Sharing

## Introduction

Very often you will find yourself adding the same variables to a view again and again. This might look something like

```python
def show(self):
    return view('dashboard', {'request': request()})

def another_method(self):
    return view('dashboard/user', {'request': request()})
```

This can quickly become annoying and it can be much easier if you can just have a variable available in all your templates. For this, we can "share" a variable with all our templates with the `View` class.

The `View` class is loaded into our container under the `ViewClass` alias. It's important to note that the `ViewClass` alias from the container points to the class itself and the `View` from the container points to the `View.render` method. By looking at the `ViewProvider` this will make more sense:

```python
from masonite.view import View
from masonite.request import Request

class ViewProvider(ServiceProvider):

    wsgi = False

    def register(self):
        view = View()
        self.app.bind('ViewClass', view)
        self.app.bind('View', view.render)
```

As you can see, we bind the view class itself to `ViewClass` and the render method to the `View` alias.

### View Sharing

We can share variables with all templates by simply specifying them in the `.share()` method like so:

```python
view.share({'request': request()})
```

The best place to put this is in a new Service Provider. Let's create one now called `ViewComposer`.

```text
$ python craft provider ViewComposer
```

This will create a new Service Provider under `app/providers/ViewComposer.py` and should look like this:

```python
class ViewComposer(ServiceProvider):

    def register(self):
        pass

    def boot(self):
        pass
```

We also don't need it to run on every request so we can set `wsgi` to `False`. Doing this will only run this provider when the server first boots up. This will minimize the overhead needed on every request:

```python
class ViewComposer(ServiceProvider):

    wsgi = False

    def register(self):
        pass

    def boot(self):
        pass
```

Great!

Since we need the request, we can throw it in the `boot` method which has access to everything registered into the service container, including the `Request` class.

```python
from masonite.view import View
from masonite.request import Request

class ViewComposer(ServiceProvider):

    wsgi = False

    def register(self):
        pass

    def boot(self, view: View, request: Request):
        view.share({'request': Request})
```

Lastly we need to load this into our `PROVIDERS` list inside our `config/application.py` file.

```python
PROVIDERS = [
    # Framework Providers
    ...
    'masonite.providers.ViewProvider.ViewProvider',
    'masonite.providers.HelpersProvider.HelpersProvider',

    # Third Party Providers

    # Application Providers
    'app.providers.UserModelProvider.UserModelProvider',
    'app.providers.MiddlewareProvider.MiddlewareProvider',
    'app.providers.ViewComposer.ViewComposer', # <- New Service Provider
]
```

And we're done! When you next start your server, the `request` variable will be available on all templates.

#### View Composing

In addition to sharing these variables with all templates, we can also specify only certain templates. All steps will be exactly the same but instead of the `.share()` method, we can use the `.compose()` method:

```python
from masonite.view import View
from masonite.request import Request

def boot(self, view: View, request: Request):
    view.compose('dashboard', {'request': request})
```

Now anytime the `dashboard` template is accessed \(the one at `resources/templates/dashboard.html`\) the `request` variable will be available.

We can also specify several templates which will do the same as above but this time with the `resources/templates/dashboard.html` template AND the `resources/templates/dashboard/user.html` template:

```python
def boot(self, view: View, request: Request):
    view.compose(['dashboard', 'dashboard/user'], {'request': request})
```

Lastly, we can compose a dictionary for all templates:

```python
def boot(self, view: View, request: Request):
    view.compose('*', {'request': request})
```

Note that this has exactly the same behavior as `view.share()`

