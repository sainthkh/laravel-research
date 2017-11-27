# Routes

## Named routes changed from 5.3.

Old:
```php
Route::get('auth/login', ['as' => 'login', 'uses' => 'Auth\AuthController@getLogin']);
```

New: 
```php
Route::get('auth/login', 'Auth\AuthController@getLogin')->name('login');
```

## Mysterious as() function

TL;DR - if you use as with group, it works as a prefix for names. 

The story starts with this simple line of code:

```php
Route::prefix('admin')->middleware(['auth', 'role:admin'])->namespace('Admin')->as('admin.')->group(function () { ...
```

I wanted to know what on earth `as()` is. First thing I did was to find the api documentation. [I found one.](https://laravel.com/api/5.5/Illuminate/Support/Facades/Route.html) But it became more mysterious. Because there was no definition. 

In Facade documentation, it says that Route Facade is the instance of [Router class](https://laravel.com/api/5.5/Illuminate/Routing/Router.html). But I went there but couldn't find the `as()` function. 

So, I tracked the history and [found this](https://github.com/guillaumebriday/laravel-blog/commit/ef56f0379082d979cc69331d9f507ade44171904#diff-7e3ce459dfcc113722bdf4667ceffc11). 

[It's called fluent routing](https://laracasts.com/series/whats-new-in-laravel-5-4/episodes/4). It's a new feature in Laravel 5.4.

Then, the problem is what is `as()`?

It was added [when he was trying to add posts resource](https://github.com/guillaumebriday/laravel-blog/commit/e05c1766dbddea1adba5aaf0b08b5e69c41a26ce#diff-7e3ce459dfcc113722bdf4667ceffc11). 

So, I learned that it's something related with `group()` function. 

Next thing I did is to find this group function. Fortunately, it was the function in [Registrar interface](https://laravel.com/api/5.5/Illuminate/Contracts/Routing/Registrar.html). 

And I wanted to know which class implemented this interface and [it was router](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Routing/Router.php). 

In router, this information is used to update `$groupStack` and load the routes. 

```php
public function group(array $attributes, $routes)
{
    $this->updateGroupStack($attributes);
    // Once we have updated the group stack, we'll load the provided routes and
    // merge in the group's attributes when the routes are created. After we
    // have created the routes, we will pop the attributes off the stack.
    $this->loadRoutes($routes);
    array_pop($this->groupStack);
}
```

And no matter which method you use, it will call `addRoute()` and `createRoute()`. And it merges current `$groupStack` attributes to route. 

Finally, in [`Route` class](https://github.com/laravel/framework/blob/183c7516f80f078c8eadea4f550d91fb9c107d2c/src/Illuminate/Routing/Route.php), `as` is used for the name. But one interesting thing is that when you use `name` function more than once, it appends new name next to the existing one. 

```php
public function name($name)
{
    $this->action['as'] = isset($this->action['as']) ? $this->action['as'].$name : $name;
    return $this;
}
```

I finally understood why there is a `.` next to `'admin'`. He used it like below. 

```php
Form::model($post, ['method' => 'DELETE', 'route' => ['admin.posts_thumbnail.destroy', $post], 'data-confirm' => __('forms.posts.delete_thumbnail')])

<a class="nav-link {{ Request::is('admin/dashboard') ? 'active' : '' }}" href="{{ route('admin.dashboard') }}">

{{ link_to_route('admin.dashboard', __('dashboard.dashboard'), [], ['class' => 'nav-link']) }}
```

## Creating API endpoints

You cannot find this in Routing, [it's in controllers](https://laravel.com/docs/5.5/controllers#resource-controllers). 