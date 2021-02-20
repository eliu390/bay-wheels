## Part 2 - Querying Data Using BigQuery CLI

### Initial Queries

1. Repeat of the first 3 queries from Part 1 using the bq command line tool:
  - What's the size of this dataset? (i.e., how many trips)
    - The dataset contains records on 983,648 trips.

  
  ```
  bq query --use_legacy_sql=false '
      SELECT COUNT(*) AS total_trips
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
  ```
  | total_trips |
  | ----------- |
  | 983648      |
   
  - What is the earliest start date and time and latest end date and time for a trip?
     - The earliest that a recorded trip started was on 08-29-2013 at 9:08am PST.
     - The latest that a recorded trip ended was on 08-31-2016 at 11:48pm PST.
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT MIN(start_date) as min_start_date
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
  ```
  |   min_start_date    |
  | ------------------- |
  | 2013-08-29 09:08:00 |

  ```
  bq query --use_legacy_sql=false '
      SELECT MAX(end_date) as max_end_date
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
  ```
  |     max_end_date    |
  | ------------------- |
  | 2016-08-31 23:48:00 |

  - How many bikes are there?
    - There are a total of 700 bikes tracked in this dataset.

  
  ```
  bq query --use_legacy_sql=false '
      SELECT COUNT(DISTINCT bike_number) as number_of_bikes
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
  ```
  | number_of_bikes |
  | --------------- |
  |             700 |

2. New query:
  - How many trips are in the morning vs in the afternoon?
    - Dataset contains:
      * 387,728 morning trips, defined as trips ending in the interval 6:00am - 11:59am PST. 
      * 379,777 afternoon trips, defined as trips ending in the interval 12:00pm - 5:59pm PST.
      * 216,143 trips ending at other times of day.
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT (CASE
          WHEN EXTRACT(
              HOUR FROM end_date
          ) BETWEEN 6 AND 11 THEN "morning"
          WHEN EXTRACT(
              HOUR FROM end_date
          ) BETWEEN 12 AND 17 THEN "afternoon"
          ELSE "other"
      END) AS times,
      COUNT(*) AS num_trips
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      GROUP BY times'
  ```
  | times     | num_trips |
  | --------- | --------- |
  | morning   | 387728    | 
  | afternoon | 379777    |
  | other     | 216143    |

### Project Questions

The main questions to answer in order to make recommendations:

- [Question 1](#Question-1---Trip-Distributions): What is the distribution of trips throughout the day for Bay Area subscribers vs. non-subscribers, on weekdays vs. weekends?

- [Question 2](#Question-2---Popular-Trips): What are the 10 most popular trips (start and end station pairs) for Bay Area subscribers vs. non-subscribers, on weekdays vs. weekends?

- [Question 3](#Question-3---Commuter-Trips): Based on results of previous questions, how are commuter trips defined? What are the 10 most popular commuter trips?
  
- [Question 4](#Question-4---Identical-Trips): Are a significant fraction of non-subscribers making identical trips (same time, same start/end station)?
  
- [Question 5](#Question-5---Bike-Shortages-and-Surpluses): Do some stations tend to have bike shortages or bike surpluses?

### Queries for Project Questions
#### Question 1 - Trip Distributions
([back to Project Questions](#Project-Questions))
- Question 1.1: How many **weekday** trips occurred in the morning, afternoon, evening, and night for Bay Area **subscribers**?
  - Answer: Bay Area subscribers took 313,257 trips in the morning (6am - noon)
  - 247,214 trips in the afternoon (noon - 6pm)
  - 144,607 trips in the evening (6pm - midnight)
  - 6,968 trips in the night (midnight - 6am)
  - SQL query:
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT (CASE
          WHEN EXTRACT(
              HOUR FROM end_date
          ) BETWEEN 6 AND 11 THEN "morning"
          WHEN EXTRACT(
              HOUR FROM end_date
          ) BETWEEN 12 AND 17 THEN "afternoon"
          WHEN EXTRACT(
              HOUR FROM end_date
          ) >= 18 THEN "evening"
          WHEN EXTRACT(
              HOUR FROM end_date
          ) <= 5 THEN "night"
          ELSE "other"
      END) AS trip_time,
      COUNT(*) AS num_trips
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE subscriber_type = "Subscriber"
      AND EXTRACT(
          DAYOFWEEK FROM end_date
      ) BETWEEN 2 AND 6
      AND SUBSTRING(zip_code, 1, 2) = "94"
      GROUP BY trip_time'
  ```
  | times     | num_trips |
  | --------- | --------- |
  | morning   | 313257    | 
  | afternoon | 247214    |
  | evening   | 144607    |
  | night     | 6968      |

- Question 1.2: How many **weekend** trips occurred in the morning, afternoon, evening, and night for Bay Area **subscribers**?
  - Answer: Bay Area subscribers took 6,072 trips in the morning (6am - noon)
  - 11,509 trips in the afternoon (noon - 6pm)
  - 4,657 trips in the evening (6pm - midnight)
  - 792 trips in the night (midnight - 6am)
  - SQL query:
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT (CASE
          WHEN EXTRACT(
              HOUR FROM end_date
          ) BETWEEN 6 AND 11 THEN "morning"
          WHEN EXTRACT(
              HOUR FROM end_date
          ) BETWEEN 12 AND 17 THEN "afternoon"
          WHEN EXTRACT(
              HOUR FROM end_date
          ) >= 18 THEN "evening"
          WHEN EXTRACT(
              HOUR FROM end_date
          ) <= 5 THEN "night"
          ELSE "other"
      END) AS trip_time,
      COUNT(*) AS num_trips
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE subscriber_type = "Subscriber"
      AND EXTRACT(
          DAYOFWEEK FROM end_date
      ) NOT BETWEEN 2 AND 7
      AND SUBSTRING(zip_code, 1, 2) = "94"
      GROUP BY trip_time'
  ```
  | times     | num_trips |
  | --------- | --------- |
  | morning   | 6072      | 
  | afternoon | 11509     |
  | evening   | 4657      |
  | night     | 792       |

- Question 1.3: How many **weekday** trips occurred in the morning, afternoon, evening, and night for Bay Area **non-subscribers**?
  - Answer: Bay Area non-subscribers took 5,629 trips in the morning (6am - noon)
  - 11,819 trips in the afternoon (noon - 6pm)
  - 5,889 trips in the evening (6pm - midnight)
  - 421 trips in the night (midnight - 6am)
  - SQL query:
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT (CASE
          WHEN EXTRACT(
              HOUR FROM end_date
          ) BETWEEN 6 AND 11 THEN "morning"
          WHEN EXTRACT(
              HOUR FROM end_date
          ) BETWEEN 12 AND 17 THEN "afternoon"
          WHEN EXTRACT(
              HOUR FROM end_date
          ) >= 18 THEN "evening"
          WHEN EXTRACT(
              HOUR FROM end_date
          ) <= 5 THEN "night"
          ELSE "other"
      END) AS trip_time,
      COUNT(*) AS num_trips
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE subscriber_type = "Customer"
      AND EXTRACT(
          DAYOFWEEK FROM end_date
      ) BETWEEN 2 AND 6
      AND SUBSTRING(zip_code, 1, 2) = "94"
      GROUP BY trip_time'
  ```
  | times     | num_trips |
  | --------- | --------- |
  | morning   | 5629      | 
  | afternoon | 11819     |
  | evening   | 5889      |
  | night     | 421       |
  
- Question 1.4: How many **weekend** trips occurred in the morning, afternoon, evening, and night for Bay Area **non-subscribers**?
  - Answer: Bay Area non-subscribers took 901 trips in the morning (6am - noon)
  - 4,579 trips in the afternoon (noon - 6pm)
  - 1,360 trips in the evening (6pm - midnight)
  - 226 trips in the night (midnight - 6am)
  - SQL query:
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT (CASE
          WHEN EXTRACT(
              HOUR FROM end_date
          ) BETWEEN 6 AND 11 THEN "morning"
          WHEN EXTRACT(
              HOUR FROM end_date
          ) BETWEEN 12 AND 17 THEN "afternoon"
          WHEN EXTRACT(
              HOUR FROM end_date
          ) >= 18 THEN "evening"
          WHEN EXTRACT(
              HOUR FROM end_date
          ) <= 5 THEN "night"
          ELSE "other"
      END) AS trip_time,
      COUNT(*) AS num_trips
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE subscriber_type = "Customer"
      AND EXTRACT(
          DAYOFWEEK FROM end_date
      ) NOT BETWEEN 2 AND 7
      AND SUBSTRING(zip_code, 1, 2) = "94"
      GROUP BY trip_time'
  ```
  | times     | num_trips |
  | --------- | --------- |
  | morning   | 901       | 
  | afternoon | 4579      |
  | evening   | 1360      |
  | night     | 226       |
  
#### Question 2 - Popular Trips
([back to Project Questions](#Project-Questions))
- Question 2.1: What were the 10 most popular **weekday** trips (start and end station pairs) for Bay Area **subscribers**?
  - Answer: Listed below.
  - SQL query:
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT
          start_station_name,
          end_station_name,
          COUNT(*) AS num_trips
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE subscriber_type = "Subscriber"
      AND EXTRACT(
          DAYOFWEEK FROM end_date
      ) BETWEEN 2 AND 6
      AND SUBSTRING(zip_code, 1, 2) = "94"
      GROUP BY 
          start_station_name,
          end_station_name
      ORDER BY num_trips DESC
      LIMIT 10'
  ```
  |              start_station_name               |             end_station_name             | num_trips |
  | --------------------------------------------- | ---------------------------------------- | --------- |
  | San Francisco Caltrain 2 (330 Townsend)       | Townsend at 7th                          |      7233 |
  | 2nd at Townsend                               | Harry Bridges Plaza (Ferry Building)     |      6366 |
  | Harry Bridges Plaza (Ferry Building)          | 2nd at Townsend                          |      5974 |
  | Townsend at 7th                               | San Francisco Caltrain 2 (330 Townsend)  |      5952 |
  | Embarcadero at Sansome                        | Steuart at Market                        |      5887 |
  | Steuart at Market                             | 2nd at Townsend                          |      5408 |
  | 2nd at South Park                             | Market at Sansome                        |      5004 |
  | Temporary Transbay Terminal (Howard at Beale) | San Francisco Caltrain (Townsend at 4th) |      4993 |
  | Harry Bridges Plaza (Ferry Building)          | Embarcadero at Sansome                   |      4960 |
  | San Francisco Caltrain (Townsend at 4th)      | Harry Bridges Plaza (Ferry Building)     |      4807 |
 
- Question 2.2: What were the 10 most popular **weekend** trips (start and end station pairs) for Bay Area **subscribers**?
  - Answer: Listed below.
  - SQL query:

  
  ```
  bq query --use_legacy_sql=false '
      SELECT
          start_station_name,
          end_station_name,
          COUNT(*) AS num_trips
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE subscriber_type = "Subscriber"
      AND EXTRACT(
          DAYOFWEEK FROM end_date
      ) NOT BETWEEN 2 AND 6
      AND SUBSTRING(zip_code, 1, 2) = "94"
      GROUP BY 
          start_station_name,
          end_station_name
      ORDER BY num_trips DESC
      LIMIT 10'
  ```
  |           start_station_name            |             end_station_name             | num_trips |
  | --------------------------------------- | ---------------------------------------- | --------- |
  | Embarcadero at Sansome                  | Harry Bridges Plaza (Ferry Building)     |       437 |
  | San Francisco Caltrain 2 (330 Townsend) | Townsend at 7th                          |       426 |
  | Harry Bridges Plaza (Ferry Building)    | Embarcadero at Sansome                   |       394 |
  | Powell Street BART                      | Market at 10th                           |       393 |
  | Embarcadero at Bryant                   | Embarcadero at Sansome                   |       388 |
  | Embarcadero at Bryant                   | Harry Bridges Plaza (Ferry Building)     |       372 |
  | Market at 10th                          | Powell Street BART                       |       339 |
  | San Francisco Caltrain 2 (330 Townsend) | 5th at Howard                            |       317 |
  | Townsend at 7th                         | San Francisco Caltrain 2 (330 Townsend)  |       309 |
  | Townsend at 7th                         | San Francisco Caltrain (Townsend at 4th) |       306 |

- Question 2.3: What were the 10 most popular **weekday** trips (start and end station pairs) for Bay Area **non-subscribers**?
  - Answer: Listed below.
  - SQL query:
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT
          start_station_name,
          end_station_name,
          COUNT(*) AS num_trips
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE subscriber_type = "Customer"
      AND EXTRACT(
          DAYOFWEEK FROM end_date
      ) BETWEEN 2 AND 6
      AND SUBSTRING(zip_code, 1, 2) = "94"
      GROUP BY 
          start_station_name,
          end_station_name
      ORDER BY num_trips DESC
      LIMIT 10'
  ```
  |            start_station_name            |           end_station_name           | num_trips |
  | ---------------------------------------- | ------------------------------------ | --------- |
  | Harry Bridges Plaza (Ferry Building)     | Embarcadero at Sansome               |       353 |
  | Harry Bridges Plaza (Ferry Building)     | Harry Bridges Plaza (Ferry Building) |       213 |
  | Embarcadero at Sansome                   | Harry Bridges Plaza (Ferry Building) |       205 |
  | Embarcadero at Sansome                   | Embarcadero at Sansome               |       184 |
  | 2nd at Townsend                          | Harry Bridges Plaza (Ferry Building) |       167 |
  | Harry Bridges Plaza (Ferry Building)     | 2nd at Townsend                      |       146 |
  | University and Emerson                   | University and Emerson               |       131 |
  | San Francisco Caltrain (Townsend at 4th) | Harry Bridges Plaza (Ferry Building) |       128 |
  | Embarcadero at Sansome                   | Steuart at Market                    |       119 |
  | Steuart at Market                        | Embarcadero at Sansome               |       118 |
  
- Question 2.4: What were the 10 most popular **weekend** trips (start and end station pairs) for Bay Area **non-subscribers**?
  - Answer: Listed below.
  - SQL query:
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT
          start_station_name,
          end_station_name,
          COUNT(*) AS num_trips
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE subscriber_type = "Customer"
      AND EXTRACT(
          DAYOFWEEK FROM end_date
      ) NOT BETWEEN 2 AND 6
      AND SUBSTRING(zip_code, 1, 2) = "94"
      GROUP BY 
          start_station_name,
          end_station_name
      ORDER BY num_trips DESC
      LIMIT 10'
  ```
  |          start_station_name          |           end_station_name           | num_trips |
  | ------------------------------------ | ------------------------------------ | --------- |
  | Harry Bridges Plaza (Ferry Building) | Embarcadero at Sansome               |       344 |
  | Harry Bridges Plaza (Ferry Building) | Harry Bridges Plaza (Ferry Building) |       282 |
  | Embarcadero at Sansome               | Embarcadero at Sansome               |       203 |
  | Embarcadero at Sansome               | Harry Bridges Plaza (Ferry Building) |       155 |
  | University and Emerson               | University and Emerson               |       145 |
  | Embarcadero at Bryant                | Embarcadero at Sansome               |       132 |
  | 2nd at Townsend                      | Harry Bridges Plaza (Ferry Building) |       128 |
  | Embarcadero at Bryant                | Embarcadero at Bryant                |       125 |
  | Mountain View Caltrain Station       | Mountain View Caltrain Station       |       115 |
  | Harry Bridges Plaza (Ferry Building) | 2nd at Townsend                      |       114 |

#### Question 3 - Commuter Trips
([back to Project Questions](#Project-Questions))
- Question 3.1: How to define commuter trips? What are the 10 most popular Bay Area commuter trips?
  - Answer: Based on the hourly distribution of rides found in [Part 3](Project_1.ipynb#q3), Bay Area commuter trips are **subscriber** trips ending between 7-10am or between 4-7pm on **weekdays**. The 10 most popular commuter trips are listed below.
  - SQL query:


  ```
  bq query --use_legacy_sql=false '
      SELECT
          start_station_name,
          end_station_name,
          COUNT(*) AS num_trips
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE subscriber_type = "Subscriber"
      AND EXTRACT(
          DAYOFWEEK FROM end_date
      ) BETWEEN 2 AND 6
      AND SUBSTRING(zip_code, 1, 2) = "94"
      AND (EXTRACT(
              HOUR FROM end_date
          ) BETWEEN 7 AND 9 
          OR EXTRACT(
              HOUR FROM end_date
          ) BETWEEN 16 AND 18)
      GROUP BY
          start_station_name,
          end_station_name
      ORDER BY num_trips DESC
      LIMIT 10'
  ```
  |              start_station_name               |             end_station_name             | num_trips |
  | --------------------------------------------- | ---------------------------------------- | --------- |
  | Harry Bridges Plaza (Ferry Building)          | 2nd at Townsend                          |      4858 |
  | 2nd at Townsend                               | Harry Bridges Plaza (Ferry Building)     |      4775 |
  | Embarcadero at Sansome                        | Steuart at Market                        |      4601 |
  | Steuart at Market                             | 2nd at Townsend                          |      4376 |
  | San Francisco Caltrain 2 (330 Townsend)       | Townsend at 7th                          |      4308 |
  | Temporary Transbay Terminal (Howard at Beale) | San Francisco Caltrain (Townsend at 4th) |      4043 |
  | San Francisco Caltrain (Townsend at 4th)      | Harry Bridges Plaza (Ferry Building)     |      4020 |
  | Steuart at Market                             | San Francisco Caltrain (Townsend at 4th) |      3988 |
  | Townsend at 7th                               | San Francisco Caltrain 2 (330 Townsend)  |      3946 |
  | Market at 10th                                | San Francisco Caltrain 2 (330 Townsend)  |      3590 |
  
- Question 3.2: What are the 10 most popular **morning** (7-10am) Bay Area commuter trips?
  - Answer: Listed below. Note the prevalence of Caltrain/BART stations among **starting** stations.
  - SQL query:
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT
          start_station_name,
          end_station_name,
          COUNT(*) AS num_trips
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE subscriber_type = "Subscriber"
      AND EXTRACT(
          DAYOFWEEK FROM end_date
      ) BETWEEN 2 AND 6
      AND SUBSTRING(zip_code, 1, 2) = "94"
      AND EXTRACT(
              HOUR FROM end_date
          ) BETWEEN 7 AND 9 
      GROUP BY
          start_station_name,
          end_station_name
      ORDER BY num_trips DESC
      LIMIT 10'
  ```
  |            start_station_name            |               end_station_name                | num_trips |
  | ---------------------------------------- | --------------------------------------------- | --------- |
  | Harry Bridges Plaza (Ferry Building)     | 2nd at Townsend                               |      4374 |
  | Steuart at Market                        | 2nd at Townsend                               |      3623 |
  | San Francisco Caltrain 2 (330 Townsend)  | Townsend at 7th                               |      2844 |
  | Market at Sansome                        | 2nd at South Park                             |      2810 |
  | Civic Center BART (7th at Market)        | Townsend at 7th                               |      2614 |
  | Steuart at Market                        | Embarcadero at Sansome                        |      2605 |
  | Harry Bridges Plaza (Ferry Building)     | Embarcadero at Sansome                        |      2564 |
  | San Francisco Caltrain (Townsend at 4th) | Harry Bridges Plaza (Ferry Building)          |      2471 |
  | San Francisco Caltrain (Townsend at 4th) | Temporary Transbay Terminal (Howard at Beale) |      2315 |
  | Mountain View Caltrain Station           | Mountain View City Hall                       |      2304 |
  
- Question 3.3: What are the 10 most popular **afternoon** (4-7pm) Bay Area commuter trips?
  - Answer: Listed below. Note the prevalence of Caltrain/BART stations among **ending** stations.
  - SQL query: 
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT
          start_station_name,
          end_station_name,
          COUNT(*) AS num_trips
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE subscriber_type = "Subscriber"
      AND EXTRACT(
          DAYOFWEEK FROM end_date
      ) BETWEEN 2 AND 6
      AND SUBSTRING(zip_code, 1, 2) = "94"
      AND EXTRACT(
              HOUR FROM end_date
          ) BETWEEN 16 AND 18
      GROUP BY
          start_station_name,
          end_station_name
      ORDER BY num_trips DESC
      LIMIT 10'
  ```
  |              start_station_name               |             end_station_name             | num_trips |
  | --------------------------------------------- | ---------------------------------------- | --------- |
  | 2nd at Townsend                               | Harry Bridges Plaza (Ferry Building)     |      3986 |
  | Embarcadero at Sansome                        | Steuart at Market                        |      3619 |
  | 2nd at South Park                             | Market at Sansome                        |      3045 |
  | Steuart at Market                             | San Francisco Caltrain (Townsend at 4th) |      2920 |
  | Temporary Transbay Terminal (Howard at Beale) | San Francisco Caltrain (Townsend at 4th) |      2859 |
  | Market at 10th                                | San Francisco Caltrain 2 (330 Townsend)  |      2805 |
  | Embarcadero at Folsom                         | San Francisco Caltrain (Townsend at 4th) |      2655 |
  | Townsend at 7th                               | San Francisco Caltrain 2 (330 Townsend)  |      2488 |
  | Townsend at 7th                               | Civic Center BART (7th at Market)        |      2342 |
  | 2nd at Townsend                               | Steuart at Market                        |      2241 |

#### Question 4 - Identical Trips
([back to Project Questions](#Project-Questions))
- Question 4.1: How many **non-subscribers** took identical trips (same time, start/end station)?
  - Answer: A total of 26,141 identical non-subscriber trips were found.
  - SQL query:
  
  
  ```
  bq query --use_legacy_sql=false '
  SELECT SUM(identical_trips) AS total_identical_trips
  FROM (
      SELECT
          start_station_name,
          end_station_name,
          start_date,
          end_date,
          COUNT(*) AS identical_trips
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE subscriber_type = "Customer"
      GROUP BY 
          start_station_name,
          end_station_name,
          start_date,
          end_date
  )
  WHERE identical_trips != 1'
  ```
  | total_identical_trips |
  | --------------------- |
  |                 26141 |
  
- Question 4.2: How many **non-subscribers** trips took place in the same period?
  - Answer: A total of 136,809 non-subscriber trips were found. Thus, identical trips represent a significant fraction of all non-subscriber trips (19%).
  - SQL query:
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT COUNT(*) AS total_trips
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE subscriber_type = "Customer"'
  ```
  | total_trips |
  | ----------- |
  |      136809 |
  
- Question 4.3: How many identical **non-subscriber** trips took place in the Bay Area?
  - Answer: A total of 4,955 non-subscriber trips were found.
  - SQL query:
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT SUM(identical_trips) AS total_identical_trips
      FROM (
          SELECT
              start_station_name,
              end_station_name,
              start_date,
              end_date,
              COUNT(*) AS identical_trips
          FROM `bigquery-public-data.san_francisco.bikeshare_trips`
          WHERE subscriber_type = "Customer"
          AND SUBSTRING(zip_code, 1, 2) = "94"
          GROUP BY 
              start_station_name,
              end_station_name,
              start_date,
              end_date
          )
      WHERE identical_trips != 1'
  ```
  | total_identical_trips |
  | --------------------- |
  |                  4955 |
  
- Question 4.4 How many total **non-subscriber** trips took place in the Bay Area in the same period?
  - Answer: A total of 38,562 non-subscriber trips were found. A lower fraction of Bay Area non-subscriber trips were identical trips (13%).
  - SQL query:


  ```
  bq query --use_legacy_sql=false '
      SELECT COUNT(*) AS total_trips
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE subscriber_type = "Customer"
      AND SUBSTRING(zip_code, 1, 2) = "94"'
  ```
  | total_identical_trips |
  | --------------------- |
  |                 38562 |

- Question 4.5: Where did the top 20 most common identical **non-subscriber** trips take place in the Bay Area?
  - Answer: Listed below.
  - SQL query:

  
  ```
  bq query --use_legacy_sql=false '
      SELECT
          start_station_name,
          end_station_name,
          SUM(identical_trips) AS total_identical_trips
      FROM (
          SELECT
              start_station_name,
              end_station_name,
              start_date,
              end_date,
              COUNT(*) AS identical_trips
          FROM `bigquery-public-data.san_francisco.bikeshare_trips`
          WHERE subscriber_type = "Customer"
          AND SUBSTRING(zip_code, 1, 2) = "94"
          GROUP BY 
              start_station_name,
              end_station_name,
              start_date,
              end_date
          )
      GROUP BY
          start_station_name,
          end_station_name
      ORDER BY total_identical_trips DESC
      LIMIT 20'
  ```
  |            start_station_name            |             end_station_name             | total_identical_trips |
  | ---------------------------------------- | ---------------------------------------- | --------------------- |
  | Harry Bridges Plaza (Ferry Building)     | Embarcadero at Sansome                   |                   697 |
  | Harry Bridges Plaza (Ferry Building)     | Harry Bridges Plaza (Ferry Building)     |                   495 |
  | Embarcadero at Sansome                   | Embarcadero at Sansome                   |                   387 |
  | Embarcadero at Sansome                   | Harry Bridges Plaza (Ferry Building)     |                   360 |
  | 2nd at Townsend                          | Harry Bridges Plaza (Ferry Building)     |                   295 |
  | University and Emerson                   | University and Emerson                   |                   276 |
  | Harry Bridges Plaza (Ferry Building)     | 2nd at Townsend                          |                   260 |
  | San Francisco Caltrain (Townsend at 4th) | Harry Bridges Plaza (Ferry Building)     |                   217 |
  | Embarcadero at Sansome                   | Steuart at Market                        |                   210 |
  | Steuart at Market                        | Embarcadero at Sansome                   |                   205 |
  | Embarcadero at Bryant                    | Embarcadero at Sansome                   |                   204 |
  | Mountain View Caltrain Station           | Mountain View Caltrain Station           |                   201 |
  | Embarcadero at Bryant                    | Embarcadero at Bryant                    |                   199 |
  | 2nd at Townsend                          | Embarcadero at Sansome                   |                   197 |
  | Harry Bridges Plaza (Ferry Building)     | San Francisco Caltrain (Townsend at 4th) |                   195 |
  | Embarcadero at Vallejo                   | Embarcadero at Sansome                   |                   193 |
  | Embarcadero at Sansome                   | 2nd at Townsend                          |                   192 |
  | Harry Bridges Plaza (Ferry Building)     | Embarcadero at Vallejo                   |                   186 |
  | 2nd at Townsend                          | 2nd at Townsend                          |                   185 |
  | Steuart at Market                        | Steuart at Market                        |                   169 |

- Question 4.6: What were the top 10 most popular starting stations for identical trips by Bay Area **non-subscribers**?
  - Answer: Listed below. A visualization can be found in [Part 3](Project_1.ipynb#q4).
  - SQL query:
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT
          start_station_name,
          SUM(identical_trips) AS total_identical_trips
      FROM (
          SELECT
              start_station_name,
              end_station_name,
              start_date,
              end_date,
              COUNT(*) AS identical_trips
          FROM `bigquery-public-data.san_francisco.bikeshare_trips`
          WHERE subscriber_type = "Customer"
          AND SUBSTRING(zip_code, 1, 2) = "94"
          GROUP BY 
              start_station_name,
              end_station_name,
              start_date,
              end_date
          )
      GROUP BY start_station_name
      ORDER BY total_identical_trips DESC
      LIMIT 10'
  ```
  |            start_station_name            | total_identical_trips |
  | ---------------------------------------- | --------------------- |
  | Harry Bridges Plaza (Ferry Building)     |                  3079 |
  | Embarcadero at Sansome                   |                  2891 |
  | San Francisco Caltrain (Townsend at 4th) |                  1818 |
  | 2nd at Townsend                          |                  1816 |
  | Steuart at Market                        |                  1368 |
  | Embarcadero at Bryant                    |                  1358 |
  | Market at 4th                            |                  1212 |
  | Embarcadero at Vallejo                   |                  1185 |
  | San Francisco Caltrain 2 (330 Townsend)  |                  1121 |
  | Market at Sansome                        |                  1086 |

#### Question 5 - Bike Shortages and Surpluses
([back to Project Questions](#Project-Questions))
- Question 5.1: Which 10 stations are the most common starting points for trips in the Bay Area?
  - Answer: Listed below. This is used in an analysis of bike deficit/surplus in [Part 3](Project_1.ipynb#q5).
  - SQL query:
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT
          start_station_name,
          COUNT(*) AS total_starts
      FROM `bigquery-public-data.san_francisco.bikeshare_trips`
      WHERE SUBSTRING(zip_code, 1, 2) = "94"
      GROUP BY start_station_name
      ORDER BY total_starts DESC
      LIMIT 10'
  ```
  |              start_station_name               | total_starts |
  | --------------------------------------------- | ------------ |
  | San Francisco Caltrain (Townsend at 4th)      |        59017 |
  | San Francisco Caltrain 2 (330 Townsend)       |        48130 |
  | Harry Bridges Plaza (Ferry Building)          |        38150 |
  | Temporary Transbay Terminal (Howard at Beale) |        37180 |
  | 2nd at Townsend                               |        35155 |
  | Steuart at Market                             |        33796 |
  | Townsend at 7th                               |        32470 |
  | Market at Sansome                             |        30561 |
  | Embarcadero at Sansome                        |        28971 |
  | Market at 10th                                |        26390 |

- Question 5.2: Which 10 stations are the most common ending points for trips in the Bay Area?
  - Answer: Listed below. This is used in an analysis of bike deficit/surplus in [Part 3](Project_1.ipynb#q5).
  - SQL query:
  
  
  ```
  bq query --use_legacy_sql=false '
        SELECT
            end_station_name,
            COUNT(*) AS total_ends
        FROM `bigquery-public-data.san_francisco.bikeshare_trips`
        WHERE SUBSTRING(zip_code, 1, 2) = "94"
        GROUP BY end_station_name
        ORDER BY total_ends DESC
        LIMIT 10'
  ```
  |               end_station_name                | total_ends |
  | --------------------------------------------- | ---------- |
  | San Francisco Caltrain (Townsend at 4th)      |      74748 |
  | San Francisco Caltrain 2 (330 Townsend)       |      50856 |
  | Harry Bridges Plaza (Ferry Building)          |      40186 |
  | 2nd at Townsend                               |      39730 |
  | Market at Sansome                             |      37066 |
  | Townsend at 7th                               |      36066 |
  | Steuart at Market                             |      33852 |
  | Temporary Transbay Terminal (Howard at Beale) |      33308 |
  | Embarcadero at Sansome                        |      30709 |
  | 2nd at South Park                             |      22285 |
  
- Question 5.3: Which 10 stations most frequently reported 0 bikes available?
  - Answer: Listed below. This is used in an analysis of bike deficit/surplus in [Part 3](Project_1.ipynb#q5-1).
  - SQL query:
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT
          name,
          latitude,
          longitude,
          COUNT(*) AS times_unavailable
      FROM `bigquery-public-data.san_francisco.bikeshare_status` AS Statuses
      JOIN `bigquery-public-data.san_francisco.bikeshare_stations` AS Stations
      ON Statuses.station_id = Stations.station_id
      WHERE bikes_available = 0
      GROUP BY 
          name,
          latitude,
          longitude
      ORDER BY times_unavailable DESC
      LIMIT 10'
  ```
  |                   name                   | latitude  |  longitude  | times_unavailable |
  | ---------------------------------------- | --------- | ----------- | ----------------- |
  | 2nd at Folsom                            | 37.785299 | -122.396236 |             44844 |
  | Commercial at Montgomery                 | 37.794231 | -122.402923 |             44728 |
  | Embarcadero at Vallejo                   | 37.799953 | -122.398525 |             35903 |
  | Embarcadero at Sansome                   |  37.80477 | -122.403234 |             32980 |
  | Clay at Battery                          | 37.795001 |  -122.39997 |             32505 |
  | San Francisco Caltrain (Townsend at 4th) | 37.776617 |  -122.39526 |             32027 |
  | Grant Avenue at Columbus Avenue          |   37.7979 | -122.405942 |             31733 |
  | Market at 4th                            | 37.786305 | -122.404966 |             30800 |
  | Howard at 2nd                            | 37.786978 | -122.398108 |             27938 |
  | Broadway St at Battery St                | 37.798541 | -122.400862 |             25496 |
  
- Question 5.4: Which 10 stations most frequently reported 0 docks available?
  - Answer: Listed below. This is used in an analysis of bike deficit/surplus in [Part 3](Project_1.ipynb#q5-1).
  - SQL query:
  
  
  ```
  bq query --use_legacy_sql=false '
      SELECT
          name,
          latitude,
          longitude,
          COUNT(*) AS times_unavailable
      FROM `bigquery-public-data.san_francisco.bikeshare_status` AS Statuses
      JOIN `bigquery-public-data.san_francisco.bikeshare_stations` AS Stations
      ON Statuses.station_id = Stations.station_id
      WHERE docks_available = 0
      GROUP BY 
          name,   
          latitude,
          longitude
      ORDER BY times_unavailable DESC
      LIMIT 10'
  ```
  |                   name                   | latitude  |  longitude  | times_unavailable |
  | ---------------------------------------- | --------- | ----------- | ----------------- |
  | San Francisco Caltrain (Townsend at 4th) | 37.776617 |  -122.39526 |             43079 |
  | Embarcadero at Bryant                    | 37.787152 | -122.388013 |             39401 |
  | Grant Avenue at Columbus Avenue          |   37.7979 | -122.405942 |             33112 |
  | Embarcadero at Sansome                   |  37.80477 | -122.403234 |             25476 |
  | San Francisco Caltrain 2 (330 Townsend)  |   37.7766 |  -122.39547 |             23605 |
  | Townsend at 7th                          | 37.771058 | -122.402717 |             18534 |
  | Harry Bridges Plaza (Ferry Building)     | 37.795392 | -122.394203 |             18036 |
  | Civic Center BART (7th at Market)        | 37.781039 | -122.411748 |             17017 |
  | Embarcadero at Vallejo                   | 37.799953 | -122.398525 |             16824 |
  | Powell Street BART                       | 37.783871 | -122.408433 |             14279 |