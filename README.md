Searchable, a search trait for Laravel
==========================================
![Filipac/searchable](https://raw.githubusercontent.com/filipac/searchable/master/searchable.jpg"Filipac/searchable")

Searchable is a trait for Laravel 4.2+ that adds a simple search function to Eloquent Models.

Searchable allows custom columns and relevance for each model.

# Installation

Simply add the package to your `composer.json` file and run `composer update`.

```
"filipac/searchable": "1.0.*"
```

## Usage

Add the trait to your model and your search rules.

```php
use Filipac\Searchable\SearchableTrait;

class User extends \Eloquent
{
	use SearchableTrait;

    /**
     * Searchable rules.
     *
     * @var array
     */
    protected $searchable = [
        ['column' => 'first_name', 'relevance' => 10],
        ['column' => 'last_name', 'relevance' => 10],
        ['column' => 'bio', 'relevance' => 2],
        ['column' => 'email', 'relevance' => 5],
    ];
}
```

Now you can search your model.

```php
// Simple search
$users = User::search($query)->get();

// Search and get relations
$users = User::search($query)
->with('photos')
->get();
```


## Search Paginated

Laravel default pagination doesn't work with this, you have to do it this way

```php
// This class is required
use Filipac\Searchable\DBHelper;
```

```php
// Get the current page values
$page = Input::get('page') ? Input::get('page') : 1;
$count = Input::get('count') ? Input::get('count') : 20; // items per page
$from = 1 + $count * ($page - 1);

// Perform the search
$data = User::search($query)
->take($count)
->skip($from - 1)
->get()
->toArray();

// Get the count of rows of the last query
$db_query_log = DB::getQueryLog();
$db_query = end($db_query_log);
$total_items = DBHelper::getQueryCount($db_query);

// Create the paginator
$users = Paginator::make($data, $total_items, $count);
```

If you want to search through relations too, you can add the following to the desired model:

```php
protected $joinable = [
    'profiles' => ['users.profile_id','profiles.id']
    ];
```
The code above would join the the ```profiles``` table on ```users.profile_id = profiles.id```. Then you can add the following to search in profiles table:
```php
/**
     * Searchable rules.
     *
     * @var array
     */
    protected $searchable = [
        ['column' => 'name', 'relevance' => 10],
        ['column' => 'profiles.name', 'relevance' => 1],
        ['column' => 'profiles.description', 'relevance' => 2],
    ];
```
