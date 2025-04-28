# Airbnb Paris Data Analysis Using SQL
# PROJECT BY Ghada SMIDA

## Overview
This project dives into the Airbnb Paris dataset using postgreSQL to uncover trends in pricing, location, room types, cleanliness, and host reputation. The analysis is driven by a set of key business questions, each mapped to a specific objective and backed by SQL queries.

## Dataset

The dataset includes properties data with features like room type, price, cleanliness, guest satisfaction, city center distance, and Superhost status. Initially, the data was stored in two separate files: paris_weekdays.csv and paris_weekends.csv. These two tables have been combined into one dataset for easier analysis.
You can access the dataset from the following URL: 

- **Dataset URL:** [Airbnb Prices in European Cities](https://www.kaggle.com/datasets/thedevastator/airbnb-prices-in-european-cities/data)

## Objectives
1.	Analyze the distribution of room types.
2.	Understand the relationship between price and guest satisfaction.
3.	Evaluate the impact of location on pricing (city center, metro station proximity).
4.	Examine cleanliness ratings and their relationship with guest satisfaction.
5.	Investigate the performance of superhosts vs non-superhosts.
6.	Assess the dynamics of properties on weekends vs weekdays.
7.	Analyze business properties in terms of pricing, guest satisfaction, and location.

## Airbnb Dataset columns

The dataset contains the following columns:

**•	price:** Price of the property.

**•	room_type:** Type of room (Private, Shared,Entire home/apt).

**•	room_shared:** Boolean indicating whether the room is shared.

**•	room_private:** Boolean indicating whether the room is private.

**•	person_capacity:** The number of people the room can accommodate.

**•	host_is_superhost:** Boolean indicating whether the host is a superhost.

**•	multi:** Boolean indicating whether the property has multiple rooms.

**•	biz:** Boolean indicating whether the property is classified as a business property.

**•	cleanliness_rating:** Rating for cleanliness.

**•	guest_satisfaction_overall:** Overall guest satisfaction.

**•	bedrooms:** Number of bedrooms in the property.

**•	city_center_distance:** Distance from the city center.

**•	metro_station_dist:** Distance to the nearest metro station.

**•	lng, lat:** Longitude and Latitude for the property.

**•	is_weekend:** Boolean indicating whether the property data corresponds to a weekend (TRUE) or a weekday (FALSE). 


## Business Problems and Solutions
### 🏠 Room Type Insights
### 1. What are the different types of rooms available and their distribution?
```sql
SELECT room_type, COUNT(*) AS total_properties
FROM airbnb_paris
GROUP BY room_type;
```
**Objective:** Understand the supply of each room type.

## 2.How do average prices differ across room types?

```sql
SELECT room_type, ROUND(AVG(price), 2) AS avg_price
FROM airbnb_paris
GROUP BY room_type
ORDER BY avg_price DESC;

```
**Objective:** Identify pricing trends per room type.

## 3. What’s the average guest satisfaction for each room type?

```sql
SELECT room_type, ROUND(AVG(guest_satisfaction_overall), 2) AS avg_satisfaction
FROM airbnb_paris
GROUP BY room_type
ORDER BY avg_satisfaction DESC;

```
**Objective:** Evaluate guest experience by room type.

## 4. Top 3 Properties with Highest Guest Satisfaction per Room Type

```sql
WITH Top_Satisfaction AS (
    SELECT room_type, price, guest_satisfaction_overall,
           ROW_NUMBER() OVER (PARTITION BY room_type ORDER BY guest_satisfaction_overall DESC) AS rank
    FROM airbnb_paris
)
SELECT room_type, price, guest_satisfaction_overall
FROM Top_Satisfaction
WHERE rank <= 3
ORDER BY room_type, guest_satisfaction_overall DESC;
```
**Objective:** Identify the top 3 properties with the highest guest satisfaction for each room type.

### 💰 Price vs. Value
## 5. Which properties offer the best quality-to-price ratio?

```sql
SELECT id, price, guest_satisfaction_overall,
       ROUND(guest_satisfaction_overall / price, 2) AS quality_price_ratio
FROM airbnb_paris
ORDER BY quality_price_ratio DESC
LIMIT 10;

```
**Objective:** Identify high-value properties.

## 6. What are the most expensive properties on the platform?

```sql
SELECT id, room_type, price
FROM airbnb_paris
ORDER BY price DESC
LIMIT 10;

```
**Objective:** Determine the highest-priced properties on the platform

## 7. Does person capacity influence the price of a property?

```sql
SELECT person_capacity, ROUND(AVG(price), 2) AS avg_price
FROM airbnb_paris
GROUP BY person_capacity
ORDER BY person_capacity;

```
**Objective:** Explore the relationship between room price and person capacity to analyze cost-efficiency.

## 8. Do multi-room properties cost more than single-room ones?

```sql
SELECT multi, ROUND(AVG(price), 2) AS avg_price
FROM airbnb_paris
GROUP BY multi;

```

**Objective:** Compare pricing based on number of bedrooms.

## 9. How can we categorize properties as ‘Premium’ or ‘Economy’?

```sql
SELECT 
  CASE 
    WHEN price > 150 AND cleanliness_rating >= 4.5 AND guest_satisfaction_overall >= 9 THEN 'Premium'
    ELSE 'Economy'
  END AS category,
  COUNT(*) AS total_properties
FROM airbnb_paris
GROUP BY category;

```
**Objective:** Categorize properties as either 'Premium' or ‘economy’ based on price, cleanliness rating, and guest satisfaction to compare different categories of properties.

### 📍 Location Impact
## 10.How does the distance from the city center affect pricing?

```sql
SELECT 
    CASE 
        WHEN city_center_distance < 1 THEN '0-1 km'
        WHEN city_center_distance BETWEEN 1 AND 5 THEN '1-5 km'
        WHEN city_center_distance BETWEEN 5 AND 10 THEN '5-10 km'
        ELSE '10+ km'
    END AS distance_range,
    AVG(price) AS avg_price
FROM airbnb_paris
GROUP BY distance_range
ORDER BY distance_range;

```
**Objective:** Average Price by Distance Range from City Center

## 11. Are properties near the city center more expensive?

```sql
SELECT 
  CASE 
    WHEN city_center_distance <= 1 THEN 'Very Close'
    WHEN city_center_distance <= 3 THEN 'Close'
    ELSE 'Far'
  END AS proximity,
  ROUND(AVG(price), 2) AS avg_price
FROM airbnb_paris
GROUP BY proximity
ORDER BY avg_price DESC;

```
**Objective:**  Compare the prices of properties based on their proximity to the city center.

## 12. How does proximity to metro stations influence pricing?

```sql
SELECT 
  CASE 
    WHEN metro_station_distance <= 0.5 THEN 'Near Metro'
    WHEN metro_station_distance <= 1 THEN 'Moderate Distance'
    ELSE 'Far from Metro'
  END AS metro_proximity,
  ROUND(AVG(price), 2) AS avg_price
FROM airbnb_paris
GROUP BY metro_proximity;

```
**Objective:** Analyze the price differences of properties based on their proximity to the nearest metro station.

### 🧼 Cleanliness & Satisfaction
## 13. What’s the relationship between cleanliness rating and guest satisfaction?

```sql
SELECT cleanliness_rating, 
       ROUND(AVG(guest_satisfaction_overall), 2) AS avg_satisfaction
FROM airbnb_paris
GROUP BY cleanliness_rating
ORDER BY cleanliness_rating DESC;

```
**Objective:** Analyze if cleaner places are better rated.

## 14. Which properties are both exceptionally clean and highly rated?

```sql
SELECT id, cleanliness_rating, guest_satisfaction_overall
FROM airbnb_paris
WHERE cleanliness_rating >= 9 AND guest_satisfaction_overall >= 95
ORDER BY guest_satisfaction_overall DESC
LIMIT 10;

```
**Objective:** Showcase properties that excel in quality.

### 🌟 Superhost Analysis
## 15. Do Superhosts have higher guest satisfaction?

```sql
SELECT host_is_superhost, 
       ROUND(AVG(guest_satisfaction_overall), 2) AS avg_satisfaction
FROM airbnb_paris
GROUP BY host_is_superhost;

```
**Objective:** Compare host type impact on ratings.

## 16.	Do Superhosts charge more?
```sql
SELECT host_is_superhost, ROUND(AVG(price), 2) AS avg_price
FROM airbnb_paris
GROUP BY host_is_superhost;
```
**Objective:**  Investigate whether superhosts tend to charge higher prices for their properties compared to non-superhosts.

## 17.	How do Superhosts compare to others in both price and satisfaction?
```sql
SELECT host_is_superhost,
       AVG(price) AS average_price,
       AVG(guest_satisfaction_overall) AS avg_guest_satisfaction
FROM airbnb_paris
GROUP BY host_is_superhost;

```
**Objective:**  Profile Superhost performance.
### 📅 Weekend vs. Weekday Dynamics
## 18.	How do prices and satisfaction differ between weekdays and weekends?
```sql
SELECT is_weekend, 
       ROUND(AVG(price), 2) AS avg_price,
       ROUND(AVG(guest_satisfaction_overall), 2) AS avg_satisfaction
FROM airbnb_paris
GROUP BY is_weekend;
```
**Objective:**  Compare the average price and guest satisfaction between weekend and weekday bookings.

## 19.	Do room type trends change depending on the day?
```sql
SELECT room_type, is_weekend,
       ROUND(AVG(price), 2) AS avg_price,
       ROUND(AVG(guest_satisfaction_overall), 2) AS avg_satisfaction
FROM airbnb_paris
GROUP BY room_type, is_weekend
HAVING COUNT(*) > 10
ORDER BY room_type, is_weekend;

```
**Objective:**  Analyze room type price and guest satisfaction trends based on whether the booking is made on a weekend or weekday.

### 💼 Business properties Analysis
## 20.	Are Business Properties More Expensive?
```sql
SELECT biz, ROUND(AVG(price), 2) AS avg_price
FROM airbnb_paris
GROUP BY biz;
```
**Objective:**  Compare the average price of business properties with non-business properties to determine pricing trends.

## 🔍 Key Findings

**• Room Types:** Entire homes/apartments dominate the market and are priced considerably higher than private or shared rooms.

**•	Best Value:** Properties with a good balance of guest satisfaction and reasonable pricing offer the best value, making them ideal for solo travelers and couples.

**•	Location's Impact:** Properties closer to the city center and metro stations are priced higher, reflecting their added convenience and desirable location.

**•	Cleanliness Drives Satisfaction:** There is a clear link between cleanliness ratings and guest satisfaction. Highly rated properties consistently maintain excellent cleanliness standards.

**•	Superhost Advantage:** Superhosts tend to receive higher guest satisfaction ratings and can charge slightly more while still receiving positive reviews.

**•	Weekend Dynamics:** Entire homes and private rooms are generally priced higher during weekdays compared to weekends, while shared rooms see a price increase on weekends. This suggests varying demand patterns and possible pricing strategies based on the type of accommodation and the day of the week.
