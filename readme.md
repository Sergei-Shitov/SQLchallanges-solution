# SQL Challanges Solutions

Hello! Here you can find my sollutions of challenges from [StrataScratch](https://platform.stratascratch.com/)

## 01

[challenge_01](https://platform.stratascratch.com/coding/10354-most-profitable-companies) by Forbes

solution

```SQL
select fg.company,
       fg.profits
from forbes_global_2010_2014 fg
order by fg.profits desc
limit 3
```

## 02

[challenge_02](https://platform.stratascratch.com/coding/10353-workers-with-the-highest-salaries) by DoorDash

solution

```SQL
select t.worker_title,
       w.salary
from worker w
join title t
on w.worker_id = t.worker_ref_id
where w.salary = (select max(salary)
                  from worker)
```

## 03

[challenge_03](https://platform.stratascratch.com/coding/10352-users-by-avg-session-time) by Meta/Facebook

solution

```SQL
with load_time as (
select fwl.user_id,
       date(fwl.timestamp) as day,
       max(fwl.timestamp) as load
from  facebook_web_log fwl
where fwl.action = 'page_load'
group by fwl.user_id, day
),
exit_time as (
select fwl.user_id,
       date(fwl.timestamp) as day,
       min(fwl.timestamp) as exit
from  facebook_web_log fwl
where fwl.action = 'page_exit'
group by fwl.user_id, day
)
select e.user_id,
       avg(e.exit-l.load)
from exit_time e,
     load_time l
where e.user_id = l.user_id
group by e.user_id
```

## 04

[challenge_04](https://platform.stratascratch.com/coding/10353-workers-with-the-highest-salaries) by Amazon

solution

```SQL
select distinct user_id
from
    (select user_id,
          created_at as order,
          lead(created_at) over (partition by user_id order by created_at) as next_order
    from amazon_transactions) a
where a.next_order is not null
and a.next_order - a.order <= 7
```

## 05

[challenge_05](https://platform.stratascratch.com/coding/10319-monthly-percentage-difference) by Amazons

solution

```SQL
with mon_sum as (
    select to_char(created_at, 'YYYY-MM') as month,
           sum(value) as value
    from sf_transactions
    group by month
    order by month
),
prep as (
select month,
       value as cur,
       lag(value) over (order by month) as prev
from mon_sum
)
select month,
       round((cur - prev)/prev * 100, 2) as MoM
from prep
```
