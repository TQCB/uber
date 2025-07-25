# Uber Driver Dispatch Optimization through Clustering

## Project Goal

The primary objective of this project is to identify optimal locations for Uber to position its drivers. This is achieved by determining clusters of ride requests for every hour of every day of the week. The key success metric is to ensure that at least 90% of riders are within a 7-minute drive (approximately 3 kilometers) of a designated cluster center at any given time. This data-driven approach aims to improve Uber's operational efficiency by minimizing rider wait times and optimizing driver distribution.

## Dataset

The analysis is based on a dataset of Uber ride requests. The raw data contains information on the latitude, longitude, and timestamp of each ride. Prior to analysis, the data was cleaned and new features such as `hour`, `day`, `year`, `month`, and `dayday` were engineered from the timestamp to facilitate time-based analysis.

## Methodology

### 1. Exploratory Data Analysis (EDA)

A preliminary analysis was conducted on a 0.5% sample of the data to visualize the geographic distribution of ride requests. The ride volume was also analyzed across different hours of the day and days of the week, revealing distinct patterns between weekdays and weekends and a general increase in rides as the week progresses.

### 2. Clustering Algorithm Evaluation

To determine the most suitable clustering algorithm for this task, several models were evaluated on a sample of 10,000 data points from a specific day and hour:

*   **DBSCAN:** This algorithm struggled with the variable density of the data, resulting in large, irregular clusters and a significant number of outliers. Its O(n^2) memory complexity also posed a challenge for a large dataset.
*   **HDBSCAN:** While theoretically more robust than DBSCAN in handling variable densities, it also produced irregular clusters that were not ideal for practical use.
*   **K-Means & Mini-Batch K-Means:** Both K-Means variants performed well, creating mostly regular-shaped clusters. Mini-Batch K-Means offered a slight speed improvement but at the cost of a performance trade-off.
*   **BIRCH (Balanced Iterative Reducing and Clustering using Hierarchies):** This algorithm was ultimately selected. Despite K-Means showing slightly better performance metrics in some cases, BIRCH was chosen for its:
    *   **Memory Efficiency:** Crucial for handling the large volume of data in this project.
    *   **Geographically Relevant Clusters:** The clusters generated by BIRCH better respected the natural geography of New York City, such as the separation of Manhattan.
    *   **Regularly Sized Clusters:** The clusters were more uniform in size, which is more practical for Uber's operational planning.

### 3. Finding Optimal Clusters

A systematic approach was developed to find the optimal number of clusters for each of the 168 hours in a week:

1.  **Distance Calculation:** A function was created to calculate the 90th percentile distance of all points in a cluster to their respective centers. This directly addresses the project's primary goal.

2.  **Binary Search for Optimal Cluster Count:** For each hour of each day, a binary search was performed to find the minimum number of clusters required to satisfy the condition that the 90th percentile distance to a cluster center is less than 3 kilometers.

3.  **Cluster Region Definition:** Since BIRCH is not a centroid-based algorithm, the region of each cluster was defined by computing the convex hull of the points within it.

The final output is a comprehensive dataset that, for every hour of every day, provides:
*   The optimal number of clusters.
*   For each cluster:
    *   The percentage of total rides.
    *   The coordinates of the cluster center.
    *   The vertices defining the cluster's geographical region (convex hull).
    *   The surface area of the region.

## Technologies Used

*   Python
*   Pandas
*   NumPy
*   Scikit-learn (for clustering algorithms)
*   Plotly (for visualizations)
*   SciPy (for convex hull computation)

## Example Output

Below is an example of the output for a Tuesday at 6 PM (Day 1, Hour 18), showing the distribution of rides across the identified clusters.

To satisfy our conditions, our algorithm found 31 clusters on day 1 and hour 18.
Here is the percentage of points assigned to each cluster (which gives UBER an idea of how to distribute their drivers):

| | points_per |
| :--- | :--- |
| **6** | 25.150 |
| **0** | 24.460 |
| **30**| 16.518 |
| **25**| 10.584 |
| **1** | 4.108 |

...and the smallest clusters:

| | points_per |
| :--- | :--- |
| **18** | 0.008 |
| **26** | 0.004 |
| **24** | 0.002 |
| **23** | 0.002 |
| **19** | 0.002 |

This information allows Uber to dynamically allocate drivers based on the expected ride demand in each defined region, thereby improving service quality and operational efficiency.