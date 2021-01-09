## Stats 306 Final Project - Version 2

**Team Members:**

Chin, Natalie Ying Li (uniqname: `nylchin`)

Guo, Walter (uniqname: `wbguo`)

Jiang, Celina (uniqname: `shuyuej`)

Yang, Zhixuan (uniqname: `yzhixuan`)

Each team member equally contributed.

### 1. Introduction

Hotel bookings provide crucial insight into the tourism industry in different geographical locations at different times. This project involves exploring a hotel dataset comprehend bookings due to arrive between the 1st of July of 2015 and the 31st of August 2017, including bookings that effectively arrived and bookings that were canceled. Consequently, this research seeks to study consumer behavior among the hotel industry over time and by geographic location. Specifically, we look at the general timeline of two different hotel bookings (Resort Hotel and City Hotel), the reservation and cancellation behavior among travellers, style of travelling with children and babies, and the budget and cost of hotel stays.

### 2a. Package Loading

To reproduce the code and results throughout this project we will need to load the following packages.


```R
library(tidyverse)
library(lubridate)      # to come up with general date time for analysis 
library(forcats)        # for reordering factor levels 
library(stats)          # for statistics summary purposes
library(dplyr)          # transforming (joining, summarizing, etc.) data
library(countrycode)    # to create a new column with countrys' full name
library(modelr)         # modelling functions that work with pipe
```

    ── [1mAttaching packages[22m ─────────────────────────────────────── tidyverse 1.3.0 ──
    
    [32m✔[39m [34mggplot2[39m 3.3.2     [32m✔[39m [34mpurrr  [39m 0.3.4
    [32m✔[39m [34mtibble [39m 3.0.3     [32m✔[39m [34mdplyr  [39m 1.0.2
    [32m✔[39m [34mtidyr  [39m 1.1.2     [32m✔[39m [34mstringr[39m 1.4.0
    [32m✔[39m [34mreadr  [39m 1.3.1     [32m✔[39m [34mforcats[39m 0.5.0
    
    ── [1mConflicts[22m ────────────────────────────────────────── tidyverse_conflicts() ──
    [31m✖[39m [34mdplyr[39m::[32mfilter()[39m masks [34mstats[39m::filter()
    [31m✖[39m [34mdplyr[39m::[32mlag()[39m    masks [34mstats[39m::lag()
    
    
    Attaching package: ‘lubridate’
    
    
    The following objects are masked from ‘package:base’:
    
        date, intersect, setdiff, union
    
    
    

### 2b. Data import


```R
df_raw <- read.csv(file = "hotel_bookings.csv", header = TRUE)
```

#### Description of variables: 

1) `hotel`: 2 types of hotel (Resort Hotel, City Hotel)

2) `is_canceled`: Value indicating if the booking was canceled (1) or not (0)

3) `lead_time`: Number of days that elapsed between the entering date of the booking into the PMS and the arrival date

4) `arrival_date_year`: Year of arrival date

5) `arrival_date_month`: Month of arrival date with 12 categories: “January” to “December”

6) `arrival_date_week_number`: Week number of the arrival date

7) `arrival_date_day_of_month`: Day of the month of the arrival date

8) `stays_in_weekend_nights`: Number of weekend nights (Saturday or Sunday) the guest stayed or booked to stay at the hotel

9) `stays_in_week_nights`: Number of week nights (Monday to Friday) the guest stayed or booked to stay at the hotel

10) `adults`: Number of adults

11) `children`: Number of children

12) `babies`: Number of babies

13) `meal`: Type of meal booked. Categories are presented in standard hospitality meal packages:	
Undefined/SC – no meal package;
BB – Bed & Breakfast;
HB – Half board (breakfast and one other meal – usually dinner);
FB – Full board (breakfast, lunch and dinner)

14) `country`: Country of origin. Categories are represented in the ISO 3155–3:2013 format

15) `market_segment`: Market segment designation. In categories, the term “TA” means “Travel Agents” and “TO” means “Tour Operators”

16) `distribution_channel`: Booking distribution channel. The term “TA” means “Travel Agents” and “TO” means “Tour Operators”

17) `is_repeated_guest`: Value indicating if the booking name was from a repeated guest (1) or not (0)

18) `previous_cancellations`: Number of previous bookings that were cancelled by the customer prior to the current booking

19) `previous_bookings_not_canceled`: Number of previous bookings not cancelled by the customer prior to the current booking

20) `reserved_room_type`: Code of room type reserved. Code is presented instead of designation for anonymity reasons

21) `assigned_room_type`: Code for the type of room assigned to the booking. Sometimes the assigned room type differs from the reserved room type due to hotel operation reasons (e.g. overbooking) or by customer request. Code is presented instead of designation for anonymity reasons

22) `booking_changes`: Number of changes/amendments made to the booking from the moment the booking was entered on the PMS until the moment of check-in or cancellation

23) `deposit_type`: Indication on if the customer made a deposit to guarantee the booking. This variable can assume three categories: No Deposit – no deposit was made;

24) `agent`: ID of the travel agency that made the booking

25) `company`: ID of the company/entity that made the booking or responsible for paying the booking. ID is presented instead of designation for anonymity reasons

26) `days_in_waiting_list`: Number of days the booking was in the waiting list before it was confirmed to the customer

27) `customer_type`: Type of booking, assuming one of four categories: 
Contract - when the booking has an allotment or other type of contract associated to it; Group – when the booking is associated to a group; Transient – when the booking is not part of a group or contract, and is not associated to other transient booking; Transient-party – when the booking is transient, but is associated to at least other transient booking

28) `adr`: defined as the "Average Daily Rate"

29) `required_car_parking_spaces`: Number of car parking spaces required by the customer

30) `total_of_special_requests`: Number of special requests made by the customer (e.g. twin bed or high floor)

31) `reservation_status`: Reservation last status, assuming one of three categories:
Canceled – booking was canceled by the customer;
Check-Out – customer has checked in but already departed;
No-Show – customer did not check-in and did inform the hotel of the reason why

32) `reservation_status_date`: Date at which the last status was set. This variable can be used in conjunction with the reservation_status to understand when was the booking canceled or when did the customer checked-out of the hotel

### 2c. Data Tidying

The original dataset was tidy since it satisfied the three conditions: 1) Each variable must have its own column. 2) Each observation must have its own row. 3) Each value must have its own cell. In terms of data cleaning and transformation, we first removed the variables `company` and `agent` because we won't be using those variables for our data analysis purposes. Secondly, we modify our arrival dates to proper ymd date formet. Next, we modify all relevant variables to factor variables for the convenience of our Exploratory Data Analysis (EDA) later. Besides that, we created a new variables called `total_nights_stayed`, where we sum up `stays_in_weekend_nights` and `stays_in_week_nights`. Then, we created a new variable called `country_full_name` to contain each country's full name for convenienve during analysis. Furthermore, we created a new variable called `cancel_lead_time`, where we find the number of days between cancellation date and the initial scheduled arrival date. This variable will help us in calculating how late do a customer cancel their bookings. Since `adr` is defined by dividing the sum of all lodging transactions by the total number of staying nights, it should be non-negative. However, we noticed that one observation had the negative adr (-6.83) and therefore we decided to remove this row. Hence, we now have the data cleaned, in a tidy format, and ready to analyze as the table below illustrates.


```R
# Remove variable `company` and `agent`
df1 <- df_raw %>% select(-c(company, agent))

# Proper arrival date
arr_month <- c("January", "February", "March", "April", "May", "June", "July", 
               "August", "September", "October", "November", "December")
df1$arrival_date_month = match(substr(df1$arrival_date_month, 1, 3), month.abb)
dates <- paste(df1$arrival_date_year,df1$arrival_date_month,df1$arrival_date_day_of_month,sep="-")
df2 <- df1 %>% unite(col = "arrival_date", sep = "-", 
                     c('arrival_date_year', 'arrival_date_month', 'arrival_date_day_of_month')) %>%
  mutate(arrival_date = as.Date(arrival_date, format='%Y-%m-%d')) %>%
  mutate(reservation_status_date = as.Date(reservation_status_date, format='%Y-%m-%d'))

# Change relevant variables to factor
df3 <- df2 %>% mutate_at(vars(hotel, meal, market_segment, distribution_channel, 
                              reserved_room_type, assigned_room_type, deposit_type,
                              customer_type, reservation_status, customer_type, deposit_type, 
                              market_segment, distribution_channel), as.factor)

# Create new variable for total nights of stay
df4 <- df3 %>% mutate(total_nights_stayed = stays_in_weekend_nights + stays_in_week_nights)

# Create new variable for country full name and make sure that adr is non-negative
df4$country[df2$country == "CN"] <- "CHN" # CN convert from iso2c to iso3c form
df5 <- df4 %>% filter(!country %in% c("NULL", "TMP")) %>% 
        filter(adr>=0) %>%
        mutate(country_full_name = countrycode(country,
                                               origin = "iso3c", destination = "country.name"))    

# Create new variable for number of days between cancellation and initial scheduled arrival date
df <- df5 %>% mutate(cancel_lead_time = ifelse(reservation_status == "Canceled",
                     day(as.period(interval(reservation_status_date, arrival_date))), NA)) 

head(df)
```


<table>
<caption>A data.frame: 6 × 31</caption>
<thead>
	<tr><th></th><th scope=col>hotel</th><th scope=col>is_canceled</th><th scope=col>lead_time</th><th scope=col>arrival_date</th><th scope=col>arrival_date_week_number</th><th scope=col>stays_in_weekend_nights</th><th scope=col>stays_in_week_nights</th><th scope=col>adults</th><th scope=col>children</th><th scope=col>babies</th><th scope=col>⋯</th><th scope=col>days_in_waiting_list</th><th scope=col>customer_type</th><th scope=col>adr</th><th scope=col>required_car_parking_spaces</th><th scope=col>total_of_special_requests</th><th scope=col>reservation_status</th><th scope=col>reservation_status_date</th><th scope=col>total_nights_stayed</th><th scope=col>country_full_name</th><th scope=col>cancel_lead_time</th></tr>
	<tr><th></th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;date&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>⋯</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;date&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>Resort Hotel</td><td>0</td><td>342</td><td>2015-07-01</td><td>27</td><td>0</td><td>0</td><td>2</td><td>0</td><td>0</td><td>⋯</td><td>0</td><td>Transient</td><td> 0</td><td>0</td><td>0</td><td>Check-Out</td><td>2015-07-01</td><td>0</td><td>Portugal      </td><td>NA</td></tr>
	<tr><th scope=row>2</th><td>Resort Hotel</td><td>0</td><td>737</td><td>2015-07-01</td><td>27</td><td>0</td><td>0</td><td>2</td><td>0</td><td>0</td><td>⋯</td><td>0</td><td>Transient</td><td> 0</td><td>0</td><td>0</td><td>Check-Out</td><td>2015-07-01</td><td>0</td><td>Portugal      </td><td>NA</td></tr>
	<tr><th scope=row>3</th><td>Resort Hotel</td><td>0</td><td>  7</td><td>2015-07-01</td><td>27</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>⋯</td><td>0</td><td>Transient</td><td>75</td><td>0</td><td>0</td><td>Check-Out</td><td>2015-07-02</td><td>1</td><td>United Kingdom</td><td>NA</td></tr>
	<tr><th scope=row>4</th><td>Resort Hotel</td><td>0</td><td> 13</td><td>2015-07-01</td><td>27</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>⋯</td><td>0</td><td>Transient</td><td>75</td><td>0</td><td>0</td><td>Check-Out</td><td>2015-07-02</td><td>1</td><td>United Kingdom</td><td>NA</td></tr>
	<tr><th scope=row>5</th><td>Resort Hotel</td><td>0</td><td> 14</td><td>2015-07-01</td><td>27</td><td>0</td><td>2</td><td>2</td><td>0</td><td>0</td><td>⋯</td><td>0</td><td>Transient</td><td>98</td><td>0</td><td>1</td><td>Check-Out</td><td>2015-07-03</td><td>2</td><td>United Kingdom</td><td>NA</td></tr>
	<tr><th scope=row>6</th><td>Resort Hotel</td><td>0</td><td> 14</td><td>2015-07-01</td><td>27</td><td>0</td><td>2</td><td>2</td><td>0</td><td>0</td><td>⋯</td><td>0</td><td>Transient</td><td>98</td><td>0</td><td>1</td><td>Check-Out</td><td>2015-07-03</td><td>2</td><td>United Kingdom</td><td>NA</td></tr>
</tbody>
</table>



### 3.1 Scientific Question - Timeline of the year (Resort Hotel & City Hotel) 

**a) Which months are considered as "off-season" and "peak-season" by the number of bookings per month?**

First of all, we'd like to define the peak season and off season based on the number of booking per month. We want our peak season and off season be three or four consecutive months. Peak season is consist of three or four months with highest number of booking and off season is another way around. If the months with the highest number of booking are not consecutive, such as May, July and August, then we will define the peak season as the time from May to August no matter we have a high number of booking in June or not. 



```R
season <- df %>%
       mutate(arrival_month = month(arrival_date)) %>%
       filter(is_canceled == 0) %>%
       group_by(arrival_month) %>%
       summarize(total_booking_per_month = n()) %>%
       arrange(desc(total_booking_per_month)) %>%
       print()
```

    `summarise()` ungrouping output (override with `.groups` argument)
    
    

    [38;5;246m# A tibble: 12 x 2[39m
       arrival_month total_booking_per_month
               [3m[38;5;246m<dbl>[39m[23m                   [3m[38;5;246m<int>[39m[23m
    [38;5;250m 1[39m             8                    [4m8[24m618
    [38;5;250m 2[39m             7                    [4m7[24m892
    [38;5;250m 3[39m             5                    [4m7[24m102
    [38;5;250m 4[39m            10                    [4m6[24m867
    [38;5;250m 5[39m             3                    [4m6[24m590
    [38;5;250m 6[39m             4                    [4m6[24m533
    [38;5;250m 7[39m             6                    [4m6[24m392
    [38;5;250m 8[39m             9                    [4m6[24m367
    [38;5;250m 9[39m             2                    [4m5[24m317
    [38;5;250m10[39m            11                    [4m4[24m632
    [38;5;250m11[39m            12                    [4m4[24m364
    [38;5;250m12[39m             1                    [4m4[24m068
    


```R
p1 <- ggplot(season, aes(x = reorder(as.character(arrival_month), -total_booking_per_month), y = total_booking_per_month, fill = total_booking_per_month) ) +
         geom_bar(stat = "identity") +
labs(title = "Number of Booking by Month", 
     x = "arrival_month", y = "Number of Booking")
p1
```


![png](output_17_0.png)


We choose the time from November to January as the off-season since they have the least amount of booking per month, and we choose the time from May to August as the peak season.

**b) Proportion of different hotel stays every month**

We create two separate datasets for both peak season and off season. We want to examine whether customers have different preference for different types of hotels. In the hotel dataset, 66% percentage of data is for resort hotels, and 34% percentage of data is for city hotels. Therefore, we cannot measure customers' preference for hotels in different seasons simply based on the number of booking for a certain type of hotels. We defined a new variable called `resort ratio`.

The variable `resort ratio` is defined as the number of resort hotel booked divided by total number of booking in a year; `peak.resort.ratio` is defined as the number of resort hotel booked divided by total number of booking during the peak season; `off.resort.ratio` is defined as the number of resort hotel booked divided by total number of booking during the off season. 


```R
peak.season <- mutate(df, arrival_month = month(arrival_date)) %>%
               filter(is_canceled == 0) %>%
               filter(arrival_month == 8 | arrival_month == 7 |arrival_month == 6 | arrival_month == 5) 
              
              

off.season <-  mutate(df, arrival_month = month(arrival_date))%>%
               filter(is_canceled == 0) %>%
               filter(arrival_month == c(11,12,1)) 
              
hotel.season <- mutate(df, arrival_month = month(arrival_date))%>%
                filter(is_canceled == 0) %>%
                filter(arrival_month == 8 | arrival_month == 7 | arrival_month == 6 | arrival_month == 5 | arrival_month == 11 | arrival_month == 12 | arrival_month == 1) %>%
                
                group_by(hotel, arrival_month) %>%
                summarize(total_booking_per_month = n()) %>%
                mutate(peak_or_not = ifelse(arrival_month == 8 | arrival_month == 7 |arrival_month == 6 | arrival_month == 5, "Peak Season", "Off Season")) %>%
                print()

hotel.season
```

    `summarise()` regrouping output by 'hotel' (override with `.groups` argument)
    
    

    [38;5;246m# A tibble: 14 x 4[39m
    [38;5;246m# Groups:   hotel [2][39m
       hotel        arrival_month total_booking_per_month peak_or_not
       [3m[38;5;246m<fct>[39m[23m                [3m[38;5;246m<dbl>[39m[23m                   [3m[38;5;246m<int>[39m[23m [3m[38;5;246m<chr>[39m[23m      
    [38;5;250m 1[39m City Hotel               1                    [4m2[24m254 Off Season 
    [38;5;250m 2[39m City Hotel               5                    [4m4[24m579 Peak Season
    [38;5;250m 3[39m City Hotel               6                    [4m4[24m365 Peak Season
    [38;5;250m 4[39m City Hotel               7                    [4m4[24m782 Peak Season
    [38;5;250m 5[39m City Hotel               8                    [4m5[24m381 Peak Season
    [38;5;250m 6[39m City Hotel              11                    [4m2[24m694 Off Season 
    [38;5;250m 7[39m City Hotel              12                    [4m2[24m391 Off Season 
    [38;5;250m 8[39m Resort Hotel             1                    [4m1[24m814 Off Season 
    [38;5;250m 9[39m Resort Hotel             5                    [4m2[24m523 Peak Season
    [38;5;250m10[39m Resort Hotel             6                    [4m2[24m027 Peak Season
    [38;5;250m11[39m Resort Hotel             7                    [4m3[24m110 Peak Season
    [38;5;250m12[39m Resort Hotel             8                    [4m3[24m237 Peak Season
    [38;5;250m13[39m Resort Hotel            11                    [4m1[24m938 Off Season 
    [38;5;250m14[39m Resort Hotel            12                    [4m1[24m973 Off Season 
    


<table>
<caption>A grouped_df: 14 × 4</caption>
<thead>
	<tr><th scope=col>hotel</th><th scope=col>arrival_month</th><th scope=col>total_booking_per_month</th><th scope=col>peak_or_not</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>City Hotel  </td><td> 1</td><td>2254</td><td>Off Season </td></tr>
	<tr><td>City Hotel  </td><td> 5</td><td>4579</td><td>Peak Season</td></tr>
	<tr><td>City Hotel  </td><td> 6</td><td>4365</td><td>Peak Season</td></tr>
	<tr><td>City Hotel  </td><td> 7</td><td>4782</td><td>Peak Season</td></tr>
	<tr><td>City Hotel  </td><td> 8</td><td>5381</td><td>Peak Season</td></tr>
	<tr><td>City Hotel  </td><td>11</td><td>2694</td><td>Off Season </td></tr>
	<tr><td>City Hotel  </td><td>12</td><td>2391</td><td>Off Season </td></tr>
	<tr><td>Resort Hotel</td><td> 1</td><td>1814</td><td>Off Season </td></tr>
	<tr><td>Resort Hotel</td><td> 5</td><td>2523</td><td>Peak Season</td></tr>
	<tr><td>Resort Hotel</td><td> 6</td><td>2027</td><td>Peak Season</td></tr>
	<tr><td>Resort Hotel</td><td> 7</td><td>3110</td><td>Peak Season</td></tr>
	<tr><td>Resort Hotel</td><td> 8</td><td>3237</td><td>Peak Season</td></tr>
	<tr><td>Resort Hotel</td><td>11</td><td>1938</td><td>Off Season </td></tr>
	<tr><td>Resort Hotel</td><td>12</td><td>1973</td><td>Off Season </td></tr>
</tbody>
</table>




```R
peak.city.num <- count(peak.season, hotel == "Resort Hotel")$n[1]
peak.resort.num <- count(peak.season, hotel == "Resort Hotel")$n[2]
peak.city.num
peak.resort.num

peak.resort.ratio <- peak.resort.num / (peak.city.num + peak.resort.num)
peak.resort.ratio

off.city.num <- count(off.season,hotel == "Resort Hotel")$n[1]
off.resort.num <- count(off.season,hotel == "Resort Hotel")$n[2]

off.resort.ratio <- off.resort.num / (off.city.num + off.resort.num)
off.resort.ratio
```


19107



10897



0.363184908678843



0.437341917682226



```R
ggplot(hotel.season) + 
  geom_col(aes(peak_or_not, total_booking_per_month
, fill = hotel)) +
  coord_flip() +
  labs(title = "Percentage of Booking for City Hotels & Resort Hotels in Peak and Off Season", 
     x = "Hotel Type", y = "Percentage of Booking")

```


![png](output_23_0.png)


As we can see from the results above, peak.resort.ratio is 36.3% and off.resort.ratio ratio is 44%.
The city hotels are more favorable by the tourists during the peak season than during the off season. Customers tend to be more indifferent to hotel types during the off season. 


**c) Do customers tend to cancel booking during peak months or off season?**


```R
cancel.peak <- mutate(df, arrival_month = month(arrival_date)) %>%
               filter(arrival_month == 8 | arrival_month == 7 |arrival_month == 6 | arrival_month == 5) 
              
              

cancel.off <-  mutate(df, arrival_month = month(arrival_date))%>%
               filter(arrival_month == c(11,12,1)) 
```

    Warning message in arrival_month == c(11, 12, 1):
    “longer object length is not a multiple of shorter object length”
    


```R
peak.total.booking <- nrow(cancel.peak)
off.total.booking <- nrow(cancel.off)

peak.season.cancel <- cancel.peak %>%
                      filter(is_canceled == 1) #%>%

peak.cancel.num <- nrow(peak.season.cancel)                    

off.season.cancel <- cancel.off %>%
                      filter(is_canceled == 1) #%>%

off.cancel.num <- nrow(off.season.cancel) 

peak.cancel.rate <- peak.cancel.num/peak.total.booking
off.cancel.rate <- off.cancel.num/off.total.booking


peak.cancel.rate
off.cancel.rate
```


0.390026225375592



0.325459317585302


Customers tend to cancel booking during the peak season. The cancellation rate is about 7% higher in the peak season than in the off season.

**d) Is the peak months trend the same worldwide?**

From the Kaggle website, we can see that about 41% of the booking took place in Portugal, 10% occurred in United Kingdom, and the rest 49% of the booking are in other countries. Therefore, our focus in this question is to study whether the peak and off seasons in Portugal, United Kingdom and other countries are the same or not.


```R
season.by.country <- df %>%
       mutate(arrival_month = month(arrival_date)) %>%
       filter(is_canceled == 0) %>%
       group_by(arrival_month,country_full_name) %>%
       summarize(total_booking_per_month = n()) %>%
       print()
```

    `summarise()` regrouping output by 'arrival_month' (override with `.groups` argument)
    
    

    [38;5;246m# A tibble: 1,006 x 3[39m
    [38;5;246m# Groups:   arrival_month [12][39m
       arrival_month country_full_name total_booking_per_month
               [3m[38;5;246m<dbl>[39m[23m [3m[38;5;246m<chr>[39m[23m                               [3m[38;5;246m<int>[39m[23m
    [38;5;250m 1[39m             1 Albania                                 3
    [38;5;250m 2[39m             1 Algeria                                 4
    [38;5;250m 3[39m             1 American Samoa                          1
    [38;5;250m 4[39m             1 Angola                                 28
    [38;5;250m 5[39m             1 Antarctica                              1
    [38;5;250m 6[39m             1 Argentina                              16
    [38;5;250m 7[39m             1 Armenia                                 1
    [38;5;250m 8[39m             1 Australia                              23
    [38;5;250m 9[39m             1 Austria                                23
    [38;5;250m10[39m             1 Belarus                                 1
    [38;5;246m# … with 996 more rows[39m
    


```R
season.prt <- season.by.country %>%
        filter(country_full_name == "Portugal") %>%
        arrange(desc(total_booking_per_month)) %>%
        print()
```

    [38;5;246m# A tibble: 12 x 3[39m
    [38;5;246m# Groups:   arrival_month [12][39m
       arrival_month country_full_name total_booking_per_month
               [3m[38;5;246m<dbl>[39m[23m [3m[38;5;246m<chr>[39m[23m                               [3m[38;5;246m<int>[39m[23m
    [38;5;250m 1[39m             8 Portugal                             [4m2[24m341
    [38;5;250m 2[39m             7 Portugal                             [4m2[24m276
    [38;5;250m 3[39m             2 Portugal                             [4m1[24m944
    [38;5;250m 4[39m            10 Portugal                             [4m1[24m881
    [38;5;250m 5[39m             3 Portugal                             [4m1[24m871
    [38;5;250m 6[39m             9 Portugal                             [4m1[24m626
    [38;5;250m 7[39m            12 Portugal                             [4m1[24m612
    [38;5;250m 8[39m             1 Portugal                             [4m1[24m600
    [38;5;250m 9[39m            11 Portugal                             [4m1[24m511
    [38;5;250m10[39m             4 Portugal                             [4m1[24m470
    [38;5;250m11[39m             5 Portugal                             [4m1[24m470
    [38;5;250m12[39m             6 Portugal                             [4m1[24m469
    


```R
season.bgr <- season.by.country %>%
        filter(country_full_name == "United Kingdom") %>%
        arrange(desc(total_booking_per_month)) %>%
        print()
```

    [38;5;246m# A tibble: 12 x 3[39m
    [38;5;246m# Groups:   arrival_month [12][39m
       arrival_month country_full_name total_booking_per_month
               [3m[38;5;246m<dbl>[39m[23m [3m[38;5;246m<chr>[39m[23m                               [3m[38;5;246m<int>[39m[23m
    [38;5;250m 1[39m             5 United Kingdom                       [4m1[24m215
    [38;5;250m 2[39m            10 United Kingdom                       [4m1[24m177
    [38;5;250m 3[39m             6 United Kingdom                       [4m1[24m089
    [38;5;250m 4[39m             7 United Kingdom                       [4m1[24m046
    [38;5;250m 5[39m             9 United Kingdom                        905
    [38;5;250m 6[39m             4 United Kingdom                        883
    [38;5;250m 7[39m             8 United Kingdom                        877
    [38;5;250m 8[39m             3 United Kingdom                        802
    [38;5;250m 9[39m             2 United Kingdom                        650
    [38;5;250m10[39m            11 United Kingdom                        427
    [38;5;250m11[39m             1 United Kingdom                        307
    [38;5;250m12[39m            12 United Kingdom                        297
    


```R
season.others <- df %>%
       mutate(arrival_month = month(arrival_date)) %>%
       filter(is_canceled == 0) %>%
       filter(country_full_name != "United Kingdom" & country_full_name != "Portugal") %>%
       group_by(arrival_month) %>%
       summarize(total_booking_per_month = n()) %>%
       arrange(desc(total_booking_per_month)) %>%
       print()
```

    `summarise()` ungrouping output (override with `.groups` argument)
    
    

    [38;5;246m# A tibble: 12 x 2[39m
       arrival_month total_booking_per_month
               [3m[38;5;246m<dbl>[39m[23m                   [3m[38;5;246m<int>[39m[23m
    [38;5;250m 1[39m             8                    [4m5[24m400
    [38;5;250m 2[39m             7                    [4m4[24m570
    [38;5;250m 3[39m             5                    [4m4[24m417
    [38;5;250m 4[39m             4                    [4m4[24m180
    [38;5;250m 5[39m             3                    [4m3[24m917
    [38;5;250m 6[39m             9                    [4m3[24m836
    [38;5;250m 7[39m             6                    [4m3[24m834
    [38;5;250m 8[39m            10                    [4m3[24m809
    [38;5;250m 9[39m             2                    [4m2[24m723
    [38;5;250m10[39m            11                    [4m2[24m694
    [38;5;250m11[39m            12                    [4m2[24m455
    [38;5;250m12[39m             1                    [4m2[24m161
    


The peak season for Portugal is from July October (by the consecutive months rule we defined in the first problem), and the off season is from April to June.
The peak season in United Kingdom is from May to July(by the consecutive months rule we defined in the first problem), and the off-season is from November until next year's January.
The peak season for other countries is from May to August, and the off-season is from November to next year's January.

**e) Which kind of room is more popular in each season?**


```R
peak.room <- peak.season %>%
             select(reserved_room_type) %>%
             group_by(reserved_room_type) %>%
             summarize(n = n()) %>%
             arrange(desc(n)) %>%
             print()
```

    `summarise()` ungrouping output (override with `.groups` argument)
    
    

    [38;5;246m# A tibble: 9 x 2[39m
      reserved_room_type     n
      [3m[38;5;246m<fct>[39m[23m              [3m[38;5;246m<int>[39m[23m
    [38;5;250m1[39m A                  [4m1[24m[4m9[24m475
    [38;5;250m2[39m D                   [4m6[24m060
    [38;5;250m3[39m E                   [4m1[24m977
    [38;5;250m4[39m F                    943
    [38;5;250m5[39m G                    650
    [38;5;250m6[39m C                    414
    [38;5;250m7[39m B                    300
    [38;5;250m8[39m H                    181
    [38;5;250m9[39m L                      4
    


```R
off.room <- off.season %>%
            select(reserved_room_type) %>%
            group_by(reserved_room_type) %>%
            summarize(n = n()) %>%
            arrange(desc(n)) %>%
            print()
```

    `summarise()` ungrouping output (override with `.groups` argument)
    
    

    [38;5;246m# A tibble: 8 x 2[39m
      reserved_room_type     n
      [3m[38;5;246m<fct>[39m[23m              [3m[38;5;246m<int>[39m[23m
    [38;5;250m1[39m A                   [4m3[24m259
    [38;5;250m2[39m D                    586
    [38;5;250m3[39m E                    259
    [38;5;250m4[39m F                     97
    [38;5;250m5[39m G                     59
    [38;5;250m6[39m B                     51
    [38;5;250m7[39m C                     23
    [38;5;250m8[39m H                     15
    


```R
p5.1 <- ggplot(peak.room, aes(x = reorder(reserved_room_type, -n), y = n, fill = reserved_room_type) ) +
         geom_bar(stat = "identity") +
labs(title = "Number of Booking by Room Type - Peak Season", 
     x = "Reserved Room Type", y = "Number of Booking")
p5.1

p5.2 <- ggplot(off.room, aes(x = reorder(reserved_room_type, -n), y = n, fill = reserved_room_type) ) +
         geom_bar(stat = "identity") +
labs(title = "Number of Booking by Room Type - Off Season", 
     x = "Reserved Room Type", y = "Number of Booking")
p5.2
```


![png](output_39_0.png)



![png](output_39_1.png)


We don't see any difference in customers' preference for room type in different seasons. Room A, Room D, Room E, Room F, and Room G have always been the most, the second, the third, the fourth and the fifth most popular room types in both peak and off seasons.

**f) Is waiting months longer in peak season than in off season? Does waiting months differ by reserved room type?**


```R
waiting.month.peak <- peak.season %>%
    mutate(waiting_month = lead_time/30) %>%
    select(waiting_month, reserved_room_type)%>%
    group_by(reserved_room_type) %>%
    summarize(avg_waiting_time_by_room = sum(waiting_month)/n()) %>%
    print()
```

    `summarise()` ungrouping output (override with `.groups` argument)
    
    

    [38;5;246m# A tibble: 9 x 2[39m
      reserved_room_type avg_waiting_time_by_room
      [3m[38;5;246m<fct>[39m[23m                                 [3m[38;5;246m<dbl>[39m[23m
    [38;5;250m1[39m A                                      3.65
    [38;5;250m2[39m B                                      4.93
    [38;5;250m3[39m C                                      2.98
    [38;5;250m4[39m D                                      3.29
    [38;5;250m5[39m E                                      3.73
    [38;5;250m6[39m F                                      2.67
    [38;5;250m7[39m G                                      2.61
    [38;5;250m8[39m H                                      2.16
    [38;5;250m9[39m L                                      0   
    


```R
waiting.month.off <- off.season %>%
    mutate(waitint_month = lead_time/30) %>%
    select(waitint_month, reserved_room_type)%>%
    group_by(reserved_room_type) %>%
    summarize(avg_waiting_time_by_room = sum(waitint_month)/n()) %>%
    print()
```

    `summarise()` ungrouping output (override with `.groups` argument)
    
    

    [38;5;246m# A tibble: 8 x 2[39m
      reserved_room_type avg_waiting_time_by_room
      [3m[38;5;246m<fct>[39m[23m                                 [3m[38;5;246m<dbl>[39m[23m
    [38;5;250m1[39m A                                     1.35 
    [38;5;250m2[39m B                                     1.38 
    [38;5;250m3[39m C                                     0.914
    [38;5;250m4[39m D                                     1.49 
    [38;5;250m5[39m E                                     1.43 
    [38;5;250m6[39m F                                     1.13 
    [38;5;250m7[39m G                                     1.39 
    [38;5;250m8[39m H                                     1.49 
    


```R
p6.1 <- ggplot(waiting.month.peak, aes(x = reorder(reserved_room_type, -avg_waiting_time_by_room), y = avg_waiting_time_by_room, fill = reserved_room_type) ) +
         geom_bar(stat = "identity") +
labs(title = "Number of Booking by Room Type - Peak Season", 
     x = "Reserved Room Type", y = "Number of Booking")
p6.1

p6.2 <- ggplot(waiting.month.off, aes(x = reorder(reserved_room_type, -avg_waiting_time_by_room), y = avg_waiting_time_by_room, fill = reserved_room_type) ) +
         geom_bar(stat = "identity") +
labs(title = "Number of Booking by Room Type - Off Season", 
     x = "Reserved Room Type", y = "Number of Booking")
p6.2
```


![png](output_44_0.png)



![png](output_44_1.png)


We define the variable "waiting month" by deviding the lead time (Number of days that elapsed between the entering date of the booking into the PMS and the arrival date) by 30 days.
The waiting months in peak season is longer than that in the off season. The waiting months for different types of room differs in two seasons. In the peak season, Room B, Room E, and Room A has the highest waiting month. However, in the off season, Room G, Room B,and Room E has the highest waiting month.

### 3.2 Scientific Question - Reservation and Cancellation behavior among travellers

**a) Average number of days between booking a hotel and arriving at hotel, for countries with more than 500 bookings**

There are many countries included in the dataset. To ensure we have a clear and interpretable boxplot, we select top countries with more bookings by filtering out countries with hotel bookings less than 500 throughout Year 2015 to 2017. We observe that for countries with more than 500 bookings, Germany has the highest number of days between booking a hotel and arriving at hotel, then followed by the United Kingdom and Norway.


```R
avg_lead_time <- df %>% 
                 select(hotel, lead_time, country_full_name) %>% 
                 add_count(country_full_name) %>% 
                 filter(n > 500) %>% 
                 mutate(country_full_name = fct_reorder(country_full_name, lead_time, mean))

ggplot(data = avg_lead_time) + 
geom_boxplot(aes(x = country_full_name, y = lead_time), color = "purple", outlier.colour = "black") + 
coord_flip() +
labs(x = "Country", 
     y = "Days", 
     title = "Box plot for average number of days between booking & scheduled
              arrival time for countries with more than 500 bookings")
```


![png](output_49_0.png)


**b) What is the percentage of cancellation of bookings, group by countries & resorts?**

As in a) above,  we select top countries with more bookings by filtering out countries with hotel bookings less than 500 throughout Year 2015 to 2017. From the bar graphs, we observe that Portugal has the highest percentage of hotel booking cancellation, while Germany has the lowest percentage of hotel booking cancellation. In almost all countries, City Hotel tend to have more pecentage of cancellation, compared to Resort Hotel. 


```R
hotel_cancel <- df %>% group_by(country_full_name, hotel) %>%
                summarise(total = n(), cancellation = sum(is_canceled)) %>%
                mutate(total_country = sum(total)) %>%
                mutate(cancel_percent = cancellation/total_country) %>%
                mutate(total_cancel_percent = sum(cancel_percent)) %>%
                filter(total_country > 500) %>% 
                ungroup()%>%
                mutate(country_full_name = fct_reorder(country_full_name, cancel_percent, sum))

ggplot(data = hotel_cancel) +
geom_col(aes(country_full_name, cancel_percent, fill = hotel)) +
coord_flip() +
labs(title = "Percentage of cancellation for City Hotels & Resort Hotels", 
     x = "Country", y = "Percentage of Cancellation")
```

    `summarise()` regrouping output by 'country_full_name' (override with `.groups` argument)
    
    


![png](output_52_1.png)


**c) Booking cancellation ratio per year, by resorts**

Indeed, as what we observe from (b) above, we confirmed that every year from 2015 to 2017, City Hotel had more booking cancellation every year, compared to Resort Hotel.


```R
cancel_ratio <- df %>% 
                mutate(arrival_date_year = year(arrival_date)) %>%
                group_by(hotel, arrival_date_year) %>% 
                count(is_canceled) %>%
                mutate(ratio = n/sum(n)) %>% print

ggplot(data = cancel_ratio) + 
geom_col(aes(x = hotel, y = n, fill = as.factor(is_canceled)), position = "fill", stat = "identity") +
facet_grid( .~ arrival_date_year) +
labs(title = "Booking Cancellation Ratio per Year",
     x = "Hotel", y = "Cancellation ratio",
     fill = "Cancelled" )
```

    [38;5;246m# A tibble: 12 x 5[39m
    [38;5;246m# Groups:   hotel, arrival_date_year [6][39m
       hotel        arrival_date_year is_canceled     n ratio
       [3m[38;5;246m<fct>[39m[23m                    [3m[38;5;246m<dbl>[39m[23m       [3m[38;5;246m<int>[39m[23m [3m[38;5;246m<int>[39m[23m [3m[38;5;246m<dbl>[39m[23m
    [38;5;250m 1[39m City Hotel                [4m2[24m015           0  [4m7[24m676 0.562
    [38;5;250m 2[39m City Hotel                [4m2[24m015           1  [4m5[24m990 0.438
    [38;5;250m 3[39m City Hotel                [4m2[24m016           0 [4m2[24m[4m2[24m731 0.596
    [38;5;250m 4[39m City Hotel                [4m2[24m016           1 [4m1[24m[4m5[24m402 0.404
    [38;5;250m 5[39m City Hotel                [4m2[24m017           0 [4m1[24m[4m5[24m817 0.575
    [38;5;250m 6[39m City Hotel                [4m2[24m017           1 [4m1[24m[4m1[24m687 0.425
    [38;5;250m 7[39m Resort Hotel              [4m2[24m015           0  [4m6[24m076 0.741
    [38;5;250m 8[39m Resort Hotel              [4m2[24m015           1  [4m2[24m120 0.259
    [38;5;250m 9[39m Resort Hotel              [4m2[24m016           0 [4m1[24m[4m3[24m390 0.732
    [38;5;250m10[39m Resort Hotel              [4m2[24m016           1  [4m4[24m910 0.268
    [38;5;250m11[39m Resort Hotel              [4m2[24m017           0  [4m9[24m052 0.691
    [38;5;250m12[39m Resort Hotel              [4m2[24m017           1  [4m4[24m047 0.309
    

    Warning message:
    “Ignoring unknown parameters: stat”
    


![png](output_55_2.png)


**d) When do people usually cancel their booking? Hence, we seek the number of days between booking cancellation and scheduled arrival by year** 

We observe that City Hotel is more likely to have more last minute cancellation, compared to Resort Hotel. In general, every year, we find that more booking cancellations were made last minute, around less than 10 days before scheduled arrival date.


```R
df_year <- df %>% mutate(arrival_date_year = year(arrival_date)) 


ggplot(df_year) + 
geom_bar(aes(x = cancel_lead_time, group = hotel, 
             fill = hotel)) +
labs(x = "Days between cancellation and scheduled arrival", 
     y = "Number of Booking Cancellations",
     title = "Last-minute cancellations") +
scale_fill_discrete(name = "Hotel")
```

    Warning message:
    “Removed 75945 rows containing non-finite values (stat_count).”
    


![png](output_58_1.png)



```R
ggplot(df_year) + 
geom_bar(aes(x = cancel_lead_time, group = as.character(arrival_date_year), 
             fill = as.character(arrival_date_year))) +
labs(x = "Days between cancellation and scheduled arrival", 
     y = "Number of Booking Cancellations",
     title = "Last-minute cancellations") +
scale_fill_discrete(name = "Year")
```

    Warning message:
    “Removed 75945 rows containing non-finite values (stat_count).”
    


![png](output_59_1.png)


**e) Number of booking arrivals on each date**


In general, the total number of arrivals of each day of the month from 2015-2017 were pretty consistent, ranging from around 3500 to 4500 number of arrivals. Surprisingly, on the 31st, it has the lowest number of arrivals. 


```R
arrival_date <- df %>% 
                 mutate(arrival_date = day(arrival_date)) %>% group_by(arrival_date) %>%
                 summarize(total_booking = n()) 

ggplot(data = arrival_date) + 
geom_line(mapping = aes(x = arrival_date, y = total_booking)) + 
labs(x = "Date", y = "Number of bookings", title = "Number of booking arrivals on each date")
```

    `summarise()` ungrouping output (override with `.groups` argument)
    
    


![png](output_62_1.png)


### 3.3 Scientific Question - Travelling with babies, children, or without children  

**a) Special request?** 

From this we can see that only around 41% of people who booked hotels put in special requests. Although it is not the majority of the population's behavior, it does represent a significant proportion.


```R
mod_df <- df %>% 
          mutate(has_child = ifelse(children > 0, 1, 0)) %>% 
          mutate(has_special_request =ifelse(total_of_special_requests > 0, 1, 0))
```


```R
special_request_percent <- mod_df  %>% 
                           summarize(percent_request = sum(has_special_request) / nrow(mod_df)) %>% 
                           print()
```

      percent_request
    1       0.4113442
    


```R
mod_df %>% filter(has_special_request > 0) %>% select(total_of_special_requests) %>%
  ggplot() + geom_bar(mapping = aes(x = total_of_special_requests)) + ggtitle("Count of Special Requests") +
                xlab("Number of Requests made by Guest") + ylab("Count")
```


![png](output_68_0.png)


This shows that the majority of people only make 1 special requests. This would make sense as it is less likely that an individual consumer will need to make multiple adjustements to the room. Special adjustments are defined as needing to make it twin beds or be moved to a higher floor. Guest wh oneed to make multiple adjustments are much more rare. 

Next we will look at if there is a change in the number of requests being made at certain times in the year


```R
#dates
mod_df %>% group_by(arrival_date) %>% summarize(mean_request = mean(total_of_special_requests)) %>%
  ggplot() + geom_line(mapping = aes(x = arrival_date, y = mean_request)) + ggtitle("Change of Mean Request from July of 2015 to July of 2017") + xlab("Year Month of Arrival") + ylab("Mean Number of Requests")

#month
mod_df %>% mutate(arrival_month = month(arrival_date)) %>%
  ggplot() + geom_bar(mapping = aes(x = arrival_month, fill = as.factor(total_of_special_requests))) + 
  scale_x_discrete(limits = c(1:12)) + labs(fill = "Number of Special Requests") + ggtitle("Number of Requests By Month") + xlab("Month of Arrival") + ylab("Quantity of Requests by Month")
  

#month year
df1 <- mod_df %>% mutate(arrival_month = month(arrival_date)) %>% mutate(arrival_year = year(arrival_date)) %>%
                  group_by(arrival_year, arrival_month) %>% summarize(mean_request = mean(total_of_special_requests)) %>% 
                  mutate(year_month = paste(arrival_year, arrival_month, "01", sep = "-"))

df1$year_month <- as_date(df1$year_month, format = "%Y-%m-%d")
ggplot(df1) + geom_line(mapping = aes(x = year_month, y = mean_request)) + ggtitle("Change of Mean Request from July of 2015 to July of 2017") + xlab("Year Month of Arrival") + ylab("Mean Number of Requests")

```

    `summarise()` ungrouping output (override with `.groups` argument)
    
    Warning message:
    “Continuous limits supplied to discrete scale.
    Did you mean `limits = factor(...)` or `scale_*_continuous()`?”
    


![png](output_71_1.png)


    `summarise()` regrouping output by 'arrival_year' (override with `.groups` argument)
    
    


![png](output_71_3.png)



![png](output_71_4.png)


The first graph is a plot of arrival dates and the number of special requests made on a given day across the entire data set. This demonstrates that over the course of 2 years, the number of requests are steadily increasing. This is corroberated by the 3rd graph. However, the first graph shows a greater degree of variation in the day to day behavior of the number of special requests being made. The bar chart of months shows us that during peak travel time, the number of special requests are also increasing. The number of requests builds up in response to increase in hotel bookings. We see that in the bar graph that the number of special requests being made by an individual being greater than or equal to 3 also increases in likelihood. 

The data suggests that the mean number of special requests per custsomer is going to increase over time as growth has been steadily trending upwards over all around the world over these two years. 

**b) Did not cancel the hotel booking/ last minute cancellations?**


```R
cancel_df <- df %>% mutate(has_child = ifelse(children > 0, 1, 0)) %>% filter(!is.na(has_child)) %>%                 group_by(has_child) %>% 
                summarize(cancel_percent = sum(reservation_status == "Canceled") / n(), 
                          last_minute_two_day = sum(cancel_lead_time <= 2) / n(), 
                          last_minute_seven_day = sum(cancel_lead_time <= 7) / n()) %>% print()
```

    `summarise()` ungrouping output (override with `.groups` argument)
    
    

    [38;5;246m# A tibble: 2 x 4[39m
      has_child cancel_percent last_minute_two_day last_minute_seven_day
          [3m[38;5;246m<dbl>[39m[23m          [3m[38;5;246m<dbl>[39m[23m               [3m[38;5;246m<dbl>[39m[23m                 [3m[38;5;246m<dbl>[39m[23m
    [38;5;250m1[39m         0          0.362                  [31mNA[39m                    [31mNA[39m
    [38;5;250m2[39m         1          0.354                  [31mNA[39m                    [31mNA[39m
    

This shows that the proportion of people who cancel their reservations does not appear to be related to whether or not they have children. Furthermore, last minute cancellations are also not significantly differentiated between the two groups under the 2 day classification of "last minute". We also see that even if we expand the definition of "last minute" to be a week before arrival time, it is shown that there is not a siginificant difference in the percentage of people who cancel between those who have Children and those who do not. 

**c) BB meal or HB meal? Do they have a preference?**

This shows that most customers just order the standard meal type BB. A small proportion of consumers order an HB meal plan. Similar percent of consumers order HB as do those who don't have a meal plan. 



```R
unique(df$meal)
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>BB</li><li>FB</li><li>HB</li><li>SC</li><li>Undefined</li></ol>

<details>
	<summary style=display:list-item;cursor:pointer>
		<strong>Levels</strong>:
	</summary>
	<style>
	.list-inline {list-style: none; margin:0; padding: 0}
	.list-inline>li {display: inline-block}
	.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
	</style>
	<ol class=list-inline><li>'BB'</li><li>'FB'</li><li>'HB'</li><li>'SC'</li><li>'Undefined'</li></ol>
</details>



```R
meal_bb <- df %>% filter(meal == "BB") %>% mutate(present = 1) %>% summarize(count = sum(present))
meal_fb <- df %>% filter(meal == "FB") %>% mutate(present = 1) %>% summarize(count = sum(present))
meal_hf <- df %>% filter(meal == "HB") %>% mutate(present = 1) %>% summarize(count = sum(present))
meal_undef <- df %>% filter(meal == "SC" | meal == "Undefined") %>% mutate(present = 1) %>% 
              summarize(count = sum(present)) 

#SC and Other are the same -- NO MEAL

total_count <- sum(meal_bb, meal_fb, meal_hf, meal_undef)

meal_breakdown <- tibble(
  "Meal Type" = c("BB" , "FB", "HB","Undefined"),
  "Number of Type" = c(meal_bb$count, meal_fb$count, meal_hf$count, meal_undef$count),
  "Percent of Whole" = c(meal_bb$count/total_count, meal_fb$count/total_count, meal_hf$count/total_count, 
                         meal_undef$count/total_count),
)
print(meal_breakdown)

ggplot(meal_breakdown, aes(x = `Meal Type`, y = `Percent of Whole`)) + geom_bar(stat = "identity") + 
ggtitle("Percent of All Meals Ordered Broken Down by Type")
```

    [38;5;246m# A tibble: 4 x 3[39m
      `Meal Type` `Number of Type` `Percent of Whole`
      [3m[38;5;246m<chr>[39m[23m                  [3m[38;5;246m<dbl>[39m[23m              [3m[38;5;246m<dbl>[39m[23m
    [38;5;250m1[39m BB                     [4m9[24m[4m1[24m864            0.773  
    [38;5;250m2[39m FB                       798            0.006[4m7[24m[4m1[24m
    [38;5;250m3[39m HB                     [4m1[24m[4m4[24m434            0.121  
    [38;5;250m4[39m Undefined              [4m1[24m[4m1[24m802            0.099[4m3[24m 
    


![png](output_79_1.png)


This shows that most customers just order the standard meal type BB. A small proportion of consumers order an HB meal plan. Similar percent of consumers order HB as do those who don't have a meal plan.

**d) Do families with kids prefer to book through agents or direct by themselves?**


```R
booking_preference <- df %>% 
                      filter(!is.na(children)) %>% 
                      mutate(has_kid = ifelse(children > 0 , 1, 0)) %>%         
                      group_by(market_segment) %>% 
                      summarize(percent = sum(has_kid)/n()) %>% 
                      arrange(desc(percent))%>% 
                      print
```

    `summarise()` ungrouping output (override with `.groups` argument)
    
    

    [38;5;246m# A tibble: 7 x 2[39m
      market_segment percent
      [3m[38;5;246m<fct>[39m[23m            [3m[38;5;246m<dbl>[39m[23m
    [38;5;250m1[39m Direct         0.126  
    [38;5;250m2[39m Online TA      0.112  
    [38;5;250m3[39m Complementary  0.059[4m9[24m 
    [38;5;250m4[39m Offline TA/TO  0.023[4m2[24m 
    [38;5;250m5[39m Corporate      0.008[4m4[24m[4m1[24m
    [38;5;250m6[39m Groups         0.002[4m3[24m[4m2[24m
    [38;5;250m7[39m Aviation       0      
    


```R
ggplot(booking_preference, aes(x = `market_segment`, y = `percent`)) + 
geom_bar(stat = "identity") + 
xlab("Booking Method") + 
ylab("Percent of Population") + 
ggtitle("Booking Method of the Population")
```


![png](output_83_0.png)


Upon closer inspection it would appear that people are more likely to book through a travel agent our tour operator if they have children than to book directly. The data set does not contain anyone who have children and book through aviation. Although it is possible to book through aviation, it is likely less appealing as they could be sold as a bundle package with flight routes and hotels stays.


```R
df %>% filter(!is.na(children)) %>% 
       mutate(arrival_month = month(arrival_date)) %>%  
       mutate(has_kid = ifelse(children > 0 , 1, 0)) %>%
       mutate(arrival_year = year(arrival_date)) %>% 
       group_by(arrival_year, arrival_month) %>% 
       #summarize(num_bookings = sum(has_kid)) %>% 
       mutate(year_month = paste(arrival_year, arrival_month, "01", sep = "-")) %>% #ungroup() %>%
       #select(year_month, num_bookings) %>% 
       ggplot() + geom_bar(aes(x = `year_month`, fill = `market_segment`)) + 
       coord_flip() + ylab("Year and Month of Arrival") + 
       ggtitle("The Breakdown of Reservation Methods by Year and Month of arrival") + 
       labs(fill = "Reservation Method")
```


![png](output_85_0.png)


Overall it does seem that the vast majority of bookings are done through TA/TO throughout the enttirety of the dataset. However The second graph shows that the booking preferences of consumers with Children is different as compared to the enitre population as the direct booking takes up a larger percentage of the total amount of bookings. However the trend of consumers predominantly using Online TA as a means of making reservations remains true in filtered data of consumers with children.

This is likely because consumers without children become consumers with children later on in their life and this event is unlikely to have changed their habits when it comes to making hotel reservations. 


**e) What kind of room do they book?**


```R
df %>% 
filter(!is.na(reserved_room_type)) %>% 
ggplot(aes(x = reserved_room_type)) + geom_bar() + 
ggtitle("Number of People by Room Type Reserved")

df %>% 
filter(!is.na(assigned_room_type)) %>% 
ggplot(aes(x = assigned_room_type)) + 
geom_bar() + 
ggtitle("Number of People by Room Type Assigned")
```


![png](output_88_0.png)



![png](output_88_1.png)


This graphs shows that the vast majority of people reserve for the room type A. When comparing between rooms assigned and rooms reserved, we see thatt there is a larger number of people who end up in room type D than people who reserved for type D. Presumably A and D are the most commonly available room types, which is why they are assigned and reserved more often. Although the data is made less clear as to what room these letters are referring to specifically, we can presume that the other letters are more likely to be larger and or pricier rooms. 

**f) Are hotel guests with children more likely to require a parking space?**


```R
df %>% 
filter(!is.na(children)) %>% 
mutate(has_kid = ifelse(children > 0 , 1, 0)) %>% 
group_by(required_car_parking_spaces) %>% 
summarize(percent_need = sum(has_kid) / n()) %>% 
print() %>% 
ggplot() + 
geom_histogram(aes(x = required_car_parking_spaces, y = percent_need), stat = "identity") + 
ggtitle("Proportion of population that want parking spots broken down by number of spots reserved") + 
xlab("Number of Spots Reserved") +
ylab("Percent of Total Population")
  
df %>% 
filter(!is.na(children)) %>% 
mutate(has_kid = ifelse(children > 0 , 1, 0)) %>% 
group_by(has_kid) %>% 
summarize(percent_need = sum(required_car_parking_spaces) / n()) %>% 
print() %>% ggplot() + geom_histogram(aes(x = has_kid, y = percent_need), stat = "identity") + 
ggtitle("Proportion that needs parking spots") + 
xlab("Has Children") + 
ylab("Percent of category")
```

    `summarise()` ungrouping output (override with `.groups` argument)
    
    

    [38;5;246m# A tibble: 5 x 2[39m
      required_car_parking_spaces percent_need
                            [3m[38;5;246m<int>[39m[23m        [3m[38;5;246m<dbl>[39m[23m
    [38;5;250m1[39m                           0       0.068[4m1[24m
    [38;5;250m2[39m                           1       0.134 
    [38;5;250m3[39m                           2       0.214 
    [38;5;250m4[39m                           3       0     
    [38;5;250m5[39m                           8       0     
    

    Warning message:
    “Ignoring unknown parameters: binwidth, bins, pad”
    `summarise()` ungrouping output (override with `.groups` argument)
    
    

    [38;5;246m# A tibble: 2 x 2[39m
      has_kid percent_need
        [3m[38;5;246m<dbl>[39m[23m        [3m[38;5;246m<dbl>[39m[23m
    [38;5;250m1[39m       0       0.057[4m7[24m
    [38;5;250m2[39m       1       0.115 
    

    Warning message:
    “Ignoring unknown parameters: binwidth, bins, pad”
    


![png](output_91_5.png)



![png](output_91_6.png)


Consumers who have children require parking spots approximately double the proportion of consumers who do not need parking spots. This would make sense as people traveling with Children are more likely to rented or need a car as traveling with a family using public transportion might be less common. Individuals who travel without children are more mobile and thus it makes sense that they utilize parking spots siginificantly less than those with children.

**g) Time series - do families with kids travel during peak travel months?**


```R
travel_behavior <- df %>% 
                   filter(children > 0) %>% 
                   group_by(arrival_date) %>% 
                   summarize(travelers = n()) %>% print()

travel_behavior_month <- df %>% 
                         filter(children > 0) %>% 
                         mutate(arrival_month = month(arrival_date)) %>% 
                         group_by(arrival_month) %>% 
                         summarize(travelers = n()) %>% 
                         print()


ggplot(travel_behavior, aes(x = arrival_date, y = travelers)) + 
geom_point() + 
ggtitle("Travel Behavior of Consumers with Children over June 2015 to June 2017") + 
xlab("Arrival Date of Consumer") + 
ylab("Number of Travelers")


ggplot(travel_behavior, aes(x = arrival_date, y = travelers)) + 
geom_bar(stat = "identity") + 
ggtitle("Travel Behavior of Consumers with Children over June 2015 to June 2017") + 
xlab("Arrival Date of Consumer") + 
ylab("Number of Travelers")
ggplot(travel_behavior_month, aes(x = arrival_month, y = travelers)) + 
geom_bar(stat = "identity") + ggtitle("Travel Behavior of Consumers with Children by month") + 
xlab("Arrival Month of Consumer") + ylab("Number of Travelers") + 
scale_x_discrete(limits = c(1:12))
```

    `summarise()` ungrouping output (override with `.groups` argument)
    
    

    [38;5;246m# A tibble: 771 x 2[39m
       arrival_date travelers
       [3m[38;5;246m<date>[39m[23m           [3m[38;5;246m<int>[39m[23m
    [38;5;250m 1[39m 2015-07-01           2
    [38;5;250m 2[39m 2015-07-02           2
    [38;5;250m 3[39m 2015-07-03           3
    [38;5;250m 4[39m 2015-07-04           5
    [38;5;250m 5[39m 2015-07-05           6
    [38;5;250m 6[39m 2015-07-06           4
    [38;5;250m 7[39m 2015-07-07           6
    [38;5;250m 8[39m 2015-07-08           2
    [38;5;250m 9[39m 2015-07-09           4
    [38;5;250m10[39m 2015-07-10           9
    [38;5;246m# … with 761 more rows[39m
    

    `summarise()` ungrouping output (override with `.groups` argument)
    
    

    [38;5;246m# A tibble: 12 x 2[39m
       arrival_month travelers
               [3m[38;5;246m<dbl>[39m[23m     [3m[38;5;246m<int>[39m[23m
    [38;5;250m 1[39m             1       316
    [38;5;250m 2[39m             2       563
    [38;5;250m 3[39m             3       486
    [38;5;250m 4[39m             4       778
    [38;5;250m 5[39m             5       557
    [38;5;250m 6[39m             6       722
    [38;5;250m 7[39m             7      [4m1[24m613
    [38;5;250m 8[39m             8      [4m1[24m939
    [38;5;250m 9[39m             9       388
    [38;5;250m10[39m            10       489
    [38;5;250m11[39m            11       204
    [38;5;250m12[39m            12       523
    


![png](output_94_4.png)


    Warning message:
    “Continuous limits supplied to discrete scale.
    Did you mean `limits = factor(...)` or `scale_*_continuous()`?”
    


![png](output_94_6.png)



![png](output_94_7.png)


From this we can conclude that individuals who travel with chidlren are more likely to travel in the later months of Peak travel season as it coincides with the start and ending months of summer vacation for students in HighSchool or below. Families with high school or below students are more likely to travel during vacations together. 

### 3.4 Scientific Question - Budget & Cost

**a) Do guests from certain countries paid more than others?** 

Now we are interested in the consumption levels of people and we are wondering what variables play an important role in the expenses in the hotels. Therefore, we mainly focus on the average daily rate as it is calculated by dividing the sum of all lodging transactions by the total number of staying nights. ADR can be viewed as a response variable and based on its definition we should also consider the total staying nights. Moreover, we only analyze the bookings that are not canceled. Firstly, we are curious whether guests from certain countries paid more than others. Unlike the mean, the median value does not depend on all the values in the dataset and consequently when some of the values are more extreme, the effect on the median is smaller. Hence we calculate the median of adr and median of the stays to compare the consumption levels of people from different countries. 


```R
adr_country <- df %>% filter(is_canceled == 0) %>%
    group_by(country_full_name) %>%
    summarize(median_adr = median(adr),
              median_stay = median(total_nights_stayed)) %>%
    top_n(10, median_adr)
```

    `summarise()` ungrouping output (override with `.groups` argument)
    
    


```R
adr_country %>% ggplot(aes(fct_reorder(country_full_name,desc(median_adr)),
            median_adr, fill = as.factor(median_stay))) + geom_col() +
        geom_text(aes(label=country_full_name, y=3), 
                  angle = 90, color = "white", size = 4.5, hjust=0) +
        labs(x = 'Country', y= "Median Average Daily Rate(ADR)",
             title = "Guests From the Top 10 Countries by Median ADR",
             fill = "Median Stay (in nights)") +
        theme(axis.text.x = element_blank(),
              axis.ticks.x = element_blank(),
              axis.text.y = element_text(size=13),
              axis.title = element_text(size = 14),
              plot.title = element_text(size=19, face="bold"),
              panel.grid = element_blank(),
              legend.position = c(0.75,0.8),
              legend.direction = "vertical",
              legend.background = element_blank())
```


![png](output_100_0.png)


The plot above shows the top ten countries ranked by median ADR. Andorra ranked first and also had high median staying nights of 4.5. Since Djibouti had the second highest median ADR with one median stay, we cannot conclude that there is a positive or negative relationship between stays and ADR.

**b) Does room type affect the cost in hotels?** 

Then we want to know if the room type affects cost in hotels. Here we consider all the values in this dataset and focus on the mean of ADR and stays. Sometimes the assigned room type differs from the reserved room type due to hotel operation reasons (e.g. overbooking) or by customer request. So we classify the bookings by assigned room type.


```R
df %>% filter(is_canceled == 0) %>%
    group_by(assigned_room_type) %>%
    summarize(average_adr = mean(adr),
              average_stay = mean(total_nights_stayed)) %>%
    arrange(desc(average_adr, average_stay))
```

    `summarise()` ungrouping output (override with `.groups` argument)
    
    


<table>
<caption>A tibble: 10 × 3</caption>
<thead>
	<tr><th scope=col>assigned_room_type</th><th scope=col>average_adr</th><th scope=col>average_stay</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>H</td><td>158.97623</td><td>3.214912</td></tr>
	<tr><td>G</td><td>158.31018</td><td>3.511086</td></tr>
	<tr><td>F</td><td>142.24091</td><td>3.331551</td></tr>
	<tr><td>E</td><td>111.76430</td><td>4.167417</td></tr>
	<tr><td>C</td><td>106.82687</td><td>3.786387</td></tr>
	<tr><td>D</td><td>101.25626</td><td>3.572363</td></tr>
	<tr><td>B</td><td> 94.46663</td><td>2.791641</td></tr>
	<tr><td>A</td><td> 92.81801</td><td>3.218492</td></tr>
	<tr><td>K</td><td> 52.77461</td><td>3.640449</td></tr>
	<tr><td>I</td><td> 40.53798</td><td>3.051136</td></tr>
</tbody>
</table>




```R
ggplot(data=filter(df, is_canceled==0), aes(x=assigned_room_type, y=adr, fill=assigned_room_type)) + 
    geom_boxplot(alpha=0.7) +
    stat_summary(fun=mean, geom="point", shape=18, size=3, color="red") +
    labs(x='Assigned Room Type', y='ADR',
         title='ADR by Assigned Room Type')+
    theme(axis.text.x = element_text(size = 14),
          axis.text.y = element_text(size = 14),
          axis.title = element_text(size = 14),
          plot.title = element_text(size=19, face="bold", hjust=0.5),
          legend.position="none")
```


![png](output_105_0.png)



```R
table(df$reserved_room_type)
```


    
        A     B     C     D     E     F     G     H     L     P 
    85599  1118   931 19172  6497  2889  2083   601     6     2 


Although type A rooms have the largest number of bookings, the table and boxplot above show that people who stay for three to four nights and are assigned with rooms G and H contribute to higher lodging transactions than other room types. Both median ADR and mean ADR of type G & H are greater than 150.  Then we wonder if those paying more overall seemed to choose these two types of rooms. And we begin to focus on the reserved room type, which may directly indicate customers' preferences and travel plans.


```R
room <- df %>% filter(is_canceled == 0,!reserved_room_type %in% c('P','L')) %>%
  group_by(reserved_room_type,country_full_name) %>%
  summarize(median_adr = median(adr),
            median_stay = median(total_nights_stayed)) %>%
  group_by(reserved_room_type) %>%
  top_n(5, wt=median_adr)
```

    `summarise()` regrouping output by 'reserved_room_type' (override with `.groups` argument)
    
    


```R
room %>% ggplot(aes(country_full_name, median_adr, fill = as.factor(median_stay))) +
    geom_col(show.legend = T) +
    facet_wrap(~reserved_room_type, scales = 'free') +
    geom_text(aes(label=country_full_name, y=3), angle=90, color="white", size=3, hjust=0) +
    labs(x = 'Country', y= "Median ADR",
       title = "ADR of Top 5 countries by Reserved Room Type",
       fill = "Median Stay (in nights)") +
    theme(axis.text.x = element_blank(),
              axis.ticks.x = element_blank(),
              axis.text.y = element_text(size = 13),
              axis.title = element_text(size = 14),
              plot.title = element_text(size=19, face="bold"),
              panel.grid = element_blank(),
              legend.position = c(0.85,0.2),
              legend.direction = "vertical",
              legend.background = element_blank()) +
    guides(fill = guide_legend(title.position = "top", nrow = 4))

```


![png](output_109_0.png)


As can be seen from the plot, customers from Andorra have highest median ADR among those who booked room G. This result is consistant with what we find before. We assume that type G & H may represent rooms with higher price compared with other types. And customers who reserved those two have big budget and therefore contribute to high ADR. This assumption makes sense since digging a little deeper reveals those paying more overall seemed to be springing for pricier rooms.

**c) What is the relationship between meal type, adr, special reuqests and stayed nights?**

As the table and pir chart illustrate, customers who book full board (breakfast, lunch and dinner) have highest average ADR as well as average staying nights. Average number of special requests made by the customer (e.g. twin bed or high floor) is below one and deos not have clear impacts on ADR when dividing the data by meal type.


```R
adr_meal <- df %>% filter(is_canceled == 0) %>%
    group_by(meal) %>%
    summarize(average_adr = mean(adr),
              average_special_requests = mean(total_of_special_requests),
              average_stay = mean(total_nights_stayed)) %>%
    arrange(desc(average_adr, average_special_requests))
adr_meal
```

    `summarise()` ungrouping output (override with `.groups` argument)
    
    


<table>
<caption>A tibble: 5 × 4</caption>
<thead>
	<tr><th scope=col>meal</th><th scope=col>average_adr</th><th scope=col>average_special_requests</th><th scope=col>average_stay</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>FB       </td><td>133.30966</td><td>0.5218750</td><td>4.668750</td></tr>
	<tr><td>HB       </td><td>118.56859</td><td>0.6487887</td><td>4.405903</td></tr>
	<tr><td>SC       </td><td> 97.44461</td><td>0.8799581</td><td>2.714713</td></tr>
	<tr><td>BB       </td><td> 97.43157</td><td>0.7160724</td><td>3.288683</td></tr>
	<tr><td>Undefined</td><td> 93.24717</td><td>0.1911263</td><td>4.410694</td></tr>
</tbody>
</table>




```R
ggplot(adr_meal, aes(x="", y=average_adr, fill=meal)) +
    geom_bar(stat="identity", width=1) +
    coord_polar("y", start=0) +
    scale_fill_brewer(palette="Blues") +
    labs(title = "Average ADR by Meal Type") +
    theme_minimal() +
    theme(plot.title = element_text(size=19, face="bold", hjust=0.5))
```


![png](output_114_0.png)


**d) What is the optimal length of stay in order to get the best daily rate?**

From the perspective of hotels, it is useful to know the optimal length of staying nights to get the best daily rate and improve their profits. So we calculate the total number of bookings and total ADR for different staying nights based on the hotel types.


```R
city_nights_stay <- filter(df, is_canceled == 0) %>%
    filter(total_nights_stayed!=0) %>%
    filter(hotel == 'City Hotel') %>%
    group_by(total_nights_stayed) %>%
    summarize(Hotel = hotel, count = n(), total_adr = sum(adr)) %>%
    unique() %>%
    arrange(desc(total_adr)) %>% print
```

    `summarise()` regrouping output by 'total_nights_stayed' (override with `.groups` argument)
    
    

    [38;5;246m# A tibble: 32 x 4[39m
    [38;5;246m# Groups:   total_nights_stayed [32][39m
       total_nights_stayed Hotel      count total_adr
                     [3m[38;5;246m<int>[39m[23m [3m[38;5;246m<fct>[39m[23m      [3m[38;5;246m<int>[39m[23m     [3m[38;5;246m<dbl>[39m[23m
    [38;5;250m 1[39m                   3 City Hotel [4m1[24m[4m1[24m893  1[4m2[24m[4m7[24m[4m8[24m806.
    [38;5;250m 2[39m                   2 City Hotel [4m1[24m[4m0[24m991  1[4m1[24m[4m6[24m[4m2[24m637.
    [38;5;250m 3[39m                   1 City Hotel  [4m9[24m169   [4m9[24m[4m6[24m[4m5[24m927.
    [38;5;250m 4[39m                   4 City Hotel  [4m7[24m703   [4m8[24m[4m2[24m[4m0[24m220.
    [38;5;250m 5[39m                   5 City Hotel  [4m3[24m221   [4m3[24m[4m4[24m[4m4[24m349.
    [38;5;250m 6[39m                   7 City Hotel  [4m1[24m251   [4m1[24m[4m3[24m[4m5[24m453.
    [38;5;250m 7[39m                   6 City Hotel  [4m1[24m116   [4m1[24m[4m2[24m[4m1[24m563.
    [38;5;250m 8[39m                   8 City Hotel   209    [4m2[24m[4m2[24m179.
    [38;5;250m 9[39m                   9 City Hotel   120    [4m1[24m[4m3[24m542.
    [38;5;250m10[39m                  10 City Hotel    83     [4m8[24m393.
    [38;5;246m# … with 22 more rows[39m
    


```R
resort_nights_stay <- filter(df, is_canceled == 0) %>%
    filter(total_nights_stayed != 0) %>%
    filter(hotel == 'Resort Hotel') %>%
    group_by(total_nights_stayed) %>%
    summarize(Hotel = hotel, count = n(), total_adr = sum(adr)) %>%
    unique() %>%
    arrange(desc(total_adr)) %>% print
```

    `summarise()` regrouping output by 'total_nights_stayed' (override with `.groups` argument)
    
    

    [38;5;246m# A tibble: 32 x 4[39m
    [38;5;246m# Groups:   total_nights_stayed [32][39m
       total_nights_stayed Hotel        count total_adr
                     [3m[38;5;246m<int>[39m[23m [3m[38;5;246m<fct>[39m[23m        [3m[38;5;246m<int>[39m[23m     [3m[38;5;246m<dbl>[39m[23m
    [38;5;250m 1[39m                   1 Resort Hotel  [4m6[24m369   [4m4[24m[4m7[24m[4m4[24m995.
    [38;5;250m 2[39m                   7 Resort Hotel  [4m4[24m427   [4m4[24m[4m3[24m[4m0[24m726.
    [38;5;250m 3[39m                   2 Resort Hotel  [4m4[24m382   [4m3[24m[4m9[24m[4m4[24m576.
    [38;5;250m 4[39m                   3 Resort Hotel  [4m3[24m796   [4m3[24m[4m5[24m[4m7[24m971.
    [38;5;250m 5[39m                   4 Resort Hotel  [4m3[24m303   [4m3[24m[4m1[24m[4m6[24m236.
    [38;5;250m 6[39m                   5 Resort Hotel  [4m1[24m889   [4m2[24m[4m0[24m[4m7[24m994.
    [38;5;250m 7[39m                   6 Resort Hotel  [4m1[24m199   [4m1[24m[4m4[24m[4m4[24m859.
    [38;5;250m 8[39m                  10 Resort Hotel   695    [4m7[24m[4m0[24m595.
    [38;5;250m 9[39m                  14 Resort Hotel   627    [4m5[24m[4m4[24m700.
    [38;5;250m10[39m                   8 Resort Hotel   507    [4m5[24m[4m4[24m159.
    [38;5;246m# … with 22 more rows[39m
    


```R
table(df$hotel)
```


    
      City Hotel Resort Hotel 
           79303        39595 



```R
stay_df <- city_nights_stay %>% 
    inner_join(resort_nights_stay, by=c('total_nights_stayed'='total_nights_stayed')) %>%
    mutate(hotel_with_higher_adr = ifelse(total_adr.x > total_adr.y, 'City Hotel', 'Resort Hotel')) 
head(stay_df)
```


<table>
<caption>A grouped_df: 6 × 8</caption>
<thead>
	<tr><th scope=col>total_nights_stayed</th><th scope=col>Hotel.x</th><th scope=col>count.x</th><th scope=col>total_adr.x</th><th scope=col>Hotel.y</th><th scope=col>count.y</th><th scope=col>total_adr.y</th><th scope=col>hotel_with_higher_adr</th></tr>
	<tr><th scope=col>&lt;int&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>3</td><td>City Hotel</td><td>11893</td><td>1278806.1</td><td>Resort Hotel</td><td>3796</td><td>357970.9</td><td>City Hotel  </td></tr>
	<tr><td>2</td><td>City Hotel</td><td>10991</td><td>1162636.9</td><td>Resort Hotel</td><td>4382</td><td>394576.4</td><td>City Hotel  </td></tr>
	<tr><td>1</td><td>City Hotel</td><td> 9169</td><td> 965926.8</td><td>Resort Hotel</td><td>6369</td><td>474994.7</td><td>City Hotel  </td></tr>
	<tr><td>4</td><td>City Hotel</td><td> 7703</td><td> 820220.2</td><td>Resort Hotel</td><td>3303</td><td>316236.3</td><td>City Hotel  </td></tr>
	<tr><td>5</td><td>City Hotel</td><td> 3221</td><td> 344348.9</td><td>Resort Hotel</td><td>1889</td><td>207993.8</td><td>City Hotel  </td></tr>
	<tr><td>7</td><td>City Hotel</td><td> 1251</td><td> 135452.8</td><td>Resort Hotel</td><td>4427</td><td>430726.2</td><td>Resort Hotel</td></tr>
</tbody>
</table>



For both City Hotel and Resort Hotel, most people choose to live from one to seven days. 11893 bookings have 3 nights stayed in the City Hotel, which contributes to the highest adr and the total adr is 1278806.06. And 6369 bookings spend one night stayed in the Resort Hotel and its highest total adr is 474995. Although the number of bookings for the City Hotel is two times greater than that of the Resort Hotel, the Resort Hotel always has higher adr when total nights stayed are more than a week.

**e) How do stays, numer of car parking spaces and special requests affect the adr?**

Through the linear regression model, we can conclude that stays in week nights have greater impacts on ADR than stays in weekend nights since with one unit increase of week night stays, we expect to see 1.26 increase in adr while it is predicted to increase 0.66 with one unit increase of weekend nights. Additionally, with one unit increase of required car parking spaces and total special requests, adr is predicted to increase 9.49 and 10.41 respectively. According to the small p-values, these four are all significant variables. The adjusted R-squared compares the explanatory power of regression models that contain different numbers of predictors. However it was samll here, which implied that we needed other more variables to predict adr.


```R
mod1 <- lm(adr ~ stays_in_weekend_nights + stays_in_week_nights +
                   required_car_parking_spaces + total_of_special_requests, data = df)
summary(mod1)
```


    
    Call:
    lm(formula = adr ~ stays_in_weekend_nights + stays_in_week_nights + 
        required_car_parking_spaces + total_of_special_requests, 
        data = df)
    
    Residuals:
       Min     1Q Median     3Q    Max 
    -166.4  -31.4   -6.5   23.3 5307.0 
    
    Coefficients:
                                Estimate Std. Error t value Pr(>|t|)    
    (Intercept)                 91.69437    0.26057 351.903  < 2e-16 ***
    stays_in_weekend_nights      0.65908    0.16639   3.961 7.47e-05 ***
    stays_in_week_nights         1.26092    0.08722  14.457  < 2e-16 ***
    required_car_parking_spaces  9.49118    0.59163  16.042  < 2e-16 ***
    total_of_special_requests   10.41159    0.18277  56.965  < 2e-16 ***
    ---
    Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
    
    Residual standard error: 49.61 on 118893 degrees of freedom
    Multiple R-squared:  0.03431,	Adjusted R-squared:  0.03428 
    F-statistic:  1056 on 4 and 118893 DF,  p-value: < 2.2e-16
    


### 4. Conclusion

We define the year into three general categories, peak season, off season and regular season. Peak season is defined by the 4 continuous months with the highest number of bookings in total. Our analysis suggests that these months are May through August, which corresponds with our analysis of what type of consumer likes to travel at certain times in the year. Peak season is mainly supported by college students beginning their summer break in May and the high volume is supported further by the beginning of K-12 summer vacation in mid-june through August. By attributing the sudden rise in volume of travelers during peak season to we can then conclude that the majority of hotel users are regular travelers and not seasonal travelers. Regular travelers include business travelers and non-seasonal based travelers. This coincides with our finding that on average consumers choose city hotels more often than resorts because city hotels are more prevalent and resort hotels target a very specific audience, which often do not include business travelers. 

Performing a country specific analysis, we concluded that hotels in Germany are less likely to deal with last minute cancellations and are much more likely to have a consistent flow of customers compared to hotels based in other countries. This means that they will need fewer adjustments to handle the changes in volume of consumers. This means that hotels in Germany are less likely to face cash flow problems as their consumer base remains relatively constant. However, hotels that do not have such low cancellation rates will need to analyze consumer behaviors and observe which months tend to generate more revenue to adjust their fiscal strategy and avoid cash flow problems in the long run. Our analysis concludes that consumer behavior is mostly cyclical and behaves in a fairly predictable manner in the long run. Adjustments with these insights in consideration would greatly improve the optimization of hotel resources, allowing management to free up resources during off season. 

Our analysis of consumer booking behavior and consumer needs also demonstrate that advertising through tour agents or travel agents is a very effective avenue as the vast majority of consumers, both travelers with kids and those without choose these methods for booking their hotels. This suggests that it might be more worthwhile to invest advertising funds into other mediums so as to reach a broader audience as the TA/TO market segment could perhaps already be too saturated. Additionally, advertising to specific target audiences through the different channels could be more effective, such as investing more into Online TA to capture a larger proportion of travelers with children. This would be impactful as travelers with children have more needs and are more likely to make special requests, which could be potential avenues for revenue generation. 

By examining the trends of consumer behavior, hotels can optimize their ADR to maximize revenue while also cutting down the amount of unnecessary expenditure through optimizations based on volumes of consumers. Increasing ADR poses a risk to many hotels as it might decrease the likelihood of consumers  choosing that hotel, however, acute awareness of supply and demand can reduce the inherent risks associated with increasing ADR. We believe that the analysis will prove to be a useful tool in helping hotel management make more informed decisions in the future. 
 
Looking at different segments of the hotel data set, such as booking amounts throughout the year, reservation and cancellation behavior, style of travelling with children and babies, the budget and cost of hotel stays, we believe that our analysis will further help the tourism and travel related industries such as  like hospitality, cruising and theme parks.


```R

```
