cf-wordpress
============

Workpress configuration to make it 'pushable' to Cloud Foundry

#### PaaS Requirements
There has to be a MySQL DB service to bind to.
Check SBaaS providers and plans
```
gcf marketplace
```

Create a service
```
$ cf create-service cleardb spark wordbress-db
# Check everything went well
$ cf service wordbress-db
```

#### Download and unpack wordpress installation
```
curl -O http://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
rm latest.tar.gz
```

#### Configure salt

```
cd wordpress
echo "<?php" > wp-salt.php
curl https://api.wordpress.org/secret-key/1.1/salt/ >> wp-salt.php
```

#### Configure database
Create wp-config.php:

```
<?php
// Fetch config from PaaS
$services = getenv("VCAP_SERVICES");
$services_json = json_decode($services,true);
$mysql_config = $services_json["mysql-5.5"][0]["credentials"];

// ** MySQL settings from resource descriptor ** //
define('DB_NAME', $mysql_config["name"]);
define('DB_USER', $mysql_config["user"]);
define('DB_PASSWORD', $mysql_config["password"]);
define('DB_HOST', $mysql_config["hostname"] . ":" . $mysql_config["port"]);

define('DB_CHARSET', 'utf8');
define('DB_COLLATE', '');
define ('WPLANG', '');
define('WP_DEBUG', false);

require('wp-salt.php');

$table_prefix  = 'wp_';

/* That's all, stop editing! Happy blogging. */

/** Absolute path to the WordPress directory. */
if ( !defined('ABSPATH') )
  define('ABSPATH', dirname(__FILE__) . '/');

/** Sets up WordPress vars and included files. */
require_once(ABSPATH . 'wp-settings.php');
```

#### Push the application to the cloud

```
cf push  --buildpack https://github.com/heroku/heroku-buildpack-php 
# PaaS doesn’t support PHP automatically, but you can use custom buildpack
# choose domain for application
# attach a MySQL service
```
