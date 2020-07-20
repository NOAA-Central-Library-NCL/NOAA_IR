#### Legal Disclaimer

*"This repository is a scientific product and is not official communication of the National Oceanic and Atmospheric Administration, or the United States Department of Commerce. All NOAA GitHub project code is provided on an 'as is' basis and the user assumes responsibility for its use. Any claims against the Department of Commerce or Department of Commerce bureaus stemming from the use of this GitHub project will be governed by all applicable Federal law. Any reference to specific commercial products, processes, or services by service mark, trademark, manufacturer, or otherwise, does not constitute or imply their endorsement, recommendation or favoring by the Department of Commerce. The Department of Commerce seal and logo, or the seal and logo of a DOC bureau, shall not be used in any manner to imply endorsement of any commercial product or activity by DOC or the United States Government."*

# NOAA Institutional Repository APIs (JSON API and OAI-PMH)

## JSON API

The NOAA Institutional Repository (NOAA IR) JSON API provides access to the NOAA IR's collections in JSON. 

Each collection in the NOAA IR API has its own endpoint, with a collection unique identifier, or pid, serving as the endpoint. The NOAA IR collections and associated pids consist of: 

```
National Environmental Policy Act (NEPA) : 1
Weather Research and Forecasting Innovation Act : 23702
Coral Reef Conservation Program (CRCP) : 3
Ocean Exploration Program (OER) : 4
National Marine Fisheries Service (NMFS) : 5
National Weather Service (NWS): 6
Office of Oceanic and Atmospheric Research (OAR) : 7
National Ocean Service (NOS) : 8               
National Environmental Satellite and Data Information Service (NESDIS) : 9
Sea Grant Publications : 11
Education and Outreach : 12
NOAA General Documents : 10031
NOAA International Agreements : 11879
Office of Marine and Aviation Operations (OMAO) : 16402
Integrated Ecosystem Assessment (IEA):22022
NOAA Cooperative Institutes: 23649
Cooperative Science Centers: 24914
```

**Notes**:
* No API key or authenication is required.
* If you query one or more collection and only are interested in the the unique item count, de-duplicate your results. Items in the NOAA IR are shared accross multiple collections, and this will be reflected in cases when multiple collections are combined into a single dataset.  

### API Base URL

The NOAA IR JSON API's base url is: 
* https://repository.library.noaa.gov/fedora/export

From the base URL you have the option to download or view JSON.

#### Download URL

* https://repository.library.noaa.gov/fedora/export/download/collection/{pid}


By choosing this option you download all items in JSON from a collection, including directly from the browser. 

#### View URL

* https://repository.library.noaa.gov/fedora/export/view/collection/{pid}

By choosing this option you retrieve items in JSON from a collection, with the default number of items returning only 100. If you want additional items to be return you need to pass the "rows" parameter with the number of items specified. Item count for each collection can be viewed in each collection's responseHeader tag:

```
"responseHeader":{
    "status":0,
    "QTime":6,
    "params":{
      "q":"fgs.state:\"Active\" AND fgs.lastModifiedDate:[* TO *] AND rdf.isMemberOf:\"1\"",
      "indent":"on",
      "fl":"PID,mods.*,dc.*,keywords,fgs.*,rdf.*",
      "rows":"100",
      "wt":"json"}},
  "response":{"numFound":896,"start":0,"docs":[
```

#### Examples

Basic examples using Python 3 to request items from a single collection using View URL

##### Example 1: Basic Query

In this example, import Python's [requests](https://requests.readthedocs.io/en/master/) library; then retrieve the collection's row numbers.

```
>>> import requests
>>>
>>> url = https://repository.library.noaa.gov/fedora/export/view/collection/4
>>> response = requests.get(url)
>>> json_d = response.json()
>>> rows = json_d['response']['numFound']
```

Once you retrieved the number of rows, you can pass them off as parameters and proceed to parse the collection data.

```
>>> response = requests.get(url, params={'rows':rows})
>>> response.url
'https://response.library.noaa.gov/fedora/export/view/collection/4?rows=718'
>>> json_d = response.json()
>>> docs = json_d['response']['docs']
```

In parsing collection data be sure your code is written to handle blank fields. 

```
>>> collection_data = []
>>> for doc in docs:
...  pid = doc['PID']
...  try:
...    abstract = doc['mods.abstract']
...  except KeyError:
...    abstract = ''
...  try:
...    doi = doc['mods.sm_digital_object_identifier']
...  except KeyError:
...    doi = ''  
...
...  collection_data.append([pid, abstact, doi])
...
>>>
```

##### Example 2: Date Filter (Query String) 

The NOAA IR JSON API allows you to filter collections according to the date items were added to the IR using the query strings 'from' and 'until' 

In using this feature, the date must be formatted as 'YYYY-MM-DDTHH:MM:SSZ'. Otherwise, the date filter will not apply all collection items will be pulled. 

```
>>> from datetime import datetime
>>> import requests
>>>
>>> url = "https://repository.library.noaa.gov/fedora/export/download/collection/4"
>>> from_d = '2020-06-01T00:00:00Z'
>>> until_d = datetime.now().strftime('%Y-%m-%dT00:00:00Z')
>>>
>>> response = requests.get(url, params={'from': from, 'until': until_d})
>>> response.url 
'https://repository.library.noaa.gov/fedora/export/download/collection/4?from=2020-06-01T00:00:00Z&until2020-07-15T00:00:00Z
>>> json_d = response.json()
>>> docs = response['response']['docs']   
```

##### Example 3: parsing with pandas

The structure of NOAA JSON API records allow you to parse them using Python's popular data analysis library, [pandas](https://pandas.pydata.org/).

Similar to other examples, begin by importing a collection and then selecting the collection's items.

```
>>> import requests
>>> import pandas as pd
>>> url = "https://repository.library.noaa.gov/fedora/export/download/collection/4"
>>> response = requests.get(url)
>>> json_d = response.json()
>>> docs = json_d['response']['docs']
```

Once the items's collection data is loaded you can load into into Pandas' DataFrame from object with a single command/

```
>>> df = pd.DataFrame(docs)
>>> type(df)
<class 'pandas.core.frame.DataFrame'>
>>>
```

When parsing the NOAA IR JSON API with pandas there are side effects to be aware of, most notably how values in some fields or columns are imported into a DataFrame as a Python list objects.

##### More examples

More detailed examples can be found in this reposistory's ```noaa_json_api``` directory. 

## OAI-PMH 

The NOAA Institutional Respository also provides access to the the IR's collection through [OAI-PMH](http://www.openarchives.org/OAI/openarchivesprotocol.html). 

More information and examples for OAI-PMH are coming soon.
