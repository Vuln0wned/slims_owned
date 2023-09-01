# The issue
Reference: `https://github.com/slims/slims9_bulian/issues/209`


# The explanation

it is straight forward that there is unsafe parsing from input to user at the code below

```php
  $reportgrid->setSQLCriteria($criteria);

    $start_date = '2000-01-01';
    if (isset($_GET['startDate'])) {
        $start_date = $_GET['startDate'];
    }
    $until_date = date('Y-m-d');
    if (isset($_GET['untilDate'])) {
        $until_date = $_GET['untilDate'];
    }
    // callbacks
    function showBiblioEntries($obj_db, $array_data)
    {
        global $start_date, $until_date;
        $_count_q = $obj_db->query('SELECT COUNT(log_id) FROM system_log WHERE log_location=\'bibliography\' AND log_type=\'staff\'
            AND log_msg LIKE \'%insert bibliographic data%\' AND id=\''.$array_data['2'].'\' AND TO_DAYS(log_date) BETWEEN TO_DAYS(\''.$start_date.'\') AND TO_DAYS(\''.$until_date.'\')');
        $_count_d = $_count_q->fetch_row();
        return $_count_d[0];
    }
```



which then passes to the function below in the same code

```php
 function showBiblioEntries($obj_db, $array_data)
    {
        global $start_date, $until_date;
        $_count_q = $obj_db->query('SELECT COUNT(log_id) FROM system_log WHERE log_location=\'bibliography\' AND log_type=\'staff\'
            AND log_msg LIKE \'%insert bibliographic data%\' AND id=\''.$array_data['2'].'\' AND TO_DAYS(log_date) BETWEEN TO_DAYS(\''.$start_date.'\') AND TO_DAYS(\''.$until_date.'\')');
        $_count_d = $_count_q->fetch_row();
        return $_count_d[0];
    }

```

and executed at the code below

```php
    $reportgrid->column_width = array(0 => '10%', 1 => '10%');
    $reportgrid->table_attr = 'class="s-table table table-sm table-bordered"';
    $reportgrid->modifyColumnContent(2, 'callback{showBiblioEntries}');
    $reportgrid->modifyColumnContent(3, 'callback{showItemEntries}');
    $reportgrid->modifyColumnContent(4, 'callback{showMemberEntries}');
    $reportgrid->modifyColumnContent(5, 'callback{showCirculation_Loan}');
    $reportgrid->modifyColumnContent(6, 'callback{showCirculation_Return}');
    $reportgrid->modifyColumnContent(7, 'callback{showCirculation_Extends}');

```

From here we can see that the input is unsanitized totally and passed to sql query without preparestatement. In order to launch the attack to exploit the bugs, all we need is a user or account with privilege to access circulation, then either capture a request to `admin/modules/reporting/customs/staff_act.php` to get the request, then use sqlmap to dump the database or perform any other various attacks to the database

# Member credits to found this vulnerability

- vigil
- Rakuzen
- Drab
