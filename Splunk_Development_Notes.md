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
- Bloom Filters: Hashed Lexicon/Dictionary terms. Ran search generates bloom filter, compares against bucket's bloom filters to avoid reading .tsidx files or entire buckets.

**Streaming vs. Non-Streaming Commands**
