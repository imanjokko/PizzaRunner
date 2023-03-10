*This is the 2nd challenge (Week 2) of the 8 Weeks SQL Challenge by DannyMa. [Click here to view the full challenge](https://8weeksqlchallenge.com/case-study-2/)*

![](https://github.com/imanjokko/PizzaRunner/blob/main/images/Logo.png)

---
# Introduction- Pizza Runner :pizza:

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

---
# Problem Statement
Danny has prepared an entity relationship diagram of his database design, but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.

---
# Business Task
The task and aim of this project is to provide Danny, the owner of Pizza Runner, with more insights into his business. And to help him optimize operations using the data he has provided.

---
# Skills Demonstrated
- Data Wrangling
- Exploratory Data Analysis
- Intermediate SQL
- Data Manipulation

---
# Entity Relationship Diagram
![](https://github.com/imanjokko/PizzaRunner/blob/main/images/ERD.png)

**This databases uses the snowflake schema model**

Follow [this link](https://github.com/imanjokko/PizzaRunner/blob/main/schema%20query.sql) to see the provided SQL code used to create the database schema, as well as the tables.

---
# CASE STUDY
1. [Data Cleaning steps](https://github.com/imanjokko/PizzaRunner/blob/main/Solutions/Data_Cleaning.md)
2. [Part A- Pizza Metrics](https://github.com/imanjokko/PizzaRunner/blob/main/Solutions/Part%20A-%20Pizza%20Metrics.md)
4. [Part B- Runner and Customer Experience](https://github.com/imanjokko/PizzaRunner/blob/main/Solutions/Part%20B-%20Runner%20and%20Customer%20Experience.md)
5. [Part C- Ingredient Optimisation](https://github.com/imanjokko/PizzaRunner/blob/main/Solutions/Part%20C-%20Ingredient%20Optimisation.md)
6. [Part D- Pricing and Ratings](https://github.com/imanjokko/PizzaRunner/blob/main/Solutions/Part%20D-%20Pricing%20and%20Ratings.md)
7. [Part E- Bonus DML Challenges (DML = Data Manipulation Language)](https://github.com/imanjokko/PizzaRunner/blob/main/Solutions/Part%20E-%20Bonus%20DML%20Challenges%20(DML%20%3D%20Data%20Manipulation%20Language).md)

---

# Recommendations
1. Look into 
- What caused restaurant cancellation? There were 2 cancellations, one initiated by the restaurant, and the other by the customer
   - It may be difficult to find out why the customer cancelled, but the restaurant should put a cancellation policy in place that protects all parties, i.e. restaurant, customers, and runners.
- Why is runner 4 not yet active? There are 4 runners, but only runners 1-3 have made any deliveries
- What caused delay in the preparation and delivery of order number 8?, it had only 1 pizza, and a much higher preparation time than other orders containing just 1 pizza. How can this delay be prevented fom re-occuring?

2. Pizza Runner HQ needs to start with marketing and advertisements, especially with the introduction of the new **Supreme pizza**, it is perfect timing.

3. Take user ratings and feedback seriously, this is important for all businesses, especially early stage ones

4. To better maximize profits, Danny could look into his overall operations cost. 
 - Are the current procurement methods as cost effective as they could be?
 - Are the pizza and delivery prices right? Taking note of what is leftover after paying runners

# Conclusion
Overall, the Pizza Runner HQ seems to be doing fairly well for a new business. There are still a lot of areas to explore and look into from a business standpoint, so I hope I did a good enough job for Danny to call me back when he decides to take the next steps at the Pizza HQ. Till then, eat pizza 🍕 and stay safe!


