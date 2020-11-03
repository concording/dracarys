

```
UPDATE
    `user` AS `a`,
    (
        select id,code,name,sum(amount) amount,avg(danjia) danjia,sum(money) money,avg(zhehoudanjia) zhehoudanjia,sum(zhehoujine) zhehoujine,
        avg(hanshuidanjia) hanshuidanjia, sum(fee) fee,sum(totalfee) totalfee from user group by code,name having count(code)>1
    ) AS `b`
SET
    a.amount=b.amount,a.money=b.money,a.fee=b.fee ,a.totalfee=b.totalfee,a.danjia=b.danjia,a.zhehoudanjia=b.zhehoudanjia,a.zhehoujine=b.zhehoujine,a.hanshuidanjia=b.`hanshuidanjia`
where `a`.`code` = `b`.`code` and     `a`.`name` = `b`.`name`;

```


```
UPDATE tableA a
INNER JOIN tableB b ON a.name_a = b.name_b
SET validation_check = if(start_dts > end_dts, 'VALID', '')
```
