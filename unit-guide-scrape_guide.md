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
  yr = []
  trimester = []
  url1 = 'https://www.deakin.edu.au/current-students/unitguides/index.php?year='
  url2 = '&semester=TRI-'
  url3 = '&unit='
  while startYr <= endYr:
    tri = 1
    url4 = url1 + str(startYr) + url2
    while tri <= 3:
      url = url4 + str(tri) + url3
      output.append(url)
      yr.append(startYr)
      trimester.append(tri)
      tri = tri + 1
    startYr = startYr + 1
  import pandas as pd
  urlZip = list(zip(yr,trimester,output))
  dfUrl = pd.DataFrame(data = urlZip, columns=['yr', 'tri', 'yrPerUrl'])
  return (dfUrl, urlZip)

yrPerUrl = yrPeriodUrl(2009, 2016)
dfYrPer = yrPerUrl[0]
listYrPer = yrPerUrl[1]
```

### List of all units for each combination of year and teaching period
```python
from bs4 import BeautifulSoup
import urllib.request as ur
def unitScrape(listYrPer):
  unitCode = []
  unitName = []
  yr = []
  tri = []
  for url in listYrPer:
    r = ur.urlopen(url[2]).read()
    soup = BeautifulSoup(r, 'html.parser')
    unitList = soup.select("#unitSelectBox option")
    units = []
    for unit in unitList:
      units.append(unit.get_text())
    for unit in units:
      unitCode.append(unit[0:6])
      unitName.append(unit[9:len(unit)])
      yr.append(url[0])
      tri.append(url[1])
  import pandas as pd
  unitsSplit = list(zip(unitCode,unitName,yr,tri))
  df = pd.DataFrame(data = unitsSplit, columns=['unitCode', 'unitName', 'yr', 'tri'])
  return df

unitGuideList = unitScrape(listYrPer)
```

### Scrape unit information from unit guides
Not all of the unit guides have the same HTML structure so a single script will
not be able to pull data from all guides. Investigation of the unit guides
shows that unit guides from the following years have the same structure:
2009-2012, 2013, 2014-onwards.

To make this work we are going to write several helper functions that can then
be used by the main function to scrape the specific information requested. The
function will work by:
1. Specify year, trimester and unit code parameters.
2. Specify what unit information is required, this could include:
    * All - all parameters
    * Unit chair
    * About unit
    * Contact Hours
    * Study commitment
    * ULOs
    * GLOs
    * Summative assessments
      * Description
      * Student output
      * Weighting
      * ULOs
      * GLOs

```python
# Test function to extract unit chair information. 2016
from bs4 import BeautifulSoup
import urllib.request as ur
import re

testUrl = "https://www.deakin.edu.au/current-students/unitguides/UnitGuide.php?year=2016&semester=TRI-1&unit=SLE111"
r = ur.urlopen(testUrl).read()
soup = BeautifulSoup(r, 'html.parser')
print(soup.prettify())

unitChairTag = soup.select("a[name=0-UNIT-CHAIR] ~ p > p")
unitChair = unitChairTag[0].get_text()
```
```python
# Test function to extract information 'About This Unit'. 2016
from bs4 import BeautifulSoup
import urllib.request as ur
import re

testUrl = "https://www.deakin.edu.au/current-students/unitguides/UnitGuide.php?year=2016&semester=TRI-1&unit=SLE111"
r = ur.urlopen(testUrl).read()
soup = BeautifulSoup(r, 'html.parser')
print(soup.prettify())

contactHoursTag = soup.select("a[name=0-CONTACT-HRS] ~ p > p")
contactHours = contactHoursTag[0].get_text()
```


```python
# Test function to extract information 'About This Unit'. 2016
from bs4 import BeautifulSoup
import requests

testUrl = "https://www.deakin.edu.au/current-students/unitguides/UnitGuide.php?year=2016&semester=TRI-1&unit=SLE111"
r = requests.get(testUrl).content
soup = BeautifulSoup(r, 'html.parser')
print(soup.prettify())

uloTableTag = soup.find(text=re.compile("ULO")).find_parent("table")
for row in uloTableTag.find_all("tr")[1:]:
    print([cell.get_text(strip=True) for cell in row.find_all("td")])
```
