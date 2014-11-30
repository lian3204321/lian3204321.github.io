---
layout: post
category: symfony
tags: [php,symfony]
---

#Databases and Propel（propel是类似doctrine的一个ORM工具）

配置数据库

配置文件在app/config/parameter.yml文件中

    parameters:
        database_driver:   mysql
        database_host:     localhost
        database_name:     test_project
        database_user:     root
        database_password: password
        database_charset:  UTF8

在配置文件app/config/config.yml文件中使用定义的配置变量

    propel:
        dbal:
            driver:   "%database_driver%"
            user:     "%database_user%"
            password: "%database_password%"
            dsn:      "%database_driver%:host=%database_host%;dbname=%database_name%;charset=%database_charset%"

根据配置文件创建数据库

    $ php app/console propel:database:create

#####在利用 PropelBundle configuration 后可以配置多个数据库

##创建一个模型类


By anni @global city  2014-12-21
