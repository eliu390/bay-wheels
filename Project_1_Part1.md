## Part 1 - Querying Data with BigQuery

### Initial Queries
- What's the size of this dataset? (i.e., how many trips)
  * The dataset contains records on 983,648 trips.
  

  ```SQL
  #standard SQL
  SELECT COUNT(*) AS total_trips
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  ```
  | total_trips |
  | ----------- |
  | 983648      |

- What is the earliest start date and time and latest end date and time for a trip?
   * The earliest that a recorded trip started was on 08-29-2013 at 9:08:00 UTC.
   * The latest that a recorded trip ended was on 08-31-2016 at 23:48:00 UTC.


  ```SQL
  #standard SQL
  SELECT MIN(start_date) as min_start_date
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  ```
  | min_start_date          |
  | ----------------------- |
  | 2013-08-29 09:08:00 UTC |
  
  ```SQL
  #standard SQL
  SELECT MAX(end_date) as max_end_date
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  ```
  | max_end_date            |
  | ----------------------- |
  | 2016-08-31 23:48:00 UTC |

- How many bikes are there?
  * There are a total of 700 bikes tracked in this dataset.
  
  
  ```SQL
  #standard SQL
  SELECT COUNT(DISTINCT bike_number) as number_of_bikes
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  ```
  | number_of_bikes |
  | --------------- |
  | 700             |

### Additional Questions
- What were the most common stations to start and end trips in 2015?
  * The Caltrain station at Townsend/4th is the most common starting and ending station.
  

  ```SQL
  #standard SQL
  SELECT start_station_name, COUNT(start_station_name) AS trip_starts
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  WHERE EXTRACT(YEAR FROM start_date) = 2015
  GROUP BY start_station_name
  ORDER BY trip_starts DESC
  LIMIT 3
  ```
  | start_station_name                       | trip_starts |
  | ---------------------------------------- | ----------- |
  | San Francisco Caltrain (Townsend at 4th) | 24827       |
  | San Francisco Caltrain 2 (330 Townsend)  | 22274       |
  | Harry Bridges Plaza (Ferry Building)     | 17344       |
  
  ```SQL
  #standard SQL
  SELECT end_station_name, COUNT(end_station_name) AS trip_ends
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  WHERE EXTRACT(YEAR FROM start_date) = 2015
  GROUP BY end_station_name
  ORDER BY trip_ends DESC
  LIMIT 3
  ```
  | end_station_name                         | trip_ends |
  | ---------------------------------------- | --------- |
  | San Francisco Caltrain (Townsend at 4th) | 32496     |
  | San Francisco Caltrain 2 (330 Townsend)  | 23782     |
  | Harry Bridges Plaza (Ferry Building)     | 17709     |

- What was the average trip duration in minutes in 2015?
  * The average trip lasted 16 minutes in 2015.
  

  ```SQL
  #standard SQL
  SELECT ROUND(AVG(duration_sec)/60) as avg_duration_min
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  WHERE EXTRACT(year FROM start_date) = 2015
  ```
  | avg_duration_min |
  | ---------------- |
  | 16.0             |

- Which zip codes had the most trips in 2015?
  * 94107 was the zip code with the most trips in 2015.


  ```SQL
  #standard SQL
  SELECT zip_code, COUNT(zip_code) AS zip_count
  FROM `bigquery-public-data.san_francisco.bikeshare_trips`
  WHERE EXTRACT(YEAR FROM start_date) = 2015
  GROUP BY zip_code
  ORDER BY zip_count DESC
  LIMIT 3
  ```
  | zip_code | zip_count |
  | -------- | --------- |
  | 94107    | 42215     |
  | 94105    | 20560     |
  | 94133    | 16038     |