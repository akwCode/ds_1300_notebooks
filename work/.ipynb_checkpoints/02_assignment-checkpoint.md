---
jupyter:
  jupytext:
    formats: ipynb,md
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.10.3
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
---

# Working with Data


>Using the data below, answer the following questions:

>1) Which entities (top 5) had the largest population density in 2020?
>2) Which entities have more water area than land area?
>3) Which entities increased in population the most in the last 10 years?
>4) What state bird accounts for the largest population as of 2020? Land area?
>5) How many entities' largest city is their capital city?
>6) Which city has the largest percent drop from their largest city to their 5th largest? 100*(1st largest - 5th largest)/(1st largest)

```python
import pandas as pd
```

```python
facts = pd.read_csv('../data/state_facts.tsv',delimiter="\t")
facts.head(5)
```

>Using the "state_dates.tsv" data, answer the remaining questions. You will need to merge the two data sets together:

>7) Of the states that joined the United States before 1790, what is the most common state flower?
>8) Which has the larger population density, the most dense US Territory or the least dense state?
>9) Make a graph that plots the populations of the largest city in each entity in the order in which they joined the US. Make the bars black
>10) Make two additional graphs like the one above but one for land area (green bars) and one for water area (blue bars)


Hint: `pd.read_csv('../data/state_dates.tsv',delimiter="\t")`

Hint: You likely want to convert the Date column to datetime. You might have to correct errors in the data as well.

Hint: `states['Date']<pd.datetime(1790,1,1)`

Hint: `pd.merge(****,****,left_on='USPS_code',right_on='Abbreviation',how='outer')`

```python
# dataframes
facts = pd.read_csv('../data/state_facts.tsv',delimiter="\t")
dates = pd.read_csv('../data/state_dates.tsv',delimiter="\t")
states = pd.merge(facts, dates, left_on='USPS_code', right_on='Abbreviation',how='outer')
```

```python
# 1
print('1. Which entities (top 5) had the largest population density in 2020?')
facts['Pop_density'] = facts['Pop_2020'] / facts['Area_land']
facts = facts.sort_values(by='Pop_density', ascending=False)
for entity in facts['State'].head(5):
    print(entity)
```

```python
# 2
print('2. Which entities have more water area than land area? (top 3)')
for entity in facts[facts['Area_land']<facts['Area_water']]['State'].head(3):
    print(entity)
```

```python
# 3 
print('3. Which entities increased in population the most in the last 10 years?')
facts['Pop_increase'] = facts['Pop_2020'] - facts['Pop_2010']
facts = facts.sort_values(by='Pop_increase', ascending=False)
for entity in facts['State'].head(3):
    print(entity)
```

```python
# 4 
print('4. What state bird accounts for the largest population as of 2020? Land area?')
birds_pop = facts[['Pop_2020', 'State_bird']].groupby(by='State_bird').sum()
birds_land = facts[['Area_land', 'State_bird']].groupby(by='State_bird').sum()
print(birds_pop.sort_values(by='Pop_2020', ascending=False).head(1).index[0])
print(birds_land.sort_values(by='Area_land', ascending=False).head(1).index[0])
```

```python
# 5
print('5. How many entities\' largest city is their capital city?')
city_capital = facts[facts['Capital']==facts['City_1']][['State','Capital','City_1']]
print(len(city_capital))

```

```python
# 6 
print('6. Which city has the largest percent drop from their largest city to their 5th largest? 100*(1st largest - 5th largest)/(1st largest)')
facts['Pop_spread'] = 100*(facts['city_1_pop']-facts['city_5_pop'])/(facts['city_1_pop'])
facts = facts.sort_values(by='Pop_spread', ascending=False)
print(facts['State'].values[0])
```

```python
# 7 
print('7. Of the states that joined the United States before 1790, what is the most common state flower?')
states['Date'] = pd.to_datetime(states['Date'])
print(states[states['Date']<pd.datetime(1790, 1 ,1)]['State_flower'].value_counts().head(1).index[0])
```

```python
# 8 
print('8. Which has the larger population density, the most dense US Territory or the least dense state?')
states['Pop_density'] = states['Pop_2020'] / states['Area_land']
sta = states[(states['Status']=='State')].sort_values(by='Pop_density', ascending=True).head(1)
ter = states[(states['Status']=='Territory')].sort_values(by='Pop_density', ascending=False).head(1)
if sta['Pop_density'].values[0] > ter['Pop_density'].values[0]:
    print('The least dense US State')
else:
    print('The most dense US Territory')
```

```python
# 9
import matplotlib.pyplot as plt
%config InlineBackend.figure_format ='retina' #This makes your plot clearer

print('9. Make a graph that plots the populations of the largest city in each entity in the order in which they joined the US. Make the bars black')
states = states.sort_values(by='Date', ascending=True)
plot = states[['city_1_pop','Abbreviation']].plot(kind='bar', 
                                                  legend=True,
                                                  width=.5,
                                                  figsize=(16, 12),
                                                  color=['black'])
plot.set_xticklabels(states['Abbreviation']);
```

```python
# 10
print('10. Make two additional graphs like the one above but one for land area (green bars) and one for water area (blue bars)')
states = states.sort_values(by='Date', ascending=True)
plot = states[['Area_land','Abbreviation']].plot(kind='bar', 
                                                  legend=True,
                                                  width=.5,
                                                  figsize=(16, 12),
                                                  color=['green'])
plot.set_xticklabels(states['Abbreviation']);

states = states.sort_values(by='Date', ascending=True)
plot = states[['Area_water','Abbreviation']].plot(kind='bar', 
                                                  legend=True,
                                                  width=.5,
                                                  figsize=(16, 12),
                                                  color=['blue'])
plot.set_xticklabels(states['Abbreviation']);


```
