# DB DESIGN

```json
users {
    id();
    uuid('uuid')->unique();
    string('first_name');
    string('last_name');
    string('email')->unique();
    string('phone_number')->nullable();
    string('secondary_phone_number')->nullable();
    unsignedTinyInteger('role')->default(3);
    boolean('status')->default(true);
    string('image')->nullable();
    date('date_of_birth')->nullable();
    string('gender')->nullable();
    string('timezone')->nullable(); 
    string('locale')->default('en_US');
    timestamp('email_verified_at')->nullable();
    string('password');
    rememberToken();
    timestamps();
}

customer_addresses { 
    id(); 
    uuid('uuid')->unique();
    string('city'); 
    string('state'); 
    string('postcode'); 
    string('country'); 
    string('phone');
    boolean('is_default')->default(false); 
    string('address_type'); // billing, shipping 
    foreignId('user_id')->constrained('users')->cascadeOnDelete(); 
    timestamps(); 
}

admin_logs {
    id();
    uuid('uuid')->unique();
    foreignId('admin_id')->nullable()->constrained('users');
    string('action'); // e.g., "Product Update", "Delete User"
    string('ip_address')->nullable();
    text('description')->nullable();
    timestamps();
}

contact_form_messages {
    id();
    uuid('uuid')->unique();
    string('name');
    string('email');
    string('phone_number');
    string('message', 2000);
    string('response', 2000)->nullable();
    string('notes')->nullable();
    boolean('is_read')->default(0);
    boolean('is_important')->default(0);
    timestamps();
}

blog_categories {
    id();
    uuid('uuid')->unique();
    string('title')->unique();
    string('slug')->unique();
    string('description')->nullable();
    string('image')->nullable();
    timestamps();
}

blogs {
    id();
    uuid('uuid')->unique();
    string('title')->unique();
    string('slug')->unique();
    text('content')->nullable();
    text('excerpt')->nullable(); // Short summary for cards, lists, SEO
    string('image')->nullable();
    json('tags')->nullable();

    $table->boolean('is_featured')->default(false);
    boolean('is_published')->default(true);
    dateTime('published_at')->nullable();
    boolean('noindex')->default(false);
    boolean('nofollow')->default(false);

    $table->unsignedInteger('reading_time')->nullable();
    unsignedInteger('word_count')->nullable();

    $table->string('meta_title')->nullable(); // < 60 chars
    string('meta_description', 500)->nullable(); // < 155 chars
    string('meta_keywords')->nullable();
    string('canonical_url')->nullable();
    json('meta_tags')->nullable();
    json('og_tags')->nullable();
    json('structured_data')->nullable(); // to store dynamic BlogPosting schema or other types, then render directly in Blade views.

    foreignId('blog_category_id')->nullable()->constrained('blog_categories')->nullOnDelete();
    foreignId('author_id')->nullable()->constrained('users')->nullOnDelete();
    timestamps();
}

blog_comments {
    id();
    uuid('uuid')->unique();
    text('content');
    boolean('is_visible')->default(true);

    foreignId('user_id')->constrained('users')->cascadeOnDelete();
    foreignId('blog_id')->constrained('blogs')->cascadeOnDelete();
    timestamps();
}

delivery_regions {
    id();
    uuid('uuid')->unique();
    string('name')->unique();
    string('slug')->unique();
    string('country')->default('KE');
    timestamps();
}

delivery_areas {
    id();
    uuid('uuid')->unique();
    string('name')->unique();
    string('slug')->unique();
    decimal('delivery_fee', 10, 2)->default(0.00);
    string('postal_code')->nullable();
    json('coordinates')->nullable();

    foreignId('delivery_region_id')->constrained('delivery_regions')->cascadeOnDelete();
    timestamps();
}

delivery_methods {
    id();
    uuid('uuid')->unique();
    string('name')->unique();
    decimal('base_price', 10, 2);
    unsignedTinyInteger('estimated_days')->nullable();
    timestamps();
}

delivery_area_delivery_methods {
    id();
    uuid('uuid')->unique();
    foreignId('delivery_area_id')->constrained('delivery_areas')->cascadeOnDelete();
    foreignId('delivery_method_id')->constrained('delivery_methods')->cascadeOnDelete();
    decimal('custom_price', 10, 2)->nullable();

    unique(['delivery_area_id', 'delivery_method_id']);
    timestamps();
}

product_categories {
    id();
    uuid('uuid')->unique();
    string('title')->unique();
    string('slug')->unique();
    string('description')->nullable();
    string('image')->nullable();
    unsignedSmallInteger('sort_order')->default(0);
    foreignId('parent_id')->nullable()->constrained('product_categories')->nullOnDelete(); // For category hierachical tree
    timestamps();
}

products {
    id();
    uuid('uuid')->unique();
    string('title')->unique();
    string('slug')->unique();
    string('product_code')->nullable();
    string('sku')->unique()->nullable();
    string('barcode')->nullable();
    boolean('is_featured')->default(false);
    boolean('is_visible')->default(true);
    decimal('production_cost', 10, 2)->default(0.00);
    decimal('selling_price', 10, 2)->default(0.00);
    decimal('discount_price', 10, 2)->default(0.00)->nullable();
    unsignedTinyInteger('discount_percentage')->default(0)->nullable();
    unsignedSmallInteger('product_measurement')->nullable();
    string('measurement_unit')->nullable();
    boolean('track_inventory')->default(true);
    unsignedSmallInteger('stock_count')->default(0)->nullable();
    unsignedSmallInteger('safety_stock')->default(0)->nullable();
    text('description')->nullable();
    json('attributes')->nullable();
    unsignedSmallInteger('sort_order')->default(0);

    // SEO
    string('meta_title')->nullable();
    string('meta_description', 500)->nullable();
    string('canonical_url')->nullable();
    json('meta_tags')->nullable();
    json('og_tags')->nullable();
    boolean('noindex')->default(false);
    boolean('nofollow')->default(false);

    foreignId('product_category_id')->nullable()->constrained('product_categories')->nullOnDelete();
    timestamps();
}

// Pivot Table (category_product)
// Allows products to have many categories.
// is_primary -> flags one category as the main one.
category_product {
    id();
    foreignId('product_id')->constrained('products')->cascadeOnDelete();
    foreignId('product_category_id')->constrained('product_categories')->cascadeOnDelete();
    boolean('is_primary')->default(false);
    unsignedSmallInteger('sort_order')->default(0);
    timestamps();

    unique(['product_id', 'product_category_id']);
});

product_variants { 
    id(); 
    uuid('uuid')->unique(); 
    string('sku')->unique(); 
    string('barcode')->nullable();
    decimal('weight', 10, 2)->default(0.00); 
    json('attributes'); // {color: 'red', size: 'XL'} 
    decimal('price', 10, 2); 
    decimal('compare_at_price', 10, 2)->nullable(); 
    integer('stock')->default(0); 
    boolean('is_default')->default(false); 
    foreignId('product_id')->constrained('products')->cascadeOnDelete(); 
    timestamps(); 
}

product_images {
    id();
    uuid('uuid')->unique();
    string('image');
    unsignedSmallInteger('sort_order')->default(5);

    foreignId('product_id')->constrained('products')->cascadeOnDelete();
    timestamps();
}

product_media { 
    id(); 
    foreignId('product_id')->constrained('products')->onDelete('cascade'); 
    enum('type', ['image', 'video'])->default('image'); 
    string('url'); 
    unsignedSmallInteger('sort_order')->default(0); 
    timestamps(); 
}

product_attributes { 
    id(); 
    uuid('uuid')->unique(); 
    string('name'); // Color, Size, etc. string('slug')->unique(); 
    string('type'); // select, color, image, etc. 
    boolean('is_filterable')->default(false); 
    timestamps(); 
}

product_attribute_values { 
    id(); 
    uuid('uuid')->unique(); 
    string('value'); 
    string('color_code')->nullable(); // For color swatches 
    string('image')->nullable(); // For image swatches 
    unsignedSmallInteger('sort_order')->default(0); 
    foreignId('attribute_id')->constrained('product_attributes')->cascadeOnDelete(); timestamps(); 
}

product_attribute_mappings { 
    id(); 
    uuid('uuid')->unique(); 
    foreignId('product_id')->constrained('products')->cascadeOnDelete(); 
    foreignId('attribute_id')->constrained('product_attributes')->cascadeOnDelete(); foreignId('attribute_value_id')->constrained('product_attribute_values')->cascadeOnDelete(); 
    timestamps(); 
}

pricing_rules { 
    id(); 
    foreignId('product_id')->constrained()->onDelete('cascade'); 
    decimal('price', 10, 2); 
    string('customer_group')->nullable(); // e.g., VIP, Wholesaler 
    string('country_code')->nullable(); 
    string('region')->nullable(); 
    timestamps(); 
}

product_reviews {
    id();
    uuid('uuid')->unique();
    unsignedTinyInteger('rating');
    string('review', 1500);
    string('image')->nullable();
    boolean('is_visible')->default(1);
    unsignedMediumInteger('sort_order')->default(0);

    foreignId('user_id')->constrained('users')->cascadeOnDelete();
    foreignId('product_id')->constrained('products')->cascadeOnDelete();
    timestamps();
}

product_views {
    id();
    uuid('uuid')->unique();
    foreignId('product_id')->constrained()->onDelete('cascade');
    foreignId('user_id')->nullable()->constrained('users');
    string('ip_address')->nullable();
    string('user_agent')->nullable();
    timestamp('viewed_at');
}

product_discounts {
    id();
    uuid('uuid')->unique();
    date('starts_at')->nullable();
    date('ends_at')->nullable();

    foreignId('product_id')->constrained('products')->cascadeOnDelete();
    timestamps();
}

discount_codes {
    id();
    uuid('uuid')->unique();
    unsignedTinyInteger('type');
    decimal('value', 10, 2);
    boolean('is_active')->default(true);
    integer('usage_limit')->nullable();
    timestamp('starts_at')->nullable();
    timestamp('ends_at')->nullable();
    string('applies_to')->nullable();
    json('conditions')->nullable();
    timestamps();
}

cart_items {
    id();
    uuid('uuid')->unique();
    unsignedSmallInteger('quantity')->default(1);

    foreignId('product_id')->constrained('products')->onDelete('cascade');
    foreignId('user_id')->constrained('users')->onDelete('cascade');
    unique(['user_id', 'product_id']);
    timestamps();
}

wishlists {
    id();
    uuid('uuid')->unique();
    foreignId('user_id')->constrained()->onDelete('cascade');
    foreignId('product_id')->constrained()->onDelete('cascade');
    timestamps();
}

wishlist_items { 
    id(); 
    uuid('uuid')->unique(); 
    foreignId('wishlist_id')->constrained('wishlists')->cascadeOnDelete(); 
    foreignId('product_id')->constrained('products')->cascadeOnDelete(); 
    timestamps(); 
}

inventory_locations {
    id(); 
    uuid('uuid')->unique(); 
    string('aisle')->nullable(); 
    string('rack')->nullable(); 
    string('shelf')->nullable(); 
    string('bin')->nullable(); 
    foreignId('inventory_id')->constrained('inventory')->cascadeOnDelete(); 
    foreignId('warehouse_id')->constrained('warehouses')->cascadeOnDelete(); 
    timestamps();
}

inventory_movements {
    id(); 
    uuid('uuid')->unique(); 
    string('sku')->unique(); 
    unsignedInteger('quantity')->default(0); 
    unsignedInteger('low_stock_threshold')->default(5); 
    boolean('in_stock')->default(true); 
    boolean('backorder_allowed')->default(false); 
    boolean('stock_managed')->default(true); 
    foreignId('product_id')->constrained('products')->cascadeOnDelete(); 
    timestamps();
}

discount_codes { 
    id(); 
    uuid('uuid')->unique(); 
    string('code')->unique(); 
    string('description')->nullable(); 
    unsignedTinyInteger('type'); // percentage, fixed_amount 
    decimal('value', 10, 2); 
    decimal('min_order_amount')->nullable(); 
    unsignedInteger('usage_limit')->nullable(); 
    unsignedInteger('usage_count')->default(0); 
    dateTime('start_date'); 
    dateTime('end_date'); 
    boolean('is_active')->default(true); 
    boolean('single_use')->default(false); 
    json('applicable_categories')->nullable(); 
    json('applicable_products')->nullable(); 
    timestamps(); 
}

orders {
    id();
    uuid('uuid')->unique();
    string('order_number')->unique();
    unsignedTInyInteger('order_status')->default(0);
    unsignedTinyInteger('order_type');

    decimal('production_cost', 10, 2)->default(0.00);
    decimal('discount_amount', 10, 2)->default(0.00);
    decimal('shipping_amount', 10, 2)->default(0.00);
    decimal('tax_amount', 10, 2)->default(0.00);
    decimal('total_amount', 10, 2)->default(0.00);
    decimal('amount_paid', 10, 2)->default(0.00);
    unsignedTinyInteger('payment_method')->nullable();

    string('currency')->default('KES');
    string('locale')->default('en_US');
    json('billing_address')->nullable();
    json('shipping_address')->nullable();
    dateTime('fulfilled_at')->nullable();
    dateTime('cancelled_at')->nullable();

    string('ip_address')->nullable();
    string('user_agent')->nullable();

    string('customer_note')->nullable(); 
    string('admin_note')->nullable();

    foreignId('discount_code_id')->nullable()->constrained('discount_codes')->nullOnDelete();
    foreignId('user_id')->nullable()->constrained('users')->nullOnDelete();
    timestamps();
}

order_items {
    id();
    uuid('uuid')->unique();
    string('title');
    unsignedSmallInteger('quantity')->default(1);
    decimal('production_cost',10,2)->default(0);
    decimal('selling_price',10,2)->default(0);

    foreignId('order_id')->constrained('sales')->cascadeOnDelete();
    foreignId('product_id')->nullable()->constrained('products')->nullOnDelete();
    timestamps();
}

order_deliveries {
    id();
    uuid('uuid')->unique();
    string('name');
    string('email');
    string('phone_number');
    string('region');
    string('area');
    string('address');
    string('additional_information')->nullalbe();
    decimal('shipping_cost');
    unsignedTinyInteger('delivery_status')->default(0);

    foreignId('order_id')->constrained('orders')->cascadeOnDelete();
    timestamps();
}

order_status_history { 
    id(); 
    uuid('uuid')->unique(); 
    unsignedTinyInteger('status'); 
    text('comment')->nullable(); 
    foreignId('sale_id')->constrained('sales')->cascadeOnDelete(); 
    foreignId('user_id')->nullable()->constrained('users')->nullOnDelete(); 
    timestamps(); 
}

payments {
    id();
    uuid('uuid')->unique();
    unsignedTinyInteger('payment_status')->default(0);
    unsignedTinyInteger('payment_method')->nullable();
    decimal('amount', 10, 2)->nullable();
    string('phone_number')->nullable();
    unsignedTinyInteger('response_code')->nullable();
    string('response_description')->nullable();
    string('merchant_request_id')->nullable();
    string('checkout_request_id')->nullable();
    string('transaction_reference')->nullable();
    text('customer_message');

    foreignId('order_id')->constrained('orders')->cascadeOnDelete();
    timestamps();
}

sale_returns {
    id();
    uuid('uuid')->unique();
    unsignedTinyInteger('status'); // 0 = Requested, 1 = Approved, 2 = Rejected, 3 = Refunded
    string('reason');

    foreignId('order_id')->constrained('orders')->onDelete('cascade');
    timestamps();
}

user_activities { 
    id(); 
    foreignId('user_id')->constrained('users')->onDelete('cascade'); 
    string('action'); // e.g., Viewed Product, Added to Cart 
    string('ip_address')->nullable(); 
    string('user_agent')->nullable(); 
    json('metadata')->nullable(); 
    timestamp('performed_at')->default(now()); 
}

settings {
    id();
    string('company_name');
    string('location');
    string('phone_number');
    string('secondary_phone_number')->nullable();
    string('email');
    string('currency')->default('KES');
    string('logo')->nullable();
    timestamps();
}
```
