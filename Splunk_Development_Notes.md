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
