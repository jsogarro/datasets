# A Brief Introduction to q and kdb+ for Analysts

## Overview

"kdb+ is a powerful database that can be used for streaming, real-time and historical data. Q is the SQL-like, general-purpose programming language built on top of kdb+."

## Launch q

Start a q console session by typing `q` at the shell prompt:

```
$ q
KDB+ 3.6 2018.10.23 Copyright (C) 1993-2018 Kx Systems
m32/ 8()core 16384MB sjt max.local 192.168.0.17 NONEXPIRE

q)
```

## Create a Table

Generate a simple table with time-series sales data:

```q
n:1000000
item:`apple`banana`orange`pear
city:`beijing`chicago`london`paris
tab:([]time:asc n?0D0;n?item;amount:n?100;n?city)
```

This creates a table with one million rows across four columns of random data.

## Simple Query

Select all rows where the item sold is a banana:

```q
select from tab where item=`banana
```

Output includes all columns when none are explicitly specified.

## Aggregate Query

Calculate the sum of amounts sold by city:

```q
select sum amount by city from tab
```

This returns a keyed table where `city` is the key column, sorted alphabetically.

## Time-Series Aggregate Query

Show sum of amounts by hour and item:

```q
select sum amount by time.hh,item from tab
```

The result is a keyed table with two key columns (`hh` and `item`), ordered by these columns.

## In-Memory Queries

### Random Data Generation

Load sample data from the `calls.q` script:

```q
/ calls.q
/ Generate some random computer statistics (CPU usage only)
n:1000; timerange:5D; freq:0D00:01; calls:3000
depts:`finance`packing`logistics`management`hoopjumping`trading`telesales
startcpu:(til n)!25+n?20 
fcn:n*fc:`long$timerange%freq

computer:([]
    time:(-0D00:00:10 + fcn?0D00:00:20)+fcn#(.z.p - timerange)+freq*til fc; 
    id:raze fc#'key startcpu
    )
computer:update `g#id from `time xasc update cpu:{
    100&3|startcpu[first x]+sums(count x)?-2 -1 -1 0 0 1 1 2
    }[id] by id from computer

/ Generate some random logged calls
calls:([] 
    time:(.z.p - timerange)+asc calls?timerange; 
    id:calls?key startcpu; 
    severity:calls?1 2 3
    )

/ Create a lookup table of computer information
computerlookup:([id:key startcpu] dept:n?depts; os:n?`win7`win8`osx`vista)
```

Load the script into your session:

```q
\l calls.q
```

### Data Overview

The generated tables contain:

- `computer`: Desktop CPU usage samples (7.2 million rows)
- `calls`: Help desk call records (3,000 rows)
- `computerlookup`: Static computer information (1,000 rows)

Sample output:

```q
q)computer
time                          id  cpu
-------------------------------------
2014.05.09D12:25:32.391350534 566 24 
2014.05.09D12:25:32.415609466 477 39 
2014.05.09D12:25:32.416150345 328 41 
```

```q
q)calls
time                          id  severity
------------------------------------------
2014.05.09D12:28:29.436601608 990 1       
2014.05.09D12:28:32.649418621 33  2       
2014.05.09D12:28:33.102242558 843 1       
```

```q
q)computerlookup
id| dept        os   
--| -----------------
0 | trading     win8 
1 | trading     osx  
2 | management  win7 
```

### Aggregation Queries

Calculate maximum, minimum, and average CPU usage by machine:

```q
select mxc:max cpu,mnc:min cpu,avc:avg cpu by id from computer
```

Aggregate by date:

```q
select mxc:max cpu,mnc:min cpu,avc:avg cpu by id,time.date from computer
```

Aggregate by hour:

```q
select mxc:max cpu,mnc:min cpu,avc:avg cpu by id,time.hh from computer
```

Aggregate by date and hour combined:

```q
select mxc:max cpu,mnc:min cpu,avc:avg cpu by id,time.date,time.hh from computer
```

Using `xbar` for flexible time bucketing:

```q
select mxc:max cpu,mnc:min cpu,avc:avg cpu by id,0D01:00:00.0 xbar time from computer
```

Custom aggregation with user-defined function:

```q
timeofday:{`0earlymorn`1midmorn`2lunch`3afternoon`4evening 00:00 07:00 12:00 13:30 17:00 bin x}
select mxc:max cpu,mnc:min cpu,avc:avg cpu by id,time.date,tod:timeofday[time.minute] from computer
```

Calculate average usage profile by time of day:

```q
select avc:sum[cpu]%sum samplecount by tod from select sum cpu,samplecount:count cpu by time.date,tod:timeofday[time.minute] from computer
```

Or simplified version (when dataset has equal records per period):

```q
select avg cpu by tod:timeofday[time.minute] from computer
```

### Joins

Left join to add static computer information:

```q
calls lj computerlookup
```

Aggregate calls by severity and department:

```q
select callcount:count i by severity,dept from calls lj computerlookup
```

Using foreign-key relationships:

```q
update `computerlookup$id from `calls
select callcount:count i by id.os,id.dept,severity from calls
```

### Time Joins

Asof join to align prevailing values at call time:

```q
aj[`id`time;calls;computer]
```

Window join to calculate statistics within time windows:

```q
p:update `p#id from `id xasc computer
wj[-0D00:10 0D00:02+\:calls.time; `id`time; calls; (p;(max;`cpu);(avg;`cpu))]
```

## On-Disk Queries

### Building the Database

Run the `buildsmartmeterdb.q` script to create an on-disk database:

```
$ q buildsmartmeterdb.q
```

The script displays configuration information and prompts for confirmation:

```
KDB+ 3.1 2014.05.03 Copyright (C) 1993-2014 Kx Systems
This process is set up to save a daily profile across 61 days for 100000 random customers with a sample every 15 minute(s). 
This will generate 9.60 million rows per day and 585.60 million rows in total
Uncompressed disk usage will be approximately 224 MB per day and 13664 MB in total
Compression is switched OFF
Data will be written to :./smartmeterDB

To modify the volume of data change either the number of customers, the number of days, or the sample period of the data.  Minimum sample period is 1 minute
These values, along with compression settings and output directory, can be modified at the top of this file (buildsmartmeterdb.q)

To proceed, type `go[]`
```

Execute the build:

```q
q)go[]
2014.05.15T09:17:42.271 Saving static data table to :./smartmeterDB/static
2014.05.15T09:17:42.291 Saving pricing tables to :./smartmeterDB/basicpricing and :./smartmeterDB/timepricing
2014.05.15T09:17:42.293 Generating random data for date 2013.08.01
2014.05.15T09:17:47.011 Saving to hdb :./smartmeterDB
2014.05.15T09:17:47.775 Save complete
```

### Running the Queries

Start the tutorial with `smartmeterdemo.q`:

```
$ q smartmeterdemo.q
```

Available commands:

```q
.tut.n[]     : run the Next example
.tut.p[]     : run the Previous example
.tut.c[]     : run the Current example
.tut.f[]     : go back to the First example
.tut.j[n]    : Jump to the specific example
.tut.db[]    : print out database statistics
.tut.res     : result of last run query
.tut.gcON[]  : turn garbage collection on after each query
.tut.gcOFF[] : turn garbage collection off after each query
.tut.help[]  : display help information
\\           : quit
```

Example query execution:

```q
q).tut.n[]

**********  Example 0  **********

Total meter usage for every customer over a 10 day period

Function meterusage has definition:

{[startdate; enddate]

 start:select first usage by meterid from meter where date=startdate;
 end:select last usage by meterid from meter where date=enddate;

 end-start}

2014.05.15T09:27:55.260 Running: meterusage[2013.08.01;2013.08.10]
2014.05.15T09:27:56.817 Function executed in 1557ms using 387.0 MB of memory

Result set contains 100000 rows.
First 10 element(s) of result set:

meterid | usage   
--------| --------
10000000| 3469.449
10000001| 2277.875
10000002| 4656.111
10000003| 2216.527
10000004| 2746.24 
10000005| 2349.073
10000006| 3599.034
10000007| 2450.384
10000008| 1939.314
10000009| 3934.089
```

### Experimentation

Try these experiments:

- Run each query multiple times to observe performance variation
- Execute queries with different parameters
- Restart the database with secondary processes using command-line flags
- Rebuild databases with varying data volumes
- Test with compression enabled to compare size and performance

### User Interface

Access the Smart Meter demo UI by starting the database on port 5600:

```
http://localhost:5600/smartmeter.html
```

Features available in the UI:

- Filter data by date, customer type, and region
- Group and aggregate by various dimension combinations
- Pivot data by selected fields
- View statistics (max, min, average, total, count)

## What Next?

Begin building your own database by loading data from CSV files, or explore other tutorials available in the documentation.
