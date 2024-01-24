# Zomato Data Exploration Analysis Project

Over View Of Project:

•	Analysed Zomato sales data, membership info, and product details, driving data-driven business decisions.
•	Extracted insights on customer spending, visit frequency, and popular items, optimizing customer satisfaction and revenue.
•	Evaluated Zomato Gold program's impact on member behaviour, loyalty, and customer engagement, guiding retention strategies.

Data Analysis Using MYSQL:

1. what is the total amount each customer spent in Zomato?

Select s.userid,sum(p.price) as Toatl_amount_spent from sales s right join product p 
on s.product_id=p.product_id group by userid order by userid;

2. How many days visited each customer on zomato?

#Solution-1
select userid, count(distinct created_date) as Distinct_days from sales group by userid order by userid;

#Solution-2
select userid,count(unique_dates)as Distinct_days from(
Select distinct(created_date)as unique_dates,userid from sales) x
 group by userid order by userid;                                                                     
 
3. What was the first product purchased by each customer?

#Solution-1 It returns product_id

select * from(
      select *, rank()over(partition by userid order by created_date) as first_product 
      from sales) x where first_product =1;  
 
 #Solution-2 It returns Product_name

select * from(
      select s.userid,p.product_name,s.created_date, row_number()over(partition by s.userid order by s.created_date) as first_product 
      from sales s inner join product p on s.product_id=p.product_id) x where first_product =1;
      
4. what was the most purchased item on the menu and how many times was it purchased by all customer?

select userid,count(product_id) count from sales where product_id= (select product_id  from sales group by product_id order by count(product_id) desc limit 1) group by userid;

5. Which item was the most popular for each customer?

select * from
(select *,row_number() over(partition by userid order by count desc) as row_no from
(select userid,product_id,count(product_id) as count from sales group by userid,product_id)a)b
where row_no=1

6. Which item was purchased first by the customer after they became a gold member?

select *from(
             select *,row_number()over(partition by userid order by created_date)as row_no from(
           select s.userid,s.product_id,s.created_date from sales s inner join goldusers_signup g 
           on s.userid=g.userid where s.created_date>g.gold_signup_date) x)y where row_no=1;
           
7. Which item was purchased just brfore the customer became a gold member?

select *from (
             select *, row_number () over (partition by userid order by created_date desc) as row_no from (
            select s. userid,s.product_id,s.created_date from sales s inner join goldusers_signup g 
           on s.userid=g.userid where s.created_date>g.gold_signup_date) x)y where row_no=1;

8. What are the total orders and amount spent for each member before they became a member?

#Solution: It clearly Gives information without confusion and output will be well ordered.

Select s.userid,count(p.product_id)as order_purchased,sum(p.price)as Total_amount_spent from
product p inner join sales s on p.product_id=s.product_id 
inner join goldusers_signup g on s.userid=g.userid 
where s.created_date<g.gold_signup_date group by userid order by userid;


9. If buying each product generates points for eg 5rs=2 zomato point and each product has 
 different purchasing points for eg for p1 5rs=1 zomato point, for p2 10rs=5 zomato point 
and p3 5rs=1 zomato point. 

Calculate points collected by each customers and for which product most points
 have been given till now. 
 
 -- Calculated points collected by each customer .  
 
select c.userid, sum(Total_points*2.5) as Total_earnings from(
 select b.userid,b.product_name,Total_amount, floor((Total_amount/points)) as Total_points from
 (select a.*, ( case when product_name='p1' then 5 
             when product_name='p2' then 2 
             when product_name='p3' then 5 else 0 end ) as points from
 (select s.userid,p.product_name,sum(p.price) as Total_amount from sales s inner join product p 
 on s.product_id=p.product_id group by s.userid,p.product_name)a)b)c group by userid order by userid;
 
 -- which product most points given till now?
 
select * from(
  select c.*,row_number() over(order by Total_points desc) as row_no from(
 select b.product_name, floor((Total_amount/points)) as Total_points from
 (select a.*, ( case when product_name='p1' then 5 
             when product_name='p2' then 2 
             when product_name='p3' then 5 else 0 end ) as points from
 (select p.product_name,sum(p.price) as Total_amount from sales s inner join product p 
 on s.product_id=p.product_id group by p.product_name)a)b)c)d where row_no=1;

10 In the first one year after a customer joins the fold program(including their join date)
irrespective of what the customer has purchased they earn 5 zomato points for every 10rs spent who earned
 more and what was their points earnings in their first year? 

 
select c.userid,c.created_date,c.product_id,c.gold_signup_date,Total_points,
row_number() over( order by Total_points desc) as row_no from(
select b.*, floor((Total_amount/points)) as Total_points from (
		select a.*,( case when product_id then 2 else 0 end) as points from
		(select s.userid,s.created_date,g.gold_signup_date,s.product_id,sum(p.price) as Total_amount from
         product p inner join sales s on p.product_id=s.product_id inner join goldusers_signup g on s.userid=g.userid
        where created_date>=gold_signup_date and created_date<=date_add(gold_signup_date,interval 1 year)
		group by userid,created_date,gold_signup_date,product_id)a)b)c ;
 
11: Rank all the transaction of the customers?
 
 Select *,rank() over(partition by userid order by created_date) as rnk from sales;
 
12: Rank all the transactions for each member whenever they are a zomato gold member 

 -- for every non gold member transactions mark as na?
 
 select b.*, (case when gold_signup_date then rnk when gold_signup_date is NULL then 'na' end)  as rank_no from
 (select a.*,rank() over(partition by userid order by created_date desc) as rnk from
 (select s.userid,s.created_date,s.product_id,g.gold_signup_date from sales s left join goldusers_signup g on s.userid=g.userid 
 and  s.created_date>=g.gold_signup_date)a)b;
