# Sam Metz Data Portfolio
Thank you for visiting my data portoflio. 
* Projects (click the link to view the project. click "home" at the end of any section to return here)
    * [Coursera Case Study](#coursera-case-study)
    * [Priority Sort Calculator](#priority-sort)
<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>

## K Means Customer Analyses
I use a k means to analyze a sample data set provided by Big Query. 

### The Dataset
I am working with the_look_ecommerce dataset in Big Query. It consists of 8 tables with sample data for a business. The tables include distribution_centers, events, inventory_items, order_items, products, thelook_ecommerce-table, and users. 

### Mission
My ultimate goal is to use K means to divide the customers in this data set in to groups based on their characteristics and purchase behaviors. This will allow my stakeholders to better understand their customer's and derive an effective marketing plan to increase revenue for the company. 

### Plan
In order to do complete this mission, I will need to combine these tables in to one table that has one row per user and columns for each characteristic that I want the model to consider. I will use the elbow method to determine the optimal number of centroids. After deciding on the number of centroids, I will run the model with that number and assign each customer to a centroid in my formatted table. I will upload this table to Power BI and create visuals to display the insights to my stakeholders.

### Preparation
The relevant tables for my analyses are order_items, users, and products. Below is a sample of each in their original format:

 * order_items: Includes information about the line items included on each order
    *  ![Order Items](Assets/order_items_table.png)
 *  users: Includes Information about each customer
    *  ![users](Assets/users_table.png)
 *  products: Includes details about the products sold by the company
    * ![Order Items](Assets/products_table.png)

I applied the following query to these tables to create my formatted table:

```sql
WITH 
Spend AS 
      (SELECT
        user_id,
        sum(coalesce(order_total,0)) AS Total_Spend,
        avg(order_total) AS avg_order_spend
       FROM
        (SELECT
          user_id,
          order_id,
          sum(coalesce(sale_price,0)) AS order_total
        FROM
          bigquery-public-data.thelook_ecommerce.order_items
        WHERE
          status <> 'Cancelled'
          GROUP BY
            user_id,
            order_id)
      GROUP BY
        user_id),
Last_Purchase AS 
      (SELECT
        DATE_DIFF(CURRENT_DATE(), DATE(MAX(created_at)), DAY) AS days_since_last_purchase,
        user_id
       FROM
            bigquery-public-data.thelook_ecommerce.order_items
        WHERE
          status <> 'Cancelled'
          GROUP BY
            user_id),
Country AS
      (SELECT
        *
        FROM
        (SELECT
          id,
          country,
          1 AS flag
         FROM
          bigquery-public-data.thelook_ecommerce.users
        )
        PIVOT (max(flag) FOR country IN ( 'Brasil',  'Japan',  'United States',  'Colombia',  'Spain',  'China',  'Australia',  'France',  'Germany',  'Belgium',  'South Korea',  'Poland',  'United Kingdom',  'Deutschland',  'Austria'
))),
Category AS 
      (SELECT
       *
       FROM
       (SELECT
        o.user_id,
        p.category,
        1 AS flag
        FROM
          bigquery-public-data.thelook_ecommerce.order_items o
        LEFT JOIN
          bigquery-public-data.thelook_ecommerce.products p
        ON
         o.product_id = p.id 
        WHERE
          o.status <> 'Cancelled'
        )
        PIVOT
          (SUM(flag) FOR Category IN( 'Accessories', 'Plus', 'Swim', 'Active', 'Socks & Hosiery', 'Socks', 'Dresses', 'Pants & Capris', 'Fashion Hoodies & Sweatshirts', 'Skirts', 'Blazers & Jackets', 'Suits', 'Tops & Tees', 'Sweaters', 'Shorts', 'Jeans', 'Maternity', 'Sleep & Lounge', 'Suits & Sport Coats', 'Pants', 'Intimates', 'Outerwear & Coats', 'Underwear', 'Leggings', 'Jumpsuits & Rompers', 'Clothing Sets')))
SELECT 
          ANY_VALUE( CASE WHEN Lower(u.gender) = 'female' THEN 1
          WHEN Lower(u.gender) = 'male' THEN 0
          ELSE NULL
          END) AS Gender,
        ANY_VALUE(u.age) AS age,
        COUNT(DISTINCT(o.order_id)) AS Order_Count,
        ANY_VALUE(s.avg_order_spend) AS avg_order_spend,
        sum(coalesce(o.sale_price,0)) AS total_spend,
                          COALESCE(ANY_VALUE(co.Brasil),0) AS Brasil,
                          COALESCE(ANY_VALUE(co.Japan),0) AS Japan,
                          COALESCE(ANY_VALUE(co.`United States`),0) AS United_States,
                          COALESCE(ANY_VALUE(co.Colombia),0) AS Colombia,
                          COALESCE(ANY_VALUE(co.Spain),0) AS Spain,
                          COALESCE(ANY_VALUE(co.China),0) AS China,
                          COALESCE(ANY_VALUE(co.Australia),0) AS Australia,
                          COALESCE(ANY_VALUE(co.France),0) AS France,
                          COALESCE(ANY_VALUE(co.Germany),0) AS Germany,
                          COALESCE(ANY_VALUE(co.Belgium),0) AS Belgium,
                          COALESCE(ANY_VALUE(co.`South Korea`),0) AS South_Korea,
                          COALESCE(ANY_VALUE(co.Poland),0) AS Poland,
                          COALESCE(ANY_VALUE(co.`United Kingdom`),0) AS United_Kingdom,
                          COALESCE(ANY_VALUE(co.Deutschland),0) AS Deutschland,
                          COALESCE(ANY_VALUE(co.Austria),0) AS Austria,


        ANY_VALUE(lp.days_since_last_purchase) AS days_since_last_purchase,
                          COALESCE(ANY_VALUE(c.Accessories), 0) AS Accessories,
                          COALESCE(ANY_VALUE(c.Plus), 0) AS Plus,
                          COALESCE(ANY_VALUE(c.Swim), 0) AS Swim,
                          COALESCE(ANY_VALUE(c.Active), 0) AS Active,
                          COALESCE(ANY_VALUE(c.`Socks & Hosiery`), 0) AS Socks_Hosiery,
                          COALESCE(ANY_VALUE(c.Socks), 0) AS Socks,
                          COALESCE(ANY_VALUE(c.Dresses), 0) AS Dresses,
                          COALESCE(ANY_VALUE(c.`Pants & Capris`), 0) AS Pants_Capris,
                          COALESCE(ANY_VALUE(c.`Fashion Hoodies & Sweatshirts`), 0) AS Fashion_Hoodies_Sweatshirts,
                          COALESCE(ANY_VALUE(c.Skirts), 0) AS Skirts,
                          COALESCE(ANY_VALUE(c.`Blazers & Jackets`), 0) AS Blazers_Jackets,
                          COALESCE(ANY_VALUE(c.Suits), 0) AS Suits,
                          COALESCE(ANY_VALUE(c.`Tops & Tees`), 0) AS Tops_Tees,
                          COALESCE(ANY_VALUE(c.Sweaters), 0) AS Sweaters,
                          COALESCE(ANY_VALUE(c.Shorts), 0) AS Shorts,
                          COALESCE(ANY_VALUE(c.Jeans), 0) AS Jeans,
                          COALESCE(ANY_VALUE(c.Maternity), 0) AS Maternity,
                          COALESCE(ANY_VALUE(c.`Sleep & Lounge`), 0) AS Sleep_Lounge,
                          COALESCE(ANY_VALUE(c.`Suits & Sport Coats`), 0) AS Suits_Sport_Coats,
                          COALESCE(ANY_VALUE(c.Pants), 0) AS Pants,
                          COALESCE(ANY_VALUE(c.Intimates), 0) AS Intimates,
                          COALESCE(ANY_VALUE(c.`Outerwear & Coats`), 0) AS Outerwear_Coats,
                          COALESCE(ANY_VALUE(c.Underwear), 0) AS Underwear,
                          COALESCE(ANY_VALUE(c.Leggings), 0) AS Leggings,
                          COALESCE(ANY_VALUE(c.`Jumpsuits & Rompers`), 0) AS Jumpsuits_Rompers,
                          COALESCE(ANY_VALUE(c.`Clothing Sets`), 0) AS Clothing_Sets

      FROM
        bigquery-public-data.thelook_ecommerce.users u
      LEFT JOIN
        bigquery-public-data.thelook_ecommerce.order_items o
      ON
        u.id = o.user_id
      LEFT JOIN
        bigquery-public-data.thelook_ecommerce.products p
      ON
        o.product_id = p.id
      LEFT JOIN
        Category c
      ON
        c.user_id = u.id
      LEFT JOIN
        Country co
      ON
        co.id = u.id
      LEFT JOIN
        Last_Purchase lp
      ON
        LP.user_id = u.id
      LEFT JOIN
        Spend s
      ON
        s.user_id = u.id
      WHERE
        o.status <> 'Cancelled'
      GROUP BY
        u.id;
```
<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>
## Priority Sort
I use Google Sheets to sort tasks based on priority creating a dynamic todo list that always brings the highest priority task to the top.

### Methodology
A task's priority is determined by it's importance category (where does the task come from), its effort level, and its due date. 

### Priority Sort Design
* Set up 2 tabs. Tab one is the actual to do list calculator that will sort tasks. Tab 2 is the points tab.
   * The points tab contains 3 separate tables: Importance table, effort table, and urgency table.
      * Importance: what category does this task fall in to? rank these categories from most to least important 
         * Assign each category an importance letter and a point value. Lower points result in higher priority rank
      * Effort: how complex is the task? how much time will it take?
         * Rank these complexity categories from least to most effort and assign a point value to each complexity category. The less complex the lower the points and higher priority.
      * Due Date: set urgency preferences: As a due date approaches, the point value decreases and the task becomes higher priority. My preference is that due date has the most impact on priority.
         * ![Priority Sort](Assets/Personal_Priorit_Sort.png)
   * The Sort Calculator tab setup:
      * Urgency Point Column setup:
         * The due date is entered manually for each task.
            * Once the due date is entered, They Day left column calculates the number of days left using the formula: =F2 - TODAY(). This finds the difference between the current date and the due date. The day left column is formatted as a whole number.
            * ![Day Left](Assets/Day_Left_Formula.png)
               * The urgency column assigns an urgency category based on the number of days left until the due date using formula: =IF(G2<=0,1, IF(G2<=3,2,IF(G2<=5,3,IF(G2<=20,4,5)))) as shown below. If the day left column contains a value less than or equal to 0 then it returns 1. If the day left column contains a value that is less than or equal to 3 then it returns 2. If the day left column contains a value that is less than or equal to 5 then it returns a 3. If the day left column contains a value that is less than or equal to 20 then it return 4. Otherwise, it returns a value of 5.
               * ![Urgency Formual](Assets/Urgency_Formula.png)
                  * The urgency point column looks up a point value by referencing the value in the urgency column and returning the corresponding point value from the urgency table in the points tab using formula: =VLOOKUP(E2,Points!I:J,2,FALSE) as shown below:
                     * ![Urgency Point](Assets/Urgency_Point_Formula.png)
      * Effort Point Column Setup:
         * The effort is entered manually for each task.
            * The effort point column looks up the effort point value by referencing the value entered in the effort column in the effort table of the points tab using formula: =VLOOKUP(D2,Points!E:F,2,FALSE) as shown below:
            * ![Effort Poing](Assets/Effort_Point_Formula.png)
      * Importance Point Column Setup:
         * The Importance category is entered manually for each task
            * The importance point column looks up the importance point value by referencing the value entered in the importance column in the importance table of the points tab using formula: =VLOOKUP(C2,Points!A:B,2,FALSE) as show below:
            * ![Importance Point](Assets/Importance_Point_Formula.png)
      * Priority Column Setup:
         * The priority column adds all of the points columns using formula: =SUM(H2:J2) as shown below:
         * ![Priority Formula](Assets/Priority_Formula.png)
      * Sorting:
         * Autofill all formulas to the cells below them
         * Sort the entire table by the priority column lowest to highest
         * The tasks that are sorted to the top of the list are the tasks that need to get done first
         * Edit the point values and categories to match your preferences.           

[Home](#sam-metz-data-portfolio)








<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>



## Coursera Case Study
I analyze the data of Fitbit users to derive marketing insights for my stakeholders. This is a case study for my Google Data Analytics Certificate. To complete this task, I used the 6 step data analyses process outlined in the course: ask, prepare, process, analyze, share, act. 
### Prompt
* You are a data analyst for a company called Bellabeat. Bellabeat makes wearable fitness devices.
* Your team has been asked to anazlyze trends in smart fitness device usage in an effort to help Bellabeat reach their target market more effectively.
* A data set about Fitbit users is provided by the company.
* Questions:
    * What are some trends in smart device usage?
    * How could these trends apply to Bellabeat's customers?
    * How could these trends influence Bellabeat's marketing strategy?
### Phase One: Ask
* Are users wearing the watch as a fashionable accessory? Do they wear it all day?
* Do users frequently log their weight in the device?
* What time of day do users typically exercise?
* How far do users go per day?
* How much sleep do the users get per night?
### Phase Two: Prepare
* Is the data reliable?
    * For purposes of this case study, I will use the dataset provided. However, it is important to note the we are trying to make a claim about Fitbit users. Therefore, our population would be 38.5 million people. If we wanted to make claims about this population with 95% certainty, we would need a sample size of at least 385 participants. If this were a real life scenario, I would recommend that we find a more representative data set.
    * Further, the source of this data is Amazon Mechanical Turk. Therefore, this dataset is not random and it was not vetted for bias.
* Is the data the original set?
    * Yes
* Is the data comprehensive?
    * The data includes enough information to allow us to answer our questions about the users included.
* Is the data current?
    * This data set is updated annually.
### Phase Three and Four: Process and Analyze
* Are users wearing the watch as a fashionable accessory? Do they wear it all day?
    * For this task, I used the heartrate_seconds_merged.csv tables. There is one for the first date range (3/12-4/11), and one for the second date range (4/12-5/12). A preview of this table is shown below. A user must be wearing the watch for their heartrate to be tracked. Therefore, this table will show us how long each user wears their watch:
    * ![Heartrate Table Example](Assets/heart_rate_table.jpg)
    * First, I cleaned this data by formatting the id’s to a number and expanded the date row to include all information.
    * Second, I uploaded the tables to Bigquery.
    * Third, I applied the following steps to the datasets for both date ranges:
         * I found the number of days that each user was wearing their fitbit during the date range.
            * To do this, I applied the following SQL query to the heartrate table:
            * ![Day Count Query](Assets/day_count_query.jpg)
            * Below is the table created by this query:
            * ![Day Count Table Example](Assets/Day_Count_Table_Example.png)
                 * I then ran the following query on the table above to find the total number of days per user during the range:
                 * ![Day Count Final Sum Query](Assets/Day_Count_Final_Sum_Query.png)
                 * Below is the table that resulted from running this query:
                 * ![Days by User Table Example](Assets/Days_By_User_Table_Example.png)
         * Now that I knew the total number of days per user for this date range, I needed to find the number of hours that each user wore the watch during the date range.
            *  To do this, I ran the following query on the original heart rate table:
            *  ![Hour Count Query1](Assets/Hour_Count_Query1.png)
            *  This resulted in the following table with the start and end time on each day for each user:
            *  ![Hour Count Table1](Assets/Hour_Count_Table1.png)
            *  Then, I ran the following query on the table above to add the total duration per user:
            *  ![Hour Count Query2](Assets/Hour_Count_Query2.png)
            *  This resulted in the following table:
            *  ![Hour Count Table2](Assets/Hour_Count_table2.png)
            *  I joined this table with the day count table using the following query:
            *  ![Hour Count Query3](Assets/Hour_Count_Query3.png)
            *  This resulted in a table that showed the Fitbit Id, total days the fitbit was used, and the total hours the fitbit was used. Below is a snip of the table for the 3/12-4/11 dataset:
            *  ![Hour Count Table3](Assets/Hour_Count_Table3.png)
         * Since I applied these steps to both date ranges, I ended up with 2 tables.
            * I combined these tables in Excel.  
            * In a new tab, I used a sumif function to total the number of hours per user in one column and the number of days per user in another.
               * I added a third column that divides the hours used by the days used to find each user's average daily use.
            * In a new sheet, I created a table that used a countif function to add the number of users that fell within each usage range as shown below:
            *  ![Final Usage Range Table](Assets/Final_Usage_Range_Table.png)
            * I used Excel to create a chart from this table as shown below:
            * ![Final Usage Range Chart](Assets/Final_Usage_Range_Chart.png)
   * This chart shows us that most Fitbit users wear their watch between 12 and 24 hours per day. My recommendation to my stakeholders would be to market their product to users who are looking for a fashionable watch that they can wear all day, not just while working out.
* How often are users utilizing the weight log function?
   * To answer this, I prepared the weight log data sets by extracting the date from the column with Date-Time using flash fill in Excel. Then, I used the following sql query to merge the two tables, and count the number of distinct id’s that fell within each date range. This showed the number of unique user Id's that used the weight log each week. I chose to measure this on a weekly basis because this is a common weigh in frequency for someone who is trying to lose weight.
   * ![Weight Log User Count Query1](Assets/Weight_Log_User_Count_Query1_Take2.jpg)
   * This query created a table that I opened in Excel. In Excel, I added a percentage column that divided the number of unique users each week by the population size (30), and created a chart that plotted the weekly percentages on a line graph as shown below:
   * ![Percent of Population Logging Weight Chart](Assets/Percent_of_Population_Logging_Weight_Chart.png)
   * The chart above shows that only a very small percentage of our population is using the weight log function. Based on this data, I would recommend that we investigate why this function is not being used. It could mean that only a small percentage of our population is trying to lose weight. Alternatively, it could mean that the weight log funtion is not easy to use. We could search for datasets that shed light on the percentage of Fitbit users that use the device for weight loss or create a survey that attempts to uncover how Fitbit users feel about the weight log function. This information could provide valuable insight about whether or not we should market to people who are trying to lose weight. Depending on what we find, it could also mean a step toward improving the weight log function for our device.
* What time of day do users exercise?
   * To answer this question, I used the hourly calories tables for both date ranges.
      * I started by opening these tables in Excel. Then, I extracted the hour from the date-hour column using flash fill.
      * Then, I downloaded the tables in to Bigquery and ran the following query on them:
      * ![Hourly Calories Query](Assets/Hourly_Calories_Query_image.png)
      * The query above combines the datasets from both date ranges, averages the calories for each hour, and groups the results by hour. I opened this table in Excel and created the following graph:
      * ![Hourly Calories Graph](Assets/Hourly_Calories_Graph.png)
   * Our population burns the most calories at 7PM each day. Based on this information, I would recommend that my stakeholders market to working professionals. Our advertisements could be timed to get the most visibility by showing them in gyms at 7PM or, right after, when these users are coming home from the gym. This information could also inform decisions about the watch design. It might make sense to create a watch that can be worn while at work so that users do not need to remember to put it on before leaving for their workout.
* How far do users go per day?
   * To answer this question, I used the daily activity datasets. These tables show the distance that each user went each day. There is one table for the date range 3/12-4/11 and one for 4/12-5/12. I wanted to make sure there weren't duplicate dates for any one user. To do this, I applied the following query to each of the tables:
   * ![Distance_Duplicate_Query](Assets/Distance_Duplicate_Query.png)
   * This confirmed that there weren't duplicate dates for any user. If there were duplicate dates per user, I would have to sum the distances for each date. 
   * Then, I ran the following query to combine the data sets from both ranges, find the average distance per user, and then count the number of users that fell within each distance range:
   * ![Distance Range Query](Assets/Distance_Range_Query.png)
   * I downloaded the table that this query created in Excel and made the folowing graph to show how many users fell within each kilometer range for an average day:
   * ![Distance Range Graph](Assets/Distance_Per_Day_Chart.png)
* Most users fall between 0-9 Km per day. The majority of users travel 3-6 Km per day. The daily activity table also has a column that shows sedentary distance. This column typicaly shows a 0 or a very small number. This means that most of the distance being tracked is active distance. Therefore, we can assume that the users are traveling these distances for their workouts or, at the very least, walking. We could use this information to market to runners or walkers that usually travel this distance. We could also design a watch that is user friendly for runners and walkers and has an effective mileage tracker.
* How much sleep do users get per night?
   * To answer this question, I ran the following query on the sleep minutes table:
   * ![Sleep Query](Assets/Sleep_Query_2.png)
   * This produced a dataset that I downloaded in to Tableau. There, I created the following viz:
   * ![Sleep Viz](Assets/Sleep_Viz.png)
   * Most users get between 5-9 hours of sleep per night. 23/30 users wore their watch to bed in this dataset. This shows that the sleep tracking function is used by the majority of Fitbit users. Bellabeat should have a sleep tracking feature and market this feature to their users.
### Phase 5 and 6: Share and Act
* Are users wearing the watch as a fashionable accessory? Do they wear it all day?
   * ![Final Usage Range Chart](Assets/Final_Usage_Range_Chart.png)
   * Yes, most users wear their watch all day. I recommend that we market our product to users who are looking for a fashionable watch. We should also make sure our watch is fashionable enough to be worn all day.
* Do users frequently log their weight in the device?
   * ![Percent of Population Logging Weight Chart](Assets/Percent_of_Population_Logging_Weight_Chart.png)
   * Only a very small percentage of Fitbit users use the weight log function. Further investigation is needed to find out why. This may not be a strong area of focus for our product until that investigation is done. 
* What time of day do users typically exercise?
   * ![Hourly Calories Graph](Assets/Hourly_Calories_Graph.png)
   * Users burn the most calories at 7PM each day. I would recommend that we market to working professionals with this time frame in mind. We should also create a watch that fits a professional dress code to accommodate these users.
* How far do users go per day?
   * ![Distance Range Graph](Assets/Distance_Per_Day_Chart.png)
   * Most users fall between 0-9 Km per day. We should design a watch that is user friendly for runners and walkers and has an effective mileage tracker.
* How much sleep do the users get per night?
   * ![Sleep Viz](Assets/Sleep_Viz.png)
   * Most users get between 5-9 hours of sleep per night. 23/30 users wore their watch to bed in this dataset. Bellabeat should have a sleep tracking feature and market this feature to their users.

[Home](#sam-metz-data-portfolio)
