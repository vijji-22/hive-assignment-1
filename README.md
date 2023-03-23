# hive-assignment-1


2. Store raw data into hdfs location

hadoop fs -put /users/thestupidmonk/downloads/sales_order_data.csv /tmp/hive/assignments/

3. Create a internal hive table "sales_order_csv" which will store csv data sales_order_csv .. make sure to skip header row while creating table

create table sales_order_data                                                                                                      
     (                                                                                                                                       
     order_number int,                                                                                                                            
     quantity_ordered int,                                                                                                                       
     price_each float,                                                                                                                         
     order_line_number int,
     sales float,                                                                                                                            
     status string,                                                                                                                       
     qtr_id int,                                                                                                                         
     month_id int,
     year_id int,                                                                                                                            
     product_line string,                                                                                                                       
     msrp int,                                                                                                                         
     product_code string,
     phone string,                                                                                                                            
     city string,                                                                                                                       
     state string,
     postal_code string,                                                                                                                       
     country string,
     territory string,                                                                                                                        
     contact_last_name string,                                                                                                                       
     contact_first_name string,                                                                                                                         
     deal_size string)                                                                                                                             
     row format delimited                                                                                                                    
     fields terminated by ','
     tblproperties("skip.header.line.count"="1");   


4. Load data from hdfs path into "sales_order_csv" 

        load data inpath '/tmp/hive/assignments/sales_data.csv' into tables sales_order_data;


5. Create an internal hive table which will store data in ORC format "sales_order_orc"

        create table sales_order_data_orc                                                                                                           
        (                                                                                                                                       
        order_number int,                                                                                                                            
        quantity_ordered int,                                                                                                                       
        price_each float,                                                                                                                         
        order_line_number int,
        sales float,                                                                                                                            
        status string,                                                                                                                       
        qtr_id int,                                                                                                                         
        month_id int,
        year_id int,                                                                                                                            
        product_line string,                                                                                                                       
        msrp int,                                                                                                                         
        product_code string,
        phone string,                                                                                                                            
        city string,                                                                                                                       
        state string,
        postal_code string,                                                                                                                       
        country string,
        territory string,                                                                                                                        
        contact_last_name string,                                                                                                                       
        contact_first_name string,                                                                                                                         
        deal_size string)
        stored as orc;                                                                                                                          
        row format delimited                                                                                                                    
        fields terminated by ','
        tblproperties("skip.header.line.count"="1"); 


6. Load data from "sales_order_csv" into "sales_order_orc"

        from sales_order_data insert overwrite table sales_order_data_orc select *;


Perform below menioned queries on "sales_order_orc" table :


a. Calculate total sales per year


        select year_id, Round(sum(sales), 3) as total_sales_per_year from sales_order_data_orc group by year_id order by year_id;


b. Find a product for which maximum orders were placed

        select MAX(x.total_order_per_product_code)
        from(
        select product_code, count(order_number) as total_order_per_product_code
        from sales_order_data_orc
        group by product_code
        order by total_order_per_product_code desc) x;


c. Calculate the total sales for each quarter

        select qtr_id, ROUND(sum(sales), 3) as total_sales_per_quarter
        from sales_order_data_orc
        group by qtr_id
        order by qtr_id;


d. In which quarter sales was minimum

        select qtr_id, ROUND(sum(sales), 3) as total_sales_per_quarter
        from sales_order_data_orc
        group by qtr_id
        order by qtr_id
        limit 1;



e. In which country sales was maximum and in which country sales was minimum

        (select country, ROUND(SUM(sales), 3) as sales_for_this_country
        from sales_order_data_orc
        group by country
        order by sales_for_this_country desc limit 1)

        UNION ALL

        (select country, ROUND(SUM(sales), 3) as sales_for_this_country
        from sales_order_data_orc
        group by country
        order by sales_for_this_country limit 1);


f. Calculate quartelry sales for each city

        select city, qtr_id, ROUND(SUM(sales), 3) as total_sales_per_quarter_for_city
        from sales_order_data_orc
        group by city, qtr_id
        order by city, qtr_id;

        select country, ROUND(SUM(sales), 3) as total_sales_per_country
        from sales_order_data_orc
        group by country
        order by total_sales_per_country;


h. Find a month for each year in which maximum number of quantities were sold

        with x as (select year_id, month_id, sum(quantity_ordered) as quantity
        from sales_order_data_orc
        group by year_id, month_id), 

        y as (select year_id, month_id, quantity,
        row_number() over(partition by year_id order by year_id, quantity desc) ranking
        from x)
	
        select year_id, month_id, quantity as quantities_sold
        from y where ranking = 1;

      
