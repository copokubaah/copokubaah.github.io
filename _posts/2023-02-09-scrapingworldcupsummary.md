---
layout: posts
# title: "FIFA WORLD CUP HISTORY"
# subtitle: "Scraping Summary Data from Wikipedia"
# background: ""
# permalink: /post/scrapingworldcup

---

# FIFA World Cup

The FIFA World Cup, often simply called the World Cup, is an international association football competition contested by the senior men's national teams of the members of the Fédération Internationale de Football Association (FIFA), the sport's global governing body. The tournament has been held every four years since the inaugural tournament in 1930, except in 1942 and 1946 when it was not held because of the Second World War. The reigning champions are Argentina, who won their third title at the 2022 tournament (From Wikipedia).

**The history of the World Cup provides interesting data for analytics, however, getting the entire data can be challenging. This Notebook describes how to scrape Wikipedia for the world cup history data from 1930 to 2022**

### Let's Begin.....

There are two main datasets:
 - The summary data for each world cup year
 - The data on every match that has been played

### 1. Scraping the Summary of Each World Cup Year

At Wikipedia, information about each world cup tournament can be found at **https://en.wikipedia.org/wiki/{year}_FIFA_World_Cup** (see link circled in red in figure below) and summary data can be located in the infobox circled in green in the figure below. 

At the infobox, we can grab details such as the Host country, Dates, Number of Teams, Champions etc.. 

![worldcup_sum1.png](attachment:worldcup_sum1.png)

To get the information from this website, we will use python libraries **requests** to get the content and **beautifulsoup** to parse the content; numpy and pandas will be used for manipulating numerical and table-like data. I found unicodedata.normalize to be very useful in getting text from scraped data. 

You can install the libraries using the following lines of code
 - pip install requests
 - pip install beautifulsoup4

After installing we will import the libraries...


```python
import numpy as np
import pandas as pd
import requests
from bs4 import BeautifulSoup
from unicodedata import normalize
```

We will use request to get content and use beautifulsoup to parse contents 


```python
year = 1974
r = requests.get(f'https://en.wikipedia.org/wiki/{year}_FIFA_World_Cup')
soup = BeautifulSoup(r.text, 'html.parser')
```

#### Now, how do we get the dat from the parsed information. At Wikipedia, the summary history data is located at the **infobox** in the figure below and tagged with ***table.infobox.vcalendar***. We will use the tag to extract that data from the parsed information

![worldcup_sum23.png](attachment:worldcup_sum23.png)


```python
summary = soup.find_all('table', {"class": "infobox vcalendar"})
```

#### Having extracted the infobox data, we can then extract each of the information we want such as Host country, Dates, Teams etc. These information are tagged with "tr", under which there are the "th.infobox-label" for the heading such as Host country and td.infobox-data for the label such as West Germany. See figure below

![worldcup_sum3.png](attachment:worldcup_sum3.png)

### We can then find all data with the tag 'tr' and iterate through them to find the headings and labels


```python
sum_contents = summary[0].find_all('tr')
```

create an empty data frame with desired columns to append data


```python
cols = ['Host country', 'Dates', 'Teams', 'Champions', 'Runners-up',
         'Third place', 'Fourth place', 'Matches played', 'Goals scored',
         'Attendance']
cupsummary = pd.DataFrame(columns = cols)
cupsummary.head()
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
      <th>Host country</th>
      <th>Dates</th>
      <th>Teams</th>
      <th>Champions</th>
      <th>Runners-up</th>
      <th>Third place</th>
      <th>Fourth place</th>
      <th>Matches played</th>
      <th>Goals scored</th>
      <th>Attendance</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>



created this function **find_delimiter** to take a string and determine whether certain delimiters are present. 
The delimiters are those found in the scraped data and will help in cleaning the data






```python
def find_delimiter(string):
    dts = [' (', '[']
    n = 0
    run = 1
    while run and n < len(dts):
        if string.find(dts[n]) == -1:
            run = 1
            n += 1
            delimiter = 'None'
        else:
            run = 0
            delimiter = dts[n]
    return delimiter
```

Now for each row, determine whether it contains the information in the columns of the dataframe. if yes, clean the text and append it to the datasets 


```python
for eachrow in sum_contents:
    gettext = eachrow.find('th', {'class': 'infobox-label', 'scope':'row'})

    if gettext is not None:

        heading = normalize('NFKD', gettext.text)

        if heading in cols:
            label = normalize('NFKD', eachrow.find('td').text)
            delimiter = find_delimiter(label)
            if delimiter != 'None':
                label = label.split(delimiter)[0]
            cupsummary.loc[0, heading] = label

cupsummary.head()
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
      <th>Host country</th>
      <th>Dates</th>
      <th>Teams</th>
      <th>Champions</th>
      <th>Runners-up</th>
      <th>Third place</th>
      <th>Fourth place</th>
      <th>Matches played</th>
      <th>Goals scored</th>
      <th>Attendance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>West Germany</td>
      <td>13 June – 7 July 1974</td>
      <td>16</td>
      <td>West Germany</td>
      <td>Netherlands</td>
      <td>Poland</td>
      <td>Brazil</td>
      <td>38</td>
      <td>97</td>
      <td>1,865,762</td>
    </tr>
  </tbody>
</table>
</div>



#### We can put everything together and get the history data for all the years from 1930 to 2022




```python
# list the years of world cup that need to be scraped 
worldcup = [1930, 1934, 1938, 1950, 1954, 1958, 1962, 1966, 1970, 1974, 1978, 1982,
            1986, 1990, 1994, 1998, 2002, 2006, 2010, 2014, 2018, 2022]

cols = ['Host country', 'Year', 'Dates', 'Teams', 'Champions', 'Runners-up',
         'Third place', 'Fourth place', 'Matches played', 'Goals scored',
         'Attendance']
cupsummary = pd.DataFrame(columns = cols)

# for each year of world cup
for i, year in enumerate(worldcup):
    
    # append world cup year
    cupsummary.loc[i, 'Year'] = year
    
    # use request to get content and use beautifulsoup to parse contents
    r = requests.get(f'https://en.wikipedia.org/wiki/{year}_FIFA_World_Cup')
    soup = BeautifulSoup(r.text, 'html.parser')
    
    # we want the table with the tag: "class: infobox vcalendar"
    summary = soup.find_all('table', {"class": "infobox vcalendar"})
    
    # find contents with tag "tr"
    sum_contents = summary[0].find_all('tr')
    
    # for each row, determine whether it contains the information in the columns of the dataframe
    # if yes, clean the text and append it to the datasets 
    for eachrow in sum_contents:
        gettext = eachrow.find('th', {'class': 'infobox-label', 'scope':'row'})
        
        if gettext is not None:
        
            heading = normalize('NFKD', gettext.text)
            
            if heading in cols or heading == 'Host countries':
                
                label = normalize('NFKD', eachrow.find('td').text)
                delimiter = find_delimiter(label)
                
                if heading not in ['Host countries', 'Dates']:
                    if delimiter != 'None':
                        label = label.split(delimiter)[0]
                else:
                    if heading == 'Host countries':
                        heading = 'Host country'
                cupsummary.loc[i, heading] = label
                
               
                
```

### view complete dataframe


```python
cupsummary
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
      <th>Host country</th>
      <th>Year</th>
      <th>Dates</th>
      <th>Teams</th>
      <th>Champions</th>
      <th>Runners-up</th>
      <th>Third place</th>
      <th>Fourth place</th>
      <th>Matches played</th>
      <th>Goals scored</th>
      <th>Attendance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Uruguay</td>
      <td>1930</td>
      <td>13–30 July 1930</td>
      <td>13</td>
      <td>Uruguay</td>
      <td>Argentina</td>
      <td>United States</td>
      <td>Yugoslavia</td>
      <td>18</td>
      <td>70</td>
      <td>590,549</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Italy</td>
      <td>1934</td>
      <td>27 May – 10 June 1934</td>
      <td>16</td>
      <td>Italy</td>
      <td>Czechoslovakia</td>
      <td>Germany</td>
      <td>Austria</td>
      <td>17</td>
      <td>70</td>
      <td>363,000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>France</td>
      <td>1938</td>
      <td>4–19 June 1938</td>
      <td>15</td>
      <td>Italy</td>
      <td>Hungary</td>
      <td>Brazil</td>
      <td>Sweden</td>
      <td>18</td>
      <td>84</td>
      <td>374,835</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Brazil</td>
      <td>1950</td>
      <td>24 June – 16 July 1950</td>
      <td>13</td>
      <td>Uruguay</td>
      <td>Brazil</td>
      <td>Sweden</td>
      <td>Spain</td>
      <td>22</td>
      <td>88</td>
      <td>1,045,246</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Switzerland</td>
      <td>1954</td>
      <td>16 June – 4 July 1954</td>
      <td>16</td>
      <td>West Germany</td>
      <td>Hungary</td>
      <td>Austria</td>
      <td>Uruguay</td>
      <td>26</td>
      <td>140</td>
      <td>768,607</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Sweden</td>
      <td>1958</td>
      <td>8–29 June 1958</td>
      <td>16</td>
      <td>Brazil</td>
      <td>Sweden</td>
      <td>France</td>
      <td>West Germany</td>
      <td>35</td>
      <td>126</td>
      <td>819,810</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Chile</td>
      <td>1962</td>
      <td>30 May – 17 June 1962</td>
      <td>16</td>
      <td>Brazil</td>
      <td>Czechoslovakia</td>
      <td>Chile</td>
      <td>Yugoslavia</td>
      <td>32</td>
      <td>89</td>
      <td>893,172</td>
    </tr>
    <tr>
      <th>7</th>
      <td>England</td>
      <td>1966</td>
      <td>11–30 July 1966</td>
      <td>16</td>
      <td>England</td>
      <td>West Germany</td>
      <td>Portugal</td>
      <td>Soviet Union</td>
      <td>32</td>
      <td>89</td>
      <td>1,563,135</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Mexico</td>
      <td>1970</td>
      <td>31 May – 21 June 1970</td>
      <td>16</td>
      <td>Brazil</td>
      <td>Italy</td>
      <td>West Germany</td>
      <td>Uruguay</td>
      <td>32</td>
      <td>95</td>
      <td>1,604,065</td>
    </tr>
    <tr>
      <th>9</th>
      <td>West Germany</td>
      <td>1974</td>
      <td>13 June – 7 July 1974</td>
      <td>16</td>
      <td>West Germany</td>
      <td>Netherlands</td>
      <td>Poland</td>
      <td>Brazil</td>
      <td>38</td>
      <td>97</td>
      <td>1,865,762</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Argentina</td>
      <td>1978</td>
      <td>1–25 June 1978</td>
      <td>16</td>
      <td>Argentina</td>
      <td>Netherlands</td>
      <td>Brazil</td>
      <td>Italy</td>
      <td>38</td>
      <td>102</td>
      <td>1,545,791</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Spain</td>
      <td>1982</td>
      <td>13 June – 11 July 1982</td>
      <td>24</td>
      <td>Italy</td>
      <td>West Germany</td>
      <td>Poland</td>
      <td>France</td>
      <td>52</td>
      <td>146</td>
      <td>2,109,723</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Mexico</td>
      <td>1986</td>
      <td>31 May – 29 June 1986</td>
      <td>24</td>
      <td>Argentina</td>
      <td>West Germany</td>
      <td>France</td>
      <td>Belgium</td>
      <td>52</td>
      <td>132</td>
      <td>2,394,031</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Italy</td>
      <td>1990</td>
      <td>8 June – 8 July 1990</td>
      <td>24</td>
      <td>West Germany</td>
      <td>Argentina</td>
      <td>Italy</td>
      <td>England</td>
      <td>52</td>
      <td>115</td>
      <td>2,516,215</td>
    </tr>
    <tr>
      <th>14</th>
      <td>United States</td>
      <td>1994</td>
      <td>June 17 – July 17, 1994</td>
      <td>24</td>
      <td>Brazil</td>
      <td>Italy</td>
      <td>Sweden</td>
      <td>Bulgaria</td>
      <td>52</td>
      <td>141</td>
      <td>3,597,042</td>
    </tr>
    <tr>
      <th>15</th>
      <td>France</td>
      <td>1998</td>
      <td>10 June – 12 July 1998</td>
      <td>32</td>
      <td>France</td>
      <td>Brazil</td>
      <td>Croatia</td>
      <td>Netherlands</td>
      <td>64</td>
      <td>171</td>
      <td>2,785,100</td>
    </tr>
    <tr>
      <th>16</th>
      <td>South KoreaJapan</td>
      <td>2002</td>
      <td>31 May – 30 June 2002</td>
      <td>32</td>
      <td>Brazil</td>
      <td>Germany</td>
      <td>Turkey</td>
      <td>South Korea</td>
      <td>64</td>
      <td>161</td>
      <td>2,705,198</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Germany</td>
      <td>2006</td>
      <td>9 June – 9 July 2006</td>
      <td>32</td>
      <td>Italy</td>
      <td>France</td>
      <td>Germany</td>
      <td>Portugal</td>
      <td>64</td>
      <td>147</td>
      <td>3,359,439</td>
    </tr>
    <tr>
      <th>18</th>
      <td>South Africa</td>
      <td>2010</td>
      <td>11 June – 11 July 2010</td>
      <td>32</td>
      <td>Spain</td>
      <td>Netherlands</td>
      <td>Germany</td>
      <td>Uruguay</td>
      <td>64</td>
      <td>145</td>
      <td>3,178,856</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Brazil</td>
      <td>2014</td>
      <td>12 June – 13 July</td>
      <td>32</td>
      <td>Germany</td>
      <td>Argentina</td>
      <td>Netherlands</td>
      <td>Brazil</td>
      <td>64</td>
      <td>171</td>
      <td>3,429,873</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Russia</td>
      <td>2018</td>
      <td>14 June – 15 July</td>
      <td>32</td>
      <td>France</td>
      <td>Croatia</td>
      <td>Belgium</td>
      <td>England</td>
      <td>64</td>
      <td>169</td>
      <td>3,031,768</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Qatar</td>
      <td>2022</td>
      <td>20 November – 18 December 2022</td>
      <td>32</td>
      <td>Argentina</td>
      <td>France</td>
      <td>Croatia</td>
      <td>Morocco</td>
      <td>64</td>
      <td>172</td>
      <td>3,404,252</td>
    </tr>
  </tbody>
</table>
</div>



#### save dataframe into csv file


```python
cupsummary.to_csv('WorldCupSummaryData.csv')
```


```python

```
