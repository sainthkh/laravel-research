# Model

* Create: php artisan make:model Name -m
* implicit id: `id`

## CRUD

### Simple Methods

* Create:
 * $m = new Model; $m->name = 'a'; $m->save();
 * [fillable, guarded and mass assignment](https://stackoverflow.com/questions/22279435/what-does-mass-assignment-mean-in-laravel)
 * firstOrCreate(where, [other properties])
* Read: 
 * App\Model::all(), App\Model::find(id), App\Model::findOrFail(id)
* Update:
 * $m = App\Model::find(id); $m->name = 'a'; $m->save();
 * updateOrCreate(where, [updated properties])
* Delete:
 * $m = App\Model::find(id); $m->delete();
 * App\Model::destory(id)

### Advanced

#### Transaction

```php
DB::transaction(function () {
    DB::table('users')->update(['votes' => 1]);

    DB::table('posts')->delete();
});
```

#### scope 

```php
function scopeActive($query) {
    return $query->where('active', 1);
}

$users = App\User::active()->orderBy('created_at')->get();
```

#### Query Builder

Eloquent ORM models are query builders.

* all: `DB::table('users')->get()`
* single row: `DB::table('users')->first()`
* list of columns: `DB::table('users')->pluck('name', 'email')`
* count: `DB::table('users')->count()`
* max: `DB::table('users')->max()`
* min: `DB::table('users')->min()`
* sum: `DB::table('orders')->sum('price')`
* avg: `DB::table('orders')->avg('price')`
* select: `DB::table('users')->select('name', 'email')->get()`
* distinct: `DB::table('users')->distinct()->get()`
* Raw: selectRaw, whereRaw, orWhereRaw, havingRaw, orHavingRaw, orderByRaw
* where: `DB::table('users')->where('votes', '>=', 100)->get()`
* other where clauses: where[Not]Between, where[Not]In, where[Not]Null, whereNotNull, whereDate/Month/Day/Year
* orWhere: `->orWhere(function($query){ ... })`
* orderBy: `DB::table('users')->orderBy('name', 'desc')->get()`
* latest/oldest: `DB::table('users')->latest()->get()`
* skip/take: `DB::table('users')->skip(10)->take(5)->get()`
* update: `DB::table('users')->where('id', 1)->update(['votes' => 1])->get()`


## Relationships

### 1-to-many

```php
// Post.php
public function comments()
{
    return $this->hasMany('App\Comment');
}

// Comment.php
public function post()
{
    return $this->belongsTo('App\Post');
}

// Save
$post = App\Post::find(1);

$post->comments()->save($comment);
$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);
$post->comments()->saveMany([
    new App\Comment(['message' => 'A new comment.']),
    new App\Comment(['message' => 'Another comment.']),
]);
$post->comments()->createMany([
    [
        'message' => 'A new comment.',
    ],
    [
        'message' => 'Another new comment.',
    ],
]);

// Associate/BelongsTo
$account = App\Account::find(10);

$user->account()->associate($account);
$user->save();

$user->account()->dissociate();
$user->save();

// Polymorphic
class Comment extends Model
{
    public function commentable()
    {
        return $this->morphTo();
    }
}

class Post extends Model
{
    public function comments()
    {
        return $this->morphMany('App\Comment', 'commentable');
    }
}

class Video extends Model
{
    public function comments()
    {
        return $this->morphMany('App\Comment', 'commentable');
    }
}
```

Eager Loading

```php
class Book extends Model
{
    public function author()
    {
        return $this->belongsTo('App\Author');
    }
}

// N+1 loading problem.
$books = App\Book::all();

foreach ($books as $book) {
    echo $book->author->name;
}

// Eager loading
$books = App\Book::with('author')->get();

foreach ($books as $book) {
    echo $book->author->name;
}
```

### Many to Many

```php
// User.php
public function roles()
{
    return $this->belongsToMany('App\Role');
}

// Role.php
public function users()
{
    return $this->belongsToMany('App\User');
}
```

```php
$user = App\User::find(1);

foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}

// Change pivot name
return $this->belongsToMany('App\Podcast')
                ->as('subscription')
                ->withTimestamps();

// Filtering with Pivot table columns
return $this->belongsToMany('App\Role')->wherePivot('approved', 1);
return $this->belongsToMany('App\Role')->wherePivotIn('priority', [1, 2]);

// Save
$user = App\User::find($roleId);
$user->roles()->attach($roleId, ['expires' => $expires]);
$user->roles()->attach([
    1 => ['expires' => $expires],
    2 => ['expires' => $expires]
]);

$user->roles()->detach($roleId);
$user->roles()->detach();
$user->roles()->detach([1, 2, 3]);


// Polymorphic
class Post extends Model
{
    public function tags()
    {
        return $this->morphToMany('App\Tag', 'taggable');
    }
}

class Video extends Model
{
    public function tags()
    {
        return $this->morphToMany('App\Tag', 'taggable');
    }
}

class Tag extends Model
{
    public function posts()
    {
        return $this->morphedByMany('App\Post', 'taggable');
    }

    public function videos()
    {
        return $this->morphedByMany('App\Video', 'taggable');
    }
}
```

### Existence/Absence

```php
$posts = App\Post::has('comments')->get();
$posts = App\Post::doesntHave('comments')->get();

$posts = Post::has('comments', '>=', 3)->get();
$posts = Post::has('comments.votes')->get();

$posts = Post::whereHas('comments', function ($query) {
    $query->where('content', 'like', 'foo%');
})->get();
$posts = Post::whereDoesntHave('comments', function ($query) {
    $query->where('content', 'like', 'foo%');
})->get();
```

### Count

```php
$posts = App\Post::withCount('comments')->get();
foreach ($posts as $post) {
    echo $post->comments_count;
}

$posts = Post::withCount([
    'comments',
    'comments as pending_comments_count' => function ($query) {
        $query->where('approved', false);
    }
])->get();
echo $posts[0]->comments_count;
echo $posts[0]->pending_comments_count;
```

### 1-to-1:

```php
// User.php
public function phone()
{
    return $this->hasOne('App\Phone');
}

// Phone.php
public function user()
{
    return $this->belongsTo('App\User');
}
```

## Migration

### Table

```php
// Create table
Schema::create('users', function (Blueprint $table) {
    $table->increments('id');
});

// Drop table
Schema::drop('users');
Schema::dropIfExists('users');
```

### Column

```php
// Create new column
Schema::create('users', function (Blueprint $table) {
    $table->increments('id');
    $table->boolean('confirmed');
    $table->char('name', 100); // CHAR
    $table->string('name', 100); // VARCHAR
    $table->longText('description');
    $table->integer('votes');
    $table->dateTime('created_at');
    $table->timestamp('added_on');
    $table->timestamps();
    $table->morphs('taggable');
    $table->rememberToken(); // Adds a nullable remember_token VARCHAR(100) equivalent column.

    // Foreign Keys
    $table->foreign('user_id')
      ->references('id')->on('users')
      ->onDelete('cascade');

    // Modifiers
    ->autoIncrement() // Set INTEGER columns as auto-increment (primary key)
    ->default($value) // Specify a "default" value for the column
    ->unique() // Create index
});

// Modify
//!!! doctrine/dbal is required !!!
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->nullable()->change();
    $table->renameColumn('from', 'to');
    $table->dropColumn(['votes', 'avatar', 'location']);
});
```

* [datetime vs. timestamp](https://stackoverflow.com/questions/409286/should-i-use-field-datetime-or-timestamp) -> Timestamp is a bit faster. 
* [text vs. longtext](https://stackoverflow.com/questions/13932750/tinytext-text-mediumtext-and-longtext-maximum-storage-sizes) -> text is 64kiB. longtext is 4GiB. 
* `Schema::create` vs. `Schema::table`

## Model Internals

[`Model` class has a lot of traits.](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Database/Eloquent/Model.php)
[Here's the list](https://github.com/laravel/framework/tree/5.5/src/Illuminate/Database/Eloquent/Concerns).

One thing I wanted to know was "why you can call some functions with just a name".

It's because of `Model`'s attribute system. In `Model` class, you can find `__get()` function and it uses [`getAttributes()` in `HasAttributes` trait](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Database/Eloquent/Concerns/HasAttributes.php). 

And `getAttributes()` interprets `$attributes` property. And `$attributes` is populated with `setAttribute()` and the values in `fillable` are filled in `fill` method in `Model` class. And `fill` method uses a lot of methods in [`GuardsAttributes`](https://github.com/laravel/framework/blob/5.5/src/Illuminate/Database/Eloquent/Concerns/GuardsAttributes.php).
