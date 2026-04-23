# Laravel Models

## Product

How it works in practice

- Admin selects multiple categories.
- Admin chooses one primary category.
- Controller validates and calls syncCategories().
- Pivot table updates automatically.
- $product->primaryCategory now always returns the main category.
- $product->categories returns all categories.

```php
class ProductCategory extends Model
{
    protected $guarded = [];

    // Relationships
    public function products()
    {
        return $this->belongsToMany(Product::class, 'category_product')
                    ->withPivot('is_primary', 'sort_order')
                    ->withTimestamps();
    }

    public function parent()
    {
        return $this->belongsTo(ProductCategory::class, 'parent_id');
    }

    public function children()
    {
        return $this->hasMany(ProductCategory::class, 'parent_id');
    }

    // Recursive method to get all ancestors (for breadcrumbs)
    public function ancestors()
    {
        $ancestors = collect();
        $current = $this;
        while ($current->parent) {
            $current = $current->parent;
            $ancestors->push($current);
        }
        return $ancestors->reverse();
    }
}



class Product extends Model
{
    protected $guarded = [];

    // Relationships
    public function categories()
    {
        return $this->belongsToMany(ProductCategory::class, 'category_product')
                    ->withPivot('is_primary', 'sort_order')
                    ->withTimestamps();
    }

    public function primaryCategory()
    {
        return $this->belongsToMany(ProductCategory::class, 'category_product')
                    ->wherePivot('is_primary', true);
    }

    // Accessor for easy usage
    // Accessing $product->primaryCategory returns the main category.
    public function getPrimaryCategoryAttribute()
    {
        return $this->primaryCategory()->first();
    }

    // Methods to manage categories
    // Handles both multiple categories and primary category.
    public function syncCategories(array $categoryIds, $primaryCategoryId = null)
    {
        // Sync all categories
        $this->categories()->sync($categoryIds);

        // Set primary category
        if ($primaryCategoryId && in_array($primaryCategoryId, $categoryIds)) {
            $this->categories()->updateExistingPivot($categoryIds, ['is_primary' => false]);
            $this->categories()->updateExistingPivot($primaryCategoryId, ['is_primary' => true]);
        } elseif (!empty($categoryIds)) {
            $this->categories()->updateExistingPivot($categoryIds[0], ['is_primary' => true]);
        }
    }
}
```
