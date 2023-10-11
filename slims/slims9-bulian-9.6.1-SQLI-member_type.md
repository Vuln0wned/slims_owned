# The issue
Reference: `https://github.com/slims/slims9_bulian/issues/216`

# The explanation

there is unsafe parsing from input to user shown at the code below

```php
$data['enable_reserve'] = $_POST['enableReserve'];
$data['reserve_limit'] = $_POST['reserveLimit'];
$data['member_periode'] = $_POST['memberPeriode'];
$data['reborrow_limit'] = $_POST['reborrowLimit']; // unsanitized input
$data['fine_each_day'] = $_POST['fineEachDay'];
$data['grace_periode'] = $_POST['gracePeriode'];
```

The Data will be passes into a function below

- Update
```php
  if ($update) {
    utility::jsToastr(__('Member Type'),__('Member Type Successfully Updated'),'success');
    // update all member expire date
    @$dbs->query('UPDATE member AS m SET expire_date=DATE_ADD(register_date,INTERVAL '.$data['member_periode'].'  DAY)
        WHERE member_type_id='.$updateRecordID);
    echo '<script type="text/javascript">parent.$(\'#mainContent\').simbioAJAX(\''.$_SERVER['PHP_SELF'].'\');</script>';
    } else { utility::jsToastr(__('Member Type'),__('Member Type Data FAILED to Save/Update. Please Contact System Administrator')."\nDEBUG : ".$sql_op->error,'error'); }
    exit();
```

- Insert
```php
  /* INSERT RECORD MODE */
  // insert the data
  if ($sql_op->insert('mst_member_type', $data)) {
    utility::jsToastr(__('Member Type'),__('New Member Type Successfully Saved'),'success');
    echo '<script type="text/javascript">parent.$(\'#mainContent\').simbioAJAX(\''.$_SERVER['PHP_SELF'].'\');</script>';
    } else { utility::jsToastr(__('Member Type'),__('Member Type Data FAILED to Save/Update. Please Contact System Administrator')."\n".$sql_op->error,'error'); }
    exit();
```


The data will be passes into a function in a `simbio_dbop.inc.php` 

- Update
```php
$this->sql_string = "UPDATE $str_table SET $_set WHERE $str_criteria";
$_update = $this->obj_db->query($this->sql_string);
```

- Insert
```php
$this->sql_string = "INSERT INTO `$str_table` ($_str_columns) "
    ."VALUES ($_str_value)";
$_insert = $this->obj_db->query($this->sql_string);
```

From the code above, we can conclude the input is unsanitized and passed to sql query without preparestatement. To launch the attack in a form of bug exploitation, we need an authorized user to capture a request from `/admin/modules/membership
/member_type.php` to get the request. After getting the authorization, use sqlmap as a tool to do some attacks, such as database dump.

# Member credits to found this vulnerability

- kisanak
- Krakenhaus
- Wrth
