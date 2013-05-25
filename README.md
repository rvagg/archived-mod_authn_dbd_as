mod_authn_dbd_as
================

A fork of Apache's [mod_authn_dbd](http://httpd.apache.org/docs/2.2/mod/mod_authn_dbd.html) that allows execution of **arbitrary SQL** for user / password matching.

With most of Apache's auth modules, you must return a password from your external (or internal) source which is then matched internally by Apache using its own [encoding methods](http://httpd.apache.org/docs/2.2/misc/password_encryptions.html). Apache's encodings are fairly limited and won't support alternative encoding methods that you may be using elsewhere.

This module aims to support alternative authentication mechanisms by offloading the password matching to your database via specially crafted SQL.

Supported databases
-------------------

Apache's [mod_dbd](http://httpd.apache.org/docs/2.2/mod/mod_dbd.html) supports a range of database drivers, including generic ODBC, MSSQL, SyBase, MySQL, Oracle, PostgreSQL and SQLite, so this module will work on top of any dbd driver you can configure.

Compiling and installing
------------------------

Download [mod_authn_dbd.c](mod_authn_dbd.c) and compile & install with apxs:

```sh
$ sudo apxs2 -iac mod_authn_dbd.c
```

(it may be simply `apxs` on your system and `sudo` may or may not be required).

The arguments `-iac` will *install* and *activate* the *compiled* output of *mod_authn_dbd.c*.

Note that you will also need to have **mod_dbd** installed and activated in Apache as this is a dependency.

Usage
-----

Configuration is the same as for the standard [mod_authn_dbd](http://httpd.apache.org/docs/2.2/mod/mod_authn_dbd.html) except you now have an added directive, **AuthDBDFullAuthQuery**:

```
# mod_dbd configuration
DBDriver mysql
DBDParams "dbname=apacheauth user=apache password=xxxxxx"

DBDMin  4
DBDKeep 8
DBDMax  20
DBDExptime 300

<Directory /usr/www/myhost/private>
  # core authentication and mod_auth_basic configuration
  # for mod_authn_dbd
  AuthType Basic
  AuthName "My Server"
  AuthBasicProvider dbd

  # core authorization configuration
  Require valid-user

  # mod_authn_dbd_as SQL query to authenticate a user and password
  AuthDBDFullAuthQuery \
    "SELECT username from user where user = %s and password = sha2(concat(%s, '::mysalt'), 256)))"
</Directory>
```

In this example, we have our passwords concatenated with `"::mysalt"` and stored as SHA-256, which MySQL can encode.

An **AuthDBDFullAuthQuery** query ***MUST*** return **one or more** results for a **correct** username / password combination and return **zero** results for an **incorrect** username / password combination. It doesn't matter what the results are, non-zero indicates a successful login.

Your query has two `"%s"` values to use. The first one is always the username the client has entered and the second one is always the password. You will need to order your query such that username comes first and password second.

The standard **AuthDBDUserPWQuery** directive from the original mod_authn_dbd will also work.

Diff
----

A full diff to the original mod_authn_dbd.c version 2.2.22 (my base) can be found **[here](https://github.com/rvagg/mod_authn_dbd_as/compare/b3204a7c29...master#diff-1)**.

License
-------

mod_authn_dbd is licensed under the Apache License, Version 2.0 http://www.apache.org/licenses/LICENSE-2.0