
# PyCity Schools Analysis

* As a whole, schools with higher budgets, did not yield better test results. By contrast, schools with higher spending per student actually (\$645-675) underperformed compared to schools with smaller budgets (<\$585 per student).

* As a whole, smaller and medium sized schools dramatically out-performed large sized schools on passing math performances (89-91% passing vs 67%).

* As a whole, charter schools out-performed the public district schools across all metrics. However, more analysis will be required to glean if the effect is due to school practices or the fact that charter schools tend to serve smaller student populations per school. 
---


```python
# Dependencies and Setup
import pandas as pd
import numpy as np
import locale as lc

# Get the list of all locale options
all_locales = lc.locale_alias
# Use US Convention
lc.setlocale(lc.LC_ALL,all_locales["en_us"])

# File to Load (Remember to Change These)
school_data_to_load = "Resources/schools_complete.csv"
student_data_to_load = "Resources/students_complete.csv"

# Read School and Student Data File and store into Pandas Data Frames
school_data = pd.read_csv(school_data_to_load)
student_data = pd.read_csv(student_data_to_load)

# Combine the data into a single dataset
school_data_complete = pd.merge(student_data, school_data, how="left", on=["school_name", "school_name"])
```

## District Summary

* Calculate the total number of schools

* Calculate the total number of students

* Calculate the total budget

* Calculate the average math score 

* Calculate the average reading score

* Calculate the overall passing rate (overall average score), i.e. (avg. math score + avg. reading score)/2

* Calculate the percentage of students with a passing math score (70 or greater)

* Calculate the percentage of students with a passing reading score (70 or greater)

* Create a dataframe to hold the above results

* Optional: give the displayed data cleaner formatting


```python
# Total Number of Schools
total_schools = school_data['school_name'].count()

# Total Number of Students
total_students = student_data['Student ID'].count()

# Total Budget
total_budget = school_data['budget'].sum()

# Average Math Score
avg_math_score = student_data['math_score'].mean()

# Average Reading Score
avg_read_score = student_data['reading_score'].mean()

# Overall Average Score 
overall_passing_rate = (avg_math_score + avg_read_score) / 2

# Passing Math Score (%)
percent_passing_math = (student_data[student_data["math_score"] >= 70]).count() / total_students
percent_passing_math = percent_passing_math.mean() * 100

# Passing Reading Score (%)
percent_passing_read = (student_data[student_data["reading_score"] >= 70]).count() / total_students
percent_passing_read = percent_passing_read.mean() * 100

# District Summary
district_summary_list = [(total_schools, total_students, total_budget, avg_math_score, avg_read_score, 
                         percent_passing_math, percent_passing_read, overall_passing_rate)]

district_summary_df = pd.DataFrame(district_summary_list, 
                                   columns=['Total Schools', 
                                            'Total Students', 
                                            'Total Budget',
                                            'Average Math Score', 
                                            'Average Reading Score', 
                                            '% Passing Math', 
                                            '% Passing Reading', 
                                            '% Overall Passing Rate'])

# Formatting
district_summary_df['Total Budget'] = district_summary_df['Total Budget'].apply(lambda x: "$"+lc.format_string("%.2f",x,True))
district_summary_df['Average Math Score'] = district_summary_df['Average Math Score'].apply(lambda x: lc.format_string("%.6f",x,True))
district_summary_df['Average Reading Score'] = district_summary_df['Average Reading Score'].apply(lambda x: lc.format_string("%.5f",x,True))
district_summary_df['% Passing Math'] = district_summary_df['% Passing Math'].apply(lambda x: lc.format_string("%.6f",x,True))
district_summary_df['% Passing Reading'] = district_summary_df['% Passing Reading'].apply(lambda x: lc.format_string("%.6f",x,True))
district_summary_df['% Overall Passing Rate'] = district_summary_df['% Overall Passing Rate'].apply(lambda x: lc.format_string("%.6f",x,True))

district_summary_df

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
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
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39170</td>
      <td>$24,649,428.00</td>
      <td>78.985371</td>
      <td>81.87784</td>
      <td>74.980853</td>
      <td>85.805463</td>
      <td>80.431606</td>
    </tr>
  </tbody>
</table>
</div>



## School Summary

* Create an overview table that summarizes key metrics about each school, including:
  * School Name
  * School Type
  * Total Students
  * Total School Budget
  * Per Student Budget
  * Average Math Score
  * Average Reading Score
  * % Passing Math
  * % Passing Reading
  * Overall Passing Rate (Average of the above two)
  
* Create a dataframe to hold the above results


```python
# Group the complete school data by school name and set it as index
school = school_data_complete.set_index('school_name').groupby('school_name')

# School Type
school_type = school_data.set_index('school_name')['type']

# Total Number of Students
total_students = school_data_complete.groupby('school_name')['Student ID'].count()

# Total School Budget
total_school_budget = school_data.set_index('school_name')['budget']

# Per Student Budget
per_student_budget = school_data.set_index('school_name')['budget'] / school_data.set_index('school_name')['size']

# Average Math Score
avg_math_score = school['math_score'].mean()

# Average Reading Score
avg_read_score = school['reading_score'].mean()

# Passing Math Score (%)
percent_passing_math = school_data_complete[school_data_complete['math_score'] >= 70].groupby('school_name')['Student ID'].count() / total_students
percent_passing_math = percent_passing_math * 100

# Passing Reading Score (%)
percent_passing_read = school_data_complete[school_data_complete['reading_score'] >= 70].groupby('school_name')['Student ID'].count() / total_students
percent_passing_read = percent_passing_read * 100

# Overall Passing Rate
overall_passing_rate = school_data_complete[(school_data_complete['reading_score'] >= 70) & (school_data_complete['math_score'] >= 70)].groupby('school_name')['Student ID'].count() / total_students

# School Summary 
school_summary_df = pd.DataFrame({
    "School Type": school_type,
    "Total Students": total_students,
    "Total School Budget": total_school_budget,
    "Per Student Budget": per_student_budget,
    "Average Math Score": avg_math_score,
    "Average Reading Score": avg_read_score,
    '% Passing Math': percent_passing_math,
    '% Passing Reading': percent_passing_read,
    "% Overall Passing Rate": overall_passing_rate
})

# Munging
school_summary_df = school_summary_df[['School Type',
                          'Total Students', 
                          'Total School Budget', 
                          'Per Student Budget', 
                          'Average Math Score', 
                          'Average Reading Score',
                          '% Passing Math',
                          '% Passing Reading',
                          '% Overall Passing Rate']]

# Formatting
school_summary_df['Total School Budget'] = school_summary_df['Total School Budget'].apply(lambda x: "$"+lc.format_string("%.2f",x,True))
school_summary_df['Average Math Score'] = school_summary_df['Average Math Score'].apply(lambda x: lc.format_string("%.6f",x,True))
school_summary_df['Average Reading Score'] = school_summary_df['Average Reading Score'].apply(lambda x: lc.format_string("%.5f",x,True))
school_summary_df['% Passing Math'] = school_summary_df['% Passing Math'].apply(lambda x: lc.format_string("%.6f",x,True))
school_summary_df['% Passing Reading'] = school_summary_df['% Passing Reading'].apply(lambda x: lc.format_string("%.6f",x,True))
school_summary_df['% Overall Passing Rate'] = school_summary_df['% Overall Passing Rate'].apply(lambda x: lc.format_string("%.6f",x,True))

school_summary_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>4976</td>
      <td>$3,124,928.00</td>
      <td>628.0</td>
      <td>77.048432</td>
      <td>81.03396</td>
      <td>66.680064</td>
      <td>81.933280</td>
      <td>0.546423</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>$1,081,356.00</td>
      <td>582.0</td>
      <td>83.061895</td>
      <td>83.97578</td>
      <td>94.133477</td>
      <td>97.039828</td>
      <td>0.913348</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>$1,884,411.00</td>
      <td>639.0</td>
      <td>76.711767</td>
      <td>81.15802</td>
      <td>65.988471</td>
      <td>80.739234</td>
      <td>0.532045</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2739</td>
      <td>$1,763,916.00</td>
      <td>644.0</td>
      <td>77.102592</td>
      <td>80.74626</td>
      <td>68.309602</td>
      <td>79.299014</td>
      <td>0.542899</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>$917,500.00</td>
      <td>625.0</td>
      <td>83.351499</td>
      <td>83.81676</td>
      <td>93.392371</td>
      <td>97.138965</td>
      <td>0.905995</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4635</td>
      <td>$3,022,020.00</td>
      <td>652.0</td>
      <td>77.289752</td>
      <td>80.93441</td>
      <td>66.752967</td>
      <td>80.862999</td>
      <td>0.535275</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087.00</td>
      <td>581.0</td>
      <td>83.803279</td>
      <td>83.81499</td>
      <td>92.505855</td>
      <td>96.252927</td>
      <td>0.892272</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>$1,910,635.00</td>
      <td>655.0</td>
      <td>76.629414</td>
      <td>81.18272</td>
      <td>65.683922</td>
      <td>81.316421</td>
      <td>0.535139</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650.00</td>
      <td>650.0</td>
      <td>77.072464</td>
      <td>80.96639</td>
      <td>66.057551</td>
      <td>81.222432</td>
      <td>0.535392</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>609.0</td>
      <td>83.839917</td>
      <td>84.04470</td>
      <td>94.594595</td>
      <td>95.945946</td>
      <td>0.905405</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>$2,547,363.00</td>
      <td>637.0</td>
      <td>76.842711</td>
      <td>80.74469</td>
      <td>66.366592</td>
      <td>80.220055</td>
      <td>0.529882</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1761</td>
      <td>$1,056,600.00</td>
      <td>600.0</td>
      <td>83.359455</td>
      <td>83.72572</td>
      <td>93.867121</td>
      <td>95.854628</td>
      <td>0.898921</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>$1,043,130.00</td>
      <td>638.0</td>
      <td>83.418349</td>
      <td>83.84893</td>
      <td>93.272171</td>
      <td>97.308869</td>
      <td>0.909480</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>$1,319,574.00</td>
      <td>578.0</td>
      <td>83.274201</td>
      <td>83.98949</td>
      <td>93.867718</td>
      <td>96.539641</td>
      <td>0.905826</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1800</td>
      <td>$1,049,400.00</td>
      <td>583.0</td>
      <td>83.682222</td>
      <td>83.95500</td>
      <td>93.333333</td>
      <td>96.611111</td>
      <td>0.903333</td>
    </tr>
  </tbody>
</table>
</div>



## Top Performing Schools (By Passing Rate)

* Sort and display the top five schools in overall passing rate


```python
top_performing_schools = school_summary_df.sort_values('% Overall Passing Rate', ascending = False)

top_performing_schools = top_performing_schools.head()
top_performing_schools
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>$1,081,356.00</td>
      <td>582.0</td>
      <td>83.061895</td>
      <td>83.97578</td>
      <td>94.133477</td>
      <td>97.039828</td>
      <td>0.913348</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>$1,043,130.00</td>
      <td>638.0</td>
      <td>83.418349</td>
      <td>83.84893</td>
      <td>93.272171</td>
      <td>97.308869</td>
      <td>0.909480</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>$917,500.00</td>
      <td>625.0</td>
      <td>83.351499</td>
      <td>83.81676</td>
      <td>93.392371</td>
      <td>97.138965</td>
      <td>0.905995</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>$1,319,574.00</td>
      <td>578.0</td>
      <td>83.274201</td>
      <td>83.98949</td>
      <td>93.867718</td>
      <td>96.539641</td>
      <td>0.905826</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>609.0</td>
      <td>83.839917</td>
      <td>84.04470</td>
      <td>94.594595</td>
      <td>95.945946</td>
      <td>0.905405</td>
    </tr>
  </tbody>
</table>
</div>



## Bottom Performing Schools (By Passing Rate)

* Sort and display the five worst-performing schools


```python
bottom_performing_schools = top_performing_schools.tail()

bottom_performing_schools = bottom_performing_schools.sort_values('% Overall Passing Rate')
bottom_performing_schools
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>$2,547,363.00</td>
      <td>637.0</td>
      <td>76.842711</td>
      <td>80.74469</td>
      <td>66.366592</td>
      <td>80.220055</td>
      <td>0.529882</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>$1,884,411.00</td>
      <td>639.0</td>
      <td>76.711767</td>
      <td>81.15802</td>
      <td>65.988471</td>
      <td>80.739234</td>
      <td>0.532045</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>$1,910,635.00</td>
      <td>655.0</td>
      <td>76.629414</td>
      <td>81.18272</td>
      <td>65.683922</td>
      <td>81.316421</td>
      <td>0.535139</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4635</td>
      <td>$3,022,020.00</td>
      <td>652.0</td>
      <td>77.289752</td>
      <td>80.93441</td>
      <td>66.752967</td>
      <td>80.862999</td>
      <td>0.535275</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650.00</td>
      <td>650.0</td>
      <td>77.072464</td>
      <td>80.96639</td>
      <td>66.057551</td>
      <td>81.222432</td>
      <td>0.535392</td>
    </tr>
  </tbody>
</table>
</div>



## Math Scores by Grade

* Create a table that lists the average Reading Score for students of each grade level (9th, 10th, 11th, 12th) at each school.

  * Create a pandas series for each grade. Hint: use a conditional statement.
  
  * Group each series by school
  
  * Combine the series into a dataframe
  
  * Optional: give the displayed data cleaner formatting


```python
# Gather average math scores for each grade
ninth_grade_math_scores = student_data[student_data['grade'] == '9th'].groupby('school_name')['math_score'].mean()
tenth_grade_math_scores = student_data[student_data['grade'] == '10th'].groupby('school_name')['math_score'].mean()
eleventh_grade_math_scores = student_data[student_data['grade'] == '11th'].groupby('school_name')['math_score'].mean()
twelfth_grade_math_scores = student_data[student_data['grade'] == '12th'].groupby('school_name')['math_score'].mean()

# Math Scores Dataframe
math_scores_by_grade_df = pd.DataFrame({"9th": ninth_grade_math_scores,
                                     "10th": tenth_grade_math_scores,
                                     "11th": eleventh_grade_math_scores,
                                     "12th": twelfth_grade_math_scores})

# Munging
math_scores_by_grade_df = math_scores_by_grade_df[['9th', '10th', '11th', '12th']]

# Formatting
math_scores_by_grade_df['9th'] = math_scores_by_grade_df['9th'].apply(lambda x: lc.format_string("%.6f",x,True))
math_scores_by_grade_df['10th'] = math_scores_by_grade_df['10th'].apply(lambda x: lc.format_string("%.6f",x,True))
math_scores_by_grade_df['11th'] = math_scores_by_grade_df['11th'].apply(lambda x: lc.format_string("%.6f",x,True))
math_scores_by_grade_df['12th'] = math_scores_by_grade_df['12th'].apply(lambda x: lc.format_string("%.6f",x,True))


# Set the index
math_scores_by_grade_df.index.name = 'School'

math_scores_by_grade_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
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
      <th>School</th>
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
    <tr>
      <th>Wright High School</th>
      <td>83.264706</td>
      <td>84.010288</td>
      <td>83.836782</td>
      <td>83.644986</td>
    </tr>
  </tbody>
</table>
</div>



## Reading Score by Grade 

* Perform the same operations as above for reading scores


```python
# Gather average reading scores for each grade 
ninth_grade_reading_scores = student_data[student_data['grade'] == '9th'].groupby('school_name')['reading_score'].mean() 
tenth_grade_reading_scores = student_data[student_data['grade'] == '10th'].groupby('school_name')['reading_score'].mean() 
eleventh_grade_reading_scores = student_data[student_data['grade'] == '11th'].groupby('school_name')['reading_score'].mean() 
twelfth_grade_reading_scores = student_data[student_data['grade'] == '12th'].groupby('school_name')['reading_score'].mean() 

# Reading Scores Dataframe 
reading_scores_by_grade_df = pd.DataFrame({"9th": ninth_grade_reading_scores, 
                                           "10th": tenth_grade_reading_scores, 
                                           "11th": eleventh_grade_reading_scores, 
                                           "12th": twelfth_grade_reading_scores}) 

# Munging 
reading_scores_by_grade_df = reading_scores_by_grade_df[['9th', '10th', '11th', '12th']] 

# Formatting 
reading_scores_by_grade_df['9th'] = reading_scores_by_grade_df['9th'].apply(lambda x: lc.format_string("%.6f",x,True)) 
reading_scores_by_grade_df['10th'] = reading_scores_by_grade_df['10th'].apply(lambda x: lc.format_string("%.6f",x,True)) 
reading_scores_by_grade_df['11th'] = reading_scores_by_grade_df['11th'].apply(lambda x: lc.format_string("%.6f",x,True)) 
reading_scores_by_grade_df['12th'] = reading_scores_by_grade_df['12th'].apply(lambda x: lc.format_string("%.6f",x,True)) 

# Set the index 
reading_scores_by_grade_df.index.name = 'School' 

reading_scores_by_grade_df 
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
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
      <th>School</th>
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



## Scores by School Spending

* Create a table that breaks down school performances based on average Spending Ranges (Per Student). Use 4 reasonable bins to group school spending. Include in the table each of the following:
  * Average Math Score
  * Average Reading Score
  * % Passing Math
  * % Passing Reading
  * Overall Passing Rate (Average of the above two)


```python
# Creating spending bins
bins = [0, 584.999, 614.999, 644.999, 999999]
group_name = ['< $585', "$585 - 614", "$615 - 644", "> $644"]
school_data_complete['spending_bins'] = pd.cut(school_data_complete['budget'] / school_data_complete['size'], bins, labels = group_name)

grouped_by_spending = school_data_complete.groupby('spending_bins')

# Average Math Score 
avg_math_score = grouped_by_spending['math_score'].mean()

# Average Reading Score
avg_read_score = grouped_by_spending['reading_score'].mean()

# Passing Math Score (%)
percent_passing_math = school_data_complete[school_data_complete['math_score'] >= 70].groupby('spending_bins')['Student ID'].count() / grouped_by_spending['Student ID'].count() 

# Passing Reading Score (%)
percent_passing_read = school_data_complete[school_data_complete['reading_score'] >= 70].groupby('spending_bins')['Student ID'].count() / grouped_by_spending['Student ID'].count() 

# Overall Passing Rate
overall_passing_rate = school_data_complete[(school_data_complete['reading_score'] >= 70) & (school_data_complete['math_score'] >= 70)].groupby('spending_bins')['Student ID'].count() / grouped_by_spending['Student ID'].count()

# Scores by Spending Dataframe
scores_by_spending_df = pd.DataFrame({'Average Math Score': avg_math_score,
                                      'Average Reading Score': avg_read_score,
                                      '% Passing Math': percent_passing_math,
                                      '% Passing Reading': percent_passing_read,
                                      '% Overall Passing Rate': overall_passing_rate})

# Munging 
scores_by_spending_df = scores_by_spending_df[['Average Math Score', 'Average Reading Score', '% Passing Math',
                                               '% Passing Reading', '% Overall Passing Rate']]

# Setting and naming the index
scores_by_spending_df.index.name = "Spending Ranges (Per Student)"
scores_by_spending_df = scores_by_spending_df.reindex(group_name)

# Formatting
scores_by_spending_df['Average Math Score'] = scores_by_spending_df['Average Math Score'].apply(lambda x: lc.format_string("%.6f",x,True))
scores_by_spending_df['Average Reading Score'] = scores_by_spending_df['Average Reading Score'].apply(lambda x: lc.format_string("%.5f",x,True))
scores_by_spending_df['% Passing Math'] = scores_by_spending_df['% Passing Math'].apply(lambda x: lc.format_string("%.6f",x,True))
scores_by_spending_df['% Passing Reading'] = scores_by_spending_df['% Passing Reading'].apply(lambda x: lc.format_string("%.6f",x,True))
scores_by_spending_df['% Overall Passing Rate'] = scores_by_spending_df['% Overall Passing Rate'].apply(lambda x: lc.format_string("%.6f",x,True))

scores_by_spending_df

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
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
      <th>Spending Ranges (Per Student)</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt; $585</th>
      <td>83.363065</td>
      <td>83.96404</td>
      <td>0.937029</td>
      <td>0.966866</td>
      <td>0.906407</td>
    </tr>
    <tr>
      <th>$585 - 614</th>
      <td>83.529196</td>
      <td>83.83841</td>
      <td>0.941241</td>
      <td>0.958869</td>
      <td>0.901212</td>
    </tr>
    <tr>
      <th>$615 - 644</th>
      <td>78.061635</td>
      <td>81.43409</td>
      <td>0.714004</td>
      <td>0.836148</td>
      <td>0.602893</td>
    </tr>
    <tr>
      <th>&gt; $644</th>
      <td>77.049297</td>
      <td>81.00560</td>
      <td>0.662308</td>
      <td>0.811094</td>
      <td>0.535288</td>
    </tr>
  </tbody>
</table>
</div>



## Scores by School Size

* Perform the same operations as above, based on school size.


```python
# Creating spending bins
bins = [0, 999, 1999, 99999999999]
group_name = ["Small (<1000)", "Medium (1000-2000)" , "Large (>2000)"]
school_data_complete['size_bins'] = pd.cut(school_data_complete['size'], bins, labels = group_name)

grouped_by_size = school_data_complete.groupby('size_bins')

# Average Math Score 
avg_math_score = grouped_by_size['math_score'].mean()

# Average Reading Score
avg_read_score = grouped_by_size['reading_score'].mean()

# Passing Math Score (%)
percent_passing_math = school_data_complete[school_data_complete['math_score'] >= 70].groupby('size_bins')['Student ID'].count() / grouped_by_size['Student ID'].count() 

# Passing Reading Score (%)
percent_passing_read = school_data_complete[school_data_complete['reading_score'] >= 70].groupby('size_bins')['Student ID'].count() / grouped_by_size['Student ID'].count() 

# Overall Passing Rate
overall_passing_rate = school_data_complete[(school_data_complete['reading_score'] >= 70) & (school_data_complete['math_score'] >= 70)].groupby('size_bins')['Student ID'].count() / grouped_by_size['Student ID'].count()

# Scores by Spending Dataframe
scores_by_size_df = pd.DataFrame({'Average Math Score': avg_math_score,
                                      'Average Reading Score': avg_read_score,
                                      '% Passing Math': percent_passing_math,
                                      '% Passing Reading': percent_passing_read,
                                      '% Overall Passing Rate': overall_passing_rate})

# Munging 
scores_by_size_df = scores_by_size_df[['Average Math Score', 'Average Reading Score', '% Passing Math',
                                               '% Passing Reading', '% Overall Passing Rate']]

# Setting and naming the index
scores_by_size_df.index.name = "School Size"
scores_by_size_df = scores_by_size_df.reindex(group_name)

# Formatting
scores_by_size_df['Average Math Score'] = scores_by_size_df['Average Math Score'].apply(lambda x: lc.format_string("%.6f",x,True))
scores_by_size_df['Average Reading Score'] = scores_by_size_df['Average Reading Score'].apply(lambda x: lc.format_string("%.5f",x,True))
scores_by_size_df['% Passing Math'] = scores_by_size_df['% Passing Math'].apply(lambda x: lc.format_string("%.6f",x,True))
scores_by_size_df['% Passing Reading'] = scores_by_size_df['% Passing Reading'].apply(lambda x: lc.format_string("%.6f",x,True))
scores_by_size_df['% Overall Passing Rate'] = scores_by_size_df['% Overall Passing Rate'].apply(lambda x: lc.format_string("%.6f",x,True))

scores_by_size_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
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
      <td>83.828654</td>
      <td>83.97408</td>
      <td>0.939525</td>
      <td>0.960403</td>
      <td>0.901368</td>
    </tr>
    <tr>
      <th>Medium (1000-2000)</th>
      <td>83.372682</td>
      <td>83.86799</td>
      <td>0.936165</td>
      <td>0.967731</td>
      <td>0.906243</td>
    </tr>
    <tr>
      <th>Large (&gt;2000)</th>
      <td>77.477597</td>
      <td>81.19867</td>
      <td>0.686524</td>
      <td>0.821252</td>
      <td>0.565740</td>
    </tr>
  </tbody>
</table>
</div>



## Scores by School Type

* Perform the same operations as above, based on school type.


```python
grouped_by_type = school_data_complete.groupby('type')

# Average Math Score 
avg_math_score = grouped_by_type['math_score'].mean()

# Average Reading Score
avg_read_score = grouped_by_type['reading_score'].mean()

# Passing Math Score (%)
percent_passing_math = school_data_complete[school_data_complete['math_score'] >= 70].groupby('type')['Student ID'].count() / grouped_by_type['Student ID'].count() 

# Passing Reading Score (%)
percent_passing_read = school_data_complete[school_data_complete['reading_score'] >= 70].groupby('type')['Student ID'].count() / grouped_by_type['Student ID'].count() 

# Overall Passing Rate
overall_passing_rate = school_data_complete[(school_data_complete['reading_score'] >= 70) & (school_data_complete['math_score'] >= 70)].groupby('type')['Student ID'].count() / grouped_by_type['Student ID'].count()

# Scores by Spending Dataframe
scores_by_type_df = pd.DataFrame({'Average Math Score': avg_math_score,
                                  'Average Reading Score': avg_read_score,
                                  '% Passing Math': percent_passing_math,
                                  '% Passing Reading': percent_passing_read,
                                  '% Overall Passing Rate': overall_passing_rate})

# Munging 
scores_by_type_df = scores_by_type_df[['Average Math Score', 'Average Reading Score', '% Passing Math',
                                               '% Passing Reading', '% Overall Passing Rate']]

# Setting and naming the index
scores_by_type_df.index.name = "School Type"

# Formatting
scores_by_type_df['Average Math Score'] = scores_by_type_df['Average Math Score'].apply(lambda x: lc.format_string("%.6f",x,True))
scores_by_type_df['Average Reading Score'] = scores_by_type_df['Average Reading Score'].apply(lambda x: lc.format_string("%.5f",x,True))
scores_by_type_df['% Passing Math'] = scores_by_type_df['% Passing Math'].apply(lambda x: lc.format_string("%.6f",x,True))
scores_by_type_df['% Passing Reading'] = scores_by_type_df['% Passing Reading'].apply(lambda x: lc.format_string("%.6f",x,True))
scores_by_type_df['% Overall Passing Rate'] = scores_by_type_df['% Overall Passing Rate'].apply(lambda x: lc.format_string("%.6f",x,True))

scores_by_type_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
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
      <td>83.406183</td>
      <td>83.90282</td>
      <td>0.937018</td>
      <td>0.966459</td>
      <td>0.905609</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.987026</td>
      <td>80.96249</td>
      <td>0.665184</td>
      <td>0.809052</td>
      <td>0.536959</td>
    </tr>
  </tbody>
</table>
</div>


