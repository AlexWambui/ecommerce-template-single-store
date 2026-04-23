# Laravel Views

Show product categories

```php
@foreach($product->categories as $category)
    <span class="badge bg-secondary">
        {{ $category->title }}
        @if($category->pivot->is_primary)
            <span class="text-warning">★</span>
        @endif
    </span>
@endforeach
```

Show category breadcrumb

```php
<nav aria-label="breadcrumb">
    <ol class="breadcrumb">
        @foreach($category->breadcrumb as $crumb)
            @if($loop->last)
                <li class="breadcrumb-item active">{{ $crumb->title }}</li>
            @else
                <li class="breadcrumb-item">
                    <a href="{{ route('categories.show', $crumb) }}">{{ $crumb->title }}</a>
                </li>
            @endif
        @endforeach
    </ol>
</nav>
```

Category tree for forms

```php
<select name="category_ids[]" multiple class="form-select">
    @foreach($categories as $category)
        <option value="{{ $category->id }}" 
                @if(in_array($category->id, $selectedCategories)) selected @endif>
            {{ str_repeat('—', $category->depth) }} {{ $category->title }}
        </option>
    @endforeach
</select>
```
