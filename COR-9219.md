Currently adding migrations with

```ruby
class AddSalesforceColumnsToBrands < ActiveRecord::Migration[6.1]
  def change
    add_column :brands, :point_of_sale_lead, :boolean, default: false 
    add_column :brands, :terms_signed, :boolean
    add_column :brands, :point_of_sale_name, :string
  end
end
```

But we run into 

```ruby
Adding a column with a non-null default blocks reads and writes while the entire table is rewritten.
Instead, add the column without a default value, then change the default.

class AddSalesforceColumnsToBrands < ActiveRecord::Migration[6.1]
  def up
    add_column :brands, :point_of_sale_lead, :boolean
    change_column_default :brands, :point_of_sale_lead, false
  end

  def down
    remove_column :brands, :point_of_sale_lead
  end
end

Then backfill the existing rows in the Rails console or a separate migration with disable_ddl_transaction!.

class BackfillAddSalesforceColumnsToBrands < ActiveRecord::Migration[6.1]
  disable_ddl_transaction!

  def up
    Brand.unscoped.in_batches do |relation| 
      relation.update_all point_of_sale_lead: false
      sleep(0.01)
    end
  end
end
```

## Similar Migrations 
```ruby
class AddEcpmColumnsToRegionAdvertisingSettings < ActiveRecord::Migration[6.0]
  def change
    add_column :region_advertising_settings, :ecpm_optimized, :boolean
    add_column :region_advertising_settings, :ecpm_burn_in_impressions, :integer
    add_column :region_advertising_settings, :ecpm_optimize_days, :integer
    add_column :region_advertising_settings, :ecpm_burn_in_default_price_value, :integer
    add_column :region_advertising_settings, :ecpm_burn_in_default_price_currency, :string

    # Is this the way?
    change_column_default :region_advertising_settings, :ecpm_optimized, from: nil, to: false
  end
end
```

## References 
https://thoughtbot.com/blog/avoid-the-threestate-boolean-problem

https://andycroll.com/ruby/always-force-booleans-to-be-true-or-false/

