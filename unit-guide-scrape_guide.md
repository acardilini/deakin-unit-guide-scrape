# Scraping Deakin University unit guide data
Deakin University maintains an online <a href="https://www.deakin.edu.au/current-students/unitguides/index.php?">repository</a>
for past and present unit guides. By selecting three parameters including a year,
teaching period and unit it is possible to find unit guides for the majority
of units run at Deakin. Some units are not include in the online repository
for copyright and privacy issues including Medicine units.

The base URL for this repository can be used to easily access all guides by
simply inputting the correct parameters.
*https://www.deakin.edu.au/current-students/unitguides/index.php?year=&semester=&unit=*

## Identifying the parameters
**Year** - The year dropdown tool allows the selection of years from 2014-2017.
If you change this attribute in the URL directly it is possible to access unit
guides from as far back as 2009, although it is a significantly reduced
range of guides.

Within the URL year is simply indicated in long form, e.g.:
*https://www.deakin.edu.au/current-students/unitguides/index.php?year=2016&semester=&unit=*

**Teaching Period** - There are six options for teaching period including three
'Residentials Period #' options and three 'Trimester #' options. The
residentials lead to masters level guides while trimester leads to undergraduate
unit guides.

Within the URL teaching period is indicated by 'TRI-#', e.g.:
*https://www.deakin.edu.au/current-students/unitguides/index.php?year=2016&semester=TRI-1&unit=*

**Unit** - There are hundreds of units available for each combination of year
and teaching period. In order to easily scrape each unit guide we will need to
know the unit codes for each unit. To identify these we will need to scrape
the list of units available for each combination of year and teaching period.

Once we have a list of all units and their associated year and teaching period
we indicate unit within the URL by including the unit code, e.g.:
*https://www.deakin.edu.au/current-students/unitguides/index.php?year=2016&semester=TRI-1&unit=SLE111*

## Defining all Unit Guide combinations
To scrape all unit guides we will first need to create a list of all URL
combinations for year and teaching period, scrape a list of all associated units
and then use all three to access each unit guide.

### URLs with all combinations of year and teaching period
```python
def yrPeriodUrl(startYr, endYr):
  output = []
  url1 = 'https://www.deakin.edu.au/current-students/unitguides/index.php?year='
  url2 = '&semester=TRI-'
  url3 = '&unit='
  while startYr <= endYr:
  	tri = 1
  	url4 = url1 + str(startYr) + url2
  	while tri <= 3:
  		url = url4 + str(tri) + url3
  		output.append(url)
  		tri = tri + 1
  	startYr = startYr + 1
  return output

test = yrPeriodUrl(2009, 2017)
print(test)
```

### List of all units for each combination of year and teaching period
Following tutorial: http://web.stanford.edu/~zlotnick/TextAsData/Web_Scraping_with_Beautiful_Soup.html
```python
from bs4 import BeautifulSoup
import urllib
r = urllib.urlopen('URL').read()
soup = BeautifulSoup(r)
print type(soup)
soup.find_all(id_="unitSelectBox", "option")

```
