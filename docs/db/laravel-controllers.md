# Controllers

```php
<?php

namespace App\Http\Controllers;

use App\Models\Products\Product;
use App\Models\Products\ProductCategory;

class ProductController extends Controller
{
    // Get products by category (including subcategories)
    public function productsByCategory($categorySlug)
    {
        $category = ProductCategory::where('slug', $categorySlug)->firstOrFail();
        
        // Get all category IDs including descendants
        $categoryIds = $this->getCategoryIdsWithDescendants($category);
        
        // Get products in any of these categories
        $products = Product::whereHas('categories', function($query) use ($categoryIds) {
            $query->whereIn('product_categories.id', $categoryIds);
        })->paginate(12);
        
        return view('products.category', compact('products', 'category'));
    }
    
    // Helper to get all descendant category IDs
    private function getCategoryIdsWithDescendants($category)
    {
        $ids = [$category->id];
        
        foreach ($category->descendants as $descendant) {
            $ids[] = $descendant->id;
        }
        
        return $ids;
    }
    
    // Create product with categories
    public function store(Request $request)
    {
        $product = Product::create($request->all());
        
        // Attach categories
        $categoryIds = $request->input('category_ids', []);
        $primaryCategoryId = $request->input('primary_category_id');
        
        $product->syncCategories($categoryIds, $primaryCategoryId);
        
        return redirect()->route('products.show', $product);
    }
    
    // Update product categories
    public function updateCategories(Request $request, Product $product)
    {
        $categoryIds = $request->input('category_ids', []);
        $primaryCategoryId = $request->input('primary_category_id');
        
        $product->syncCategories($categoryIds, $primaryCategoryId);
        
        return response()->json(['success' => true]);
    }
}


// Example 2 for the controller:
public function store(Request $request)
{
    $data = $request->validate([
        'title' => 'required|string|max:255',
        'category_ids' => 'required|array|min:1',
        'category_ids.*' => 'exists:product_categories,id',
        'primary_category_id' => 'required|exists:product_categories,id',
        // other product fields...
    ]);

    $product = Product::create($data);

    // Sync categories
    $product->syncCategories($data['category_ids'], $data['primary_category_id']);

    return redirect()->route('products.index')->with('success', 'Product created.');
}

public function update(Request $request, Product $product)
{
    $data = $request->validate([
        'title' => 'required|string|max:255',
        'category_ids' => 'required|array|min:1',
        'category_ids.*' => 'exists:product_categories,id',
        'primary_category_id' => 'required|exists:product_categories,id',
        // other product fields...
    ]);

    $product->update($data);

    // Sync categories
    $product->syncCategories($data['category_ids'], $data['primary_category_id']);

    return redirect()->route('products.index')->with('success', 'Product updated.');
}
```
