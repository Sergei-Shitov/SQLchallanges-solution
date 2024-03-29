# SQL Challanges Solutions

Hello! Here you can find my sollutions of challenges from [StrataScratch](https://platform.stratascratch.com/)

## 01

[challenge_01](https://platform.stratascratch.com/coding/10354-most-profitable-companies) by Forbes

*difficulty: medium*

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

*difficulty: medium*

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

*difficulty: medium*

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

*difficulty: medium*

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

*difficulty: hard*

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

*difficulty: easy*

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

*difficulty: hard*

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

*difficulty: easy*

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

## 09

[challenge_09](https://platform.stratascratch.com/coding/10285-acceptance-rate-by-date) by Meta/Facebook

What is the overall friend acceptance rate by date? Your output should have the rate of acceptances by the date the request was sent. Order by the earliest date to latest.

*difficulty: medium*

solution

```SQL
select date,
       count(accepted)::float / count(action) as acc_rate
from
    (select date,
           action,
           lead(action) over (partition by user_id_sender) as accepted
    from fb_friend_requests) a
where action = 'sent'
group by date
order by date
```

## 10

[challenge_10](https://platform.stratascratch.com/coding/10284-popularity-percentage) by Meta/Facebook

Find the popularity percentage for each user on Meta/Facebook. The popularity percentage is defined as the total number of friends the user has divided by the total number of users on the platform

*difficulty: hard*

solution

```SQL
with friends as(
select user1,
       count(user2) as count_friends
from
    (select user1, user2
    from facebook_friends
    union
    select user2, user1
    from facebook_friends) a
group by user1
order by count_friends desc
), total_users as (
select count(user1) as total
from friends
)
select f.user1,
       (f.count_friends::float / tu.total)*100 as rating
from friends f,
     total_users tu
order by rating desc
```

## 11

[challenge_11](https://platform.stratascratch.com/coding/10064-highest-energy-consumption) by Meta/Facebook

Find the date with the highest total energy consumption from the Meta/Facebook data centers.

*difficulty: medium*

solution

```SQL
with totals as(
select date,
       sum(consumption) as tot_cons
from
    (select * 
    from fb_eu_energy
    union all
    select * 
    from fb_asia_energy
    union all
    select * 
    from fb_na_energy) a
group by date
order by tot_cons desc
)
select *
from totals
where tot_cons = (select max(tot_cons)
                  from totals)
```

## 12

[challenge_12](https://platform.stratascratch.com/coding/10061-popularity-of-hack) by Meta/Facebook

Find the average popularity of the Hack per office location

*difficulty: easy*

solution

```SQL
select e.location,
       avg(hs.popularity)
from facebook_employees e
join facebook_hack_survey hs
on e.id = hs.employee_id
group by e.location
```

## 13

[challenge_13](https://platform.stratascratch.com/coding/10060-top-cool-votes) by Yelp

Find the review_text that received the highest number of 'cool' votes.

*difficulty: medium*

solution

```SQL
select business_name,
       review_text
from yelp_reviews
where cool = (select max(cool)
              from yelp_reviews);
```

## 14

[challenge_14](https://platform.stratascratch.com/coding/10049-reviews-of-categories) by Yelp

Find the top business categories based on the total number of reviews

*difficulty: medium*

solution

```SQL
select unnest(string_to_array(categories, ';')) as cat,
       sum(review_count) as count
from yelp_business
group by cat
order by count desc
```

## 15

[challenge_15](https://platform.stratascratch.com/coding/10046-top-5-states-with-5-star-businesses) by Yelp

Find the top 5 states with the most 5 star businesses.

*difficulty: hard*

solution

```SQL
with ranks as(
    select state,
           count(*) as count,
           rank() over (order by count(*) desc) as ranking
    from yelp_business
    where stars = 5
    group by state
)
select state,
       count
from ranks
where ranking <= 5
order by count desc, state
```

## 16

[challenge_16](https://platform.stratascratch.com/coding/9917-average-salaries) by Salesforce

Compare each employee's salary with the average salary of the corresponding department.

*difficulty: easy*

solution

```SQL
select e.department,
       e.first_name,
       e.salary,
       avg(e.salary) over(partition by department) as avg
from employee e
```

## 17

[challenge_17](https://platform.stratascratch.com/coding/9915-highest-cost-orders) by Amazon

Find the customer with the highest daily total order cost between 2019-02-01 to 2019-05-01.

*difficulty: medium*

solution

```SQL
with calc as(
select c.first_name,
       sum(o.total_order_cost) as order_sum,
       o.order_date
from customers c
join orders o
on c.id = o.cust_id
where o.order_date between '2019-02-01' and '2019-05-01'
group by c.first_name,
         o.order_date
)
select *
from calc c
where c.order_sum = (select max(cs.order_sum)
                     from calc cs)
```

## 18

[challenge_18](https://platform.stratascratch.com/coding/9913-order-details) by Amazon

Find order details made by Jill and Eva.

*difficulty: easy*

solution

```SQL
select c.first_name,
       o.order_date,
       o.order_details,
       o.total_order_cost
from customers c
join orders o
on c.id = o.cust_id
where c.first_name in ('Jill', 'Eva')
order by c.id
```

## 19

[challenge_19](https://platform.stratascratch.com/coding/9905-highest-target-under-manager) by Salesforce

Find the highest target achieved by the employee or employees who works under the manager id 13.

*difficulty: medium*

solution

```SQL
select se.first_name,
       se.target
from salesforce_employees se
where se.manager_id = 13
and se.target = (select max(ses.target)
                 from salesforce_employees ses
                 where ses.manager_id = 13)
```

## 20

[challenge_20](https://platform.stratascratch.com/coding/9897-highest-salary-in-department) by Twitter

Find the employee with the highest salary per department.

*difficulty: medium*

solution

```SQL
select e.department,
       e.first_name,
       e.salary
from employee e
join (select es.department,
             max(es.salary) as max_salary
      from employee es
      group by es.department) ess
on e.department = ess.department
and e.salary = ess.max_salary
```

## 21

[challenge_21](https://platform.stratascratch.com/coding/9894-employee-and-manager-salaries) by DropBox

Find employees who are earning more than their managers. Output the employee's first name along with the corresponding salary.

*difficulty: medium*

solution

```SQL
select e.first_name,
       e.salary
from employee e
where e.salary > (select es.salary
                  from employee es
                  where e.manager_id = es.id)
```

## 22

[challenge_22](https://platform.stratascratch.com/coding/9891-customer-details) by Amazon

Find the details of each customer regardless of whether the customer made an order.

*difficulty: easy*

solution

```SQL
select c.first_name,
       c.last_name,
       c.city,
       o.order_details
from customers c
left join orders o
on c.id = o.cust_id
order by 1,4
```

## 23

[challenge_23](https://platform.stratascratch.com/coding/9782-customer-revenue-in-march) by Meta/Facebook

Calculate the total revenue from each customer in March 2019

*difficulty: medium*

solution

```SQL
select cust_id,
       sum(total_order_cost) as revenue
from orders
where order_date between '2019-03-01' and '2019-03-31'
group by cust_id
order by revenue desc
```

## 24

[challenge_24](https://platform.stratascratch.com/coding/9728-inspections-that-resulted-in-violations) by City of San Francisco

Count the number of violation in an inspection in 'Roxanne Cafe' for each year

*difficulty: medium*

solution

```SQL
select extract(year from inspection_date) as year,
       count(*) as n_inspections
from sf_restaurant_health_violations
where business_name = 'Roxanne Cafe'
group by year
order by year
```

## 25

[challenge_25](https://platform.stratascratch.com/coding/9726-classify-business-type) by City of San Francisco

Classify each business as either a restaurant, cafe, school, or other

*difficulty: medium*

solution

```SQL
select distinct 
       business_name,
       case
        when lower(business_name) like '%restaurant%' then 'restaurant'
        when lower(business_name) like any(array['%cafe%', '%café%', '%coffee%']) then 'cafe'
        when lower(business_name) like '%school%' then 'School'
        else 'other'
       end as class
from sf_restaurant_health_violations
```
