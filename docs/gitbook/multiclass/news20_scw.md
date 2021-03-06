| Algorithm | Accuracy |
|:-----------|------------:|
| PA2 | 0.8204357625845229 |
| SCW1 | 0.8314550463310794 |
| AROW | 0.8474830954169797 |
| SCW2 |  0.8482344102178813 |
| CW |  0.850488354620586 |
---

Preparation
=========

## UDF preparation
```sql
delete jar /home/myui/tmp/hivemall.jar;
add jar /home/myui/tmp/hivemall.jar;

source /home/myui/tmp/define-all.hive;
```

---
#[CW]

## training
```sql
drop table news20mc_cw_model1;
create table news20mc_cw_model1 as
select 
 label, 
 cast(feature as int) as feature,
 -- voted_avg(weight) as weight -- [hivemall v0.1]
 argmin_kld(weight, covar) as weight -- [hivemall v0.2 or later]
from 
 (select 
     -- train_multiclass_cw(addBias(features),label) as (label,feature,weight) -- [hivemall v0.1]
     train_multiclass_cw(addBias(features),label) as (label,feature,weight,covar)    -- [hivemall v0.2 or later]
  from 
     news20mc_train_x3
 ) t 
group by label, feature;
```

## prediction
```sql
create or replace view news20mc_cw_predict1 
as
select 
  rowid, 
  m.col0 as score, 
  m.col1 as label
from (
select
   rowid, 
   maxrow(score, label) as m
from (
  select
    t.rowid,
    m.label,
    sum(m.weight * t.value) as score
  from 
    news20mc_test_exploded t LEFT OUTER JOIN
    news20mc_cw_model1 m ON (t.feature = m.feature)
  group by
    t.rowid, m.label
) t1
group by rowid
) t2;
```

## evaluation
```sql
create or replace view news20mc_cw_submit1 as
select 
  t.label as actual, 
  pd.label as predicted
from 
  news20mc_test t JOIN news20mc_cw_predict1 pd 
    on (t.rowid = pd.rowid);
```

```
select count(1)/3993 from news20mc_cw_submit1 
where actual == predicted;
```

> 0.850488354620586

## Cleaning

```sql
drop table news20mc_cw_model1;
drop table news20mc_cw_predict1;
drop view news20mc_cw_submit1;
```

---
#[AROW]

## training
```sql
drop table news20mc_arow_model1;
create table news20mc_arow_model1 as
select 
 label, 
 cast(feature as int) as feature,
 -- voted_avg(weight) as weight -- [hivemall v0.1]
 argmin_kld(weight, covar) as weight -- [hivemall v0.2 or later]
from 
 (select 
     -- train_multiclass_arow(addBias(features),label) as (label,feature,weight) -- [hivemall v0.1]
     train_multiclass_arow(addBias(features),label) as (label,feature,weight,covar) -- [hivemall v0.2 or later]
  from 
     news20mc_train_x3
 ) t 
group by label, feature;
```

## prediction
```sql
create or replace view news20mc_arow_predict1 
as
select 
  rowid, 
  m.col0 as score, 
  m.col1 as label
from (
select
   rowid, 
   maxrow(score, label) as m
from (
  select
    t.rowid,
    m.label,
    sum(m.weight * t.value) as score
  from 
    news20mc_test_exploded t LEFT OUTER JOIN
    news20mc_arow_model1 m ON (t.feature = m.feature)
  group by
    t.rowid, m.label
) t1
group by rowid
) t2;
```

## evaluation
```sql
create or replace view news20mc_arow_submit1 as
select 
  t.label as actual, 
  pd.label as predicted
from 
  news20mc_test t JOIN news20mc_arow_predict1 pd 
    on (t.rowid = pd.rowid);
```

```
select count(1)/3993 from news20mc_arow_submit1 
where actual == predicted;
```

> 0.8474830954169797

## Cleaning

```sql
drop table news20mc_arow_model1;
drop table news20mc_arow_predict1;
drop view news20mc_arow_submit1;
```

---
#[SCW1]

## training
```sql
drop table news20mc_scw_model1;
create table news20mc_scw_model1 as
select 
 label, 
 cast(feature as int) as feature,
 -- voted_avg(weight) as weight -- [hivemall v0.1]
 argmin_kld(weight, covar) as weight -- [hivemall v0.2 or later]
from 
 (select 
     -- train_multiclass_scw(addBias(features),label) as (label,feature,weight) -- [hivemall v0.1]
     train_multiclass_scw(addBias(features),label) as (label,feature,weight,covar) -- [hivemall v0.2 or later]
  from 
     news20mc_train_x3
 ) t 
group by label, feature;
```

## prediction
```sql
create or replace view news20mc_scw_predict1 
as
select 
  rowid, 
  m.col0 as score, 
  m.col1 as label
from (
select
   rowid, 
   maxrow(score, label) as m
from (
  select
    t.rowid,
    m.label,
    sum(m.weight * t.value) as score
  from 
    news20mc_test_exploded t LEFT OUTER JOIN
    news20mc_scw_model1 m ON (t.feature = m.feature)
  group by
    t.rowid, m.label
) t1
group by rowid
) t2;
```

## evaluation
```sql
create or replace view news20mc_scw_submit1 as
select 
  t.label as actual, 
  pd.label as predicted
from 
  news20mc_test t JOIN news20mc_scw_predict1 pd 
    on (t.rowid = pd.rowid);
```

```
select count(1)/3993 from news20mc_scw_submit1 
where actual == predicted;
```

> 0.8314550463310794

## Cleaning

```sql
drop table news20mc_scw_model1;
drop table news20mc_scw_predict1;
drop view news20mc_scw_submit1;
```

---
#[SCW2]

## training
```sql
drop table news20mc_scw2_model1;
create table news20mc_scw2_model1 as
select 
 label, 
 cast(feature as int) as feature,
 -- voted_avg(weight) as weight -- [hivemall v0.1]
 argmin_kld(weight, covar) as weight -- [hivemall v0.2 or later]
from 
 (select 
     -- train_multiclass_scw2(addBias(features),label) as (label,feature,weight) -- [hivemall v0.1]
     train_multiclass_scw2(addBias(features),label) as (label,feature,weight,covar) -- [hivemall v0.2 or later]
  from 
     news20mc_train_x3
 ) t 
group by label, feature;
```

## prediction
```sql
create or replace view news20mc_scw2_predict1 
as
select 
  rowid, 
  m.col0 as score, 
  m.col1 as label
from (
select
   rowid, 
   maxrow(score, label) as m
from (
  select
    t.rowid,
    m.label,
    sum(m.weight * t.value) as score
  from 
    news20mc_test_exploded t LEFT OUTER JOIN
    news20mc_scw2_model1 m ON (t.feature = m.feature)
  group by
    t.rowid, m.label
) t1
group by rowid
) t2;
```

## evaluation
```sql
create or replace view news20mc_scw2_submit1 as
select 
  t.label as actual, 
  pd.label as predicted
from 
  news20mc_test t JOIN news20mc_scw2_predict1 pd 
    on (t.rowid = pd.rowid);
```

```
select count(1)/3993 from news20mc_scw2_submit1 
where actual == predicted;
```

> 0.8482344102178813

## Cleaning

```sql
drop table news20mc_scw2_model1;
drop table news20mc_scw2_predict1;
drop view news20mc_scw2_submit1;
```