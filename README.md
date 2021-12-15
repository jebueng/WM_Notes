# WM_Notes
Notes collected while developing on the platform 

## Recent jots

### Console
 `grep -R --exclude-dir=node_modules "string" ./`
 
 Use that command to grep the whole codebase and exclude certain directories.
 
 
 ### Rails
 `safety assured` migration block is a no no. Works oppose you think.
 
 `change_column_default` apparently works after the fact? 
 
 `change_column_null` also disables columns from being initialized null?
 
#### Seeding Databases
When in doubt, `make clean` and then leverage `rake dev:prime` to set up the databases. 

### Connecting to cloud environments

```shell
#this opens up teleport
wmtsh weedmaps acceptance

# this will get a list 
kubectl config get-contexts

# using AUTHINFO, pick the right id then
kubectl config use-context teleport.internal-weedmaps.com-eks-weedmaps-acceptance

# this will list pods
kubectl -n core get pods

# choose the one with UTILITY
kubectl -n core exec -ti core-utility-798cfccbb4-jjqtx -- /vault/vault-env bundle exec rails c
```

#### Requesting on PROD
```sh
wmtsh weedmaps production prod-edit 
```
You will need to get approved. after approval, need to grab the `request-id` from the response. 

Then log in with 
```sh
tsh login --request-id=e615c7df-66f3-4483-be61-5da8b6480c28
```

From there you can proceed normally as above. 


#### VPN Stability issues
It's normal behavior for VPN to be inconsistent thus far. Keep retrying before esculating. 

### React Admin
Had to not use the `${}` when configuring `~/.npmrc` 
