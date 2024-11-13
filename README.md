# Bike_Share_Data_Project
## Project Mission
Cyclistic - a bike sharing company wants to understand the differences in usage and riding pattern of the two types of users of their bikes: Casual Riders and Annual Members. The company targets to increase its profit by maximizing the number of annual memberships, which has been concluded to be more profitable than casual rides. This project aims to process and analyze 12 months of user data to find out the main differences between the two types of riders, enabling the company to come up with a new marketing strategy to convert casual riders into annual members.

## What sets us apart:
1. 5,824 bicycles that are geotracked and locked into a network of 692 stations across Chicago.
2. The bikes can be unlocked from one station and returned to any other station.
3. Flexibility of its pricing plans: Single-ride passes,Full-day passes,Annual Memberships.
4. Reclining bikes, hand tricycles, and cargo bikes, making bike-share more inclusive to people with disabilities and riders who can’t use a standard two-wheeled bike.

5. ## Business Task:
The company wants develop an effective marketing strategy for the casual riders to convert them to annual members, in order to increase their profit.

The main task of this project is finding an answer to the question: How do annual members and casual riders use Cyclistic bikes differently?
# installing the packages needed for the analysis
```{r}library(tidyverse) 
library("dplyr")
library(gridExtra)
```
## Loading and Combining Datasets

In this project, data from the Divvy bike-sharing service is used, covering several months of rides in 2021 and 2022. Each month’s data is loaded as a separate CSV file and then combined into a single dataset for analysis. 

### Steps to Load and Combine Data:

1. **Load Monthly Datasets:** Each month's data is stored in a separate CSV file, and each file is read into R using `read.csv`. These files are organized by month and year.

    ```r
    t4 <- read.csv("/kaggle/input/cyclistic-bike-share/202104-divvy-tripdata.csv")
    t5 <- read.csv("/kaggle/input/cyclistic-bike-share/202105-divvy-tripdata.csv")
    t6 <- read.csv("/kaggle/input/cyclistic-bike-share/202106-divvy-tripdata.csv")
    t7 <- read.csv("/kaggle/input/cyclistic-bike-share/202107-divvy-tripdata.csv")
    t8 <- read.csv("/kaggle/input/cyclistic-bike-share/202108-divvy-tripdata.csv")
    t9 <- read.csv("/kaggle/input/cyclistic-bike-share/202109-divvy-tripdata.csv")
    t10 <- read.csv("/kaggle/input/cyclistic-bike-share/202110-divvy-tripdata.csv")
    t11 <- read.csv("/kaggle/input/cyclistic-bike-share/202111-divvy-tripdata.csv")
    t12 <- read.csv("/kaggle/input/cyclistic-bike-share/202112-divvy-tripdata.csv")
    t1 <- read.csv("/kaggle/input/cyclistic-bike-share/202201-divvy-tripdata.csv")
    t2 <- read.csv("/kaggle/input/cyclistic-bike-share/202202-divvy-tripdata.csv")
    t3 <- read.csv("/kaggle/input/cyclistic-bike-share/202203-divvy-tripdata.csv")
    ```

2. **Combine Datasets:** Once each month’s data is loaded, all datasets are combined into a single data frame called `data_combined` using the `rbind` function. This allows for efficient analysis across the entire period.

    ```r
    data_combined <- rbind(t4, t5, t6, t7, t8, t9, t10, t11, t12, t1, t2, t3)
    ```

This combined dataset, `data_combined`, now contains all records for the period April 2021 through March 2022, enabling a comprehensive analysis of bike-sharing trends over the year.




## Finding the number of rows and columns in our dataset
```{r}dim(data_combined)
```
## Preview of the dataset
```{r}
head(data_combined)
```
## Basic summary of our dataset:
```{r}
print ("=========GLIMPSE==============")
```

glimpse(data_combined)
```{r}
print ("==========SUMMARY==============")
```
summary(data_combined)
# Cleaning and Processing the Data
## Dropping the NA value containing rows
```{r}
clean_data_combined <-  drop_na(data_combined)
dim(clean_data_combined)
```
## Chekcking the number of character in the first row
```{r}
nchar(clean_data_combined$ride_id[1])
```
## Filtering only those rows that have a valid ride id
```{r}
clean_data_combined <- clean_data_combined %>%
  filter(nchar(ride_id)==16)
``` 

## Checking the number of rows that had invalid ride id  
```{r}
dim(clean_data_combined)
```
## Checking the types of inputs we have for member_casual and for the rideable type columns
```[r}
table(clean_data_combined$Membership.Type)

table(clean_data_combined$Type)
```
## Chaging the data type of the started_at and ended_at variable from character to POSIXct
```{r}
clean_data_combined$started_at <- as.POSIXct(clean_data_combined$started_at, format = "%Y-%m-%d %H:%M:%S")
clean_data_combined$ended_at <- as.POSIXct(clean_data_combined$ended_at, format = "%Y-%m-%d %H:%M:%S")
```
## Extracting start hour and end hour from the started_at and ended_at columns
```{r}

clean_data_combined$start_hour <- format(clean_data_combined$started_at, "%H")
clean_data_combined$end_hour <- format(clean_data_combined$ended_at, "%H")
```
## Converting the start_time and end_time datatype from char to POSIXct and creating a new column for Travel time
```{r}
clean_data_combined$start_time <- as.POSIXct(clean_data_combined$start_time,format = "%H:%M:%S")
clean_data_combined$end_time <- as.POSIXct(clean_data_combined$end_time,format = "%H:%M:%S")
clean_data_combined$travel_time <- abs(difftime(clean_data_combined$end_time, clean_data_combined$start_time, units= "mins"))

```
## Finding out the Travel Distance
```{r}


library("geosphere")
clean_data_combined$travel_distance <- distGeo(matrix(c(clean_data_combined$start_lng, clean_data_combined$start_lat), ncol = 2), matrix(c(clean_data_combined$end_lng, clean_data_combined$end_lat), ncol = 2))
clean_data_combined$travel_distance <- clean_data_combined$travel_distance/1000
```
## Removing the rows where travel distance is negative
```{r}
clean_data_combined_v2 <- clean_data_combined[clean_data_combined$travel_distance > 0.0,]
dim(clean_data_combined_v2)
clean_data_combined_v2 <- clean_data_combined_v2[clean_data_combined_v2$travel_time > 0.0,]
dim (clean_data_combined_v2)
```
## Finding out the Travel Speed
```{r}
clean_data_combined_v2 <- clean_data_combined_v2 %>%
  mutate(travel_time = as.numeric(travel_time),
         travel_time_hours = travel_time / 60,
         speed = travel_distance / travel_time_hours)
```
## Preview Our Final Dataset
```{r}
colnames(clean_data_combined_v2)
head(clean_data_combined_v2)
```
# Analysis Phase
## Analyzing Travel Time and Distance by User Type

In this section, we analyze the travel time, distance, and speed of rides based on user type. The following R code calculates the mean, median, maximum, and minimum travel times, and then summarizes the data by user type.

```r
# Basic statistics for travel time
mean(clean_data_combined_v2$travel_time) # Straight average (total ride length / rides)
median(clean_data_combined_v2$travel_time) # Midpoint number in the ascending array of ride lengths
max(clean_data_combined_v2$travel_time) # Longest ride
min(clean_data_combined_v2$travel_time) # Shortest ride

# Finding the mean travel_time for each user type
usertype_meantime <- clean_data_combined_v2 %>%
  group_by(Membership.Type) %>%
  summarize(mean_time = mean(travel_time), 
            mean_distance = mean(travel_distance), 
            mean_speed = mean(speed))

print(usertype_meantime)

# Creating visualizations
library(ggplot2)
library(gridExtra)

# Mean Ride Time by User Type
a <- ggplot(data = usertype_meantime) +
  geom_col(mapping = aes(x = Membership.Type, y = mean_time, fill = Membership.Type), position = "dodge") +
  labs(title = "Mean Ride Time by User Type", x = "User  Type", y = "Mean Travel Time (mins)")

# Mean Ride Distance by User Type
b <- ggplot(data = usertype_meantime) +
  geom_col(mapping = aes(x = Membership.Type, y = mean_distance, fill = Membership.Type), position = "dodge") +
  labs(title = "Mean Ride Distance by User Type", x = "User  Type", y = "Mean Travel Distance (km)")

# Mean Ride Speed by User Type
c <- ggplot(data = usertype_meantime) +
  geom_col(mapping = aes(x = Membership.Type, y = mean_speed, fill = Membership.Type), position = "dodge") +
  labs(title = "Mean Ride Speed by User Type", x = "User  Type", y = "Mean Travel Speed (km/hr)")

# Arrange plots in a grid
grid.arrange(a, b, c, nrow = 2, ncol = 2)
```
