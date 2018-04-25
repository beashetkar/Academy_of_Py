

```python
# Dependencies
import pandas as pd
import numpy as np
import os
```


```python
# Create a reference the CSV schools_complete.csv
schools_csv_path = "raw_data/schools_complete.csv"

# Read the schools CSV into a Pandas DataFrame
schools_df = pd.read_csv(schools_csv_path)
```


```python
# Create a reference the CSV students_complete.csv
students_csv_path = "raw_data/students_complete.csv"

# Read the students CSV into a Pandas DataFrame
students_df = pd.read_csv(students_csv_path)
# students_df.columns
```


```python
# Check if students does not have missing data
# students_df.count()

```


```python
#  Rename column 'name'of schools dataframe and merge both dataframes (tables) 

schools2_df = schools_df.rename(columns={'name': 'school'})
students_schools_df = pd.merge(students_df, schools2_df, on="school")
# students_schools_df.head()

```


```python
# PART I.- District Summary
#
# Create a high level snapshot (in table form) of the district's key metrics, including:

# Total Schools
total_schools_d = schools_df['School ID'].count()

# Total Students
total_students_d = students_schools_df['Student ID'].count()

# Total Budget
total_budget_d = schools_df['budget'].sum()

# Average Math Score
average_math_d = students_schools_df['math_score'].mean()

# Average Reading Score
average_read_d = students_schools_df['reading_score'].mean()   

# % Passing Math  & % Passing Reading (passing requires a score of 70% or greater)

# Sort out any Reading Score and Math Score that Fail 
D_passmath_df = students_schools_df.loc[(students_schools_df["math_score"] >= 70)]
D_passread_df = students_schools_df.loc[(students_schools_df["reading_score"] >= 70)]

# % Passing Math
percentage_math_d = ( D_passread_df['Student ID'].count() /
                      students_schools_df['Student ID'].count())*100    

# % Passing Reading
percentage_read_d = ( D_passmath_df['Student ID'].count() /
                      students_schools_df['Student ID'].count())*100  

# Overall Passing Rate (Average of the above two)

pass_rate_d = (percentage_math_d + percentage_read_d)/2
```


```python
# Place all of the data  of District Summary found into a summary DataFrame

summary_district_df = pd.DataFrame ({ "Total Schools":['{:,.0f}'.format(total_schools_d)],
                                      "Total Students":['{:,.0f}'.format(total_students_d)],
                                      "Total Budget":['${:,.2f}'.format(total_budget_d)],
                                      "Average Math Score":['{:.2f}'.format(average_math_d)],
                                      "Average Reading Score":['{:.2f}'.format(average_read_d)],
                                      "% Passing Math":['{:.2f}%'.format(percentage_math_d)],
                                      "% Passing Reading":['{:.2f}%'.format(percentage_read_d)],
                                      "Overall Passing Rate":['{:.2f}%'.format(pass_rate_d)]
                                    })

summary_district_df = summary_district_df[["Total Schools","Total Students","Total Budget", "Average Math Score",
                                           "Average Reading Score","% Passing Math", "% Passing Reading",
                                           "Overall Passing Rate"  ]]


summary_district_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Schools</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39,170</td>
      <td>$24,649,428.00</td>
      <td>78.99</td>
      <td>81.88</td>
      <td>85.81%</td>
      <td>74.98%</td>
      <td>80.39%</td>
    </tr>
  </tbody>
</table>
</div>




```python

# PART II.- School Summary

# Create an overview table that summarizes key metrics about each school, including:
# - School Name
# - School Type
# - Total Students
# - Total School Budget
# - Per Student Budget
# - Average Math Score
# - Average Reading Score
# - % Passing Math
# - % Passing Reading
# - Overall Passing Rate (Average of the above)

# Use schools_df_idx dataframe for adding results, School Name is unique value and used as an index.
schools_summary_df = schools_df.set_index('name')
```


```python
# Add a column to schools_df_idx_name student_budget 
schools_summary_df['student_budget'] = (schools_summary_df['budget']/schools_summary_df['size'])

# Add a column to schools_df_idx_name average_math for Average Math Score
schools_summary_df['average_math'] = students_schools_df.groupby('school')["math_score"].mean()

# Add a column to schools_df_idx_name average_read for Average Reading Score
schools_summary_df['average_reading'] = students_schools_df.groupby('school')["reading_score"].mean()

# % Passing Math  & % Passing Reading (passing requires a score of 70% or greater)

# Leave out the scores are below 70% and group by school_id
reduced_math_school_df = students_schools_df.loc[(students_schools_df["math_score"] >= 70)]

reduced_read_school_df = students_schools_df.loc[(students_schools_df["reading_score"] >= 70)]

# Total_math_school and total_read_school are lists
total_math_school = reduced_math_school_df.groupby('School ID')['Student ID'].count()
total_read_school = reduced_read_school_df.groupby('School ID')['Student ID'].count()

# Add columns with total_math_school & total_read_school from respectively series
schools_summary_df['total_math_school'] = pd.Series(total_math_school).values
schools_summary_df['total_read_school'] = pd.Series(total_read_school).values

# Calculate and Add columns perc_math_school and perc_read_school
schools_summary_df['perc_math_school'] = (  pd.Series(total_math_school).values /
                                            schools_summary_df['size'])*100

schools_summary_df['perc_read_school'] = (  pd.Series(total_read_school).values /
                                            schools_summary_df['size'])*100

# Add Overall Passing Rate
schools_summary_df['over_pass_rate'] = ( schools_summary_df['perc_math_school'] + 
                                         schools_summary_df['perc_read_school'])/2
```


```python
# Reorganizing the columns using double brackets, type is School Type, size is Total Students, 
# and budget is Total School Budget.

# Using .rename(columns={}) in order to rename columns for desire output
school_summary_df = schools_summary_df.rename( columns=
                                             {  "type":"School Type", "size":"Total Students", 
                                                "budget":"Total School Budget", "student_budget": "Per Student Budget",
                                                "average_math": "Average Math Score", "average_reading": "Average Reading Score",
                                                "perc_math_school": "% Passing Math", "perc_read_school": "% Passing Reading",
                                                "over_pass_rate": "% Overall Passing Rate"
                                             })

output_schools_df = school_summary_df.loc[: ,[ "School Type", "Total Students", "Total School Budget", "Per Student Budget",
                                               "Average Math Score", "Average Reading Score","% Passing Math", 
                                               "% Passing Reading","% Overall Passing Rate"]]

# Formatting or mapping
output_schools_df['Total Students'] = output_schools_df['Total Students'].map('{:,.0f}'.format)
output_schools_df['Total School Budget'] = output_schools_df['Total School Budget'].map('${:,.2f}'.format)
output_schools_df['Per Student Budget'] = output_schools_df['Per Student Budget'].map('${:,.2f}'.format)
output_schools_df['Average Math Score'] = output_schools_df['Average Math Score'].map('{:.2f}'.format)
output_schools_df['Average Reading Score'] = output_schools_df['Average Reading Score'].map('{:.2f}'.format)
output_schools_df['% Passing Math'] = output_schools_df['% Passing Math'].map('{:.2f}%'.format)
output_schools_df['% Passing Reading'] = output_schools_df['% Passing Reading'].map('{:.2f}%'.format)
output_schools_df['% Overall Passing Rate'] = output_schools_df['% Overall Passing Rate'].map('{:.2f}%'.format)

output_schools_df 
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2,917</td>
      <td>$1,910,635.00</td>
      <td>$655.00</td>
      <td>76.63</td>
      <td>81.18</td>
      <td>65.68%</td>
      <td>81.32%</td>
      <td>73.50%</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2,949</td>
      <td>$1,884,411.00</td>
      <td>$639.00</td>
      <td>76.71</td>
      <td>81.16</td>
      <td>65.99%</td>
      <td>80.74%</td>
      <td>73.36%</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1,761</td>
      <td>$1,056,600.00</td>
      <td>$600.00</td>
      <td>83.36</td>
      <td>83.73</td>
      <td>93.87%</td>
      <td>95.85%</td>
      <td>94.86%</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4,635</td>
      <td>$3,022,020.00</td>
      <td>$652.00</td>
      <td>77.29</td>
      <td>80.93</td>
      <td>66.75%</td>
      <td>80.86%</td>
      <td>73.81%</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1,468</td>
      <td>$917,500.00</td>
      <td>$625.00</td>
      <td>83.35</td>
      <td>83.82</td>
      <td>93.39%</td>
      <td>97.14%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2,283</td>
      <td>$1,319,574.00</td>
      <td>$578.00</td>
      <td>83.27</td>
      <td>83.99</td>
      <td>93.87%</td>
      <td>96.54%</td>
      <td>95.20%</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1,858</td>
      <td>$1,081,356.00</td>
      <td>$582.00</td>
      <td>83.06</td>
      <td>83.98</td>
      <td>94.13%</td>
      <td>97.04%</td>
      <td>95.59%</td>
    </tr>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>4,976</td>
      <td>$3,124,928.00</td>
      <td>$628.00</td>
      <td>77.05</td>
      <td>81.03</td>
      <td>66.68%</td>
      <td>81.93%</td>
      <td>74.31%</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087.00</td>
      <td>$581.00</td>
      <td>83.80</td>
      <td>83.81</td>
      <td>92.51%</td>
      <td>96.25%</td>
      <td>94.38%</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>$609.00</td>
      <td>83.84</td>
      <td>84.04</td>
      <td>94.59%</td>
      <td>95.95%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1,800</td>
      <td>$1,049,400.00</td>
      <td>$583.00</td>
      <td>83.68</td>
      <td>83.95</td>
      <td>93.33%</td>
      <td>96.61%</td>
      <td>94.97%</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3,999</td>
      <td>$2,547,363.00</td>
      <td>$637.00</td>
      <td>76.84</td>
      <td>80.74</td>
      <td>66.37%</td>
      <td>80.22%</td>
      <td>73.29%</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4,761</td>
      <td>$3,094,650.00</td>
      <td>$650.00</td>
      <td>77.07</td>
      <td>80.97</td>
      <td>66.06%</td>
      <td>81.22%</td>
      <td>73.64%</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2,739</td>
      <td>$1,763,916.00</td>
      <td>$644.00</td>
      <td>77.10</td>
      <td>80.75</td>
      <td>68.31%</td>
      <td>79.30%</td>
      <td>73.80%</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1,635</td>
      <td>$1,043,130.00</td>
      <td>$638.00</td>
      <td>83.42</td>
      <td>83.85</td>
      <td>93.27%</td>
      <td>97.31%</td>
      <td>95.29%</td>
    </tr>
  </tbody>
</table>
</div>




```python

# PART III .- Top Performing Schools (By Passing Rate)

# Create a table that highlights the top 5 performing schools based on Overall Passing Rate. Include:
output_schools_df.sort_values(by="% Overall Passing Rate", ascending=False).head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1,858</td>
      <td>$1,081,356.00</td>
      <td>$582.00</td>
      <td>83.06</td>
      <td>83.98</td>
      <td>94.13%</td>
      <td>97.04%</td>
      <td>95.59%</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1,635</td>
      <td>$1,043,130.00</td>
      <td>$638.00</td>
      <td>83.42</td>
      <td>83.85</td>
      <td>93.27%</td>
      <td>97.31%</td>
      <td>95.29%</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1,468</td>
      <td>$917,500.00</td>
      <td>$625.00</td>
      <td>83.35</td>
      <td>83.82</td>
      <td>93.39%</td>
      <td>97.14%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>$609.00</td>
      <td>83.84</td>
      <td>84.04</td>
      <td>94.59%</td>
      <td>95.95%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2,283</td>
      <td>$1,319,574.00</td>
      <td>$578.00</td>
      <td>83.27</td>
      <td>83.99</td>
      <td>93.87%</td>
      <td>96.54%</td>
      <td>95.20%</td>
    </tr>
  </tbody>
</table>
</div>




```python

# PART IV .- Bottom Performing Schools (By Passing Rate)

# Create a table that highlights the top 5 performing schools based on Overall Passing Rate. Include:

output_schools_df.sort_values(by="% Overall Passing Rate").head()

```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3,999</td>
      <td>$2,547,363.00</td>
      <td>$637.00</td>
      <td>76.84</td>
      <td>80.74</td>
      <td>66.37%</td>
      <td>80.22%</td>
      <td>73.29%</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2,949</td>
      <td>$1,884,411.00</td>
      <td>$639.00</td>
      <td>76.71</td>
      <td>81.16</td>
      <td>65.99%</td>
      <td>80.74%</td>
      <td>73.36%</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2,917</td>
      <td>$1,910,635.00</td>
      <td>$655.00</td>
      <td>76.63</td>
      <td>81.18</td>
      <td>65.68%</td>
      <td>81.32%</td>
      <td>73.50%</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4,761</td>
      <td>$3,094,650.00</td>
      <td>$650.00</td>
      <td>77.07</td>
      <td>80.97</td>
      <td>66.06%</td>
      <td>81.22%</td>
      <td>73.64%</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2,739</td>
      <td>$1,763,916.00</td>
      <td>$644.00</td>
      <td>77.10</td>
      <td>80.75</td>
      <td>68.31%</td>
      <td>79.30%</td>
      <td>73.80%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# PART V .- Math Scores by Grade 

# Create a table that lists the average Math Score for students of each grade level (9th, 10th, 11th, 12th) at each school.

# Using merged_df which joins students_complete.csv and schools_complete.csv

students_schools_df_idx = students_schools_df.set_index(['School ID','Student ID'])

# Get 9th, 10th, 11th and 12th grades and group by school id

grade_9th_df = students_schools_df.loc[students_schools_df["grade"] == "9th", :]
grade_10th_df = students_schools_df.loc[students_schools_df["grade"] == "10th", :]
grade_11th_df = students_schools_df.loc[students_schools_df["grade"] == "11th", :]
grade_12th_df = students_schools_df.loc[students_schools_df["grade"] == "12th", :]

# Group by school id

math_9th_df = grade_9th_df.groupby(['school']).mean()['math_score']
math_10th_df = grade_10th_df.groupby(['school']).mean()['math_score']
math_11th_df = grade_11th_df.groupby(['school']).mean()['math_score']
math_12th_df = grade_12th_df.groupby(['school']).mean()['math_score']

# Create a Dataframe that lists the average Math Score for students of each grade level (9th, 10th, 11th, 12th) at each school.

MathScoresByGrade_df = pd.DataFrame({ "9th" : math_9th_df, "10th" : math_10th_df, 
                                     "11th" : math_11th_df, "12th": math_12th_df
                                    })

MathScoresByGrade_df = MathScoresByGrade_df [[ "9th", "10th", "11th", "12th"]]

MathScoresByGrade_df.head(14)

```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>77.083676</td>
      <td>76.996772</td>
      <td>77.515588</td>
      <td>76.492218</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.094697</td>
      <td>83.154506</td>
      <td>82.765560</td>
      <td>83.277487</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.403037</td>
      <td>76.539974</td>
      <td>76.884344</td>
      <td>77.151369</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.361345</td>
      <td>77.672316</td>
      <td>76.918058</td>
      <td>76.179963</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>82.044010</td>
      <td>84.229064</td>
      <td>83.842105</td>
      <td>83.356164</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.438495</td>
      <td>77.337408</td>
      <td>77.136029</td>
      <td>77.186567</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.787402</td>
      <td>83.429825</td>
      <td>85.000000</td>
      <td>82.855422</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>77.027251</td>
      <td>75.908735</td>
      <td>76.446602</td>
      <td>77.225641</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.187857</td>
      <td>76.691117</td>
      <td>77.491653</td>
      <td>76.863248</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.625455</td>
      <td>83.372000</td>
      <td>84.328125</td>
      <td>84.121547</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.859966</td>
      <td>76.612500</td>
      <td>76.395626</td>
      <td>77.690748</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.420755</td>
      <td>82.917411</td>
      <td>83.383495</td>
      <td>83.778976</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.590022</td>
      <td>83.087886</td>
      <td>83.498795</td>
      <td>83.497041</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.085578</td>
      <td>83.724422</td>
      <td>83.195326</td>
      <td>83.035794</td>
    </tr>
  </tbody>
</table>
</div>




```python
# PART VI .- Reading Scores by Grade

# Create a table that lists the average Reading Score for students of each grade level (9th, 10th, 11th, 12th) at each school.

reading_9th_df = grade_9th_df.groupby(['school']).mean()['reading_score']
reading_10th_df = grade_10th_df.groupby(['school']).mean()['reading_score']
reading_11th_df = grade_11th_df.groupby(['school']).mean()['reading_score']
reading_12th_df = grade_12th_df.groupby(['school']).mean()['reading_score']

ReadingScoresByGrade_df = pd.DataFrame({ "9th" : reading_9th_df, "10th" : reading_10th_df, 
                                        "11th" : reading_11th_df, "12th": reading_12th_df
                                    })

ReadingScoresByGrade_df = ReadingScoresByGrade_df [[ "9th", "10th", "11th", "12th"]]

ReadingScoresByGrade_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>81.303155</td>
      <td>80.907183</td>
      <td>80.945643</td>
      <td>80.912451</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.676136</td>
      <td>84.253219</td>
      <td>83.788382</td>
      <td>84.287958</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.198598</td>
      <td>81.408912</td>
      <td>80.640339</td>
      <td>81.384863</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>80.632653</td>
      <td>81.262712</td>
      <td>80.403642</td>
      <td>80.662338</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.369193</td>
      <td>83.706897</td>
      <td>84.288089</td>
      <td>84.013699</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>80.866860</td>
      <td>80.660147</td>
      <td>81.396140</td>
      <td>80.857143</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.677165</td>
      <td>83.324561</td>
      <td>83.815534</td>
      <td>84.698795</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.290284</td>
      <td>81.512386</td>
      <td>81.417476</td>
      <td>80.305983</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>81.260714</td>
      <td>80.773431</td>
      <td>80.616027</td>
      <td>81.227564</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.807273</td>
      <td>83.612000</td>
      <td>84.335938</td>
      <td>84.591160</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.993127</td>
      <td>80.629808</td>
      <td>80.864811</td>
      <td>80.376426</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>84.122642</td>
      <td>83.441964</td>
      <td>84.373786</td>
      <td>82.781671</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.728850</td>
      <td>84.254157</td>
      <td>83.585542</td>
      <td>83.831361</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.939778</td>
      <td>84.021452</td>
      <td>83.764608</td>
      <td>84.317673</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.833333</td>
      <td>83.812757</td>
      <td>84.156322</td>
      <td>84.073171</td>
    </tr>
  </tbody>
</table>
</div>




```python
# PART VII .- Scores by School Spending

# Create a table that breaks down school performances based on average Spending Ranges (Per Student). 
# Use 4 reasonable bins to group school spending. Include in the table each of the following:

# 1. Average Math Score
# 2. Average Reading Score
# 3. % Passing Math
# 4. % Passing Reading
# 5. Overall Passing Rate (Average of the above two)

per_student_budget_df=school_summary_df.sort_values(by="Per Student Budget")

```


```python
# Create bins in which to place values based upon Per Student Budget
budget_bins = [0, 585, 615, 645, 675]

# Create labels for these bins
budget_labels = ["<$585", "$585-615", "$615-645", "$645-675"]

# Slice the data and place the data series into a new column inside of the DataFrame
per_student_budget_df["Spending Ranges (Per Student)"] = pd.cut(per_student_budget_df["Per Student Budget"],
                                                      budget_bins,labels=budget_labels)

math_scores = per_student_budget_df.groupby(["Spending Ranges (Per Student)"]).mean()["Average Math Score"]
reading_scores = per_student_budget_df.groupby(["Spending Ranges (Per Student)"]).mean()["Average Reading Score"]
passing_math = per_student_budget_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Passing Math"]
passing_reading = per_student_budget_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Passing Reading"]
passing_rate = (passing_math + passing_reading) / 2

# Create  a data frame to list the Performance based on School Spending

school_spending_summary_df = pd.DataFrame({"Average Math Score" : math_scores, "Average Reading Score": reading_scores,
                                           "% Passing Math": passing_math, "% Passing Reading": passing_reading,
                                           "% Overall Passing Rate": passing_rate})

school_spending_summary_df = school_spending_summary_df[["Average Math Score", "Average Reading Score", 
                                                        "% Passing Math", "% Passing Reading", "% Overall Passing Rate"]]

school_spending_summary_df

```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Spending Ranges (Per Student)</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;$585</th>
      <td>83.455399</td>
      <td>83.933814</td>
      <td>93.460096</td>
      <td>96.610877</td>
      <td>95.035486</td>
    </tr>
    <tr>
      <th>$585-615</th>
      <td>83.599686</td>
      <td>83.885211</td>
      <td>94.230858</td>
      <td>95.900287</td>
      <td>95.065572</td>
    </tr>
    <tr>
      <th>$615-645</th>
      <td>79.079225</td>
      <td>81.891436</td>
      <td>75.668212</td>
      <td>86.106569</td>
      <td>80.887391</td>
    </tr>
    <tr>
      <th>$645-675</th>
      <td>76.997210</td>
      <td>81.027843</td>
      <td>66.164813</td>
      <td>81.133951</td>
      <td>73.649382</td>
    </tr>
  </tbody>
</table>
</div>




```python
# PART VIII .- Scores by School Size

# Repeat the above breakdown, but this time group schools based on a reasonable approximation 
# of school size (Small, Medium, Large).

per_school_size_df=school_summary_df.sort_values(by="Total Students")

```


```python
# Create bins in which to place values based upon School Size
school_bins = [0, 1000, 2000, 5000]

# Create labels for these bins
school_labels = ["Small (<1000)", "Medium (1000-2000)", "Large (2000-5000)"]

# Slice the data and place the data series into a new column inside of the DataFrame
per_school_size_df["School Size"] = pd.cut(per_school_size_df["Total Students"], school_bins,labels=school_labels)

# Do calculations
math_scores_school = per_school_size_df.groupby(["School Size"]).mean()["Average Math Score"]
reading_scores_school = per_school_size_df.groupby(["School Size"]).mean()["Average Reading Score"]
passing_math_school = per_school_size_df.groupby(["School Size"]).mean()["% Passing Math"]
passing_reading_school = per_school_size_df.groupby(["School Size"]).mean()["% Passing Reading"]
passing_rate_school = (passing_math_school + passing_reading_school) / 2

# Create  a data frame to list performance based on School Size

school_size_summary_df = pd.DataFrame({"Average Math Score" : math_scores_school, "Average Reading Score": reading_scores_school,
                                           "% Passing Math": passing_math_school, "% Passing Reading": passing_reading_school,
                                           "% Overall Passing Rate": passing_rate_school})

school_size_summary_df = school_size_summary_df[["Average Math Score", "Average Reading Score", "% Passing Math", 
                                                 "% Passing Reading", "% Overall Passing Rate"]]

school_size_summary_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Size</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small (&lt;1000)</th>
      <td>83.821598</td>
      <td>83.929843</td>
      <td>93.550225</td>
      <td>96.099437</td>
      <td>94.824831</td>
    </tr>
    <tr>
      <th>Medium (1000-2000)</th>
      <td>83.374684</td>
      <td>83.864438</td>
      <td>93.599695</td>
      <td>96.790680</td>
      <td>95.195187</td>
    </tr>
    <tr>
      <th>Large (2000-5000)</th>
      <td>77.746417</td>
      <td>81.344493</td>
      <td>69.963361</td>
      <td>82.766634</td>
      <td>76.364998</td>
    </tr>
  </tbody>
</table>
</div>




```python
# PART IX .- Scores by School Type

# Repeat the above breakdown, but this time group schools based on school type (Charter vs. District).

```


```python
grouped_school_type_df = school_summary_df.groupby("School Type")

math_scores_type = grouped_school_type_df["Average Math Score"].mean()
reading_scores_type = grouped_school_type_df["Average Reading Score"].mean()
passing_math_type = grouped_school_type_df["% Passing Math"].mean()
passing_reading_type = grouped_school_type_df["% Passing Reading"].mean()
passing_rate_type = (passing_math_type + passing_reading_type) / 2

# Create  a data frame to list performance based on School Type

school_type_summary_df = pd.DataFrame({"Average Math Score" : math_scores_type, "Average Reading Score": reading_scores_type,
                                       "% Passing Math": passing_math_type, "% Passing Reading": passing_reading_type,
                                       "% Overall Passing Rate": passing_rate_type})

school_type_summary_df = school_type_summary_df[["Average Math Score", "Average Reading Score", "% Passing Math", 
                                                 "% Passing Reading", "% Overall Passing Rate"]]
school_type_summary_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Charter</th>
      <td>83.473852</td>
      <td>83.896421</td>
      <td>93.620830</td>
      <td>96.586489</td>
      <td>95.103660</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.956733</td>
      <td>80.966636</td>
      <td>66.548453</td>
      <td>80.799062</td>
      <td>73.673757</td>
    </tr>
  </tbody>
</table>
</div>


