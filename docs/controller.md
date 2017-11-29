# Controller

## Request

```php
$uri = $request->path(); // https://example.com/foo/bar => foo/bar
$url = $request->url(); // Without Query String...
$url = $request->fullUrl(); // With Query String...

// Inputs
$input = $request->all();
$name = $request->input('name', 'default name'); // input
$name = $request->query('name', 'default name'); // query

// Check input exists
if($request->has('name')) { /* ... */ }

// Cookie
$request->cookie('name');

// File
$file = $request->file('photo');
if ($request->file('photo')->isValid()) { /* ... */ }
$path = $request->photo->storeAs('images', 'filename.jpg', 's3');

```

## Response

```php
return response('Hello World', 200)->header('Content-Type', 'text/plain');

// Redirect
return redirect('home/dashboard');
return redirect()->route('profile', ['id' => 1]);
return redirect()->action('UserController@profile', ['id' => 1]);

// Cookie
return response('Hello World')->cookie('name', 'value', $minutes);

// JSON
return response()->json(['name' => 'Abigail', 'state' => 'CA']);

// File
return response()->download($pathToFile);
return response()->file($pathToFile);
```

## Authorization

### Gate

```php
// Define gates in app/Providers/AuthServiceProvider::boot()
Gate::define('update-post', function ($user, $post) {
    return $user->id == $post->user_id;
});

// Usage
if (Gate::allows('update-post', $post)) {
    // The current user can update the post...
}

if (Gate::denies('update-post', $post)) {
    // The current user can't update the post...
}
```

### Policy

```php
// Create with 
// php artisan make:policy PostPolicy
// You can find the class in app/Policies
// Register them in app/Providers/AuthServiceProvider
protected $policies = [
    Post::class => PostPolicy::class,
];

// Writing Policies
class PostPolicy
{
    // All access
    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }

    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id;
    }

    public function create(User $user)
    {
        //
    }
}

// Usage

// if
if ($user->can('update', $post)) { /* ... */ }
if ($user->can('create', Post::class)) { /* ... */ }

// middleware
Route::put('/post/{post}', function (Post $post) {
    // The current user may update the post...
})->middleware('can:update,post');
Route::post('/post', function () {
    // The current user may create posts...
})->middleware('can:create,App\Post');

// Controller helper
public function update(Request $request, Post $post)
{
    $this->authorize('update', $post); // if not, it throws AuthorizationException

    // The current user can update the blog post...
}
public function create(Request $request)
{
    $this->authorize('create', Post::class);

    // The current user can create blog posts...
}

// Blade
@can('update', $post)
    <!-- The Current User Can Update The Post -->
@elsecan('create', App\Post::class)
    <!-- The Current User Can Create New Post -->
@endcan

@cannot('update', $post)
    <!-- The Current User Can't Update The Post -->
@elsecannot('create', App\Post::class)
    <!-- The Current User Can't Create New Post -->
@endcannot
```

## Validation

### Try again.

```php
// simpliest.
$validatedData = $request->validate([
    'title' => 'bail|required|unique:posts|max:255', // bail -> stop on the first failure
    'body' => 'required',
]);

// Complex
// php artisan make:request StoreBlogPost
class CommentsRequest extends FormRequest
{
    public function authorize()
    {
        $comment = Comment::find($this->route('comment'));
        return $comment && $this->user()->can('update', $comment);
    }

    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

    public function messages()
    {
        return [
            'title.required' => 'A title is required',
            'body.required'  => 'A message is required',
        ];
    }

    // Custom rule
    public function withValidator($validator)
    {
        $validator->after(function ($validator) {
            if ($this->somethingElseIsInvalid()) {
                $validator->errors()->add('field', 'Something is wrong with this field!');
            }
        });
    }
}
```

### Rules

* alpha, alpha\_dash, alpha\_num
* after[\_or\_equal]:date, before[\_or\_equal]:date -> date will be used in [strtotime()](http://php.net/manual/kr/function.strtotime.php).
* accepted -> for terms of service
* date
* size: string=>length, number=>integer value, array=>count, file=>file size
* email
* exists:table,column -> ex) exists:posts,id
* image
* json
* nullable
* unique:table,column,except,idColumn -> ex) unique:users,email_address
* url


### Error MessageBag

```php
@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

{{ $errors->login->first('email') }} // named messagebag

$errors->first('email');
$errors->get('email'); // all error messages for a field
```

### Redirect/Multiple forms

```php
$validator = Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
], [
    'same'    => 'The :attribute and :other must match.',
    'size'    => 'The :attribute must be exactly :size.',
    'between' => 'The :attribute value :input is not between :min - :max.',
    'in'      => 'The :attribute must be one of the following types: :values',
    'email.required' => 'We need to know your e-mail address!',
]);

$validator->after(function ($validator) {
    if ($this->somethingElseIsInvalid()) {
        $validator->errors()->add('field', 'Something is wrong with this field!');
    }
});

if ($validator->fails()) {
    return redirect('post/create')
                ->withErrors($validator, 'login')
                ->withInput();
}

```
