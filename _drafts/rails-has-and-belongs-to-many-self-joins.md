#### Many-to-many relationship without a join table.
##### Migration
```ruby
class CreateJoinTableSKUKits < ActiveRecord::Migration[6.0]
  def change
    create_table :sku_kits, id: false do |t|
      t.references :sku, null: false, foreign_key: true, index: false
      t.references :kit, null: false, foreign_key: { to_table: :skus }, index: false
      t.index [:sku_id, :kit_id], unique: true
      t.index [:kit_id, :sku_id]
    end
  end
end
```
```ruby
  has_and_belongs_to_many :kits,
    join_table: :skus,
    class_name: 'SKU',
    foreign_key: 'id',
    association_foreign_key: 'kit_id'
```

#### Many-to-many with help of a join table
##### Migration
```ruby
class AddKitIdToSkus < ActiveRecord::Migration[6.0]
  def change
    add_column :skus, :kit_id, :bigint
  end
end
```

```ruby
  has_and_belongs_to_many :kits,
    join_table: :sku_kits,
    class_name: 'SKU',
    foreign_key: :sku_id,
    association_foreign_key: :kit_id
```

#### Queries

```ruby
irb(main):014:0> SKU.joins(:kits).where(sku_kits: { kit_id: 706786073}).explain
  SKU Load (1.3ms)  SELECT "skus".* FROM "skus" INNER JOIN "sku_kits" ON "sku_kits"."sku_id" = "skus"."id" INNER JOIN "skus" "kits_skus" ON "kits_skus"."id" = "sku_kits"."kit_id" WHERE "sku_kits"."kit_id" = $1  [["kit_id", 706786073]]
=>
EXPLAIN for: SELECT "skus".* FROM "skus" INNER JOIN "sku_kits" ON "sku_kits"."sku_id" = "skus"."id" INNER JOIN "skus" "kits_skus" ON "kits_skus"."id" = "sku_kits"."kit_id" WHERE "sku_kits"."kit_id" = $1 [["kit_id", 706786073]]
                                                     QUERY PLAN
--------------------------------------------------------------------------------------------------------------------
 Nested Loop  (cost=15.03..40.97 rows=9 width=103)
   ->  Index Only Scan using skus_pkey on skus kits_skus  (cost=0.15..8.17 rows=1 width=8)
         Index Cond: (id = '706786073'::bigint)
   ->  Hash Join  (cost=14.88..32.72 rows=9 width=111)
         Hash Cond: (skus.id = sku_kits.sku_id)
         ->  Seq Scan on skus  (cost=0.00..16.20 rows=620 width=103)
         ->  Hash  (cost=14.76..14.76 rows=9 width=16)
               ->  Bitmap Heap Scan on sku_kits  (cost=4.22..14.76 rows=9 width=16)
                     Recheck Cond: (kit_id = '706786073'::bigint)
                     ->  Bitmap Index Scan on index_sku_kits_on_kit_id_and_sku_id  (cost=0.00..4.22 rows=9 width=0)
                           Index Cond: (kit_id = '706786073'::bigint)
(11 rows)
```

```ruby
irb(main):022:0> SKU.first.kits.explain
  SKU Load (1.0ms)  SELECT "skus".* FROM "skus" ORDER BY "skus"."id" ASC LIMIT $1  [["LIMIT", 1]]
  SKU Load (1.1ms)  SELECT "skus".* FROM "skus" INNER JOIN "sku_kits" ON "skus"."id" = "sku_kits"."kit_id" WHERE "sku_kits"."sku_id" = $1  [["sku_id", 218662599]]
=>
EXPLAIN for: SELECT "skus".* FROM "skus" INNER JOIN "sku_kits" ON "skus"."id" = "sku_kits"."kit_id" WHERE "sku_kits"."sku_id" = $1 [["sku_id", 218662599]]
                                                  QUERY PLAN
--------------------------------------------------------------------------------------------------------------
 Hash Join  (cost=14.88..32.72 rows=9 width=103)
   Hash Cond: (skus.id = sku_kits.kit_id)
   ->  Seq Scan on skus  (cost=0.00..16.20 rows=620 width=103)
   ->  Hash  (cost=14.76..14.76 rows=9 width=8)
         ->  Bitmap Heap Scan on sku_kits  (cost=4.22..14.76 rows=9 width=8)
               Recheck Cond: (sku_id = '218662599'::bigint)
               ->  Bitmap Index Scan on index_sku_kits_on_sku_id_and_kit_id  (cost=0.00..4.22 rows=9 width=0)
                     Index Cond: (sku_id = '218662599'::bigint)
(8 rows)
```
