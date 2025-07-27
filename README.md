**Note:**  This Project documentations, processes and files can be viewed through this Google Drive link and the .pbix file can be downlaoded and viewed via Power BI desktop App. **https://drive.google.com/drive/folders/1IrnEqYQ8DwhyJx8JTCWCvaLsvbV20E3J?usp=drive_link**




2024-2025 Global Ride Sharing Project Report
 
Title: Analyzing Global Ride Sharing dataset for 2024-2025 to discover insights, patterns and correlations to drive business goals

Introduction:

Data Source: Kaggle public datasets

About Dataset:
This dataset contains synthetic data generated to simulate a ride-sharing platform's operations. It includes detailed records across four key tables: fact_rides, vouchers, drivers, and users. The dataset contains 4 csv files, 39 columns, and 25000(K) records which comes across the globe.

Expected data update frequency: Never

1. fact rides: Captures information about individual ride transactions, including ride IDs, timestamps, distances, durations, and fare amounts.
2. vouchers: Contains details about promotional vouchers, discount percentage, and dates.
3. drivers: Provides profiles of drivers, including their IDs, names, and vehicle details.
4. users: Includes user profiles.

The dataset is ideal for data analysis, machine learning model development, and exploration of trends and behaviors in a ride-sharing context. Since it's generated synthetically, it is free from privacy concerns, making it perfect for public use and academic research.



A. Data Preparation and Importation into MS Power BI Model:

All 4 csv files were imported through the MS Power BI data import wizard, prepped in Power Query and loaded to the Power BI Model. All tables were connected with their unique keys to the fact table to have all necessary data in one table. There are Many to Many and One to Many relationships. In order to maintain data privacy, there are columns that were removed as they either contain personal information(PI), PII, or any data that could easily help identify a person.

The csv files were recorded in json format whereby the columns contain up to five data points in a single column. However, I used the Power Query's Split and Merge columns tool to separate and join the columns to have a better look at the data and for seamless analysis. Consistency is key, so I split the columns by either their comma(,) in between, colon(;), brackets({}) and some times by space. Whenever I split the column and I dropped the columns that are not needed or does not have meaning (e.g. columns with ;,{}) and merge columns that are separated by these delimiters. Finally, columns that are not needed for analysis that may or may not have these delimiters were removed from the query for optimization and smooth loading of the entire dashboard. Finally the data is loaded to the MS Power BI data model. 

Some noticeable values from the dataset are;
1. 25000 records in total
2. payment methods were in cash, mobile wallet and credit cards
3. Different Genders(which were concatenated to 3 groups)
4. Use of different Car makes

Removed columns that are not required for analysis include:

1. from driver table-- first name, last name, email, phone number, license plate
2. from user table --- username, last name, first name, email, password hash, phone number
3. from voucher table- start date, end date, updated at(these columns were in text format and if changed to "date", it results in error)
4. from ride table-- start location, end location

Data Profiling, Privacy, Governance and Validation:

i. Numerical columns: Data distribution and Column Statistics, checks for Minimum, Maximum, Average, Count, and Standard deviation of each numerical columns, values for all numerical columns, Checks for Unique values, Error or Empty cells, Distinct values and Value distribution were done with MS Power BI "Power Query's Data Preview tool" from the View tab. Column quality is 100% complete and no duplicate in key fields. All empty records were removed for data consistency. All data validation were checked and assigned appropriate formats. Data Privacy is crucial here, so all data that may lead or help identify a user has been removed to maintain and respect personal rights and choices.

ii. Categorical Columns: Unique and Distinct are checked to ensure there is no whitespaces, misspelt words, trailing spaces. Column statistics was viewed to ensure proper data distribution, this helped in ensuring there is no anomalies, outliers or any record out of context in the data.


B. Data Analysis:

Research Questions:

1. What are the most popular cities for rides?
2. What is the average ride duration and distance by city?
3. What is the distribution of ride ratings?
4. What are the most common payment methods used by customers?
5. How effective are vouchers in driving revenue and customer engagement?
6. What are the demographics of our customer base?
7. How does driver performance impact customer satisfaction and retention?

-- Analysis Procedures:
Power Query and DAX functions were used to perform all calculations in the analysis. Custom columns was used to extract ratings column into a new table and grouped into bins of ratings as string{5, 4, 3, 2, 1} having all count of each customer rating. KPIs were calculated with the following DAX functions in addition to other functions that are used for "time intelligence analysis". The functions include, AVERAGE, SUM, MAXX, COUNTROWS, COUNT, CALCULATE, DATESYTD, DISTINCTCOUNT, FILTER, VAR, MAX, MIN, DIVIDE, SUMMARIZE, TOPN, DISTINCTCOUNTNOBLANK, VALUES, TRUE, LOWER, TRIM, SWITCH. Some exemplary calculations can be drawn from the analysis for demonstration purposes;

1. Creating Calendar table

CalendarTable = 
ADDCOLUMNS(
    CALENDAR(
        MIN(rides[ride_date]),
        MAX(rides[ride_date])
    ),
    "Year", YEAR([Date]), -- this will create a column with year extracted from the date column, the format is (2000)
    "Month", FORMAT([Date], "MMMM"), -- this will create a column with month extracted from the date column, the format is (January)
    "Day", DAY([Date]), -- this will create a column with day extracted from the date column, the format is (1)
    "Weekday", FORMAT([Date], "dddd") -- this will create a column with weekdays extracted from the date column, the format is (Monday)
)


2. Calculating Average Distance per Ride in Kilometers 

AVG Distance per Ride(km) = AVERAGE(rides[ride_distance_miles])


3. This function is calculating Average Revenue per User(ARPU)

AVG Revenue per User ($) = 
VAR TotalRevenue = SUM(rides[fare_amount])
VAR TotalCustomers = DISTINCTCOUNT(rides[customer_id])
RETURN DIVIDE(TotalRevenue, TotalCustomers, 0)


4. This function is calculating Overall Average Rating

Customer Average Rating (%) = 
CALCULATE(
    AVERAGE(ratings[ratings]),
    DATESYTD(rides[datetime].[Date])
)


5. Calculating Average Retention Rate

CustomerRetentionRate = 
VAR TotalCustomers = DISTINCTCOUNT(rides[customer_id])
VAR ReturningCustomers = 
CALCULATE(
    DISTINCTCOUNT(rides[customer_id]),
    FILTER(
        rides,
        CALCULATE(COUNT(rides[ride_id])) > 1
    )
)
RETURN DIVIDE(TotalCustomers, ReturningCustomers, 0)


6. Calculating Driver Retention Rate

Driver Retention Rate (%) = 
VAR TotalDrivers = DISTINCTCOUNT(drivers[driver_id])
VAR ReturningDrivers = 
    CALCULATE(
        DISTINCTCOUNT(drivers[driver_id]),
        FILTER(
            drivers,
            CALCULATE(COUNT(rides[ride_id])) > 1
        )
    )
RETURN DIVIDE(ReturningDrivers, TotalDrivers, 0)


7. Calculating most used customer payment type

Most Popular Payment Type ($) = 
VAR BestPaymentType = 
    TOPN(
        1,
        SUMMARIZE(rides, rides[payment_method], "PaymentCount", COUNT(rides[ride_id])),
        [PaymentCount],
        DESC
    )
RETURN
    MAXX(BestPaymentType, rides[payment_method])


8. Calculating Peak Hour Utilization Rate for Drivers

Peak Hour Utilization Rate (%) = 
DIVIDE(
    [PeakHourDrivers],
    [Total Drivers],
    1
) * 100


9. Calculating Ride Peak hours across hours of day

PeakHourDrivers = 
CALCULATE(
    DISTINCTCOUNT(rides[driver_id]),
    FILTER(
        rides,
        rides[Hour of Day] >= 7 && rides[Hour of Day] <= 9
    )
)


10. Calculating Total cities

Total Cities = DISTINCTCOUNTNOBLANK(rides[city])


11. Calculating top 10 cities

Top10 Cities = 
TOPN(
    10,
    VALUES(rides[city]),
    [Total Cities], DESC
)


12. Calculating Total Revenue

Total Revenue ($) = SUM(rides[fare_amount])


13. Calculating Total Rides made by customer

Total Rides = COUNTROWS(rides)


14. Calculating Voucher Usage Rate
 
Voucher Usage Rate = CALCULATE(DIVIDE([Total Vouchers], DISTINCTCOUNTNOBLANK(rides[voucher_code]), 0))


15. This function is normalizing and grouping the gender column into specified bins(Male, Female, Non-binary) for consistency

Gender Group = 
SWITCH(
    TRUE(),
    LOWER(TRIM([gender])) = "male", "Male",
    LOWER(TRIM([gender])) = "female", "Female",
    "Non-Binary"
)



D. Data Visualizations and Reports:

For easier understanding of analyzed data, charts are created and arranged into dashboard of 2 pages to represent the data well 

a. Global Ride sharing Dashboard Overview Page 1

-- 1. Daily Ride Rush Hours: Area chart
-- 2. Top Revenue Cities: Line and Stacked Column chart
-- 3. Popular Payment Methods: Pie chart
-- 4. Weekly Ride Patterns: Line chart
-- 5. Monthly Ride Growth: Area chart

KPIs:
1. Total Rides(K)
2. Total Revenue($)
3. Average Driver Rating(%)
4. Average Revenue per User($)
5. Most Popular City


b. Global Ride sharing Dashboard Drilldown Page 2

-- 1. Rides by Gender: Pie chart
-- 2. Ratings Distribution: Pie chart
-- 3. Customer Engagement vs. Vouchers Usage: Pie chart
-- 4. Driver Performance Distribution: Pie chart
-- 5. Revenue Analysis by Car Make: Stacked Bar chart
-- 6. Customer Segmentation by Gender: Pie chart
-- 7. Peak Hour Utilization & Revenue Analysis by Car Make: Table
-- 8. Revenue by Country: Stacked Bar chart

KPIs:
1. Average Ride Time(Minutes)
2. Average Revenue per Driver($)
3. Customer Average Rating(%)
4. Driver Retention Rate(%)
5. Most Popular Payment type($)
6. Average Ride Distance(kilometers)


Dashboard Page Navigation:
The dashboard pages are interactive, allowing users to navigate and dig deep through the data by selecting through the Date(Quarter), Country slicers or through the charts. A slicer has been added to the first page, providing dropdown options for both Country and Date(quarter). An information icon is placed in Page 1 which will give users ability to navigate to Page 2 and while a back icon is placed in the Drilldown(Page 2) which returns the user to Page 1. These icons have been configured using the Action feature in the icon formatting pane. 

Slicer and filter options are configured to apply across both pages using the Show Panes group under the View tab. Additionally, features such as Drill-through, Cross-report, and Apply all filters have been enabled to provide a seamless dashboard navigation and to give more comprehensive insights to the user.

Note: On some systems, holding the Ctrl key may be required when clicking the icons.


E. Insights and findings:

1.Peak Ride Hours: Ride usage peaks between midnight–1:00 AM, 7:00–10:00 AM, and 3:00–5:00 PM, which aligns with office and school commute times. Additional spikes occur at 7:00 PM (likely linked to social outings) and 10:00–11:00 PM (returning home). The overall hourly average is 1,060 rides.
2. Revenue by Region: European cities—Stockholm, Oslo, Goteborg, and Copenhagen—generate the highest average ride revenue ($1,898). In contrast, U.S. cities like San Francisco and Washington have the lowest average ride revenue ($857).
3. Day and Payment Preferences: Rides are more frequent on Mondays and Thursdays. All payment methods are used fairly evenly, each contributing approximately 33% of total usage.
4. User Demographics: Male riders slightly dominate usage at 45%, followed closely by females at 44%, indicating nearly equal adoption across genders.
5. Driver Rating Trends: Most rides receive a rating of 4 stars (38%), followed by 5-star ratings (31%), indicating generally favorable driver performance with room for improvement.
6. Car Make Utilization: Vehicles from Ford, GMC, Mazda, and Dodge exhibit the highest ride counts, peak-hour usage, and revenue generation, suggesting strong performance and preference among riders.

Recommendations:
1. Allocate more drivers during peak commute hours and late evenings to meet demand and reduce rider wait times. This will help optimize driver availability.
2. Focus marketing and service enhancements in high-revenue cities like Stockholm and Oslo, and investigate factors causing low revenue in San Francisco and Washington.
3. Launch targeted promotions or loyalty rewards on Mondays and Thursdays to encourage repeat usage as it willl help engage more users.
4. Given the near-equal gender usage, design campaigns and product features that appeal to both male and female demographics.
5. Focus on raising average driver ratings by offering training programs and incentives to improve service quality and achieve more 5-star ratings.
6. Continual investing in and maintaining high-performing vehicle brands (Ford, GMC, Mazda, Dodge) that are proven to generate more revenue and satisfy customers.


Summary:
The analysis of ride-sharing data reveals strong usage patterns tied to daily routines and social behaviors, with Europe outperforming the U.S. cities in revenue generation. Usage trends show balanced gender participation and a clear preference for certain vehicle brands. Ratings are generally positive, but optimization opportunities exist in timing, regional strategy, vehicle selection, and driver service quality.

Conclusion:
The ride-sharing platform demonstrates significant potential, particularly in European cities and peak commuting hours. To sustain and scale this performance, strategic investments in driver experience, fleet quality, and regional engagement are crucial. With nearly balanced usage across gender and payment types, efforts should also focus on inclusive and seamless user experiences. Enhancing these key areas will help maximize revenue and customer satisfaction rates across markets.
