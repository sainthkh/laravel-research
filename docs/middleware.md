# Middleware

You can find the middlewares at `app/Http/Kernel.php`.

## `auth` Middleware

### TL;DR
* `auth` middleware is used to guard the content from unauthenticated visitors. 
* If you want to change the redirection behavior for the unauthenticated visitors, override `unauthenticated` method in `app/Exceptions/Handler`.

### Beginning

It is used in almost every system. So, I researched it.  

I began with the [`Authenticate` middleware](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Auth/Middleware/Authenticate.php) because I could find it in the `app/Http/Kernel.php`.

The code made me wonder 2 things:

1. what is guards?
2. how does it redirect unauthenticated users to `/login` page?


### What are guards

To understand guards, I read the [authentication document](https://laravel.com/docs/5.5/authentication). 

It says that:

* Guard: How users are authenticated for each request. 
* Providers: How users are retrieved from your persistent storage. 

In other words, 

* Guard: How user information is saved in client. And how to validate it. 
* Providers: How users are saved in the server and how to retrieve it. 

It's still somewhat vague, but I moved on. Then, how the `auth` middleware redirects visitors?


### Redirection

The interesting thing I found in projects is that I cannot find any `view('login')` statement. 

And in many projects, they use methods like `showLoginForm` or `register` for authentication, but I couldn't find them in `LoginController` or `RegisterController`.

So, I started my journey to find them. 

#### Auth Routes

First thing, I learned is that these routes are added to the project after hitting `php artisan make:auth` command. It addes views, routes and controllers to the project. 

I wanted to know what happens exactly. So, I read [the code about the `make:auth` command](https://github.com/laravel/framework/blob/bd352a0d2ca93775fce8ef02365b03fc4fb8cbb0/src/Illuminate/Auth/Console/AuthMakeCommand.php). 

And I found this code:

```php
file_put_contents(
    base_path('routes/web.php'),
    file_get_contents(__DIR__.'/stubs/make/routes.stub'),
    FILE_APPEND
);
```

OK. It says that it adds the content in `routes.stub`. So, I opened [the file](https://github.com/laravel/framework/blob/bd352a0d2ca93775fce8ef02365b03fc4fb8cbb0/src/Illuminate/Auth/Console/stubs/make/routes.stub). It was simple.

```php

Auth::routes();

Route::get('/home', 'HomeController@index')->name('home');
```

It uses facade again. So, I opened the [`Auth` facade](https://github.com/laravel/framework/blob/5.4/src/Illuminate/Support/Facades/Auth.php). And it says that I should go to [`router`](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Routing/Router.php) again. And I finally found the routes. 

```php
public function auth()
{
    // Authentication Routes...
    $this->get('login', 'Auth\LoginController@showLoginForm')->name('login');
    $this->post('login', 'Auth\LoginController@login');
    $this->post('logout', 'Auth\LoginController@logout')->name('logout');
    // Registration Routes...
    $this->get('register', 'Auth\RegisterController@showRegistrationForm')->name('register');
    $this->post('register', 'Auth\RegisterController@register');
    // Password Reset Routes...
    $this->get('password/reset', 'Auth\ForgotPasswordController@showLinkRequestForm')->name('password.request');
    $this->post('password/email', 'Auth\ForgotPasswordController@sendResetLinkEmail')->name('password.email');
    $this->get('password/reset/{token}', 'Auth\ResetPasswordController@showResetForm')->name('password.reset');
    $this->post('password/reset', 'Auth\ResetPasswordController@reset');
}
```

I finally realized how one project created those routes without using those `get`, `post` functions. It simply used `Auth::routes()`. I finally could see the line. 

#### Auth Default Controllers

But it didn't solve my question. 

>> Where is `showLoginForm` method?

I wondered they are in the [Controller class](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Routing/Controller.php) in the framework. I know it's insane. But I checked. 

No, they weren't there. 

So, I searched it and [found this](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Foundation/Auth/AuthenticatesUsers.php). 

Yes, it has everything I needed. But the declaration was strange. `trait`?

I searched it and learned that [it's a way of reusing code](http://php.net/manual/en/language.oop5.traits.php). It's added in PHP 5.4.

You can use `use` keyword in class definition to include the trait. 

Now, I could see this line in the classes. 

```php
// LoginController.php
use AuthenticatesUsers;
```

```php
// RegisterController.php
use RegistersUsers;
```

```php
// ResetPasswordController.php
use ResetsPasswords;
```

I thought the `use` in the class definition is the same `use` when we include `namespace`s into a php file. But it wasn't. It was a overloaded keyword. 

You can read how they are implemented by clicking below:

* [AuthenticatesUsers](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Foundation/Auth/AuthenticatesUsers.php)
* [RegistersUsers](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Foundation/Auth/RegistersUsers.php)
* [ResetsPasswords](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Foundation/Auth/ResetsPasswords.php)


#### Redirection

When I'm here, I wondered where the journey started. What I wanted to know was "how does Laravel redirect unauthenticated visitors". 

And I googled and found some answers. There was a simple answer like ["use `auth` middleware"](https://laracasts.com/discuss/channels/general-discussion/redirect-to-authlogin-if-user-is-a-guest-not-logged-in-l5). But I found that you can change it with [`unauthenticated` method in `app/Exceptions/Handler`](https://laracasts.com/discuss/channels/laravel/default-redirect-login-page-if-not-authenticate-in-54?page=1). So, I read the document about [Error handling](https://laravel.com/docs/5.5/errors). Unfortunately, I couldn't find the method. 

But I learned that it handles every exception the program raises. And I remembered the `AuthenticationException` I saw in the [`Authenticate` middleware](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Auth/Middleware/Authenticate.php). I visited [`AuthenticationException`](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Auth/AuthenticationException.php). But there was nothing interesting. So, I went to the [`Handler` class](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Foundation/Exceptions/Handler.php). And there it was. 

```php
protected function unauthenticated($request, AuthenticationException $exception)
{
    return $request->expectsJson()
                ? response()->json(['message' => $exception->getMessage()], 401)
                : redirect()->guest(route('login'));
}
```

It routes when the visitor is a guest. 

There is [one more method](https://laracasts.com/discuss/channels/laravel/how-do-you-use-redirectifauthenticated-middleware):

Use `RedirectIfAuthenticated` middleware. 

```php
// RedirectIfAuthenticated.php
class RedirectIfAuthenticated
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @param  string|null  $guard
     * @return mixed
     */
    public function handle($request, Closure $next, $guard = null)
    {
        if (Auth::guard($guard)->check()) {
            return redirect('/home');
        }

        return $next($request);
    }
}

// web.php
Route::get('/', function index ()
{
    return view('landing');
})->middleware('guest');
```


### More about Guards

#### How middleware works

Yes, the useful research was finished. But it still made me wonder what exactly `guard` is. And I wanted to know how `guards` are passed to `auth` middleware. 

To understand what's going on, I had to understand how the middleware works in Laravel. And that lead me to [`Kernel` class](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Foundation/Http/Kernel.php).

It uses `handle()` method to handle the incoming HTTP request and the middlewares are handled in `sendRequestThroughRouter()` function. And `sendRequestThroughRouter()` uses the [`Pipeline` class](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Pipeline/Pipeline.php).

In Pipeline class, it uses `carry()` function to call middlewares. And `parsePipeString()` parses the middleware commands in routing codes. 

In conclusion, guards are added to `Authenticate` when we use `auth` middleware like `auth:session`. And it is `... $guards` because [you can state multiple parameters](https://stackoverflow.com/questions/38712282/how-to-pass-multiple-parameters-to-middleware-with-or-condition-in-laravel-5-2) (in this case, guards) like `auth:session, token`. 

#### How guards created.

And it led me to wonder how guards are created. 

First of all, you can configure guards in `config/auth.php`. 

And guards are created in [`AuthManager`](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Auth/AuthManager.php).

It has function `guard()` and it uses `resolve()` to create guards. 

Every [`Manager`](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Support/Manager.php) has a driver.

You can learn about [the methods in guards here](https://github.com/laravel/framework/blob/bd352a0d2ca93775fce8ef02365b03fc4fb8cbb0/src/Illuminate/Auth/GuardHelpers.php).


## Role Middleware

I found this in [Laravel Blog](https://github.com/guillaumebriday/laravel-blog). 

```php
class RoleMiddleware
{
    public function handle($request, Closure $next, $role)
    {
        if (! $request->user()->hasRole($role)) {
            return redirect()->route('home')->withErrors(__('auth.not_authorized'));
        }

        return $next($request);
    }
}
```

And I found this in [Laravel Quick LMS](https://github.com/LaravelDaily/QuickLMS).

```php
class AdminMiddleware
{
    public function handle($request, Closure $next, $guard = null)
    {
        if (!Auth::check()) {
            return redirect('/');
        }
        if (Auth::user()->role()->where('title', 'Student')->count() > 0) {
            return redirect('/');
        }

        return $next($request);
    }
}
```

I found this in [Laravel LMS](https://github.com/LMS-Laravel/LMS-Laravel).

```php
class Authenticate
{
    protected $auth;

    public function __construct(Guard $auth)
    {
        $this->auth = $auth;
    }

    public function handle($request, Closure $next)
    {
        if ($this->auth->guest()) {
            if ($request->ajax()) {
                return response('Unauthorized.', 401);
            } else {
                return redirect()->guest('/');
            }
        }

        return $next($request);
    }
}
```
=> This code is a bit verbose. Because we can do the same with `Auth::check()`.
`Auth::check() !== auth->guest()` [link](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Auth/GuardHelpers.php)

Personally, I prefer Laravel Blog method. 



## Other middlewares

* `EncryptCookies`: List the cookies that should not be encrypted
* `TrimStrings`: List the attribute names that should not be trimmed.
* `VerifyCsrfToken`: List the URIs that will be excluded from CSRF verification. 

