# SQL--PROJECTS
My SQL projects

----------------------------------1st Sql project -- ZOMATO CASE STUDY FOR GOLD MEMBERSHIP CUSTOMERS AND NON GOLD MEMBERSHIP CUSTOMERS------------

DB AND TABLE CREATION-----------------------------------------------------------------------------------------------------------------------------------------
drop table if exists goldusers_signup;
CREATE TABLE goldusers_signup(userid integer,gold_signup_date date); 

INSERT INTO goldusers_signup(userid,gold_signup_date) 
 VALUES (1,'09-22-2017'),
(3,'04-21-2017');

drop table if exists users;
CREATE TABLE users(userid integer,signup_date date); 

INSERT INTO users(userid,signup_date) 
 VALUES (1,'09-02-2014'),
(2,'01-15-2015'),
(3,'04-11-2014');

drop table if exists sales;
CREATE TABLE sales(userid integer,created_date date,product_id integer); 

INSERT INTO sales(userid,created_date,product_id) 
 VALUES (1,'04-19-2017',2),
(3,'12-18-2019',1),
(2,'07-20-2020',3),
(1,'10-23-2019',2),
(1,'03-19-2018',3),
(3,'12-20-2016',2),
(1,'11-09-2016',1),
(1,'05-20-2016',3),
(2,'09-24-2017',1),
(1,'03-11-2017',2),
(1,'03-11-2016',1),
(3,'11-10-2016',1),
(3,'12-07-2017',2),
(3,'12-15-2016',2),
(2,'11-08-2017',2),
(2,'09-10-2018',3);


drop table if exists product;
CREATE TABLE product(product_id integer,product_name text,price integer); 

INSERT INTO product(product_id,product_name,price) 
 VALUES
(1,'p1',980),
(2,'p2',870),
(3,'p3',330);


select * from sales;
select * from product;
select * from goldusers_signup;
select * from users;

Questions -------------------------------------------------------------------------------------------------------------------------

1.What is the total amount each customer spent on zomato ?
select a.userid,sum(b.price) as total_amount_spent from sales a inner join product b on a.product_id =b.product_id
group by a.userid;

2.How many days each customer visited zomato app?
select userid,count(distinct created_date) as distinct_days from sales group by userid;

3.What was first product purchased by each customer ?-- will help in future new product launch to target those users
select * from 
(select *,rank() over (partition by userid order by created_date) rnk from sales)  #-- created _date wil be sorted in desc order to check 1st product
t where rnk =1;

4. What is the most purchased food item on the menu and how many times it got purchased by each customers?
select product_id,count(product_id) count_of_product from sales group by product_id order by count(product_id) desc ; -just cout wrt to  product_id
select userid,product_id,count(product_id) from sales group by product_id ,userid order by count(product_id) desc ; -- wrt to each customer 

for top product---1st part
select product_id,count(product_id) count_of_product from sales group by product_id order by count(product_id) desc limit 1;

select top 1 product_id,count(product_id) count_of_product from sales group by product_id order by count(product_id) desc ; -- in ssn sql where limit dont work
how many times it got purchased by each customers --2nd part 
select userid,count(product_id) cnt from sales where product_id =
(select product_id from sales group by product_id order by count(product_id) desc limit 1 )
group by userid;

5. Which item was most popular for each customers?
select * from (
select *,rank() over(partition by userid order by cnt desc )rnk  from                     #wrt cnt each user how many product bought for res product_id
(select userid,product_id,count(product_id) cnt from sales group by userid ,product_id)x)y  
where rnk =1;

6.Which items was purchased first by the customer when they became gold member ?
select * from 
(select x.* ,rank() over (partition by userid order by created_date ) rnk from
(select a.userid,a.created_date,a.product_id ,b.gold_signup_date from sales a inner join goldusers_signup b on a.userid=b.userid and a.created_date >=b.gold_signup_date) x)y where rnk =1;

7.Which item was purchased just before the customer became a member ? 
select * from 
(select x.* ,rank() over (partition by userid order by created_date desc ) rnk from
(select a.userid,a.created_date,a.product_id ,b.gold_signup_date from sales a inner join goldusers_signup b on a.userid=b.userid and created_date <= gold_signup_date) x)y where rnk =1;

8.What is the total orders and amount spent for each member before they become a gold member?

select userid,count(created_date),sum(price) from 
(select x.*, y.price from
(select a.userid,a.created_date,a.product_id ,b.gold_signup_date from sales a inner join goldusers_signup b on a.userid=b.userid and created_date <= gold_signup_date )x inner join product y on x.product_id =y.product_id) z
group by userid;


9.If buying each product generates reward points for e.g -5rs =2 points i.e 2.5 for1 point and each product have differnt purchasing points e.g- p1 -5rs =1 zomato point ,p2 =10rd =5 zomato pints and p3 -5rs =1 zomato point so calculate total point collected by each customer and for which product max points given till now.

calculate total point collected by each customer ?
select userid ,sum(total_points)*2.5  total_money_earned from 
(select e.*, amount/reward_points total_points from
(select d.*, case when product_id =1 then 5 when product_id =2 then 2 when product_id =3 then 5 else 0 end as reward_points from
(select c.userid,c.product_id ,sum(price) amount from
(select a.*, b.price from sales a inner join product b on a.product_id =b.product_id)c group by userid,product_id)d)e)f group by userid ;


calculate for which product max points given till now ?
select * from 
(select * , rank() over(order by total_points_earned desc) rnk from
(select product_id, sum(total_points) total_points_earned from
(select e.*, amount/reward_points total_points from
(select d.*, case when product_id =1 then 5 when product_id =2 then 2 when product_id =3 then 5 else 0 end as reward_points from
(select c.userid,c.product_id ,sum(price) amount from
(select a.*, b.price from sales a inner join product b on a.product_id =b.product_id )c group by userid,product_id)d)e)f group by product_id )f) g where rnk =1;

10.In the first one year after a customer joins the gold program (including their join date ) irrespective of what the customer has purchased they earn 5 zomto points for every 10 rs spent who earned user 1 or user 3 and what was their points earnings in their first yr ?

0.5 points =1 rs  multiline comment /*.......*/

e.g --date cant add with date directly so use dateadd func
select c.*,d.price*0.5 total_points_earned from 
(select a.userid ,a.created_date ,a.product_id ,b.gold_signup_date from sales a inner join goldusers_signup b on a.userid=b.userid and created_date>= gold_signup_date and created_date<=DATEADD(year,1,gold_signup_date))c inner join product d on c.product_id=d.product_id;

11. Rank all cutomers transactions?

select *, rank() over (partition by userid order by created_date ) from sales;   --created_date is in asc order

12 .Rank all the transactions foe each member whenever they are a zomato gold member for every non gold members marked as na -- means rank will be their for member and for non member rank will be na


select d.*,case when rnk=0 then 'NA' else rnk end as ranking from       # cast is used to convert int rnk to varchar and left join used as non gold member also included
(select c.*, cast (case when gold_signup_date is null then 0 else rank() over (partition by userid order by created_date desc) end as varchar ) as rnk from
(select a.userid,a.created_date,a.product_id, b.gold_signup_date from sales a left join goldusers_signup b on a.userid=b.userid and created_date>=gold_signup_date)c)d;

Conclusion --

We have analysed the zomato data set with multiple data and 4 tables and we got inlighted to sales and analysis of non member and gold member customers which will be helpful to increase sales in future to provide offer to target customers for their fav items.
