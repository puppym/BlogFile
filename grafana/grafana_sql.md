1. 每天的区块数目统计

```sql

SELECT
  _timestamp AS "time",
 date_trunc('day',to_timestamp(_timestamp)),
  count(2)
FROM blocks
WHERE
  to_timestamp(_timestamp) between '2015-07-30 03:00:00' and '2019-11-25 06:30:00'
GROUP BY 2;

改正之后：

SELECT
 date_trunc('day',to_timestamp(_timestamp)) AS "time",
  count(1)
FROM blocks
WHERE
  to_timestamp(_timestamp) between '2015-07-30 03:00:00' and '2019-11-25 06:30:00'
GROUP BY 1;

```

2. 每天的交易数量统计

```sql
SELECT
 date_trunc('day',to_timestamp(_timestamp)) AS "time",
  count(1)
FROM transactions
WHERE
  to_timestamp(_timestamp) between '2015-07-30 03:00:00' and '2019-11-25 06:30:00'
GROUP BY 1;

```

3. 每天创建地址的数量

```sql
SELECT
 date_trunc('day',to_timestamp(_timestamp)) AS "time",
  count(1)
FROM transfers
WHERE
  to_timestamp(_timestamp) between '2015-07-30 03:00:00' and '2019-11-25 06:30:00' and left(transferType,2)=left('CREATE',2)
GROUP BY 1;
```

4. 每天调用各类trace的数量

```sql
explain analyse select op,count(*) from traces  where to_timestamp(_timestamp) between '2015-07-31 09:23:00' and '2015-08-31 09:23:00' group by op;

SELECT
 date_trunc('day',to_timestamp(_timestamp)) AS "time",
  count(1)
FROM traces
WHERE
  to_timestamp(_timestamp) between '2015-07-30 03:00:00' and '2019-11-25 06:30:00' and op='CALL'
GROUP BY 1;
```

5. 每天调用失败的数量

```sql
explain analyse select err,count(*) from 
(select * from traces where to_timestamp(_timestamp) between '2015-07-31 09:23:00' and '2017-08-31 09:23:00') as f
where f.err!='' group by err; 

SELECT
 date_trunc('day',to_timestamp(_timestamp)) AS "time",
  count(1)
FROM traces
WHERE
  to_timestamp(_timestamp) between '2015-07-30 03:00:00' and '2019-11-25 06:30:00' and err!=''
GROUP BY 1;
```

6. 预编译合约调用统计

```sql
explain analyse select _to,count(*) from (select * from precompileds where to_timestamp(_timestamp) between '2015-07-31 09:23:00' and '2015-08-31 09:23:00') as f  group by _to;

SELECT
 date_trunc('day',to_timestamp(_timestamp)) AS "time",
  count(1)
FROM precompileds
WHERE
  to_timestamp(_timestamp) between '2018-07-30 03:00:00' and '2019-11-25 06:30:00' 
GROUP BY 1;

```

7.主流交易所交易统计

```sql
explain analyse select row_number()over(),split_part(topics,'_',2) as _from,split_part(topics,'_',3) as _to,eventData,blockNumber,blockHash,txHash,txIndex,logIndex,_timestamp 
from events where to_timestamp(_timestamp) between '2015-07-31 09:23:00' and '2019-08-31 09:23:00' 
and left(eventAddress,10)='0xdAC17F958D2ee523a2206206994597C13D831ec7' and left(topics,10)='0xddf252ad';

SELECT
 date_trunc('day',to_timestamp(_timestamp)) AS "time",
  count(1)
FROM events
where to_timestamp(_timestamp) between '2018-07-31 09:23:00' and '2019-11-25 09:23:00' and left(eventAddress,10)=left('0xdAC17F958D2ee523a2206206994597C13D831ec7',10) and left(topics,10)='0xddf252ad' 
GROUP BY 1;


```

8. 每一天活跃地址的数量

```sql
select f._from, count(f._from),row_number()over(order by count(*) desc) from (select _from ,_timestamp from transfers where to_timestamp(_timestamp) between '2015-07-31 09:23:00' and '2016-08-31 09:23:00' union all select _to ,_timestamp from transfers where to_timestamp(_timestamp) between '2015-07-31 09:23:00' and '2016-08-31 09:23:00') as f group by f._from;

统计from地址每一天的地址总数

SELECT
  date_trunc('day',to_timestamp(f._timestamp)) AS "time",
  distinct on (_from),
  count(1)
FROM (select distinct on (_from) ,_timestamp from transfers where to_timestamp(_timestamp) between '2019-10-30 09:23:00' and '2019-11-25 09:23:00') as f
group by 1;

每天每个地址交易的次数，每天一共有多少个地址活跃
select date_trunc('day',timee) AS "time" ,count(*) from 
(SELECT
  date_trunc('day',to_timestamp(_timestamp)) AS timee,
  _from,
  count(*) as cnt
FROM transfers 
WHERE  to_timestamp(_timestamp) between '2019-10-30 09:23:00' and '2019-11-25 09:23:00'
group by 1,2) as f group by 1;
```

9. USDT每天(一段时间内)调用数量

```sql
explain analyse select _from,count(_from) from transfers where to_timestamp(_timestamp) between '2015-07-31 09:23:00' and '2017-08-31 09:23:00' and left(_to,10)='0xa0b86991' group by 1 order by 2 limit 20;

SELECT
  date_trunc('day',to_timestamp(_timestamp)) AS "time",
  count(1)
FROM transfers
where to_timestamp(_timestamp) between '2018-07-31 09:23:00' and '2019-11-25 09:23:00' and left(_to,10)=left('0xdAC17F958D2ee523a2206206994597C13D831ec7',10)
GROUP BY 1;
```

10. USDT每天活跃地址数

```sql
SELECT date_trunc('day',timee) AS "time" ,count(*) from (SELECT
  date_trunc('day',to_timestamp(_timestamp)) AS "timee",
  _from,
  count(*) as cnt
FROM transfers
where to_timestamp(_timestamp) between '2019-07-31 09:23:00' and '2019-11-25 09:23:00' and left(_to,10)=left('0xdAC17F958D2ee523a2206206994597C13D831ec7',10)
GROUP BY 1,2) as f group by 1;
```

11. USDT每天(一段时间内)调用最活跃最活跃地址前20

