# PHP PDO MySQL Cheat sheet guide

A guide on the basics for using PDO PHP for MySQL with pre-prepared statements.

# Table of Contents

1. [Creating connection](#connection)

   1a [inline](#connection)

   1b [function](#connectionfunc)

   1c [class example](#classexample)

2. [SELECT queries](#select)

   2a [Loop](#loop)

   2b [One row](#onerow)

   2c [One column](#onecolumn)

   2d [Count](#count)

   2e [If exists](#ifexists)

3. [INSERT queries](#insert)

   3a [insert](#insert)

   3b [insert short form from array](#insertshortfromarray)

   3c [on duplicate key update](#ondupeupdate)

   3d [bind types](#bindtypes)

   3e [get last inserted id](#lastid)

4. [UPDATE queries](#update)

   4a [update column/s](#update)

   4b [get amount of rows updated](#rowsupdated)

5. [DELETE queries](#delete)

<a name="connection"></a>

## Creating the database connection

##### Method 1: Inline

```php
$db = new PDO('mysql:host=127.0.0.1;dbname=database;charset=utf8mb4', 'username', 'password');
$db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
```

<a name="connectionfunc"></a>

##### Method 2: Function

```php
function db_connect(): PDO
{
    $host = '127.0.0.1';
    $db_name = 'database';
    $db_user = 'username';
    $db_password = 'password';
    $db = "mysql:host=$host;dbname=$db_name;charset=utf8mb4";
    $options = array(PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION);
    return new PDO($db, $db_user, $db_password, $options);
}

$db = db_connect();//Assign to variable before use
```

#### Options:

See [here](https://www.php.net/manual/en/pdo.setattribute.php) for options.

```ATTR_ERRMODE``` throws exceptions on errors.

<a name="select"></a>

## SELECT

<a name="loop"></a>

#### loop

```php
$select = $db->prepare("SELECT `column`, `column2` FROM `table`");
$select->execute();
while ($row = $select->fetch(PDO::FETCH_ASSOC)) {
    $db_col = $row['column'];
    $db_col2 = $row['column2'];
    echo "$db_col $db_col2<br>";
}
```

loop with where

```php
$status = 1;

$select = $db->prepare("SELECT `column`, `column2` FROM `table` WHERE `column3` = ?;");
$select->execute([$status]);
while ($row = $select->fetch(PDO::FETCH_ASSOC)) {
    $db_col = $row['column'];
    $db_col2 = $row['column2'];
    echo "$db_col $db_col2<br>";
}
```

<a name="onerow"></a>

#### One row

Selecting one row

```php
$user_id = 37841;

$select = $db->prepare("SELECT `name`, `email`, `age` FROM `users` WHERE `uid` = ? LIMIT 1;");
$select->execute([$user_id]);
$row = $select->fetch(PDO::FETCH_ASSOC);
$name = $row['name'];
$email = $row['email'];
$age = $row['age'];
```

Alternate placeholder binding method:

```php
$user_id = 37841;
$status = 1;

$select = $db->prepare("SELECT `name`, `email`, `age` FROM `users` WHERE `uid` = :uid AND `status` = :status LIMIT 1;");
$select->execute(array(':uid' => $user_id, ':status' => $status));
$row = $select->fetch(PDO::FETCH_ASSOC);
$name = $row['name'];
$email = $row['email'];
$age = $row['age'];
```

<a name="onecolumn"></a>

#### One column

Selecting one column only

```php
$user_id = 37841;

$select = $db->prepare("SELECT `name` FROM `users` WHERE `uid` = ? LIMIT 1;");
$select->execute([$user_id]);
$name = $select->fetchColumn();

```

<a name="count"></a>

#### count

Count the returned rows amount

```php
$age = 50;

$select = $db->prepare("SELECT `name` FROM `users` WHERE `age` > ?;");
$select->execute([$age]);
$row_count = $select->rowCount();//Row count
```

<a name="ifexits"></a>

#### if exists

Check if row found for the query

```php
$user_id = 37841;

$select = $db->prepare("SELECT `name` FROM `users` WHERE `uid` = ? LIMIT 1;");
$select->execute([$user_id]);
$row = $select->fetch(PDO::FETCH_ASSOC);
if (!empty($row)) {//Row found
    echo $row['name'];
} else {//NO row found
    echo "DOES NOT EXIST";
}
```

<a name="insert"></a>

## INSERT

```php
$insert = $db->prepare("INSERT INTO `table` (`col`, `col2`) VALUES (?, ?)");
$insert->execute([$value1, $value2]);
```

Or insert ignore

```php
$insert = $db->prepare("INSERT IGNORE INTO `table` (`col`, `col2`) VALUES (?, ?)");
$insert->execute([$value1, $value2]);
```

Alternate value binding:

```php
$insert = $db->prepare('INSERT INTO `table` (`col`, `col2`, `col3`) VALUES (:value, :value2, :value3)');
$insert->execute([
    'value' => 1,
    'value2' => $val2,
    'value3' => $val3,
]);
```

<a name="insertshortfromarray"></a>
Insert short form from array

```php
$users_array = array(
  ['uid' => 1, 'name' => 'Mike', 'age' => 42],
  ['uid' => 2, 'name' => 'John', 'age' => 36],
  ['uid' => 3, 'name' => 'Tony', 'age' => 51]
); 

$db->beginTransaction();
$insert = $db->prepare("INSERT INTO `users` (`uid`, `name`, `age`) VALUES (?, ?, ?)");
foreach ($users_array as $user) {
    $insert->execute(array(
        $user->uid,
        $user->name,
        $user->age,
    ));
}
$db->commit();
```

<a name="ondupeupdate"></a>

#### Insert on duplicate key update

```php
$query = $db->prepare('INSERT INTO `table` (id, name, price, quantity) VALUES(:id, :name, :price, :quantity)
    ON DUPLICATE KEY UPDATE `quantity` = :quantity2, `price` = :price2');
$query->bindParam(':id', $id, PDO::PARAM_INT);
$query->bindParam(':name', $name, PDO::PARAM_STR);
$query->bindParam(':price', $price, PDO::PARAM_STR);
$query->bindParam(':quantity', $quantity, PDO::PARAM_INT);
$query->bindParam(':price2', $price, PDO::PARAM_STR);
$query->bindParam(':quantity2', $quantity, PDO::PARAM_STR);
$query->execute();
```

<a name="bindtypes"></a>
Common bindParam values: ```PARAM_BOOL, PARAM_NULL, PARAM_INT & PARAM_STR```

***Note there is NO float type.***

without binding:

```php
$query = $db->prepare('INSERT INTO `table` (id, name, price, quantity) VALUES(?, ?, ?, ?)
    ON DUPLICATE KEY UPDATE `quantity` = ?, `price` = ?');
$query->execute([$id, $name, $price, $quantity, $price, $quantity]);
```

<a name="lastid"></a>

#### Getting last inserted id

```php
$last_id = $db->lastInsertId();
```

<a name="update"></a>

## UPDATE

Update column/s

```php
$score = 453;
$user_id = 37841;

$update = $db->prepare("UPDATE `users` SET `score` = ? WHERE `uid` = ? LIMIT 1;");
$update->execute([$score, $user_id]);
```

<a name="rowsupdated"></a>
Get amount of rows affected/updated:

```php
$status = 1;

$update = $db->prepare("UPDATE `users` SET `status` = ? WHERE `score` > 75;");
$update->execute([$status]);
$updated_rows = $update->rowCount();//Returns rows amount that got updated
```

<a name="delete"></a>

## DELETE

Deleting a row

```php
$user_id = 37841;

$delete = $db->prepare("DELETE FROM `users` WHERE `uid` = ? LIMIT 1;");
$delete->execute([$user_id]);
```

<a name="classexample"></a>

## Class example

PDO connection class example where you won't need to keep setting and creating a connection

```php
class test_class
{
    protected const HOSTNAME = '127.0.0.1';
    protected const DATABASE = 'database';
    protected const USERNAME = 'root';
    protected const PASSWORD = 'thepassword';
    protected PDO $db;

    public function __construct()
    {
        $db = "mysql:host=" . self::HOSTNAME . ";dbname=" . self::DATABASE . ";charset=utf8mb4";
        $options = array(PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION);
        $this->db = new PDO($db, self::USERNAME, self::PASSWORD, $options);
    }

    public function nameForUID(int $id)
    {
        $select = $this->db->prepare("SELECT `name` FROM `users` WHERE `uid` = ? LIMIT 1;");
        $select->execute([$id]);
        $row = $select->fetch(PDO::FETCH_ASSOC);
        if (!empty($row)) {//Row found
            return $row['name'];
        } else {//NO row found
            return 'Error: No name found';
        }
    }

    //..........
}
```

#### usage:

```php
$test_db = new test_class();

echo $test_db->nameForUID(1);
```
