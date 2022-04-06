# Laravel 6.*|7.*|8.*|9.* : Laravel Pretty Pagination 

[![Latest Stable Version](https://poser.pugx.org/web-ruslan/laravel-pretty-pagination/v/stable)](https://packagist.org/packages/web-ruslan/laravel-pretty-pagination)
[![Total Downloads](https://poser.pugx.org/web-ruslan/laravel-pretty-pagination/downloads)](https://packagist.org/packages/web-ruslan/laravel-pretty-pagination)
[![Build Status](https://scrutinizer-ci.com/g/web-ruslan/laravel-pretty-pagination/badges/build.png?b=master)](https://scrutinizer-ci.com/g/web-ruslan/laravel-pretty-pagination/build-status/master)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/web-ruslan/laravel-pretty-pagination/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/web-ruslan/laravel-pretty-pagination/?branch=master)
[![Code Intelligence Status](https://scrutinizer-ci.com/g/web-ruslan/laravel-pretty-pagination/badges/code-intelligence.svg?b=master)](https://scrutinizer-ci.com/code-intelligence)
[![License](https://poser.pugx.org/web-ruslan/laravel-pretty-pagination/license)](https://packagist.org/packages/web-ruslan/laravel-pretty-pagination)

This package adds the `paginate` route method to support pagination via custom routes instead of query strings. This also allows for easily translatable pagination routes ex. `/news/page/2`, `/nieuws/pagina/2`.
## Install
Via Composer
``` bash
composer require web-ruslan/laravel-pretty-pagination
```

First register the service provider and facade in your application.

```php
// config/app.php

'providers' => [
    ...
    'Web-ruslan\PaginateRoute\PaginateRouteServiceProvider',
];

'aliases' => [
    ...
    'PaginateRoute' => 'Web-ruslan\PaginateRoute\PaginateRouteFacade',
];
```

Then register the macros in `App\Providers\RouteServiceProvider::boot()`.

```php
// app/Providers/RouteServiceProvider.php

use Web-ruslan\PaginateRoute\PaginateRouteFacade as PaginateRoute;

// ...

public function boot()
{
    PaginateRoute::registerMacros(); // Add this line

    parent::boot();
}
```

## Usage

The `paginate` route macro will register two routes for you.

```php
// app/Http/routes.php

// Generates /users & /users/page/{page}
Route::paginate('users', 'UsersController@index');

```

In your route's action you can just use Laravel's regular pagination methods.

```php
// app/Http/Controllers/UsersController.php

public function index()
{
    return view('users.index', ['users' => \App\User::simplePaginate(5)]);
}
```

If you want to customize paginator base url just use setPath function

```php
public function index()
{
    $users = \App\User::simplePaginate(5);
    $users->setPath('custom/url');
    return view('users.index', ['users' => $users]);
}
```

If you want to customize or add translations for the "page" url segment, you can publish the language files.

``` bash
php artisan vendor:publish --provider="Web-ruslan\PaginateRoute\PaginateRouteServiceProvider"
```

### Generating Url's

Since Laravel's paginator url's will still use a query string, PaginateRoute has it's own url generator and page helper functions.

```

@if(PaginateRoute::hasPreviousPage())
  <a href="{{ PaginateRoute::previousPageUrl() }}">Previous</a>
@endif

@if(PaginateRoute::hasNextPage($users))
  <a href="{{ PaginateRoute::nextPageUrl($users) }}">Next</a>
@endif
```

The `nextPage` functions require the paginator instance as a parameter, so they can determine whether there are any more records.

```php
/**
 * @param  \Illuminate\Contracts\Pagination\Paginator $paginator
 * @return int|null
 */
public function nextPage(Paginator $paginator)
```

```php
/**
 * @param  \Illuminate\Contracts\Pagination\Paginator $paginator
 * @return bool
 */
public function hasNextPage(Paginator $paginator)
```

```php
/**
 * @param  \Illuminate\Contracts\Pagination\Paginator $paginator
 * @return string|null
 */
public function nextPageUrl(Paginator $paginator)
```

```php
/**
 * @param  \Illuminate\Contracts\Pagination\Paginator $paginator
 * @return int|null
 */
public function previousPage(Paginator $paginator)
```

```php
/**
 * @return bool
 */
public function hasPreviousPage()
```

```php
/**
 * @param  bool $full
 * @return string|null
 */
public function previousPageUrl($full = false)
```

```php
/**
 * @param  \Illuminate\Contracts\Pagination\Paginator $paginator
 * @param int  $page
 * @param bool $full
 * @return string
 */
public function pageUrl(Paginator $paginator, $page, $full = false)
```

If `$full` is true, the first page will be a fully qualified url. Ex. `/users/page/1` instead if just `/users` (this is the default).

To retrieve the url of a specific page of a paginated route, that isn't the current route, there's the `addPageQuery` function.

```php
/**
 * @param string $url
 * @param int $page
 * @param bool $full
 * @return string
 */
public function addPageQuery($url, $page, $full = false)
```

You can also retrieve an array with all available urls. These can be rendered as a plain html list with page numbers. Note that these functions require a `LengthAwarePaginator`.

```php
/**
 * @param  \Illuminate\Contracts\Pagination\LengthAwarePaginator $paginator
 * @param  bool $full
 * @return array
 */
public function allUrls(LengthAwarePaginator $paginator, $full = false)
```

```php
/**
 * @param  \Illuminate\Contracts\Pagination\LengthAwarePaginator $paginator
 * @param  bool $full
 * @param  array $styles
 * @param  bool $additionalLinks
 * @return string
 */
public function renderPageList(LengthAwarePaginator $paginator, $full = false, $styles = null, $additionalLinks = false)

```

```html
/**
 * Example : {!! PaginateRoute::renderPageList($items,true,null,true) !!}
 */
<!-- Example output: -->
<ul>
    <li><a href="http://example.com/news">1</a></li>
    <li><a href="http://example.com/news/page/2">2</a></li>
    <li class="active"><a href="http://example.com/news/page/3">3</a></li>
    <li><a href="http://example.com/news/page/4">4</a></li>
    <li><a href="http://example.com/news/page/4">&raquo;</a></li>
</ul>
```
You can add styles for paginator like this:

```html
/**
 * Example : {!! PaginateRoute::renderPageList($companies, false,
 * [
 *    'ul' => 'pagination-list',
 *    'li' => 'pagination-list-item',
 *    'a' => 'pagination-list-link',
 *    'previous_a' => 'pagination-prev',
 *    'next_a' => 'pagination-next',
 *    'active_a' => 'pagination-active',
 *    'previous_label' => '<i class="fas fa-chevron-left"></i>',
 *    'next_label' => '<i class="fas fa-chevron-right"></i>',
 * ],
 * true) !!}
 */
<!-- Example output: -->
<ul class="pagination-list">
    <li class="pagination-list-item">
        <a href="http://example.com/news" class="pagination-prev"><i class="fas fa-chevron-left"></i></a>
    </li>
    <li class="pagination-list-item">
        <a href="http://example.com/news/page/2" class="pagination-list-link pagination-active">2</a>
    </li>
    <li class="pagination-list-item">
        <a href="http://example.com/news/page/3" class="pagination-list-link">3</a>
    </li>
    <li class="pagination-list-item">
        <a href="http://example.com/news/page/4" class="pagination-list-link">4</a>
    </li>
    <li class="pagination-list-item">
        <a href="http://example.com/news/page/5" class="pagination-list-link">4</a>
    </li>
    <li class="pagination-list-item">
        <a href="http://example.com/news/page/3" class="pagination-next"><i class="fas fa-chevron-right"></i></a>
    </li>
</ul>
```

You can render link tags to mark previous and next page for SEO. Note that these functions require a `LengthAwarePaginator`.

```php
/**
 * @param  \Illuminate\Contracts\Pagination\LengthAwarePaginator $paginator
 * @param  bool $full
 * @return string
 */
public function renderRelLinks(LengthAwarePaginator $paginator, $full = false)
```

```html
<!-- Example output: -->
<link rel="prev" href="http://example.com/news/page/2" />
<link rel="next" href="http://example.com/news/page/4" />
```

## Tests

The package contains some integration/smoke tests, set up with Orchestra. The tests can be run via phpunit.

```
$ phpunit
```

## Comming Soon
 - Bootstrap 4.* supported pagination rendering.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Security

If you discover any security related issues, please email webruslan@gmail.com instead of using the issue tracker.

## Credits

- [Sebastian De Deyne](https://github.com/sebastiandedeyne)
- [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
