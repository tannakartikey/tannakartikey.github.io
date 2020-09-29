---
layout: post
title:  "Many-to-many self join in Rails"
tags: rails
---

Let's say we have "products" and we want to prepare "kits" of those products. Kits are nothing but the group of the products.  

We can use many-to-many relationship here because products can be in many kits, and kits can be associated with many products.  

Also, since the kits are just grouping the products, we can use self-joins. There are multiple ways we can implement self-joins.

### Using has\_and\_belongs\_to\_many

You can read more about [has_and_belongs_to_many](https://guides.rubyonrails.org/association_basics.html#the-has-and-belongs-to-many-association){:target="_blank"} on Rails docs.
#### Migration
```ruby
class CreateJoinTableProductKits < ActiveRecord::Migration[6.0]
  def change
    create_table :product_kits, id: false do |t|
      t.references :product, null: false, foreign_key: true, index: false
      t.references :kit, null: false, foreign_key: { to_table: :products }, index: false
      t.index [:product_id, :kit_id], unique: true
    end
  end
end
```

#### Model
```ruby
class Product < ApplicationRecord
  has_and_belongs_to_many :kits,
    join_table: :product_kits,
    class_name: 'Product',
    association_foreign_key: 'kit_id'
end
```

### Using has\_many through
This approach is better because later on in your project you can add more fields and validations in `ProductKit` model.
As you know, our projects are always dynamic and most of the time(all the time) we end up modifying the flow. So, it is
better to be prepared and use `has_many :through` from the beginning.

More on, [`has_many :through`](https://guides.rubyonrails.org/association_basics.html#the-has-many-through-association){:target="_blank"} on Rails docs.

#### Migration
```ruby
class CreateJoinTableProductKits < ActiveRecord::Migration[6.0]
  def change
    create_table :product_kits do |t|
      t.references :product, null: false, foreign_key: true, index: false
      t.references :kit, null: false, foreign_key: { to_table: :products }, index: false
      t.index [:product_id, :kit_id], unique: true

      t.timestamps
    end
  end
end
```

#### Model `app/models/product.rb`
```ruby
class Product < ApplicationRecord
  has_many :product_kits, dependent: :destroy
  has_many :kits, through: :product_kits
end
```

#### Model `app/models/product_kit.rb`
```ruby
class ProductKit < ApplicationRecord
  belongs_to :product
  belongs_to :kit, class_name: 'Product'
end
```

