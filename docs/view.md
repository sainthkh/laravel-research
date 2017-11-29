# View

## Blade

```php
// Layout
@section('name')
@endsection
@section('name', 'value')
@yield('name')

// Component
<!-- /resources/views/alert.blade.php -->
<div class="alert alert-danger">
    <h2>{{$title}}</h2>
    {{ $slot }}
</div>
@component('alert', ['foo' => 'bar'])
    @slot('title')
        Forbidden
    @endslot
    <strong>Whoops!</strong> Something went wrong!
@endcomponent

// Data
{{ $usual }}
{!! $unescaped !!}
@json($array)
@{{ $forFrameworks}}
@verbatim
    <div class="long-code-for-framework">
        {{ $someValue }}
    </div>
@endverbatim

// If
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I dont have any records!
@endif
@unless (Auth::check())
    You are not signed in.
@endunless
@isset($records)
    // $records is defined and is not null...
@endisset
@empty($records)
    // $records is "empty"...
@endempty
@auth
    // The user is authenticated...
@endauth
@guest
    // The user is not authenticated...
@endguest

// Loop
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach

@forelse ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>Im looping forever.</p>
@endwhile

// Others
{{-- This comment will not be present in the rendered HTML --}}
@php
    //
@endphp
@include('view.name', ['some' => 'data'])
@includeIf('view.name', ['some' => 'data']) // check exists
@includeFirst(['custom.admin', 'admin'], ['some' => 'data'])
@includeWhen($boolean, 'view.name', ['some' => 'data'])
@each('view.name', $jobs, 'dataBagName', 'view.empty')
@push('scripts')
    <script src="/example.js"></script>
@endpush
@stack('scripts')
```

Loop Variable
* $loop->index	The index of the current loop iteration (starts at 0).
* $loop->iteration	The current loop iteration (starts at 1).
* $loop->first
* $loop->last
* $loop->parent	When in a nested loop, the parent's loop variable.


## Composer 

Retrieving data that many views will share.

```php
// 1. Adding view composer service
class ComposerServiceProvider extends ServiceProvider
{
    public function boot()
    {
        // Using class based composers...
        View::composer(
            ['profile', 'dashboard'], 'App\Http\ViewComposers\ProfileComposer'
        );

        // Using Closure based composers...
        View::composer('dashboard', function ($view) {
            //
        });
    }

    // ...
}

// 2. Add composer
class ProfileComposer
{
    protected $users;

    public function __construct(UserRepository $users)
    {
        // Dependencies automatically resolved by service container...
        $this->users = $users;
    }

    public function compose(View $view)
    {
        $view->with('count', $this->users->count());
    }
}
```

## Create URL

```php
route('post.show', ['post' => 1]);
route('post.show', ['post' => $post]);
action('HomeController@index');
action('UserController@profile', ['id' => 1]);
```

## Laravel HTML

It was removed from 5.0. You can get it from [Laravel Collective](https://github.com/LaravelCollective/html).

```php
Form::open(['url' => 'foo/bar'])
Form::model($user, ['route' => ['user.update', $user->id]])
<form action="{{ route('user.update', $user->id) }}">

Form::label('email', 'E-Mail Address');
Form::text('email', 'example@gmail.com');
Form::password('password', ['class' => 'awesome']);
Form::email($name, $value = null, $attributes = []);
Form::file($name, $attributes = []);
Form::number('name', 'value');
```