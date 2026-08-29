# **Title:** Smart Animal Weight Log: A Seasonal Trend Detector

### **Description**
Excel logbook of animal weights that extracts a seasonal trend curve and evaluating how well it fits the data, and how thoroughly the samples cover a needed input period. User-friendly as a regular Excel logbook, but with analytical power in its hidden columns.


### **The Problem**
The staff at my local nature center wanted to check which animals in their Excel weight logs displayed seasonal weight changes. To help them, I needed a tool that:
*   Could reliably extract seasonal changes from time-series data
*   Could handle inconsistent intervals and gaps
*   Would score the input volume and distribution, as well as the seasonal curve’s fit
*  Must be accessible to nonprogrammers (ideally built in Excel)
*  Doesn't use Macros (whose Excel security warnings might discourage use)

### **Solution**
I created an Excel worksheet which uses hidden helper columns to run a time series decomposition pipeline.
*   Helper cells split the data into a multi-year trend, a seasonal trend, and residual noise, while the user sees only the input and results.
*    Using a dropdown, the user can choose from any workbook in the document, “loading in” their columns for analysis.

<br>

# **1.** Decomposition and Trend Detection
### **Generating the Trend Curve**
Nadaraya-Watson kernel regression was used to generate a smooth trend curve from the logged points. I chose a broad 720-day moving window so short-term changes would be ignored.

The regression uses a custom kernel function for the falloff curve: **(1-(1-x)³)⁵**
*   A near-symmetric sigmoidal curve that’s aggressive near the edges and gentle around the middle.

The curve points are calculated on the same days as the weight logs so they can be subtracted to find the detrended data. Sparsely logged periods are automatically corrected: The fewer the points, the more they sway the trend curve, and the less they impact the detrended seasonal curve.

There is some **boundary bias** along the edges, as the window’s input points diminish. I chose to leave this in to prioritize the data integrity. The weight logs are noisy and irregular enough that extrapolating slopes or point density would introduce more bias than it would fix.

<br>

### **Generating the Seasonal Curve**
To get the seasonal data, I added a “Day of Year” column, which stored a “day-of-year” number from 1-366 (leap years were included.) The detrended weights were used to keep long-term changes from swaying the results.

To keep the seasonal plots comparable and consistent, the data points were resampled on regular, 7.5-day intervals using *Nadaraya-Watson kernel regression.*

**I plotted two kernel regressions:**
*  The first, with a 25-day window, condenses the messy overlapping into a single curve.
    *  This curve shows how different clusters of points or outliers did (or did not) shape the results, helping the user bridge the visual gap.
*  The second, with a 120-day window, is the robust seasonal curve.

Long gaps between logs can yield empty sample windows, creating false dips if left unchecked.
*   In cases where 0 adjacent points are detected, the window size is reset to the maximum time distance between log day-of-years.

<br>

### **Generating the Combined Trend Curve**
The best way to show the user the graphs’ accuracy is to plot the logged time series against the combined trend curve (the trend curve added to a loop of the seasonal curve.)
*   Since the trend and seasonal curves are on different intervals, an extra pass of resampling was needed to combine everything.

![Description of image](https://raw.githubusercontent.com/DrSteeeve/Data-Sci-Portfolio/refs/heads/main/SmartAnimalWeightLog/img/WeightTrendAndCombined.png)
*A) Weight over Time plotted against Trend Curve &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; B) Weight over Time against Combined Trend Curve*

<br>

# **2.** Evaluating the Seasonal Curve and Input Data
### **Scoring the Seasonal Curve’s Fit**
I used the R2 score to represent how well the seasonal curve fits the data.


### **Scoring the Average Weighted Deviation**
I used the RMSE score to represent deviation.


### **Scoring the Input Coverage**
**Objective:**
There is no perfect way to quantify the thoroughness of data coverage, so I developed a heuristic built around the nature center’s goals. Here a good seasonal score should cover multiple years, and a certain number of points per year, and an even distribution of points for each time of year.

<br>

**Base Evaluation Metric:**
For this goal, I chose a quota of 3 years and 10 points per year. The latter was derived from the *“one-in-ten”* rule in statistics, which states that one variable can be predicted for every 10 data points.

This provided a manageable way to earn a perfect score, and doesn’t penalize poorly-logged years.
*   If one staff member logs an animal for years and then leaves for a different position, their score will stand.
*   While more detailed studies would demand higher standards, this approach:
      *   Is sufficient for checking if individual animals show seasonal changes.
      *   Encourages naturalists to meet a certain quota while removing causes of frustration.

<br>

**Scoring System:**
I also wanted to measure the evenness of date distribution, so I created a “jar of marbles” scoring system.
*   There are 12 “jars,” one for each month. Each year can add a maximum of one marble (weight log) per jar. Each jar can hold a maximum of three marbles.
*   The final score is “total marbles” / 36

<br>

**Downweighing Sparsely-Logged Years:**
I wanted to downweigh less-logged years, since it’s harder to distinguish seasonal shifts when there’s fewer reference points. To do this, I assigned each year is assigned a "multiplier score", determined by its log count. The values are determined by the custom falloff score from before: (1-(1-x)³)⁵

*   *10 logs: Mult = 1* &emsp; &emsp; *5 logs: Mult = 0.513* &emsp; &emsp; *1 log: Mult = 0.001*
*   *(To avoid confusion, a full multiplier log chart is provided for the user.)*

Each year’s monthly tally is multiplied by that score.

*   A year with 5 logs total and 2 in October would have an October score of 1:
*   *5 logs yields a multiplier of 0.513. 0.513\*2 = 1.026, then trimmed down to the maximum 1.*


<img src="https://raw.githubusercontent.com/DrSteeeve/Data-Sci-Portfolio/refs/heads/main/SmartAnimalWeightLog/img/SeasonalWeightPlot.png" alt="Cute Dog" width="50%">

*The Seasonal Weight Trend*

<br>

# **3.** Additional Features
### **Confidence Input**
To accommodate naturalist doubt around particular entries, I provided a “Confidence” section can be manually downweighed. This score interpolates the logged weight with the average of the previous and subsequent valid point.

### **Custom Time Windows**
Next to the output, there is a section where the user can adjust the kernel regression’s time windows to better fit their data.
*   Recommended values are included as a reference point.


### **Visualizations**
In addition to showing the seasonal trend curve and weight over time, I wanted to provide context for what those graphs mean. Alongside the time series I showed:
*   The trend curve to show where the seasonal values come from
*   The combined trend curve (trend + looping seasonal) to show how well the curves explain the data.
*   An area plot showing how much of the animals’ weight was changing vs static.
I also added some general stats, such as time elapsed, log count by year, and a guide to the yearly multiplier scores.


### **AI Usage**
AI was used as an engineering co-pilot for optimization and code design.
*   Consulted on Excel syntax and function performance to optimize Excel’s multi-thread calculations.
*   Assisted in designing complex single-cell modules using multi-phase LET, XLOOKUP, and interpolation operations.

![Description of image](https://raw.githubusercontent.com/DrSteeeve/Data-Sci-Portfolio/main/SmartAnimalWeightLog/img/AnalysisDashboard.png)
*The Weight Pattern Analysis Dashboard*

<br>

# Connect with Me
**Author:** Steven Ostuni

**LinkedIn:** [linkedin.com/in/stevenostuni](https://www.linkedin.com/in/stevenostuni/)

**Portfolio Website:** [github.com/DrSteeeve](https://github.com/DrSteeeve/Data-Sci-Portfolio)