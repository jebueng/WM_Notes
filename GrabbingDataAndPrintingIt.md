orgIds = Listing.where(wmid: listing_ids).where('package_level > ?', 0).where.not(organization: nil).group(:organization_id).pluck(:organization_id)

```ruby
orgIds_map = orgIds.map do |id|
    {
        "name": "orgId",
        "value": "#{id}",
        "match_type": "exact",
        "type": "custom_attribute"
    }
end
```

f = File.new(Rails.root + 'org_ids_with_package.json', 'w')
  f << JSON.pretty_generate(orgIds_map.as_json)
f.close

usernames_w_package = Listing.where(wmid: listing_ids).where('package_level > ?', 0).where(organization: nil).joins(:user).pluck(:username)

```ruby
usernames_w_package_map = listing_users_uniq.map do |user|
    {
        "name": "username",
        "value": "#{user.first}",
        "match_type": "exact",
        "type": "custom_attribute"
    }
end
```

f = File.new(Rails.root + 'listing_users_optimizely_uniq.json', 'w')
  f << JSON.pretty_generate(listing_users_uniq_map.as_json)
f.close

  usernames = Listing.where(wmid: listing_ids).where('package_level <> ?', 0).where(organization: nil).joins(:user).pluck(:username).uniq.map do |username|
    {
      "name": "username",
      "value": "#{username}",
      "match_type": "exact",
      "type": "custom_attribute"
    }
  end
  
  f = File.new(Rails.root + 'usernames.json', 'w')
  f << JSON.pretty_generate(usernames.as_json)  
  f.close
  
  ### Doing it with PSQL
  ```sql
    -- save wmids to file
  /connect insights
  COPY (
    SELECT "insights"."homepage_analytics"."wmid"
    FROM "insights"."homepage_analytics"
    WHERE (report_date = '2021-12-13' AND roas_last_7 < 3.0)
  ) TO '/tmp/query.csv' (format csv, delimiter ';')

  /connect weedmaps
  -- copy results into a temp table
  CREATE TABLE csv(wmid integer);
  -- load in csv data
  COPY csv(wmid) FROM '/tmp/query.csv' DELIMITER ',';

  -- run query to generate json
  SELECT
    json_build_object(
        'name', 'orgId',
        'a', json_agg(listings.organization_id),
        'match_type', 'exact',
        'type', 'custom_attribute'
    )
  FROM "listings"
  WHERE wmid in (SELECT wmid from csv)
    AND (package_level > 0) AND "listings"."organization_id" IS NOT NULL
  GROUP BY "listings"."organization_id"

  -- clean up table
  DROP TABLE csv;
  ```
