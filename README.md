ovv.wallabag
============

[![Build Status](https://travis-ci.org/ovv/ansible-role-wallabag.svg?branch=master)](https://travis-ci.org/ovv/ansible-role-wallabag)

Ansible role to install and configure [Wallabag](https://wallabag.org/en).

Requirements
------------

A PHP, Nginx and PostgreSQL installation are required. We recommend using [ovv.php7](https://github.com/ovv/ansible-role-php7),
[pyslackers.nginx](https://github.com/pyslackers/ansible-role-nginx) and [pyslackers.postgres](https://github.com/pyslackers/ansible-role-postgres).

Installation
------------

To install this roles clone it into your roles directory.

```bash
$ git clone https://github.com/ovv/ansible-role-wallabag.git ovv.wallabag
```

If your playbook already reside inside a git repository you can clone it by using git submodules.

```bash
$ git submodule add -b master https://github.com/ovv/ansible-role-wallabag.git ovv.wallabag
```

Role Variables
--------------

* `wallabag_db_name`: Postgresql database name
* `wallabag_db_user`: Postgresql database connecting user.
* `wallabag_db_password`: Postgresql database password.

* `wallabag_url`: Full URL of your wallabag instance.
* `wallabag_secret`: Unique secret string 

* `wallabag_admin`: Wallabag admin username.
* `wallabag_admin_email`: Wallabag admin email.
* `wallabag_admin_password`: Wallabag admin password.

Example Playbook
----------------

```yml
- hosts: localhost
  roles:
    - pyslackers.postgres
    - ovv.php7
    - ovv.wallabag
    - pyslackers.nginx
  vars:
    wallabag_db_name: wallabag
    wallabag_db_user: wallabag
    wallabag_db_password: databasepassword
    wallabag_url: https://wallabag.example.com
    
    wallabag_admin: admin
    wallabag_admin_email: admin@example.com
    wallabag_admin_password: password

    # ovv.php
    custom_php_packages:
      - php7.0-json
      - php7.0-xml
      - php7.0-pgsql
      - php7.0-tidy
      - php7.0-bcmath
      - php7.0-gd

    php_pools:
      wallabag:
        socket: /var/run/php7.0-fpm-wallabag.sock
        user: wallabag
        working_dir: /opt/wallabag

    # pyslackers.nginx
    nginx_sites:
      wallabag:
        directory: /opt/wallabag/web
        locations:
          - location: /
            custom: try_files $uri /app.php$is_args$args;
          - location: ~ ^/app\.php(/|$)
            custom: |
              fastcgi_pass unix:{{ php_pools['wallabag']['socket'] }};
              fastcgi_split_path_info ^(.+\.php)(/.*)$;
              include fastcgi_params;
              fastcgi_param  SCRIPT_FILENAME  $realpath_root$fastcgi_script_name;
              fastcgi_param DOCUMENT_ROOT $realpath_root;
              internal;
          - location: ~ \.php$
            custom: deny all;

    # pyslackers.postgres
    postgres_users:
      wallabag:
        password: databasepassword
```

License
-------

MIT
