

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


```
UPDATE
    `test` AS `a`,
    (
        select invoice,amount from test3
    ) AS `b`
SET
    a.amount2=b.amount
where `a`.`invoice` = `b`.`invoice`;
```

```
update test set flag =
case when amount2=0 then '数据不存在'
     when `diffamount`=0 then '数据金额一致'
     else '数据金额不匹配' end;

```
