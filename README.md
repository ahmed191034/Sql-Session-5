# Sql-Session-5
The script employs subqueries, partitioning, and CTEs to analyze customer data. It identifies customers with above-average subscription lengths and recent interactions, ranks them based on subscriptions and ratings, and uses a CTE to find those exceeding average data usage. The approach enhances query readability and maintainability.
# Mutiple row subquires in, not in, any,all
 # find customer whose subscription lengths are longer than average subscription length of
 # all customer
 
 
 select * from subscriptions;
 
 select distinct c.customer_id,c.first_name,c.last_name
 from customer c
 join subscriptions s on c.customer_id=s.customer_id
 where datediff(s.end_date,s.start_date)>all(select avg(datediff(sub.end_date,sub.start_date))
 from subscriptions sub
 group by sub.customer_id); 
 
 # Write a query to find customer whose last interaction date is  more recent than 
 # the average last interaction date
 select * from customer;
 
select customer_id,First_Name,Last_name
from customer 
where last_interaction_date>(select avg(last_interaction_date) from customer);


# corelated subquries
# inner quries will refernce col from oute query- it cannot be executed independent of 
#the outer quey 

# partition
# find the number of feedback entries for each service type for each customer

select customer_id,service_impacted,count(feedback_id)
over (partition by customer_id,service_impacted) as feedback_count
from feedback;


# calculate the average data used for each service type for each customer
select customer_id,service_type,round(avg(data_used),1) as Avg_Data_Used
from service_usage
group by customer_id,service_type,data_used order by customer_id asc;

# By Partition
select customer_id,service_type, avg(data_used)
over (partition by customer_id,service_type) as Average_Data_Used
from service_usage order by customer_id asc;


# rank and dense 
# rank customer according to number of services the ahve subscribed
select * from subscriptions;

select customer_id,count(subscription_id) as total_subs,
rank() over (order by count(subscription_id) desc) as rank_subs
from subscriptions
group by customer_id;

select customer_id,count(subscription_id) as total_subs,
dense_rank() over (order by count(subscription_id) desc) as rank_subs
from subscriptions
group by customer_id;


# rank customer based the total sum of their rating  they have ever given

select fed.customer_id,sum(fed.rating),concat(c.First_Name," ",c.Last_Name ) as Full_Name,
dense_rank() over (order by sum(rating) desc) as Rank_Rating
from feedback fed
inner join customer c on c.customer_id=fed.customer_id
group by customer_id ,rating order by Rank_Rating desc ;



# common table expression(cte)
# cte offer a way to structure complex quries  for clarify and maintainability
# Properties
# temporary result set create which you can reference late in the same quey
# selct,insert,update,delete
# make complex queries more readable by breaking them into parts



# find customer who uses more data then average
with AverageDataUsage as ( select avg(data_used) as avg_Data from service_usage)
select su.customer_id from service_usage su,AverageDataUsage adu
where su.data_used>adu.avg_data
group by su.customer_id; 
