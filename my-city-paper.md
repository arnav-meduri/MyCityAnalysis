---
title: City Crime Model
date: 2022-1-22
authors:
  - name: Arnav Meduri
    affiliations: 
      - North Carolina School of Science and Mathematics  
  - name: Abhinav Meduri 
    affiliations:
      - North Carolina School of Science and Mathematics
---
+++ {"part": "abstract"}

Crime is a common issue in nearly all major metropolitan areas, and My City, a fictional city surrounded by a large metropolitan area, is no exception. We have developed a mathematical model to quantify safety in My City based on various attributes related to crime incidents. The factors considered in our model include:
- The severity of given types of crime.
- The prevalence of specific types of crime.
- The proportion of perpetrators arrested for a given crime.

For each individual crime in My City, we assigned a score based on these factors. To assess the relative safety, we compared these scores to a national score calculated using a similar methodology. This comparison allowed us to determine a crime score differential for each type of crime. Summing these differentials, we arrived at an overall safety index for My City, which was -71.13.

While our primary goal was to establish a quantitative measure for My City, the significance of this score lies in its comparison with other cities across the United States. Placing My City’s safety within this broader context provides valuable insights into its safety ranking relative to other cities. To maintain consistency, we tested our model on four other large cities, each with a population of over 300,000. Our comparative analysis revealed that Baltimore was the least safe city with a safety index of -101.36, while Virginia Beach ranked as the safest city, with a safety index of -0.98. My City fell in the middle as the second most dangerous city, closely followed by Chicago with an index of -60.34. Austin emerged as the second safest city with an index of -6.08.

To assess the sensitivity and robustness of our model to limited crime data, we reduced the number of crimes included in our analysis. This reduction aimed to determine if it significantly affected the overall safety index, leading to a simplified model. Most large U.S. cities have limited crime data, typically comprising seven major crimes. However, since our model relied on 12 major factors, we sought to understand the impact of using only the crimes reported in the FBI’s Uniform Crime Report—a representation of the most relevant crimes affecting safety in the United States. In addition to constraining our model to this limited crime data, we adjusted the weight of the arrest factor to reflect its importance in comparison to non-arrested crimes.

Furthermore, we conducted an analysis to evaluate safety across different regions of My City by examining the territorial distribution of crimes, as well as the proportion of domestic crimes in various areas. We found that while My City’s safety rating was lower than the national average, only a few regions experienced very high crime rates, significantly impacting the city’s overall safety.

+++

### Analysis of the Problem

We are given a log of all the crimes to occur in My City over a 2 week period from July 5th to July 18th, 2014, which contains the case number, date of occurrence, primary description, secondary description, location description, whether an arrest was made, whether the crime was domestic or not, and the beat number of the crime. Using this log, we determined the frequency of each type of crime, as well as the arrests relative to each type of crime, shown below. 
```python
import matplotlib.pyplot as plt
import pandas as pd

# Re-creating the graph in one code block

# Load the data
file_path = "./My_City_Crime_Data.csv"
data = pd.read_csv(file_path)

# Cleaning the data
data = data.drop(data.tail(5).index) # Remove informational rows
data.columns = [col.strip() for col in data.columns] # Strip spaces from column names

# Calculating the frequency of each type of crime and the number of arrests for each type of crime
crime_counts = data['PRIMARY DESCRIPTION'].value_counts()
arrests_per_crime_type = data[data['ARREST'] == 'Y']['PRIMARY DESCRIPTION'].value_counts()

# Creating a data frame for visualization
crime_arrest_df = pd.DataFrame({'Total Crimes': crime_counts, 'Arrests': arrests_per_crime_type}).fillna(0)

# Creating the horizontal bar chart
plt.figure(figsize=(12, 10));
crime_arrest_df.sort_values('Total Crimes', ascending=True).plot(kind='barh', color=['blue', 'red']);
plt.title('Frequency of Each Type of Crime and Number of Arrests');
plt.xlabel('Count');
plt.ylabel('Crime Type');
plt.tight_layout();
plt.show();
```
![](#my-cell) 

In addition to analyzing the frequency of each type of crime and the number of arrests, we also examined the number of crime incidents that were categorized as domestic versus non-domestic and those that resulted in an arrest versus no arrest, as shown in the charts below.
```python 
data = pd.read_csv(file_path)

# Cleaning the data
data = data.drop(data.tail(5).index) # Remove informational rows
data.columns = [col.strip() for col in data.columns] # Strip spaces from column names
data['ARREST'] = data['ARREST'].map({'Y': True, 'N': False})

# Calculating the frequency of domestic and non-domestic crimes
domestic_counts = data['DOMESTIC'].value_counts().rename({True: 'Domestic', False: 'Non-Domestic'})

# Calculating the number of arrest vs no arrest cases
arrest_counts = data['ARREST'].value_counts().rename({True: 'Arrest', False: 'No Arrest'})

# Creating DataFrames for visualization
domestic_df = pd.DataFrame({'Count': domestic_counts})
arrest_df = pd.DataFrame({'Count': arrest_counts})

# Setting up the figure and axes
fig, axes = plt.subplots(nrows=2, ncols=1, figsize=(10, 12))

# Plot for Domestic vs Non-Domestic Crimes
domestic_df.plot(kind='bar', color=['blue', 'red'], ax=axes[0], legend=False)
axes[0].set_title('Number of Domestic vs Non-Domestic Crimes')
axes[0].set_xlabel('Crime Type')
axes[0].set_ylabel('Count')
axes[0].set_xticklabels(['Non-Domestic', 'Domestic'], rotation=0)
axes[0].grid(axis='y')

# Plot for Arrest vs No Arrest Cases
arrest_df.plot(kind='bar', color=['red', 'blue'], ax=axes[1], legend=False)
axes[1].set_title('Number of Arrest vs No Arrest Cases')
axes[1].set_xlabel('Case Type')
axes[1].set_ylabel('Count')
axes[1].set_xticklabels(['No Arrest', 'Arrest'], rotation=0)
axes[1].grid(axis='y')

# Adjust layout and display
plt.tight_layout()
plt.show()
```
![](#my-cell-2)

We also analyzed the number of total crimes over the 14-day period, shown below in **Figure 3**:
```python
import matplotlib.dates as mdates

data = pd.read_csv(file_path)

# Convert 'DATE  OF OCCURRENCE' to datetime format
data['DATE  OF OCCURRENCE'] = pd.to_datetime(data['DATE  OF OCCURRENCE'], errors='coerce')

# Filtering data for a two-week time span
start_date = pd.to_datetime("2014-07-01")
end_date = pd.to_datetime("2014-07-14")
two_week_data = data[(data['DATE  OF OCCURRENCE'] >= start_date) & (data['DATE  OF OCCURRENCE'] <= end_date)]

# Counting the number of crimes for each day
daily_crime_counts = two_week_data['DATE  OF OCCURRENCE'].dt.date.value_counts().sort_index()

# Plotting the daily crime counts
plt.figure(figsize=(12, 6))
plt.plot(daily_crime_counts, marker='o', color='blue')
plt.title('Number of Crimes Over Each Day of a Two-Week Time Span')
plt.xlabel('Date')
plt.ylabel('Number of Crimes')
plt.xticks(rotation=45)
plt.gca().xaxis.set_major_formatter(mdates.DateFormatter('%Y-%m-%d'))
plt.gca().xaxis.set_major_locator(mdates.DayLocator())
plt.grid(True)
plt.tight_layout()
plt.show()
```
![](#my-cell-3)

### Design and Justification of Model

We first wanted to consider that not all crimes are equally severe. For example, homicide is a significantly more harmful crime than theft. A crime’s impact on safety depends on its severity, so we assigned a severity score for each crime. We used the average sentencing length from the United States Bureau of Justice Statistics to determine how severe a crime is, as this will account for variation between incidents of a particular crime. For instance, possessing a drug is less severe than distributing it. We also chose sentencing length rather than time spent in prison so penalties for repeat offenses would be ignored, and shorter prison sentences due to parole would also be ignored since these are unrelated to the crime. The severity of each crime is a constant and will be the same for all cities. The severity of each crime is shown in the table below.

:::{table} Severity scores were assigned to various offenses based on the average sentencing length for each crime, as issued by the United States Bureau of Justice Statistics in 2014.
:label: table
:align: center

| Offense | Severity Score |
| --- | --- |
| Theft | 3.7 |
| Narcotics | 4.0 |
| Motor Vehicle Theft | 4.0 |
| Criminal Damage | 4.5 |
| Weapons Violation | 4.6 |
| Assault | 5.6 |
| Burglary | 5.8 |
| Arson | 7.7 |
| Robbery | 9.0 |
| Battery | 10.1 |
| Criminal Sexual Assault | 12.2 |
| Homicide | 40.6 |

:::

Additionally, we wanted to consider the frequency of crime in our model, as more crimes generally indicate that a city is less safe. Since cities with higher populations generally have more crimes than smaller cities, we used a frequency count of crimes per 100,000 people, as this is the standard metric for measuring crime prevalence across varying populations. To account for datasets covering varying lengths of time, we used the count per 100,000 people per day (CHPD) to measure crime prevalence in our model. See Equation {eq}`my-equation-1`.

```{math}
:label: my-equation-1
CHPD = \frac{100,000 \times I}{PD}
```
CHPD = Crime count per 100,000 people per day \
I = Total number of crime incidents \
P = Population \
D = Number of days 

A city with more arrests is generally safer than one with fewer arrests, since it potentially has fewer perpetrators to commit crimes. Our model accounts for the influence of how many crimes result in arrests by incorporating an “arrest mitigation factor.” Crimes that result in arrests are weighed less than crimes in which the perpetrator is not cleared for an arrest.

Using the equation below, we combined these factors to create a “crime score” for each crime category. We chose to calculate a score for each category to provide more granularity. This could be useful to government agencies attempting to analyze the impact of a specific type of crime, or to civilians trying to avoid certain types of crime. See Equation {eq}`my-equation-2`.

```{math}
:label: my-equation-2
\text{CS} = S(0.8N A + N(1 - A))
```
**CS** = Crime score \
**S** = Severity of crime \
**N** = Incident frequency in CHPD \
**D** = Arrest percentage

We separate the crimes based on how many resulted in arrests by multiplying the frequency by the arrest percentage. Crimes with arrests are multiplied by a mitigation factor of 0.8 since they are generally considered less impactful on the community, as stated in Assumption 3.

To make our model easily interpretable, we subtracted the crime score calculated for a specific crime in a given city from the corresponding national score for that crime. The national crime score, CS{sub}`n`, for each crime category, was determined using the same formula applied at the city level; however, for the national calculation, CHPD was computed using the entire U.S. population in 2015 (**P = 320 million**) and the total number of days in the year (**D = 365**). This approach is based on the nature of the data from the FBI’s Uniform Crime Report for 2015, which encompasses an entire year. Additionally, since the UCR data includes reported crime incidents for roughly 30% of the total U.S. population, we adjusted the frequency of incidents to reflect the size of the entire population by scaling the incident numbers by a factor of 10/3. 

In our model, a negative value compared to the national average for a specific crime indicates that the city is less safe for a given crime than the national average, and a positive value compared to the national average indicates that the city is safer for a specific crime. See Equation {eq}`my-equation-3`

```{math}
:label: my-equation-3
CSD = CS_n - CS_x
```
CSD = Crime score differential \
CS{sub}`n` = National crime score \
N = Crime score for any city 

The overall safety index for a given city is the sum of the CSDs of all the significant crimes. See Equation {eq}`my-equation-4`.

```{math}
:label: my-equation-4
SI = \sum CSD
```
The complete model is shown below, where the sum is for all 12 crimes. See Equation {eq}`my-equation-5`. 

```{math}
:label: my-equation-5
SI = \sum (CS_n - S(0.8N A + N(1 - A)))
```
:::{table} National crime score (CS{sub}`n`) values for each of the 12 significant crimes. Theft has the highest CS{sub}`n` value (17.01) whereas arson has the lowest CSn value (0.28). The CS{sub}`n` values will be used as a benchmark to determine how far a city’s individual crimes are from the national average. 
:label: severity scores table
:align: center

| Offense | Severity Score |
| --- | --- |
| Theft | 17.01 |
| Narcotics | 5.39 |
| Motor Vehicle Theft | 2.04 |
| Criminal Damage | 8.84 |
| Weapons Violation | 1.00 |
| Assault | 9.51 |
| Burglary | 7.22 |
| Arson | 0.28 |
| Robbery | 1.64 |
| Battery | 4.32 |
| Criminal Sexual Assault | 1.47 |
| Homicide | 0.49 |

:::

The step-by-step process to calculate the safety index of a city is outlined in the figure below. 

```{figure} ./flowchart.png
:name: myFigure
:alt: Image depicting methodology
:align: center

Flowchart of the process to determine safety index. Our model takes as inputs the number of incidents of each significant crime, population, time period in days, arrest percentage, and severity of each crime. It calculates various intermediate values, such as crime per hundred thousand people per day, and the crime score for each crime. Then, it calculates a crime score differential for a given type of crime, based on the national score for that crime. The crime score differentials for each type of crime are summed together to determine the overall safety index of a city.
```
### Application and Testing of Model

Our model indicates that My City has a Safety Index of -71.13. This negative value suggests that My City is less safe than the national average. To properly interpret this value, comparing it to real-life cities and their known safety reputations is essential. Given My City’s large population of 2.8 million, we chose to compare it with other large cities. We defined a large city as one with a population of over 300,000, following the standard used by Forbes. We selected Chicago for comparison due to its similar population size of 2.7 million. We also included Baltimore, as it is ranked the third most dangerous city in the country by Forbes and had more available data than the two cities ranked as less safe. Additionally, we chose Austin, Texas, and Virginia Beach, Virginia, since Forbes ranked them as the 15th and 2nd safest large cities, respectively. These cities were selected because they had more available data than other relatively safe large cities. The Crime per Hundred People Data (CHPD) and Safety Indices for these cities are presented in Table 4.

:::{table} Comparative safety indices for four cities: My City, Chicago, Austin, and Virginia Beach. Virginia Beach has the highest safety index, followed by Austin, Chicago, and My City. The relative ranking provided by our model aligns with Forbes’ published ranking.
:label: CSD values
:align: center

| Crime Category           | Baltimore CSD | My City CSD | Chicago CSD | Austin CSD | Virginia Beach CSD |
|--------------------------|---------------|-------------|-------------|------------|--------------------|
| Criminal Damage          | 7.13          | -3.19       | -5.01       | 8.11       | 0.92               |
| Theft                    | -16.18        | -6.92       | -5.53       | 4.79       | -3.68              |
| Battery                  | -23.57        | -45.54      | -44.96      | 1.69       | 1.66               |
| Assault                  | -9.39         | 0.21        | 0.41        | -8.34      | -3.59              |
| Narcotics                | 0.12          | -4.05       | 4.68        | 3.15       | -2.73              |
| Robbery                  | -25.15        | -7.38       | -6.57       | 0.16       | 0.64               |
| Weapons Violation        | -0.69         | -0.36       | -0.55       | 0.97       | -0.75              |
| Motor Vehicle Theft      | -9.48         | -1.74       | -1.64       | -0.30      | 0.00               |
| Homicide                 | -10.59        | -1.42       | -2.56       | -0.08      | -0.03              |
| Burglary                 | -13.36        | -0.77       | 0.12        | -15.70     | 5.25               |
| Arson                    | -0.73         | -0.17       | -0.16       | -0.06      | 0.16               |
| Criminal Sexual Assault  | 0.52          | 0.20        | 1.41        | -0.46      | 1.16               |
| Safety Index             | **-101.37**       | **-71.13**      | **-60.33**      | **-6.07**     | **-0.98**              |

:::

The table above shows that the safety rating of My City is most similar to that of Chicago, which has a similar population. Our model identifies Baltimore as the least safe city among those we analyzed. It is important to note that our model interprets negative safety indices as indicating less safe cities than the national average. Yet, it is crucial to remember that these are large cities. Large cities inherently have higher crime rates than smaller towns and rural areas, even after adjusting for population through Crime per Hundred People Data (CHPD). This can be attributed to various socioeconomic factors, including population density and economic disparity ({cite}`10.1016/j.socscimed.2007.07.014`). Therefore, even the safest large cities are generally less safe than the national average, predominantly consisting of smaller towns and rural areas.