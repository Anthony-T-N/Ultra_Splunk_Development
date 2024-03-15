## Using Fields

**Using the Fields Sidebar**
- Self-explanatory
Fields = Searchable key/value pairs

**Using Fields in Search**
```
index=magical_fields sourcetype=evil_linux
| ``` '!=' and 'NOT' difference ```
| hostname!=broken_server (Only checks hostname fields for value)
| NOT hostname=broken_server (Includes events without hostname field)
| ``` Check mutiple values in field ```
| hostname IN ("host1", "host2", "host3")
| ``` Filtering early == best practice ```
| fields hostname
| ``` Renaming to make fields descriptive ```
| rename hostname as "Company Devices"
```
**Fields in Search Results**
- Indexer automatically extracts fields (Metadata fields == host, sourcetype, source, _time, _raw)
- During Search-time: Field discovery extracts fields from raw event data. 
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
Search Trigger Action 

**Creating Scheduled Reports**
Craft search -> Save As -> Report -> Schedule -> Schedule Report Checkbox (Select frequency, Range, Priority) -> Trigger Actions
- Schedule Priority: Prioritisation between reporting 
- Schedule Window: Delay within time window when concurrent scheduled reporting exists (Concurrent reporting increase demand on system hardware)

**Managing Reports**
- Self-explanatory (Usage of GUI, edit options to schedule reports)
- Key notes: Power/Admin set report display for Self/App. Admin set report for all Apps
- Permissions can be given to specific reports. Report Embedding can be viewed by anyone (URL)

**Alerts & Creation & Actions & Management** - Create notifications when defined conditions are met based on a completed search and perform action.
- Alert types: Scheduled & Real-time
- Trigger once: Alert created once within specified timerange.
- Throttle Checkbox = Alert suppression.

## Visualizations 
Tables | Charts | Transformation Commands

**Using Formatting Commands**
```
index=complex_dataset sourcetype=ocean_sensors asian_sensors=*
| fields SENSORID coordinates brand_name offset #Field extraction == Expensive part of search. Specifying fields == efficient
| table SENSORID coordinates brand_name offset #Display in table in order of specified fields.
| dedup SENSORID coordinates # Removes duplicate events with combinations of field 1 & 2.
| ``` Total column with sum of rows. Also creating a row with sum of columns (col=true) and labeling them accordingly ```
| addtotals col=true label="Column Total" labelfield="SENSORID" fieldname="Row_total" #Specifying column (labelfield) to place custom row name (label) with addition of new column with label (fieldname)
| ``` Overwrite Row_Total column to have values given "<Whatever>" and commas ```
| fieldformat Row_Total = "<Whatever>" + tostring(Row_Total, "commas")
```

**Visualizating Data** - Difficult
```
``` Transforming Commands to Support Visualisation ```
| top field field2 limit=x OR 0 countfield="Count" showperc=true percentfield="Fieldname" useother=true #Count of most common values of field.
| rare #Least common values of field (Same options as top)
| stats #Common functions: count, distinct count, sum, average, min, max, list, values
| chart count over computer_name #Y-axis over X-axis.
| timechart span=5hr sum(coins) by vending_machine limit=0 #Uses _time to display events over time and treated as the X-axis
| trendline wma6(field) as trend #Three arguments (Trendtype, time period (6 days), field) #Trendtype: sma (Simple), ema (Exponential), wma (Weighted) ... moving average
```
- "By" clause: Split by another field.
  - `| rare fruits by stall limit=10 showperc=true # Show 10 rarest fruits from each stall`
  - ``` 
    index=company sourcetype=access_points usage!=Fun
    | chart count over office_area by usage #Counts events based on location and splits based on usage.
    ```
- `| stats count as "Magic Clock" by magical_stalls #Count of events based on magical_stalls renamed as Magic Clock`

**Generating Maps**
- Marker Maps (Interactive markers on map)
```
| iplocation ip_address_field #Adds new location fields based on field with IP addresses (Subject to third-party database so not all values exist)
| geostats latfield=lat longfield=lon count by vendor globallimit=0 #Uses same functions as stats command (Accepts single "by" clause arguments). Uses fields from iplocation command.
```
- Choropleth Maps (Metrics shown through shading)
  - Requires keyhole markup language file.
  - `| chart count by magical_states`
  - `| geom <kmz_file> featureIdField=magical_states #KMZ File == FeatureCollection. Requirement to use field mapping back to featurecollection.`

**Single Value Visualizations**
- Single Value Graph:
  - Caption text, colour & number formatting
  - Trellis Layout == Split visualisation by selected field.
- Gauge Graph (Radial, Filler, Marker):
  - `| gauge Total 0 100 200 300 #Optional to set gauge ranges. GUI based`

**Formatting Visualizations**
- Format Option (After a transformation Command: Wrap Results, Row Numbers, Click Selection (Cell/Row), Data Overlay (Heat map, High/Low Values), Totals, Percentages
- Chart Overlay (Useful with trendlines: Format Option -> Chart Overlay -> Select field for overlay -> Creates line graph over existing visualisation. 

## Working with Time
Time searches | Time based functions/commands | Timezones

**Searching with Time**
- _time field, one of fields (index, host, source, sourcetype) stored prior search time.
- _time expressed as unix/epoch time. At search time = Time converted to human-readable

- Timeline
    - (GUI) Click-drag across timeline to select specific time range.
    - (GUI) Select "+ Zoom to Selection" to view entire selected range.
    - (GUI) Time-range picker (Self-explanatory) 
    - Real-time search == Resource intensive. "Continuously update search results as the events arrive" - Re-word. Schedule report as alternative

- "earliest" & "latest" Time Modifiers 
    - `Syntax: earliest|latest=[+|-<timeInt><timeUnit>@<timeUnit>]`
    - Overrides Time Ranger Picker (GUI) settings. 
    - `@<timeUnit>` | Always rounds/snaps to beginning of timeUnit (s,m,h,d,w,w[0-6],mon,q,y).
    - Query can be copied to either search or advance time range picker.
 
- Default Time Fields
    - Timestamps within raw data is extracted and used to generate default date_* fields & time fields.
    - Time field fallbacks to index time when no timestamp exist in raw data. 

- bin Command 
    - `| bin _time [span=<int> OR <timescale>] [as <newfield>]`
    - Organise groupings of events based on time (BIN 1 = 1:00AM to 1:59AM, BIN 2 = 21:00AM to 2:59AM). span adjust size of bin/bucket.
    - Chunking a number of actions occuring every 5mins or per day.

**Formatting Time**
- eval command
    - Expression calculation and stores result in new/existing field.
    ```
    | eval field = now() #Time search started
    | eval field = time() #Time processed by eval command
    | eval field = relative_time(A,B) #A == Number (EPOCH time) B == Time modifier 
    | Example: eval = yesterday = relative_time(now(),"-1d@h")
    | EPOCH time for yesterday.
    
    | eval friendly_time = strftime(now(), "F %H:%M") # Converts result in a friendlier format. # 2021-04-20 16:00

    | eval field = strptime(friendly_time, "%Y-%m-%d %H:%M:%S,%N") #Converts friendly format to epoch/unix time.
    ```

**Using Time Commands**
- Timechart Command
    - `| timechart <stats-func>(<field>) by <field> [span=<int><timescale>] [limit=<int>]`
- Timewrap Command (Follows timechart command)
    - `| timechart span=1day count as "Scans"`
    - `| timewrap 1w`
    - Comparing multiple (Dependent on time range specified in timewrap command) periods on same graph. Breaks down a 2 week timechart down to 1 week with two lines.

**Working with Time Zones**
- date_* field values extracted directly from raw event (Does no consider local time settings).
- Display time based on user's time zone preference.
    - `eval my_hour = strftime(_time, "%H")`

**Working with Time END**
```
... earliest=-7d@w1+9h+13m latest=@w5-30m date_hour>=9 date_hour<17
| eval "Week_Middle" = relative_time(now(),"-3d@d")
| eval "Extracted_Keys" = strftime(Week_Middle,"%a %B %d %H:%M:%S.%N %Y")
| timechart span=1d count by usage
| timewrap 1w
```

## Statistical Processing
Single/Multi/Time-series | Transforming commands | Statistical Visualisations
- Data series (3-types) == Sequence of related data points plotted in a visualisation

**Chart Command**
- `| chart count(field) over row-split-field by [column-split-field] [span=4 limit=5 useother=f usenull=true]`

  ```
  # Achieves the same result
  chart count OVER field_A BY field_B
  chart count BY field_A field_B
  ```
  
**Timechart Command**
- `| timechart count(field) by column-split-field [span=5h limit=3]`
- (GUI): "Format" -> "General" -> "Multi-series Mode": Separate series into individual swimlanes.

**Top/Rare Command**
- `| top/rare field_A by field_B [countfield="Count_name" limit=7 showperc=t]`
- `| top/rare field_A field_B` #Top combinations.
- (GUI): Select field from "Field side-bar" and select "Top values"

**Stats Command**
- `| stats count(field_A) [as field_A_renamed by field_B]`
- Allows continuous split of data.
- `| stats count(field_A) as renamed_field_A, count(field_B) as total_events #Multiple stats functions separated by commas`
- `| stats count by stall fruit_type expiry #Order of fields important`

**Functions of the Stats Command**
- Statistical Functions Categories:
    - Aggregate:
    - Event Order: 
    - Multivalue:
    - Time:

```
| stats dc(field) 
| stats sum(field)
| stats min/max/avg
| stats values(unique_value_fields)
| stats list(all_value_fields)
```

**Transforming Commands Summary**
```
| chart/timechart limit=10 userother=t usenull=t span=10
| stats #can't use the above args.
```

**Eval Command**
- Assign one or more (Separated by commas) expressions/calculations to a newly created field.
- Supported operators (Arithmetic, Concatenation, Boolean, Comparison)
- Syntax: 
    - `| eval travel_member_status = case(travel<=10,"Low priority", travel>10,"Superstar")`
    - String values are case-sensitive and "double-quoted". Not numbers/int
    - Field names are unquoted/single quoted.
    - Period (.) to concat strings/numbers together.
- `| eval magic_number = 100/2, magic_number = magic_number + 5 #Calling multiple eval statements in one command`

**Functions of the Eval Command** - RETURN
- Common Eval Functions
    - pow()
    - round()
    - max()
    - min()
    - random()
- `| eval rounded_large_number = round(pow(7,3),2)`
- `| eval give_me_max_out_of_min_list = max(min(int_field, 4), min(11, 98), random())`

**Eval as a Function**
- ` stats count(eval(wand_type="Magical")) AS Magical_Wand`
- Eval used as function within count function. AS clause required.

**Rename Command**
- `| rename magical_wizards AS default_wizards, stick as "Magical wand"`
- `device* AS technology*` # Field (Example: device_abc_123) converted to new field (Example: technology_abc_123)

**Sort Command**
-`| sort (-|+) Field_A [limit=0]`
- Sorted lexicographically (Uppercase appears before lowercase)
```
| sort - States, Zones # Sorts both fields/columns in descending order.
| sort -States, Zones # Only sort "States" column in descending order, while "Zones" sorted in ascending order.
```

```
| stats count(tv_blocks) as pixels
| rename pixels AS blocks, television AS tv
| sort - blocks limit=10
```

- `| lookup zoo_animal_list.csv animal_ID [OUTPUT|OUTPUTNEW] colour animal_name`

## Leveraging Lookups and Subsearches
Lookups | Subsearches Correlations | Return

- Lookups: Addition of values/fields to events not indexed.
- Lookup types:
    - File-based (CSV File)
    - External (Scripts/Executables)
    - Key-value based pairs (KV Store)
    - Geo-spatial information (KMZ File)
 
**Using Lookups Commands**
- Lookups: Data pulled from standalone files based on specific field during search time and added to search results (Additional fields and field values)

**Inputlookup Command**
- `| inputlookup large_list.csv.gz #Loads data from file into Splunk`

**Lookup Command**
- `| lookup zoo_animal_list.csv animal_ID [OUTPUT|OUTPUTNEW] colour animal_name`
    - Referring to the zoo_animal_list.csv file or lookup definition. Uses animal_ID as input field. Data pulled from CSV file to populate event associated with the input field.
- OUTPUT == Overwrite existing fields. OUTPUTNEW == Create new fields.
- (GUI) Lookup table files (Upload new file) + Lookup Definitions (Configuration to support lookup command and is mandatory)

**Outputlookup Command**
- `| outputlookup zoo_animal_list_output.csv [createinapp=t/f]`
    - Writing results to the csv file or lookup definition. Argument "createinapp" tells Splunk to create lookup for system lookups directory.
- csv file continuously updated on a scheduled report or alert.

**Adding a Subsearch**
- `| [search/tstats] ... ` 
    - Subsearch runs first, starts with generating commands (Search/tstats) and enclosed in square brackets. Implicit AND operator between search and sub-search.
```
``` Adds NOT operator on all field-value pairs in csv ```
 index=wizard_list sourcetype=magic_tower
  NOT [inputlookup magic_list.csv]
```
```
```Main search applies against returned results from sub-search.```
index=company_A_network sourcetype=guest_ap
[search index=company_A_network sourcetype=guest_ap ext_ip!=192*
| fields ext_ip]

# EXPANDED AS:

index=company_A_network sourcetype=guest_ap AND ((ext_ip="172.111.111.111") OR (ext_ip="172.222.222.222") OR (ext_ip="172.333.333.333"))
```

**When to Use Subsearch**
- Subsearch can be an expensive operation due to resource usage (CPU/Memory).
- Limitations (Adjustable by Administrator):
    - Default 60s execution limit. (Results return regardless of completion) 
    - Default limit of 10,000 results.
- Real time search == All time by default (Time modifiers to control timeframe)
- stats and eval in single search == Enhanced performance.

**Troubleshooting Subsearch**
- eval/stats == Enhanced performance.
- Run outer search and subsearch separately to validate results are returning.
- (GUI) Bracket enclosed check.

**Return Command**
```
| return [10] new_field_A=field_A #Fieldname changed.
| return [1] field_A #Returns field-value pair.
| return [44] $field_A #Returns value without fieldname.
```
- Why use return command over fields?
    - Return is restrictive over results returned. Fields returns all key-value pairs.
    - Removes need for "Fields, rename, format, dedup, head" commands.

## Intro to Knowledge Objects

**What are Knowledge Objects?**

- Essentially tools to "discover and analyse data".
- Primary Types:
    - Fields (Field-value pairs)
    - Field extractions (Regex/Delimiters to Manually extract fields)
    - Field Aliases (Alternative assignment of names to existing fields)
    - Calculated Fields (Calculations based on existing fields)
    - Lookups (Data from CSV append to search)
    - Event Types (Saved search queries)
    - Tags (Labels for data and used in search)
    - Workflow Actions (HTTP Get/Post method interact with external resources or back into Splunk)
    - Reports/Alerts (Alert based on search conditions)
    - Macros (Advanced event types. Allows piping)
    - Data Models (Structured datasets 1-Events 2-Searches 3-Transactions. Pivot: Alternative interface to interact data with no knowledge of SPL)


**Managing Knowledge Objects**
- (GUI) Settings -> Knowledge -> Actions (Editing/Moving/Deleting Objects/Permissions)
- (ADMIN - GUI) - Reassign Knowledge Objects Options

## Search Optimization
Commands Search Optimization | Accelerated Search & Datamodels | Accessing Datamodels

- Search Modes 
    - (GUI) Fast (Field Discovery Disabled), Smart, Verbose (All extracted fields returned, event list and timeline for every search)
    - Time == Efficient way to filter events
    - Default fields (index, host, source, sourcetype) stored prior search time / extracted during index time.
    - Inclusion > Exclusion search statements.
    - OR and IN operators > Wildcards
    - Filtering commands early.

**Splunk Search Scheduler**

**Search Acceleration Overview**
- Accelerating datasets within data models
- Persistent and ad-hoc
- .tsidx file summaries for the data model bulit continuously 
  
**Datamodel Command**
` | datamodel [modelName] [objectNameItem/datasetID] [search/flat] [summariesonly=t]`

- Returns data model that are accessible (Without Data Model ID) structure (In JSON) and description of data model and its objects and can be searched against.
- [search] argument returns events under either data model or its associated datasetID.
- [Flat] arugment strips parent/Hierarchical information on field side bar (wizard.city > city)
- [summariesonly=t] argument returns data covered by accelerated summaries. Without it, Splunk will reach out to raw events to cover time-range. Increases efficiency and search as only searching against summaries.
- `allow_old_summaries = false` = Old and current data summaries
- Datasets within data models can be searched `datamodel.dataset`
- Summary only = Controls summarized and unsummarised data searched for data model
This argument applies only to accelerated data models.
- Unaccelerated data models.
- Datamodel name case sensitive. 

**Tstats Command**
`| tstats <stats-func> [summariesonly=<bool>] [from datamodel=<data_model_name] [where <search_query>] [by field-list] `

- Search limited to indexed fields but can search fields with dataset prefix (Dot notation: "application.method") in data models.
- Applicable to both tsidx files and data models (Using from clause).
- No performance gains when searching unaccelerated data models using tstats. 
- Without span, dynamic time span/bucket chosen.
- Wildcard fieldnames are not supported. Only used for field values `| tstats count where hostname=123* vs | tstats count where hostname=123 by source*`
- Stats + Data model = Automatic usage of tstats command.
    - Default: Automatic stats to tstats conversion disabled. (Affects summary indexing when enabled)

## Search Under the Hood
Data Storage | Crafting efficient searches | Troubleshooting commands

**Search Job Inspector**
- Tool displaying stats for a search.
- "Search job properties" overall stats of search (eventCount, earliestTime, optimizedSearch).

**SPL Commenting**
```
index=bad_security sourcetype=evil_linux
| ``` Backticks commenting here. No pipe required. ```
| timechart span=1h count() by host
| ``` Useful for commenting out previous used SPL ```
```
**Splunk Architecture**
- Search Head (Search requester to indexers and result merger)
- Data buckets: Hot -> Warm -> Cold -> Frozen (Rolls over after conditions are met)
- Buckets contains both journal.gz (Raw event data) & .tsidx files (References to slices of raw event data)
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
| ``` NOT efficient as "rename" forced to execute on the search head (Lacks distributed processing) because of transforming command. ```
```
Goal: Distributable Streaming Command before Centralised Streaming Commands == More efficient

**Breakers and Segmentation**
- BLUF: Deep dive into how unique terms are extracted from raw events to create bloom filters
- Major breakers: Isolate terms by dividing on: `[](){}!?;,'"&`
- Minor breakers: Isolate terms further on: `/:.-$`
- 192.168.0.1, Host, [Device] -> 192 168 0 1 Host Device -> Tokens/Terms for .tsidx lexicon.
- Using job inspector, "base lispy" == Expression used to create tokens/terms for lexicon.
- GOAL: Avoid breakers to avoid individual terms being searched against.
- field=TERM(EXACT_VALUE) #Must not contain major breakers and will not work on alias fields.
- Wildcards before search term will not be tokenized and all raw events will be searched.

**Makeresults Command**
- BLUF: Create fake data (Practising commands, regexes)
```
| makeresults
| eval raw = "YYYY-MM-DD <Rest of event details>"
| rex field=raw "<REGEX to extract value to new field>"
```
**Fieldsummary Command**
- BLUF: Performs common stats calculations (Saves trouble of specifying common stats commands) and returns as results table.
```
index=bad_security sourcetype=evil_linux
``` Only returns stats for two specific fields ```
| fieldsummary hostname user
```
**Informational Functions**
- BLUF: Certain functions provides more context behind certain values assigned to fields such as whether it is null or data type of value.
```
index=companys_honey_pot sourcetype=apple_products
``` Checks if host field is null and assigns null, if not, assign data type (Likely string in this scenario) ```
| eval host_data_type = if(isnull(host), "Null", typeof(host)) | table host, host_data_type
```
