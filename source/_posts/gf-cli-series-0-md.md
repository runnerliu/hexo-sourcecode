---
title: GF-CLI学习系列
date: 2020-10-27 15:13:50
tags:
 - gf
 - gf-cli
categories:
 - gf-cli
---

#### gen

该命令用以自动化从数据库直接生成模型文件。

该命令将会根据数据表名（注意：所以要先在数据里建好表）生成对应的目录，该目录名称即数据表包名。

目录下自动生成3个文件：

- 数据表名.go 自定义文件，开发者可以自由定义填充的代码文件，仅会生成一次，每一次模型生成不会覆盖
- 数据表名\_entity.go 表结构文件，根据数据表结构生成的结构体定义文件，包含字段注释。数据表在外部变更后，可使用gen命令重复生成更新该文件
- 数据表名\_model.go 表模型文件，为数据表提供了许多便捷的CURD操作方法，并可直接查询返回该表的结构体对象。数据表在外部变更后，可使用gen命令重复生成更新该文件

使用方式：`gf gen model ./app/model -c config/config.toml -p sys_ -t sys_users`

命令说明：

- ./app/model：在model生成的路径
- -c config/config.toml：在这个配置里找database数据库连接配置 需要写好mysql的配置信息
- -p sys_：去除生成文件目录的sys前缀 如果不加这个参数就会按数据库名生成目录和文件名 如：sys_users
- -t sys_users：要生成model的数据表文件名