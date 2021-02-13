# PHP PDO MySQL Cheat sheet guide

A guide on the basics for using PDO PHP for MySQL.

## Creating the database connection

##### Method 1: Inline

```php
$db = new PDO('mysql:host=127.0.0.1;dbname=database;charset=utf8mb4', 'username', 'password');
$db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
```

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

## SELECT

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

Alternate binding method:

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

#### One column

Selecting one column only

```php
$user_id = 37841;

$select = $db->prepare("SELECT `name` FROM `users` WHERE `uid` = ? LIMIT 1;");
$select->execute([$user_id]);
$name = $select->fetchColumn();

```

#### count

Count the returned rows amount

```php
$age = 50;

$select = $db->prepare("SELECT `name` FROM `users` WHERE `age` > ?;");
$select->execute([$age]);
$row_count = $select->rowCount();//Row count
```

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

Common bindParam values: ```PARAM_BOOL, PARAM_NULL, PARAM_INT & PARAM_STR```

***Note there is NO float type.***

without binding:

```php
$query = $db->prepare('INSERT INTO `table` (id, name, price, quantity) VALUES(?, ?, ?, ?)
    ON DUPLICATE KEY UPDATE `quantity` = ?, `price` = ?');
$query->execute([$id, $name, $price, $quantity, $price, $quantity]);
```

#### Getting last inserted id

```php
$last_id = $db->lastInsertId();
```

## UPDATE

Update column/s

```php
$score = 453;
$user_id = 37841;

$update = $db->prepare("UPDATE `users` SET `score` = ? WHERE `uid` = ? LIMIT 1;");
$update->execute([$score, $user_id]);
```

Get amount of rows affected/updated:

```php
$status = 1;

$update = $db->prepare("UPDATE `users` SET `status` = ? WHERE `score` > 75;");
$update->execute([$status]);
$updated_rows = $update->rowCount();//Returns rows amount that got updated
```

## DELETE

Deleting a row

```php
$user_id = 37841;

$delete = $db->prepare("DELETE FROM `users` WHERE `uid` = ? LIMIT 1;");
$delete->execute([$user_id]);
```
