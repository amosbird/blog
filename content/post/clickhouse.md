+++
title = "ClickHouse 测试"
lastmod = 2017-09-29T00:28:26+08:00
draft = true
+++

## 机器配置 {#机器配置}

测试环境
-----------------------------------------------------------------
Linux Kernel: 3.10.0-514.26.2.el7.x86\_64 x86\_64
System: Dell product: PowerEdge R720xd
CPU: 2 Hexa core Intel Xeon E5-2620 0s (-HT-MCP-SMP-) cache: 30720 KB
Network: Intel I350 Gigabit Network Connection driver: igb
Drive:  Raid-5 12 SATA 3TB disks
Memory: 80GB
ClickHouse Commit Hash: df42f234f19ad21c0507d29185cdc1dc5ca4ce8f


## OnTime 测试集 {#ontime-测试集}

原始 CSV 文件大小：65GB

数据量：175043894 条

加载时间：(24m 28s 262ms)

磁盘占用量：14.17GB

```sql
SELECT formatReadableSize(sum(bytes)) FROM system.parts WHERE table = 'ontime_merge' AND active
```


### 建表语句 {#建表语句}

```sql
CREATE TABLE ontime_merge (
`Year` UInt16,
`Quarter` UInt8,
`Month` UInt8,
`DayofMonth` UInt8,
`DayOfWeek` UInt8,
`FlightDate` Date,
`UniqueCarrier` FixedString(7),
`AirlineID` Int32,
`Carrier` FixedString(2),
`TailNum` String,
`FlightNum` String,
`OriginAirportID` Int32,
`OriginAirportSeqID` Int32,
`OriginCityMarketID` Int32,
`Origin` FixedString(5),
`OriginCityName` String,
`OriginState` FixedString(2),
`OriginStateFips` String,
`OriginStateName` String,
`OriginWac` Int32,
`DestAirportID` Int32,
`DestAirportSeqID` Int32,
`DestCityMarketID` Int32,
`Dest` FixedString(5),
`DestCityName` String,
`DestState` FixedString(2),
`DestStateFips` String,
`DestStateName` String,
`DestWac` Int32,
`CRSDepTime` Int32,
`DepTime` Int32,
`DepDelay` Int32,
`DepDelayMinutes` Int32,
`DepDel15` Int32,
`DepartureDelayGroups` String,
`DepTimeBlk` String,
`TaxiOut` Int32,
`WheelsOff` Int32,
`WheelsOn` Int32,
`TaxiIn` Int32,
`CRSArrTime` Int32,
`ArrTime` Int32,
`ArrDelay` Int32,
`ArrDelayMinutes` Int32,
`ArrDel15` Int32,
`ArrivalDelayGroups` Int32,
`ArrTimeBlk` String,
`Cancelled` UInt8,
`CancellationCode` FixedString(1),
`Diverted` UInt8,
`CRSElapsedTime` Int32,
`ActualElapsedTime` Int32,
`AirTime` Int32,
`Flights` Int32,
`Distance` Int32,
`DistanceGroup` UInt8,
`CarrierDelay` Int32,
`WeatherDelay` Int32,
`NASDelay` Int32,
`SecurityDelay` Int32,
`LateAircraftDelay` Int32,
`FirstDepTime` String,
`TotalAddGTime` String,
`LongestAddGTime` String,
`DivAirportLandings` String,
`DivReachedDest` String,
`DivActualElapsedTime` String,
`DivArrDelay` String,
`DivDistance` String,
`Div1Airport` String,
`Div1AirportID` Int32,
`Div1AirportSeqID` Int32,
`Div1WheelsOn` String,
`Div1TotalGTime` String,
`Div1LongestGTime` String,
`Div1WheelsOff` String,
`Div1TailNum` String,
`Div2Airport` String,
`Div2AirportID` Int32,
`Div2AirportSeqID` Int32,
`Div2WheelsOn` String,
`Div2TotalGTime` String,
`Div2LongestGTime` String,
`Div2WheelsOff` String,
`Div2TailNum` String,
`Div3Airport` String,
`Div3AirportID` Int32,
`Div3AirportSeqID` Int32,
`Div3WheelsOn` String,
`Div3TotalGTime` String,
`Div3LongestGTime` String,
`Div3WheelsOff` String,
`Div3TailNum` String,
`Div4Airport` String,
`Div4AirportID` Int32,
`Div4AirportSeqID` Int32,
`Div4WheelsOn` String,
`Div4TotalGTime` String,
`Div4LongestGTime` String,
`Div4WheelsOff` String,
`Div4TailNum` String,
`Div5Airport` String,
`Div5AirportID` Int32,
`Div5AirportSeqID` Int32,
`Div5WheelsOn` String,
`Div5TotalGTime` String,
`Div5LongestGTime` String,
`Div5WheelsOff` String,
`Div5TailNum` String
) ENGINE = MergeTree(FlightDate, (Year, FlightDate), 8192)
```


### 测试结果 {#测试结果}

QueryID | SQL Text                                                                                                                                            | Query Time (Seconds) | Query Time Hot (Seconds)
--------|-----------------------------------------------------------------------------------------------------------------------------------------------------|----------------------|-------------------------
        |                                                                                                                                                     |                      |
Q0      | SELECT avg(c1) from (select Year, Month, count(\*) as c1 from ontime group by Year, Month);                                                         | 1.27                 | 0.13
Q1      | SELECT DayOfWeek, count(\*) AS c FROM ontime WHERE Year >= 2000 AND Year <= 2008 GROUP BY DayOfWeek ORDER BY c DESC;                                | 1.23                 | 0.21
Q2      | SELECT DayOfWeek, count(\*) AS c FROM ontime WHERE DepDelay>10 AND Year >= 2000 AND Year <= 2008 GROUP BY DayOfWeek ORDER BY c DESC;                | 1.4                  | 0.25
Q3      | SELECT Origin, count(\*) AS c FROM ontime WHERE DepDelay>10 AND Year >= 2000 AND Year <= 2008 GROUP BY Origin ORDER BY c DESC LIMIT 10;             | 5.2                  | 0.29
Q4      | SELECT Carrier, count(\*) FROM ontime WHERE DepDelay>10  AND Year = 2007 GROUP BY Carrier ORDER BY count(\*) DESC;                                  | 1.0                  | 0.09
Q5      | SELECT Carrier, avg(DepDelay > 10) \* 1000 AS c3 FROM ontime WHERE Year = 2007 GROUP BY Carrier ORDER BY Carrier;                                   | 0.12                 | 0.04
Q6      | SELECT Carrier, avg(DepDelay > 10) \* 1000 AS c3 FROM ontime WHERE Year >= 2000 AND Year <= 2008 GROUP BY Carrier ORDER BY Carrier;                 | 0.42                 | 0.26
Q7      | SELECT Year, avg(DepDelay > 10) FROM ontime GROUP BY Year ORDER BY Year;                                                                            | 6.92                 | 0.36
Q8      | SELECT DestCityName, uniqExact(OriginCityName) AS u FROM ontime WHERE Year >= 2000 and Year <= 2010 GROUP BY DestCityName ORDER BY u DESC LIMIT 10; | 7.96                 | 1.23
Q9      | SELECT Year, count(\*) as c1 from ontime group by Year;                                                                                             | 0.25                 | 0.23
Q10     | SELECT avg(cnt) FROM (SELECT Year,Month,count(\*) AS cnt FROM ontime WHERE DepDel15=1 GROUP BY Year,Month);                                         | 4.84                 | 0.36
Q11     | SELECT avg(c1) from (select Year,Month,count(\*) as c1 from ontime group by Year,Month);                                                            | 0.45                 | 0.47
Q12     | SELECT DestCityName, uniqExact(OriginCityName) AS u FROM ontime GROUP BY DestCityName ORDER BY u DESC LIMIT 10;                                     | 10.01                | 2.63
Q13     | SELECT OriginCityName, DestCityName, count() AS c FROM ontime GROUP BY OriginCityName, DestCityName ORDER BY c DESC LIMIT 10;                       | 3.06                 | 2.79
Q14     | SELECT OriginCityName, count() AS c FROM ontime GROUP BY OriginCityName ORDER BY c DESC LIMIT 10;                                                   | 1.11                 | 1.06


## NYC Taxi 测试集 {#nyc-taxi-测试集}

原始 CSV 文件大小：570GB
数据量：1368105519 条加载时间： (2h 19m 34s 746ms)
磁盘占用量：173GB

```sql
SELECT formatReadableSize(sum(bytes)) FROM system.parts WHERE table = 'trips_log' AND active
```


### 建表语句 {#建表语句}

```sql
CREATE TABLE trips_log
(
trip_id                 UInt32,
vendor_id               String,
pickup_datetime         Nullable(DateTime),
dropoff_datetime        Nullable(DateTime),
store_and_fwd_flag      Nullable(FixedString(1)),
rate_code_id            Nullable(UInt8),
pickup_longitude        Nullable(Float64),
pickup_latitude         Nullable(Float64),
dropoff_longitude       Nullable(Float64),
dropoff_latitude        Nullable(Float64),
passenger_count         Nullable(UInt8),
trip_distance           Nullable(Float64),
fare_amount             Nullable(Float32),
extra                   Nullable(Float32),
mta_tax                 Nullable(Float32),
tip_amount              Nullable(Float32),
tolls_amount            Nullable(Float32),
ehail_fee               Nullable(Float32),
improvement_surcharge   Nullable(Float32),
total_amount            Nullable(Float32),
payment_type            Nullable(String),
trip_type               Nullable(UInt8),
pickup                  Nullable(String),
dropoff                 Nullable(String),
cab_type                Nullable(String),
precipitation           Nullable(Float32),
snow_depth              Nullable(Float32),
snowfall                Nullable(Float32),
max_temperature         Nullable(Int8),
min_temperature         Nullable(Int8),
average_wind_speed      Nullable(Float32),
pickup_nyct2010_gid     Nullable(UInt8),
pickup_ctlabel          Nullable(String),
pickup_borocode         Nullable(UInt8),
pickup_boroname         Nullable(String),
pickup_ct2010           Nullable(String),
pickup_boroct2010       Nullable(String),
pickup_cdeligibil       Nullable(FixedString(1)),
pickup_ntacode          Nullable(String),
pickup_ntaname          Nullable(String),
pickup_puma             Nullable(String),
dropoff_nyct2010_gid    Nullable(UInt8),
dropoff_ctlabel         Nullable(String),
dropoff_borocode        Nullable(UInt8),
dropoff_boroname        Nullable(String),
dropoff_ct2010          Nullable(String),
dropoff_boroct2010      Nullable(String),
dropoff_cdeligibil      Nullable(String),
dropoff_ntacode         Nullable(String),
dropoff_ntaname         Nullable(String),
dropoff_puma            Nullable(String)
) ENGINE = Log;
```


### 查询语句 {#查询语句}

QueryID | SQL Text                                                                                                                                                                                         | Query Time (Seconds) | Query Time Hot (Seconds)
--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------|-------------------------
        |                                                                                                                                                                                                  |                      |
Q0      | SELECT cab\_type, count(\*) FROM trips\_log GROUP BY cab\_type;                                                                                                                                  | 10.14                | 11.57
Q1      | SELECT passenger\_count, avg(total\_amount) FROM trips\_log GROUP BY passenger\_count;                                                                                                           | 12.00                | 6.27
Q2      | SELECT passenger\_count, toYear(pickup\_datetime) AS year, count(\*) FROM trips\_log GROUP BY passenger\_count, year;                                                                            | 10.45                | 7.23
Q3      | SELECT passenger\_count, toYear(pickup\_datetime) AS year, round(trip\_distance) AS distance, count(\*) FROM trips\_log GROUP BY passenger\_count, year, distance ORDER BY year, count(\*) DESC; | 13.03                | 10.80
