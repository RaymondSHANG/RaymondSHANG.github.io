---
layout: post
title: "config of a xamp or xapp server"
subtitle: "manually"
date: 2022-11-08 16:27:44
header-style: text
catalog: true
author: "Yuan"
tags: [Apache,SQL, MySQL, Postgresql, PHP]
---
{% include linksref.html %}

I have used apache/tomcat-mysql-php many years ago. Recently, I need to configure my own local database management system, to manage EHR datasets based on OMOP schema. So, I picked this system up.

The two key config files are httpd.conf in Apache and php.in in PHP. I did not use the mature wamp because I'm not sure which database I would use finally, and I have much more flexibility if I configed by hand. And most importantly, it's easy.

# Step-by-Step configs in windows
- Check vs distributions
1. Go to control panel. ...
2. Now under Programs click uninstall a program.
3. Now you can see all the list of programs installed in your PC.
4. Now find Microsoft Visual C++ in the list.
5. In the last column you can see the version of the program.

You may need to update/install Visual C++ redistributables to the latest version or at least 2019 version.

- Get Apache and config
1. Get [2.4.54](https://www.apachelounge.com/download/VS16/binaries/httpd-2.4.54-win64-VS16.zip) version from https://www.apachelounge.com/download/
2. Extract to C:\Web
3. Config C:\Web\Apache24\conf\httpd.conf
    a. Define SRVROOT "c:\Web\Apache24"  #(Around line 37)
    b. ServerAdmin admin@localhost    #(Around line 218
    c. ServerName localhost:80    #(Around line 227)
    d. Save
4. Check Apache
    a. Run 'cmd' as adminstrator
    b. in cmd terminal, run 
    ```bash
    C:/Web/Apache24/bin/httpd.exe -S
    ```
    Something like below:
    VirtualHost configuration:
    ServerRoot: "C:/Web/Apache24"
    Main DocumentRoot: "C:/Web/Apache24/htdocs"
    Main ErrorLog: "C:/Web/Apache24/logs/error.log"
    Mutex default: dir="C:/Web/Apache24/logs/" mechanism=default
    PidFile: "C:/Web/Apache24/logs/httpd.pid"
    Define: DUMP_VHOSTS
    Define: DUMP_RUN_CFG
    Define: SRVROOT=c:\Web\Apache24
5. Install httpd.exe
    ```bash
    C:/Web/Apache24/bin/httpd.exe -k install
    ```
    "Allow Access to Private"
6. Start Apache2.4 Service
   Run WIndows "Service" and find "Apache2.4", right click and 'Start'
   Open "Chrome", and go to "localhost", should come out "It works"
   
- Get Postgresql
If Postgresql is installed. Just ignore this part.
Some helpful hints to specify your data directory and register your database to windows 'Service':
```bash
makdir yourpgsql\data
initdb -D yourpgsql\data -U postgres
pg_ctl register -N "PostgreSQL-x64-15" yourpgsql\data
psql -U postgres
```

- Get and config php
1. Download php8.1.12, extract to C:\web\php-8.1.12
2. Go to C:\web\php-8.1.12, copy "php.ini-development" to "php.ini"
3. Edit C:\web\Apache2.4\config\httpd.conf, appending:

```
LoadModule php_module "C:\Web\php-8.1.12\php8apache2_4.dll"
AddHandler application/x-httpd-php .php
PHPIniDir "C:\Web\php-8.1.12"
LoadFile "C:\Web\php-8.1.12\libpq.dll"
```

   And Change line 285 to:
   
```
    DirectoryIndex index.html index.php
```
 Save,check by runing "C:\Web\Apache24\bin\httpd.exe -S" and restart apache service

- Test hello.php
1. write a helloworld.php and save it to C:\web\Apache24\htdocs
2. in your brower, check: "localhost/helloworld.php", should show conents in your helloworld.php

- Config postgresql
config php.in, 

line763, uncomment

```
extension_dir = "C:\Web\php-8.1.12\ext"
```

line937 and line939, uncomment:

```
extension=pdo_pgsql
extension=pgsql
```

restart service httpd and test!

-dbconnectsample.php
```php
<?php
$conn = pg_connect("host=localhost port=5432 user=xxx@localhost password=xxxx123 dbname=dbtest");
if (!$conn){
    die("PostgreSQL failed");
}
echo "PostgreSQL connected";
?>
```

# Config for MySQL
1. Download MySQL
2. ConfigMySQL
   - Config my.ini, and put it to C:\Web\mysql-8.0.31-winx64\my.ini
    ```
    [mysqld]
    # set basedir to your installation path
    basedir=C:/Web/mysql-8.0.31-winx64
    # set datadir to the location of your data directory
    datadir=C:/Web/mysql-8.0.31-winx64/data
    # set table name case sensitive
    lower_case_table_names=2
    ```

3. Start MySQL
    ```
    #Init mysql
    .\mysqld.exe --defaults-file=C:\Web\mysql-8.0.31-winx64\my.ini --initialize --console
    #a password for root will be generated, need to write it down!

    #start MySQL service
    .\mysqld.exe --install MySQL --defaults-file=C:\Web\mysql-8.0.31-winx64\my.ini
    ```
4. Reset root password
   ```
   #login sql
   mysql.exe -u root -passwordgeneratedabove
   ```

    ```sql
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'newpassword';
    flush privileges;
    exit;
    ```

5. Import Data
   
   ```bash
   mysql.exe -u root -p -h localhost < C:/Web/Apache24/htdocs/sql/mydb_schema.sql
   #Or
   mysql.exe -u username -p schemaname -h localhost < C:/Web/Apache24/htdocs/sql/mydb_schema.sql
   ```

6. Some useful MySQL commands:
   ```sql
   show databases;
   use db;
   show tables;
   DESCRIBE usertable;
   ```
7. Test MySQL connectsion
   - conmsql.php
    ```php
    // Connect
    //DB_HOST,DB_PORT,DB_USER,DB_PASS,DB_SCHEMA,DB_PORT
    define('DB_HOST', "localhost");
    define('DB_PORT', "3306");
    define('DB_USER', "username");
    define('DB_PASS', "password");
    define('DB_SCHEMA', "dbname");

    $db = mysqli_connect(DB_HOST, DB_USER, DB_PASS, DB_SCHEMA, DB_PORT);

    if (mysqli_connect_errno())
    {
        echo "Failed to connect to MySQL: " . mysqli_connect_error() . NEWLINE;
        echo "Running on: ". DB_HOST . ":". DB_PORT . '<br>' . "Username: " . DB_USER . '<br>' . "Password: " . DB_PASS . '<br>' ."Database: " . DB_SCHEMA;
        phpinfo();   //unsafe, but verbose for learning. 
        exit();
    }
    echo "connect success";
    // Query
    $query = sprintf("SELECT * FROM users WHERE user='%s' AND password='%s'",
                mysql_real_escape_string($user),
                mysql_real_escape_string($password));
    $result = mysqli_query($db, $query);
    $count = mysqli_num_rows($result);
    if (!empty($result) && ($count > 0) ) {
        echo "do somethin";
        while($row = mysqli_fetch_arrays($result)){
            echo "do somethin";
        }
    }
    // Insert
    $query = "INSERT INTO tablename (email, username, date_insertion) " .
				 "VALUES ('{$_SESSION['email']}', '$username',  NOW())";
    $queryID = mysqli_query($db, $query);
    
    if ($queryID  == False) {  //INSERT, UPDATE, DELETE, DROP return True on Success  / False on Error
        //insertion fail
        array_push($error_msg, "INSERT ERROR: xxxxx! <br>" . __FILE__ ." line:". __LINE__ );
    } 
    // Update
    $email = mysqli_real_escape_string($db, $_GET['username']);

	$query = "UPDATE tablename " .
			 "SET date_insertion = NOW() " .
			 "WHERE email = '{$_SESSION['email']}' " .
			 "AND username = '$username'";

	$result = mysqli_query($db, $query);

    if (mysqli_affected_rows($db) == -1) {
        //update fail
        array_push($error_msg,  "UPDATE ERROR: tablename ... <br>".  __FILE__ ." line:". __LINE__ );
	}
    // Delete
    $email = mysqli_real_escape_string($db, $_GET['email']);

	$query = "DELETE FROM tablename " .
			 "WHERE email = '$email'";
    
	$result = mysqli_query($db, $query);
    if (mysqli_affected_rows($db) == -1) {
        //deletion fail
        array_push($error_msg,  "DELETE ERROR: tablename".$email."...<br>" . __FILE__ ." line:". __LINE__ );
	}
    // A very useful function
    $showQueries=True;
    $showCounts=True;
    $query_msg=[];
    define('NEWLINE',"<br/>");
    if($showQueries){
        if(is_bool($result)) {
            array_push($query_msg,  $query . ';' . NEWLINE);
            
            if( mysqli_errno($db) > 0 ) {
                array_push($error_msg,  'Error# '. mysqli_errno($db) . ": " . mysqli_error($db));
            }
        } else {          
            if($showCounts){
            array_push($query_msg,  $query . ';');
            array_push($query_msg,  "Result Set Count: ". mysqli_num_rows($result). NEWLINE);
            } else {
                array_push($query_msg,  $query . ';'. NEWLINE);
            }
        } 
    }
    ```
---
