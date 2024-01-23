# The issue
Reference: `https://github.com/slims/slims9_bulian/issues/229`


# The explanation

The code can be found [/slims/slims9_bulian/blob/master/admin/moduels/master_file/pop_scope_vocabolary.php](https://github.com/slims/slims9_bulian/blob/master/admin/modules/master_file/pop_scope_vocabolary.php), we can see that it called for external feature from `simbio_dbop` which are `update`, `insert` and `delete`. We can see there that we can control the `vocabolary_id` via the update function.

```php
    $itemID = (integer)isset($_GET['itemID'])?$_GET['itemID']:0;
    if (isset($_POST['save'])) {
    $data['topic_id'] = (integer)$_POST['topic_id'];
    $data['scope'] = trim($dbs->escape_string(strip_tags($_POST['scope'])));

    $sql_op = new simbio_dbop($dbs);

    if (!empty($_POST['vocabolary_id'])) {
        $save = $sql_op->update('mst_voc_ctrl', $data, 'vocabolary_id='.$_POST['vocabolary_id']);
    } else {
        $save = $sql_op->insert('mst_voc_ctrl', $data);
    }

    if (isset($_POST['delete'])) {
        $save = $sql_op->delete('mst_voc_ctrl', 'vocabolary_id='.$_POST['vocabolary_id']);
    }

    if ($save) {
        $alert_save  = '<script type="text/javascript">';
        $alert_save .= 'alert(\''.__('Data saved!').'\');';
        $alert_save .= 'parent.setIframeContent(\'itemIframe\', \''.MWB.'master_file/iframe_vocabolary_control.php?itemID='.$data['topic_id'].'\');';
        $alert_save .= 'top.jQuery.colorbox.close();';
        $alert_save .= '</script>';
        echo $alert_save;
    } else {
        toastr(__('Failed to save data!'))->error();
    }
```

We will look further into the `update` function in [simbio2/simbio_DB/simbio_dbop.inc.php](https://github.com/slims/slims9_bulian/blob/master/simbio2/simbio_DB/simbio_dbop.inc.php), as we can see here, the update doesn't really sanitize the input provided by the POST request.

```php
    /**
     * Method to update table records based on $str_criteria
     *
     * @param   string  $str_table
     * @param   array   $array_update
     * @param   string  $str_criteria
     * @return  boolean
     */
    public function update($str_table, $array_update, $str_criteria)
    {
        if (!is_array($array_update)) {
            return false;
        } else {
            $_set = '';
            foreach ($array_update as $column => $new_value) {
                if ($new_value == '') {
                    $_set .= ", `$column` = ''";
                } else if ($new_value === 'NULL' OR $new_value == null) {
                    $_set .= ", `$column` = NULL";
                } else if (is_string($new_value)) {
                    if (preg_match("/^literal{.+}/i", $new_value)) {
                        $new_value = preg_replace("/literal{|}/i", '', $new_value);
                        $_set .= ", `$column` = $new_value";
                    } else {
                        $_set .= ", `$column` = '$new_value'";
                    }
                } else {
                    $_set .= ", `$column` = $new_value";
                }
            }
            $_set = substr_replace($_set, '', 0, 1);
        }

        try {
            $this->sql_string = "UPDATE $str_table SET $_set WHERE $str_criteria";
            $_update = $this->obj_db->query($this->sql_string);
            $this->affected_rows = $this->obj_db->affected_rows;
        } catch (Exception $e) {
             $this->error = isDev() ? $e->getMessage() . ' : ' . $this->sql_string : ''; 
             return false; 
        }
```

From the source code analysis before, we can safely assume that this code is vulnerable to SQL Injection Vulnerability since the input was not sanitized and no preparestatement to prevent this vulnerability. In order to test the attack to exploit the bugs, we need a user or account with the privilege to access the option `master file` and then create and capture the request to `admin/modules/master_file/pop_scope_vocabolary.php` to get the request form. Last thing is to use sqlmap to do the SQL Injection attack and testing.

# Member credits to found this vulnerability

- KaCeWaffle
- Kiinzu
