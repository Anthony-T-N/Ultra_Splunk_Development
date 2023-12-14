## Search Under the Hood
Data Storage | Crafting efficient searches | Troubleshooting commands

**Search Job Inspector**
- "Search job properties" overall stats of search (eventCount, earliestTime, optimizedSearch)

**SPL Commenting**
```
index=bad_security sourcetype=evil_linux
| ``` Backticks commenting here ```
| timechart span=1h count() by host
| ``` Useful for commenting out previous used SPL ```
```

**Splunk Architecture**

Search Head -> Indexers (Data  buckets: Hot, Warm, Cold, Frozen)


**Streaming vs. Non-Streaming Commands**
