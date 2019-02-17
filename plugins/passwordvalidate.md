# validate_password插件

 validate_password插件是为了检验密码和提升安全， validate_password拥有一系列的variable来设置密码策略。



## 安装

- 在线：INSTALL PLUGIN validate_password SONAME 'validate_password.so';

- 离线：

  ```
  [mysqld]
  plugin-load-add=validate_password.so
  ```

  

## validate_password选项和变量

```
mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password_check_user_name    | OFF    |
| validate_password_dictionary_file    |        |
| validate_password_length             | 8      |
| validate_password_mixed_case_count   | 1      |
| validate_password_number_count       | 1      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 1      |
+--------------------------------------+--------+
```

- validate_password_check_user_name = [on|off],默认不可用

  当这个参数可用，有以下影响：

  - 所有validate_password插件被使用的地方都将检验密码和用户名。包含ALTER USER或者SET PASSWORD来改变当前用户的密码,或者调用PASSWORD()和VALIDATE_PASSWORD_STRENGTH()
  - 用于比较的用户名来自当前session的user()或current_user().**言外之意，一个有足够权限的用户可以将其它用户的密码设置成和帐号一样**
  - 仅仅user()或current_user()的名字部分被使用，不包括host部分
  - 如果密码和账户名称一样或者密码的倒序和帐号一样，此时匹配上，密码被拒绝。
  - 比对过程中大小写敏感。
  - 如果匹配上，[`VALIDATE_PASSWORD_STRENGTH()`](https://dev.mysql.com/doc/refman/5.7/en/encryption-functions.html#function_validate-password-strength) 返回0

  

- validate_password_length：检验密码的长度

- 

  