# MySQL基本操作

## 创建表

```mysql
CREATE TABLE `nano_order` (
  `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT 'id',
  `nano_number` VARCHAR(32) NOT NULL DEFAULT '' COMMENT 'nano单号',
  `tracking_no` VARCHAR(32) NOT NULL DEFAULT '' COMMENT '物流单号',
  `delivery_way` VARCHAR(20) NOT NULL DEFAULT '' COMMENT '物流方式',
  `feature` JSON DEFAULT NULL COMMENT '扩展字段',
  `parcel_price` DECIMAL(10,2) DEFAULT NULL COMMENT '包裹价格',
  `order_name` VARCHAR(24) NOT NULL DEFAULT '' COMMENT '订单号',
  `is_deleted` TINYINT(4) UNSIGNED DEFAULT '0' COMMENT '是否删除',
  `create_time` DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间 UTC',
  `update_time` DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间 UTC',
  PRIMARY KEY (`id`),
  KEY `idx_order_name` (`order_name`),
  KEY `idx_nano_order` (`nano_number`),
  KEY `idx_update_time` (`update_time`),
  UNIQUE KEY `uidx_tracking_no_delivery_way` (`tracking_no`,`delivery_way`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT 'nano_order表';
```

## 增加索引

```mysql
ALTER TABLE
  seller_address_registration DROP INDEX address_id,
ADD
  UNIQUE INDEX idx_addr_id_delivery_way(address_id, delivery_way) 
```

或者

```mysql
ALTER TABLE
  order_nano_rel
ADD
  INDEX idx_carrier_order(carrier_order),
ADD
  INDEX idx_order_name(order_name);
```

## 修改表数据

```mysql
UPDATE
  sale_order
SET
  shipping_state = "Punjab",
  shipping_city = "Jallandhur"
WHERE
  order_name = "SO275146782";
```

## 插入数据

```mysql
INSERT INTO
  cargo_proxy(
    cargo_proxy,
    email_collection,
    warehouse,
    report_type_collection
  )
VALUES
  (
    '盛凡',
    "sandy@sfanair.com",
    'hangzhou2',
    "summary_from_manifest"
  ),
  (
    '盛凡',
    "sandy@sfanair.com",
    'dongguan2',
    "summary_from_manifest"
  )
```

## 清空表

```mysql
TRUNCATE TABLE cargo_proxy
```

## 更改默认值

```mysql
ALTER TABLE carrier_company ALTER COLUMN status SET DEFAULT 1;
```

