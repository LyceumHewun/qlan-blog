---
title: "Seata 部署指南"
date: 2022-06-04T18:25:11+08:00
tags: []
categories: []
author: "谦兰君"
draft: true
description: ""
weight: 100
---

## 环境

- Docker - 1.13.1
- docker-compose - 1.23.2
- seata-server - 1.4.2
- Nacos - 2.0.4

## 步骤

1. 新建seata目录

```sh
mkdir /opt/seata
mkdir /opt/seata/config
```

2. 新建seata服务端配置文件`registry.conf`

```sh
vi /opt/seata/config/registry.conf
```

编写以下配置

```
registry {
  type = "nacos"
  
  nacos {
    # 服务名称
    application = "seata-server"
    serverAddr = "127.0.0.1:18848"
    # 命名空间ID
    namespace = ""
    group = "SEATA_GROUP"
    # 集群名称
    cluster = "default"
  }
}

config {
  type = "nacos"
  
  nacos {
    serverAddr = "127.0.0.1:18848"
    namespace = ""
    group = "SEATA_GROUP"
    # 配置文件dataId
    dataId = "seata-server.properties"
  }
}
```

3. 在nacos新建`seata-server.properties`配置

> 注意配置文件的Group要与上一步配置的一致

```properties
store.mode=db
store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.cj.jdbc.Driver
store.db.url=jdbc:mysql://192.168.1.213:3309/seata-server?useUnicode=true&characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false
store.db.user=root
store.db.password=123456
```

4. 初始化数据库

新建上一步配置的数据库`seata-server`，执行`seata-server.sql`

```sql
-- -------------------------------- The script used when storeMode is 'db' --------------------------------
-- the table to store GlobalSession data
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_status_gmt_modified` (`status` , `gmt_modified`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

-- the table to store BranchSession data
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

-- the table to store lock data
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(128),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `status`         TINYINT      NOT NULL DEFAULT '0' COMMENT '0:locked ,1:rollbacking',
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_status` (`status`),
    KEY `idx_branch_id` (`branch_id`),
    KEY `idx_xid_and_branch_id` (`xid` , `branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

CREATE TABLE IF NOT EXISTS `distributed_lock`
(
    `lock_key`       CHAR(20) NOT NULL,
    `lock_value`     VARCHAR(20) NOT NULL,
    `expire`         BIGINT,
    primary key (`lock_key`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('AsyncCommitting', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('RetryCommitting', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('RetryRollbacking', ' ', 0);
INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('TxTimeoutCheck', ' ', 0);
```



5. 编写`docker-compose.yaml`

```sh
vi /opt/seata/docker-compose.yaml
```

编写以下配置

```yaml
version: "3.1"
services:
  alum-seata:
    image: seataio/seata-server:1.4.2
    hostname: alum-seata
    ports:
      - "18091:18091"
    environment:
      - SEATA_PORT=18091
      - SEATA_IP=127.0.0.1
      - SEATA_CONFIG_NAME=file:/root/seata-config/registry
    volumes:
      - "/opt/seata/config:/root/seata-config"
```

6. 启动seata-server

```sh
cd /opt/seata/
docker-compose up -d
```

## 客户端配置

1. 新建seata-client配置

```properties
seata.tx-service-group=sfc_tx_group
seata.service.vgroup-mapping.sfc_tx_group=default
seata.registry.type=nacos
seata.registry.nacos.server-addr=127.0.0.1:18848
seata.registry.nacos.application=seata-server
seata.registry.nacos.group=SEATA_GROUP
```

2. 数据新建undo_log表，执行`undo_log.sql`

```sql
-- 注意此处0.3.0+ 增加唯一索引 ux_undo_log
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

