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

We are given a log of all the crimes to occur in My City over a 2 week period from July 5th to July 18th, 2014, which contains the case number, date of occurrence, primary description, secondary description, location description, whether an arrest was made, whether the crime was domestic or not, and the beat number of the crime. Using this log, we determined the frequency of each type of crime, as well as the arrests relative to each type of crime, shown in Figure 1.
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
![](#my-cell) - This is a cross-reference to a notebook cell
**Figure 1.** *Frequency of offenses and corresponding arrests in My City. Theft records the highest number of incidents (2658), while narcotics offenses lead in arrest frequency (1178). Conversely, concealed carry violations show the lowest incident frequency (1) and no arrests (0).*

### Design & Justification of the Model

We first wanted to consider that not all crimes are equally severe. For example, homicide is a significantly more harmful crime than theft. A crime’s impact on safety depends on its severity, so we assigned a severity score for each crime. We used the average sentencing length from the United States Bureau of Justice Statistics to determine how severe a crime is, as this will account for variation between incidents of a particular crime. For instance, possessing a drug is less severe than distributing it. We also chose sentencing length rather than time spent in prison so penalties for repeat offenses would be ignored, and shorter prison sentences due to parole would also be ignored since these are unrelated to the crime. The severity of each crime is a constant and will be the same for all cities. The severity of each crime is shown in **Table 1**.

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

Additionally, we wanted to consider the frequency of crime in our model, as more crimes generally indicate that a city is less safe. Since cities with higher populations generally have more crimes than smaller cities, we used a frequency count of crimes per 100,000 people, as this is the standard metric for measuring crime prevalence across varying populations. To account for datasets covering varying lengths of time, we used the count per 100,000 people per day (CHPD) to measure crime prevalence in our model. 

```{math}
:label: my-equation
CHPD = \frac{100,000 \times I}{PD}
```
CHPD = Crime count per 100,000 people per day \
I = Total number of crime incidents \
P = Population \
D = Number of days 

A city with more arrests is generally safer than one with fewer arrests, since it potentially has fewer perpetrators to commit crimes. Our model accounts for the influence of how many crimes result in arrests by incorporating an “arrest mitigation factor.” Crimes that result in arrests are weighed less than crimes in which the perpetrator is not cleared for an arrest.

Using the equation below, we combined these factors to create a “crime score” for each crime category. We chose to calculate a score for each category to provide more granularity. This could be useful to government agencies attempting to analyze the impact of a specific type of crime, or to civilians trying to avoid certain types of crime.

```{math}
:label: my-equation
\text{CS} = S(0.8N A + N(1 - A))
```
**CS** = Crime score \
**S** = Severity of crime \
**N** = Incident frequency in CHPD \
**D** = Arrest percentage

We separate the crimes based on how many resulted in arrests by multiplying the frequency by the arrest percentage. Crimes with arrests are multiplied by a mitigation factor of 0.8 since they are generally considered less impactful on the community, as stated in Assumption 3.

To make our model easily interpretable, we subtracted the crime score calculated for a specific crime in a given city from the corresponding national score for that crime. The national crime score, CS{sub}`n`, for each crime category, was determined using the same formula applied at the city level; however, for the national calculation, CHPD was computed using the entire U.S. population in 2015 (**P = 320 million**) and the total number of days in the year (**D = 365**). This approach is based on the nature of the data from the FBI’s Uniform Crime Report for 2015, which encompasses an entire year. Additionally, since the UCR data includes reported crime incidents for roughly 30% of the total U.S. population, we adjusted the frequency of incidents to reflect the size of the entire population by scaling the incident numbers by a factor of 10/3. 

In our model, a negative value compared to the national average for a specific crime indicates that the city is less safe for a given crime than the national average, and a positive value compared to the national average indicates that the city is safer for a specific crime.

```{math}
:label: my-equation
CSD = CS_n - CS_x
```
CSD = Crime score differential \
CS{sub}`n` = National crime score \
N = Crime score for any city 

The overall safety index for a given city is the sum of the CSDs of all the significant crimes.

```{math}
:label: my-equation
SI = \sum CSD
```
The complete model is shown below, where the sum is for all 12 crimes. 

```{math}
:label: my-equation
SI = \sum (CS_n - S(0.8N A + N(1 - A)))
```
:::{table} National crime score (CS{sub}`n`) values for each of the 12 significant crimes. Theft has the highest CS{sub}`n` value (17.01) whereas arson has the lowest CSn value (0.28). The CS{sub}`n` values will be used as a benchmark to determine how far a city’s individual crimes are from the national average. 
:label: table
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

