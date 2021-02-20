# Project Bay Wheels
---

This project is an analysis of the [Lyft Bay Wheels](https://www.lyft.com/bikes/bay-wheels) program using:
  - The **static** public bikeshare dataset: [bigquery-public-data.san_francisco](https://console.cloud.google.com/marketplace/product/san-francisco-public-data/sf-bike-share?filter=solution-type:dataset&q=san%20francisco%20ford%20gobike%20share&project=w205-301508)
  - GCP: [BigQuery](https://cloud.google.com/bigquery/), [AI Platform Notebooks](https://cloud.google.com/ai-platform-notebooks)
  - [SciPy](https://www.scipy.org/index.html): NumPy, pandas, matplotlib
  - [Geopandas](https://geopandas.org/)
  - [Descartes](https://pypi.org/project/descartes/)

The bikeshare program offers various deals and options that change over time. Visit the [website](https://help.baywheels.com/hc/en-us/sections/360006816911-Passes-memberships) to view current offers and other marketing information. Frequent offers include:
  * Single Ride 
  * Monthly Membership
  * Annual Membership
  * Bike Share for All
  * Access Pass
  * Corporate Membership
  * etc.

### Project Outline
---

The goal of this project is to answer the following questions: 
  - What are the 5 most popular "commuter trips"? 
  - Are there any additional promotional deals that are likely to be successful?

This project is divided into 3 parts:
  - [Part 1](Project_1_Part1.md): Basic exploratory queries using BigQuery UI
  - [Part 2](Project_1_Part2.md): Main project queries in AI Platform Notebook
  - [Part 3](Project_1.ipynb): Further analysis and data visualization
  
### Summary of Results
---

- Commuter trips are subscriber trips that end between 7-10am or 4-7pm on weekdays. 
   - The [5 most popular commuter trips](Project_1_Part2.md#Question-3---Commuter-Trips) are all located within San Francisco.
- [Proposal 1](Project_1.ipynb#p1): Group discount for Access Pass
- [Proposal 2](Project_1.ipynb#p2): Extended ride time for certain trips
- [Proposal 3](Project_1.ipynb#p3): Adjust rates based on current or predicted stress