# 🚄 Performances

Tested ws4sqlite v0.9.0 with JMeter, on a Windows system (Dell XPS 13 9305).

10000 requests in 100 bursts of 100 requests, 500ms ramp-up between requests. Single SELECT from a table with 1000 records, by primary key. Stored Statement used.

```
# Samples	10000
Average		1
Median		1
90% Line	2
95% Line	3
99% Line	14
Minimum		0
Maximum		24
Error %		0.0
```
