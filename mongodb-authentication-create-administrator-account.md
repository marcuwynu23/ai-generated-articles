CREATE ADMINISTRATOR USER MONGODB SERVER AUTHENTICATION
# use admin database
use admin

# create User administrator with password and roles root for admin database
```js
db.createUser(
  {
    user: "administrator",
    pwd:  "<your administrator password here>",
    roles: [ { role: "root", db: "admin" } ]
  }
)
```



