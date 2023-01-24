# SQL Challanges Solutions

Hello! Here you can find my sollutions of challenges from [StrataScratch](https://platform.stratascratch.com/)

## 01

[challenge_01](https://platform.stratascratch.com/coding/10354-most-profitable-companies) by Forbes

difficulty: medium

Find the 3 most profitable companies in the entire world.

solution:

```SQL
select fg.company,
       fg.profits
from forbes_global_2010_2014 fg
order by fg.profits desc
limit 3
```

## 02

[challenge_02](https://platform.stratascratch.com/coding/10353-workers-with-the-highest-salaries) by DoorDash

difficulty: medium

Find the titles of workers that earn the highest salary

solution:

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

difficulty: medium

Calculate each user's average session time.

solution:

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

[challenge_04](https://platform.stratascratch.com/coding/10322-finding-user-purchases) by Amazon

Write a query that'll identify returning active users. A returning active user is a user that has made a second purchase within 7 days of any other of their purchases.

difficulty: medium

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

[challenge_05](https://platform.stratascratch.com/coding/10319-monthly-percentage-difference) by Amazon

Given a table of purchases by date, calculate the month-over-month percentage change in revenue

difficulty: hard

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

## 06

[challenge_06](https://platform.stratascratch.com/coding/10308-salaries-differences) by Dropbox

Write a query that calculates the difference between the highest salaries found in the marketing and engineering departments.

difficulty: easy

solution

```SQL
select abs(max(salary) filter (where department = 'marketing') - 
           max(salary) filter (where department = 'engineering')) 
from db_employee e
join db_dept d on e.department_id = d.id
```

## 07

[challenge_07](https://platform.stratascratch.com/coding/10300-premium-vs-freemium) by Microsoft

Find the total number of downloads for paying and non-paying users by date.

difficulty: hard

solution

```SQL
with downloads as (
select mdf.date,
       sum(mdf.downloads) filter (where mad.paying_customer = 'no') as non_pay,
       sum(mdf.downloads) filter (where mad.paying_customer = 'yes') as pay
from ms_download_facts mdf,
     ms_acc_dimension mad,
     ms_user_dimension mud
where mdf.user_id = mud.user_id
and mud.acc_id = mad.acc_id
group by mdf.date
)
select *
from downloads
where non_pay > pay
order by date
```

## 08

[challenge_08](https://platform.stratascratch.com/coding/10299-finding-updated-records) by Microsoft

Find the current salary of each employee assuming that salaries increase each year

difficulty: easy

solution

```SQL
select id,
       first_name,
       last_name,
       max(salary) as salary,
       department_id
from ms_employee_salary
group by id,
         first_name,
         last_name,
         department_id
order by id
```
