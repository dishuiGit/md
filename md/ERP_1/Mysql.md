1,向表中插入数据

```sql
INSERT tbl_emp (username,pwd,NAME) VALUES('admin',MD5('admin'),'滴水');
```

```sql
CREATE TABLE `tbl_goods` (
  `uuid` bigint(20) NOT NULL auto_increment,
  `name` varchar(30) NOT NULL,
  `origin` varchar(30) NOT NULL,
  `producer` varchar(30) NOT NULL,
  `unit` varchar(30) NOT NULL,
  `inPrice` double(10,2) NOT NULL,
  `outPrice` double(10,2) NOT NULL,
  `goodsTypeUuid` bigint(20) NOT NULL,
  PRIMARY KEY  (`uuid`)
)
```