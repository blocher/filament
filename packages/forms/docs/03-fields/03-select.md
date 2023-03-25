---
title: Select
---

The select component allows you to select from a list of predefined options:

```php
use Filament\Forms\Components\Select;

Select::make('status')
    ->options([
        'draft' => 'Draft',
        'reviewing' => 'Reviewing',
        'published' => 'Published',
    ])
```

![](https://user-images.githubusercontent.com/41773797/147612885-888dfd64-6256-482d-b4bc-840191306d2d.png)

## Searching options

You may enable a search input to allow easier access to many options, using the `searchable()` method:

```php
use Filament\Forms\Components\Select;

Select::make('author_id')
    ->label('Author')
    ->options(User::all()->pluck('name', 'id'))
    ->searchable()
```

![](https://user-images.githubusercontent.com/41773797/147613023-cb7d1907-e4d3-4a33-aa86-1c25d780c861.png)

### Returning custom search results

If you have lots of options and want to populate them based on a database search or other external data source, you can use the `getSearchResultsUsing()` and `getOptionLabelUsing()` methods instead of `options()`.

The `getSearchResultsUsing()` method accepts a callback that returns search results in `$key => $value` format. The current user's search is available as `$search`, and you should use that to filter your results.

The `getOptionLabelUsing()` method accepts a callback that transforms the selected option `$value` into a label. This is used when the form is first loaded when the user has not made a search yet. Otherwise, the label used to display the currently selected option would not be available.

Both `getSearchResultsUsing()` and `getOptionLabelUsing()` must be used on the select if you want to provide custom search results:

```php
Select::make('author_id')
    ->searchable()
    ->getSearchResultsUsing(fn (string $search): array => User::where('name', 'like', "%{$search}%")->limit(50)->pluck('name', 'id')->toArray())
    ->getOptionLabelUsing(fn ($value): ?string => User::find($value)?->name),
```

## Multi-select

The `multiple()` method on the `Select` component allows you to select multiple values from the list of options:

```php
use Filament\Forms\Components\Select;

Select::make('technologies')
    ->multiple()
    ->options([
        'tailwind' => 'Tailwind CSS',
        'alpine' => 'Alpine.js',
        'laravel' => 'Laravel',
        'livewire' => 'Laravel Livewire',
    ])
```

![](https://user-images.githubusercontent.com/41773797/147613070-cd82703a-fa05-4f29-b0ac-3eb03b542077.png)

These options are returned in JSON format. If you're saving them using Eloquent, you should be sure to add an `array` [cast](https://laravel.com/docs/eloquent-mutators#array-and-json-casting) to the model property:

```php
use Illuminate\Database\Eloquent\Model;

class App extends Model
{
    protected $casts = [
        'technologies' => 'array',
    ];

    // ...
}
```

If you're [returning custom search results](#returning-custom-search-results), you should define `getOptionLabelsUsing()` instead of `getOptionLabelUsing()`. `$values` will be passed into the callback instead of `$value`, and you should return a `$key => $value` array of labels and their corresponding values:

```php
Select::make('technologies')
    ->multiple()
    ->searchable()
    ->getSearchResultsUsing(fn (string $search): array => Technology::where('name', 'like', "%{$search}%")->limit(50)->pluck('name', 'id')->toArray())
    ->getOptionLabelsUsing(fn (array $values): array => Technology::whereIn('id', $values)->pluck('name', 'id')->toArray()),
```

## Integrating with an Eloquent relationship

> If you're building a form inside your Livewire component, make sure you have set up the [form's model](../adding-a-form-to-a-livewire-component#setting-a-form-model). Otherwise, Filament doesn't know which model to use to retrieve the relationship from.

You may employ the `relationship()` method of the `Select` to configure a `BelongsTo` relationship to automatically retrieve options from. The `titleAttribute` is the name of a column that will be used to generate a label for each option:

```php
use Filament\Forms\Components\Select;

Select::make('author_id')
    ->relationship(relationshipName: 'author', titleAttribute: 'name')
```

The `multiple()` method may be used in combination with `relationship()` to use a `BelongsToMany` relationship. Filament will load the options from the relationship, and save them back to the relationship's pivot table when the form is submitted. If a `relationshipName` is not provided, Filament will use the field name as the relationship name:

```php
use Filament\Forms\Components\Select;

Select::make('technologies')
    ->multiple()
    ->relationship(titleAttribute: 'name')
```

### Searching relationship options across more than one column

By default, if the select is also searchable, Filament will return search results for the relationship based on title column of the relationship. If you'd like to search across more than one column, you can pass an array of columns to the `searchable()` method:

```php
use Filament\Forms\Components\Select;

Select::make('author_id')
    ->relationship(relationshipName: 'author', titleAttribute: 'name')
    ->searchable(['name', 'email'])
```

### Preloading relationship options

If you'd like to populate the searchable options from the database when the page is loaded, instead of when the user searches, you can use the `preload()` method:

```php
use Filament\Forms\Components\Select;

Select::make('author_id')
    ->relationship(relationshipName: 'author', titleAttribute: 'name')
    ->searchable()
    ->preload()
```

### Customizing the relationship query

You may customize the database query that retrieves options using the third parameter of the `relationship()` method:

```php
use Filament\Forms\Components\Select;
use Illuminate\Database\Eloquent\Builder;

Select::make('author_id')
    ->relationship(
        relationshipName: 'author',
        titleAttribute: 'name',
        modifyOptionsQueryUsing: fn (Builder $query) => $query->withTrashed(),
    )
```

### Customizing the relationship option labels

If you'd like to customize the label of each option, maybe to be more descriptive, or to concatenate a first and last name, you could use a virtual column in your database migration:

```php
$table->string('full_name')->virtualAs('concat(first_name, \' \', last_name)');
```

```php
use Filament\Forms\Components\Select;

Select::make('author_id')
    ->relationship(relationshipName: 'author', titleAttribute: 'full_name')
```

Alternatively, you can use the `getOptionLabelFromRecordUsing()` method to transform an option's Eloquent model into a label:

```php
use Filament\Forms\Components\Select;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

Select::make('author_id')
    ->relationship(
        relationshipName: 'author',
        modifyOptionsQueryUsing: fn (Builder $query) => $query->orderBy('first_name')->orderBy('last_name'),
    )
    ->getOptionLabelFromRecordUsing(fn (Model $record) => "{$record->first_name} {$record->last_name}")
    ->searchable(['first_name', 'last_name'])
```

### Creating a new option in a modal

You may define a custom form that can be used to create a new record and attach it to the `BelongsTo` relationship:

```php
use Filament\Forms\Components\Select;
use Illuminate\Database\Eloquent\Model;

Select::make('author_id')
    ->relationship(relationshipName: 'author', titleAttribute: 'name')
    ->createOptionForm([
        Forms\Components\TextInput::make('name')
            ->required(),
        Forms\Components\TextInput::make('email')
            ->required()
            ->email(),
    ]),
```

The form opens in a modal, where the user can fill it with data. Upon form submission, the new record is selected by the field.

### Editing the selected option in a modal

You may define a custom form that can be used to edit the selected record and save it back to the `BelongsTo` relationship:

```php
use Filament\Forms\Components\Select;
use Illuminate\Database\Eloquent\Model;

Select::make('author_id')
    ->relationship(relationshipName: 'author', titleAttribute: 'name')
    ->editOptionAction([
        Forms\Components\TextInput::make('name')
            ->required(),
        Forms\Components\TextInput::make('email')
            ->required()
            ->email(),
    ]),
```

The form opens in a modal, where the user can fill it with data. Upon form submission, the data from the form is saved back to the record.

### Handling `MorphTo` relationships

`MorphTo` relationships are special, since they give the user the ability to select records from a range of different models. Because of this, we have a dedicated `MorphToSelect` component which is not actually a select field, rather 2 select fields inside a fieldset. The first select field allows you to select the type, and the second allows you to select the record of that type.

To use the `MorphToSelect`, you must pass `types()` into the component, which tell it how to render options for different types:

```php
use Filament\Forms\Components\MorphToSelect;

MorphToSelect::make('commentable')
    ->types([
        MorphToSelect\Type::make(Product::class)
            ->titleAttribute('name'),
        MorphToSelect\Type::make(Post::class)
            ->titleAttribute('title'),
    ])
```

### Customizing the option labels for each morphed type

The `titleAttribute()` is used to extract the titles out of each product or post. If you'd like to customize the label of each option, you can use the `getOptionLabelFromRecordUsing()` method to transform the Eloquent model into a label:

```php
use Filament\Forms\Components\MorphToSelect;

MorphToSelect::make('commentable')
    ->types([
        MorphToSelect\Type::make(Product::class)
            ->getOptionLabelFromRecordUsing(fn (Product $record): string => "{$record->name} - {$record->slug}"),
        MorphToSelect\Type::make(Post::class)
            ->titleAttribute('title'),
    ])
```

### Customizing the relationship query for each morphed type

You may customize the database query that retrieves options using the `modifyOptionsQueryUsing()` method:

```php
use Filament\Forms\Components\MorphToSelect;
use Illuminate\Database\Eloquent\Builder;

MorphToSelect::make('commentable')
    ->types([
        MorphToSelect\Type::make(Product::class)
            ->titleAttribute('name')
            ->modifyOptionsQueryUsing(fn (Builder $query) => $query->whereBelongsTo($this->team)),
        MorphToSelect\Type::make(Post::class)
            ->titleAttribute('title')
            ->modifyOptionsQueryUsing(fn (Builder $query) => $query->whereBelongsTo($this->team)),
    ])
```

> Many of the same options in the select field are available for `MorphToSelect`, including `searchable()`, `preload()`, `allowHtml()`, and `optionsLimit()`.

## Allowing HTML in the option labels

By default, Filament will escape any HTML in the option labels. If you'd like to allow HTML, you can use the `allowHtml()` method:

```php
use Filament\Forms\Components\Select;

Select::make('technology')
    ->options([
        'tailwind' => '<span class="text-blue-500">Tailwind</span>',
        'alpine' => '<span class="text-green-500">Alpine</span>',
        'laravel' => '<span class="text-red-500">Laravel</span>',
        'livewire' => '<span class="text-pink-500">Livewire</span>',
    ])
    ->searchable()
    ->allowHtml()
```

Be aware that you will need to ensure that the HTML is safe to render, otherwise your application will be vulnerable to XSS attacks.

## Disable placeholder selection

You can prevent the placeholder (null option) from being selected using the `disablePlaceholderSelection()` method:

```php
use Filament\Forms\Components\Select;

Select::make('status')
    ->options([
        'draft' => 'Draft',
        'reviewing' => 'Reviewing',
        'published' => 'Published',
    ])
    ->default('draft')
    ->disablePlaceholderSelection()
```

## Disabling specific options

You can disable specific options using the `disableOptionWhen()` method. It accepts a closure, in which you can check if the option with a specific `$value` should be disabled:

```php
use Filament\Forms\Components\Select;

Select::make('status')
    ->options([
        'draft' => 'Draft',
        'reviewing' => 'Reviewing',
        'published' => 'Published',
    ])
    ->default('draft')
    ->disableOptionWhen(fn (string $value): bool => $value === 'published')
```

## Adding affix text aside the field

You may place text before and after the input using the `prefix()` and `suffix()` methods:

```php
use Filament\Forms\Components\Select;

Select::make('domain')
    ->prefix('https://')
    ->suffix('.com')
```

### Using icons as affixes

You may place an [icon](https://blade-ui-kit.com/blade-icons?set=1#search) before and after the input using the `prefixIcon()` and `suffixIcon()` methods:

```php
use Filament\Forms\Components\Select;

Select::make('domain')
    ->prefixIcon('heroicon-m-arrow-top-right-on-square')
    ->suffixIcon('heroicon-m-arrow-top-right-on-square')
```

## Setting a custom loading message

When you're using a searchable select or multi-select, you may want to display a custom message while the options are loading. You can do this using the `loadingMessage()` method:

```php
use Filament\Forms\Components\Select;

Select::make('author_id')
    ->relationship(relationshipName: 'author', titleAttribute: 'name')
    ->searchable()
    ->loadingMessage('Loading authors...')
```

## Setting a custom no search results message

When you're using a searchable select or multi-select, you may want to display a custom message when no search results are found. You can do this using the `noSearchResultsMessage()` method:

```php
use Filament\Forms\Components\Select;

Select::make('author_id')
    ->relationship(relationshipName: 'author', titleAttribute: 'name')
    ->searchable()
    ->noSearchResultsMessage('No authors found.')
```

## Setting a custom search prompt

When you're using a searchable select or multi-select, you may want to display a custom message when the user has not yet entered a search term. You can do this using the `searchPrompt()` method:

```php
use Filament\Forms\Components\Select;

Select::make('author_id')
    ->relationship(relationshipName: 'author', titleAttribute: 'name')
    ->searchable(['name', 'email'])
    ->searchPrompt('Search authors by their name or email address')
```

## Setting a custom searching message

When you're using a searchable select or multi-select, you may want to display a custom message while the search results are being loaded. You can do this using the `searchingMessage()` method:

```php
use Filament\Forms\Components\Select;

Select::make('author_id')
    ->relationship(relationshipName: 'author', titleAttribute: 'name')
    ->searchable()
    ->searchingMessage('Searching authors...')
```

## Tweaking the search debounce

By default, Filament will wait 1000 milliseconds (1 second) before searching for options when the user types in a searchable select or multi-select. It will also wait 1000 milliseconds between searches, if the user is continuously typing into the search input. You can change this using the `searchDebounce()` method:

```php
use Filament\Forms\Components\Select;

Select::make('author_id')
    ->relationship(relationshipName: 'author', titleAttribute: 'name')
    ->searchable()
    ->searchDebounce(500)
```

Ensure that you are not lowering the debounce too much, as this may cause the select to become slow and unresponsive due to a high number of network requests to retrieve options from server.

## Limiting the number of options

You can limit the number of options that are displayed in a searchable select or multi-select using the `optionsLimit()` method. The default is 50:

```php
use Filament\Forms\Components\Select;

Select::make('author_id')
    ->relationship(relationshipName: 'author', titleAttribute: 'name')
    ->searchable()
    ->optionsLimit(20)
```

Ensure that you are not raising the limit too high, as this may cause the select to become slow and unresponsive due to high in-browser memory usage.

## Select validation

As well as all rules listed on the [validation](../validation) page, there are additional rules that are specific to selects.

### Selected items validation

You can validate the minimum and maximum number of items that you can select in a [multi-select](#multi-select) by setting the `minItems()` and `maxItems()` methods:

```php
use Filament\Forms\Components\Select;

Select::make('technologies')
    ->multiple()
    ->options([
        'tailwind' => 'Tailwind CSS',
        'alpine' => 'Alpine.js',
        'laravel' => 'Laravel',
        'livewire' => 'Laravel Livewire',
    ])
    ->minItems(1)
    ->maxItems(3)
```