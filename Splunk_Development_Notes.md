## Using Fields
Fields = Searchale key/value pairs

**Using the Fields Sidebar**
- Self-explanatory

**Using Fields in Search**
```
index=magical_fields sourcetype=evil_linux
| ``` != and NOT difference ```
| hostname!=broken_server (Only checks hostname fields for value)
| NOT hostname=broken_server (Includes events without hostname field)
| ``` Check mutiple values in field ```
| hostname IN ("host1", "host2", "host3")
| ``` Filtering early == best practise ```
| fields hostname
| ``` Renaming to make fields descriptive ```
| rename hostname as "Company Devices"
```
**Fields in Search Results**
- Indexer automatically extract fields (Metadata fields == host, sourcetype, source, _time, _raw)
- During search-time: Field-discovery extracts fields from raw event data. 
- Temporary fields:
`| eval calculated_sales_field = sales_price/2`
- Field Extraction
  Automatically creates regex based on provided examples:
  - `| erex new_field_name fromfield=_raw examples="123, Hello"`
  - Job Icon Selection -> See Regex used for search.
- Use regex against _raw field (Notes: Default using extracted fields, performance issues with just raw):
  - `| rex field=_raw "<REGEX HERE>"`

**Enriching Data**
- (GUI) Calculated fields can store eval commands/expression and will create new field on every Splunk Search. (Should reference fields already extracted)
- (GUI) Fields aliases (Hostname OR Device = Host)
- (GUI) Lookups (Append context at search time)
- See Knowledge Objects.

## Scheduling Reports & Alerts
Search Trigger Action |  | 

**Creating Scheduled Reports**
Craft search -> Save As -> Report -> Schedule -> Schedule Report Checkbox (Select frequency, Range, Priority) -> Trigger Actions
- Schedule Priority: Prioritisation between reporting 
- Schedule Window: Delay within time window when concurrent scheduled reporting exists (Concurrent reporting increase demand on system hardware)

**Managing Reports**
- Self-explanatory (Usage of GUI, edit options to schedule reports)
- Key notes: Power/Admin set report display for Self/App. Admin set report for All Apps
- Permissions can be given to specific reports. Report Embedding can be viewed by anyone

**Alerts & Creation & Actions & Management** - Create notifications when defined conditions met based on a search completed and perform action.
- Alert types: Scheduled & Real-time
- Trigger once: Alert created once within specified timerange.
- Throttle Checkbox = Alert surpression.

## Visualizations 
Tables | Charts | Transformation Commands

**Using Formatting Commands**
```
index=complex_dataset sourcetype=ocean_sensors asian_sensors=*
| fields SENSORID coordinates brand_name offset #Field extraction == Expensive part of search. Specifying fields == efficient
| table SENSORID coordinates brand_name offset #Display in table.
| dedup SENSORID coordinates # Removes duplicate events with combinations of field 1 & 2.
| ``` Total column with sum of rows. Also creating a row with sum of column (col=true) and labeling them accordingly ```
| addtotals col=true label="Column Total" labelfield="SENSORID" fieldname="Row_total" #NOT CLEAR ENOUGH
| ``` Overwrite Row_Total column to have values given "<Whatever>" and commas ```
| fieldformat Row_Total = "<Whatever>" + tostring(Row_Total, "commas")
```

**Visualizating Data** - UNDERDEVELOPED
```
``` Transforming Commands to Support Visualisation ```
| top field field2 limit=x OR 0 #Most common values of field.
| rare #Least common values of field.
| stats #Common functions: count, distinct count, sum, average, min, max, list, values
| chart count over computer_name #Y-axis over X-axis 
| timechart span=5hr sum(coins) by vending_machine limit=0 #Uses _time to display events over time.
| trendline
```

**Generating Maps**
- Marker Maps (Interactive markers on map)
```
| iplocation ip_address #Adds new location fields (Dependent on third-party database so not all values exist)
| geostats latfield=lat longfield=lon count by vendor globallimit=3 #Uses same functions as stats command (Accepts single "by" clause arguments). Uses fields from iplocation command.
```
- Choropleth Maps (Metrics shown through shading)
  - Requires keyhole markup language file.
  - `| geom <kmz_file> featureIdField=Country #KMZ File == FeatureCollection. Requirement to use field mapping back to featurecollection.`

**Single Value Visualizations**
- Single Value Graph:
  - Caption text, colour & number formatting
  - Trellis Layout == Split visualisation by selected field.
- Gauge Graph (Radial, Filler, Marker):
  - `| gauge Total 0 100 200 300 #Optional to set gauge ranges. GUI based`

**Formatting Visualizations**
- Format Option (After a transformation Command: Wrap Results, Row Numbers, Click Selection (Cell/Row), Data Overlay (Heat map, High/Low Values), Totals, Percentages
- Chart Overlay (Useful with trendlines: Format Option -> Chart Overlay -> Select field for overlay -> Creates line graph over existing visualisation. 

## Search Under the Hood
Data Storage | Crafting efficient searches | Troubleshooting commands

**Search Job Inspector**
- A tool showing stats for a search.
- "Search job properties" overall stats of search (eventCount, earliestTime, optimizedSearch)

**SPL Commenting**
```
index=bad_security sourcetype=evil_linux
| ``` Backticks commenting here ```
| timechart span=1h count() by host
| ``` Useful for commenting out previous used SPL ```
```
**Splunk Architecture**
- Search Head (Search requester to indexers and result merger)
- Data buckets: Hot -> Warm -> Cold -> Frozen
- Buckets contains journal.gz (Raw event data) & .tsidx (References to slices of raw event data)
- Extract unique terms from raw events -> Lexicon/Dictionary referencing slices containing unique terms.
- Bloom Filters (Created when roll over to hot to warm): Hashed Lexicon/Dictionary terms from .tsidx file. Ran Splunk search generates bloom filter, compares against bucket's bloom filters to avoid reading .tsidx files or entire buckets.

**Streaming vs. Non-Streaming Commands**
BLUF: Types of commands affect search efficiency based on where they are executed. 
- Transforming Commands (Operate on entire result set of data) - Executed on search head
  - Examples: stats, timechart, chart, top, rare
- Streaming Commands
  - Centralised Streaming Commands (Applies transformations on each event in order) - Executed on search head
    - Examples: transaction, streamstats
  - Distributable Streaming Commands (Applies transformations on each event individually & disregards event order) - Executes on indexers
    - Examples: rename, eval, fields, regex
```
index=bad_security sourcetype=evil_linux
| ``` eval == Distributable Streaming Command == Executed on indexers ```
| eval ...
| ``` Timechart == Transforming command == Executed on search head ```
| timechart count by hostname
| ``` Indexers to search head == Efficient ```

index=bad_security sourcetype=evil_linux
| ``` Transforming Command ```
| stats ...
| ``` Distributable Streaming Command ```
| rename
| ``` NOT efficient because "rename" forced to execute on the search head (Lacks distributed processing) because of transforming command ```
```
**Breakers and Segmentation**
BLUF: Deep dive into how unique terms are extracted from raw events to create bloom filters
- Major breakers: Isolate terms by dividing on: `[](){}!?;,'"&`
- Minor breakers: Isolate terms further on: `/:.-$`
- 192.168.0.1, Host, [Device] -> 192 168 0 1 Host Device -> Tokens/Terms for .tsidx lexicon.
- Using job inspector, "base lispy" == Expression used to create tokens/terms for lexicon.
- GOAL: Avoid breakers to avoid individual terms being searched against.
- field=TERM(EXACT_VALUE) #Must not contain major breakers and will not work on alias fields.
- Wildcards before search term will not be tokenized and all raw events will be searched.

**Makeresults Command**
BLUF: Creates fake data (Practising commands, regexes)
```
| makeresults
| eval raw = "YYYY-MM-DD <Rest of event details>"
| rex field=raw "<REGEX to extract value to new field>"
```
**Fieldsummary Command**
BLUF: Performs common stats calculations (Saves trouble of specifying common stats commands) and returns as results table.
```
index=bad_security sourcetype=evil_linux
``` Only returns stats for two specific fields ```
| fieldsummary hostname user
```
**Informational Functions**
BLUF: Certain functions provides more context behind certain values assigned to fields such as whether it is null or data type of value.
```
index=companys_honey_pot sourcetype=apple_products
``` Checks if host field is null and assigns null, if not, assign data type (Likely string in this scenario) ```
| eval host_data_type = if(isnull(host), "Null", typeof(host)) | table host, host_data_type
```
