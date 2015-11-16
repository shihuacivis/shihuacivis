title: mysql按中文首字母进行排序
---

用gbk编码进行排序，其中name是用户中文名字段

```sql
    SELECT * from `tb_member` 
    WHERE 1 
    ORDER BY CONVERT( `name` USING gbk ) 
    COLLATE gbk_chinese_ci ASC;
```
`written in 2014/10 本文搬运自LOFTER，让LOFTER纯粹po图！ `