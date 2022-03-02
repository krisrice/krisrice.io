---
layout: post
published: true
title: 'Strava/Zwift and Autonomous Database'
date: '2022-03-02'
---

# Visualize and Analyze Strava Data on Autonomous Database

A new quickstart was published on Oracle Cloud Console's homepage this week. This new quick start uses one of our Product Manager's Strava data to show off how easy the new Dashboaring and Charting feature work in Database Actions ( sqldev-web ) with Autonomous Database.

![](/img/strava_quickstart.png)

This quick start creates an Always Free Autonomous Database, loads the Strava Data, then produces a dashboard over it all. 

![](/img/strava_dashboard.png)

# Free


[Oracle ALWAYS Free Tier](https://www.oracle.com/cloud/free/) is free, always offers many services including a free Oracle Autonomous Database. So for someone wanting to try this out for free, just sign up and kick the tires. There's a number of other free services including compute, storage, function , containers,....
![](/img/always_free_products.png)


# Use your Strava Data

After seeing this a number of Oracle Employees asked how to do this on their own data.

### Getting the data from Strava

This is the scariest part because Strava for some reason connects Downloading your data to Deleting the Profile completely! They clearly did not consider the Developer Experience in folks wanting to get at their own data. This is a nice read on what the Developer Experience should be.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">icymi - i wrote a thing about developer experience. <a href="https://t.co/6CGrnYNxJ2">https://t.co/6CGrnYNxJ2</a></p>&mdash; Go on then, drop it! (@monkchips) <a href="https://twitter.com/monkchips/status/1496169202717270019?ref_src=twsrc%5Etfw">February 22, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

In the account page, there's section at the bottom. Click the Getting Started.

![](/img/strava_download_delete.png)

Now in section 2, click "Request your Archive"

![](/img/strava_request_data.png)

### Wait for an email
![](/img/waiting.jpeg)

### The email

Obviously click the download button.

![](/img/strava_download_ready.png)


# Now Load the Data

I'm recommend make a new directory named strava and unzip into that. There's everything in this zip including the gpx/fit/tcx files, images, authorized apps, every aspect of the Strava account.

![](/img/strava_unzip.png)

## LOAD into the database.

Now that you have the data in `csv` there's many many ways to load the data. Some options are outlined [here](https://www.thatjeffsmith.com/archive/2019/10/batch-loading-csv-to-a-table-in-oracle-autonomous-database-using-autorest-api/) and [here](https://docs.oracle.com/en/cloud/paas/autonomous-database/adbsa/load-data.html)

SQLcl has for a long time had a load csv option. You can learn more about the generic from Jeff's blog here from '19 [SQLcl Load Command](https://www.thatjeffsmith.com/archive/2019/09/sqlcl-and-the-load-csv-command/)

This is the sql script which will load the data. It's quite simple in that the 1st line `set load scan_rows 10000` tells sqlcl how many rows to scan for auto calculating the data type and size. Without this if the first 10 rows column X is say 100 chars long then row 11 is 101, the load would blow out. This let's it scan more and work out the type and length by scanning more data. 

The next is 1 line per csv file `load strava_activities activities.csv new` The command `load` takes in the new table name, the file to load, and `new` meaning make a new table automatically.

<script src="https://gist.github.com/krisrice/2b06df3ad4b51de278fe53672831f5ea.js"></script>

SQLcl has always been able to run remote scripts making this gist running able straigh off github using the `raw` link or of course save to a local file and run it.

```
SQL> @https://gist.githubusercontent.com/krisrice/2b06df3ad4b51de278fe53672831f5ea/raw/99792be40b8881f0799700afb66ac991ed356512/load_strava.sql
```


SQLcl will scrub all the column names into Oracle Database column names such as this for the `visibility_settings.csv`. Again Developer Experience here is lacking w/Strava and these column names. The tool will report the before and after of the names such as>

```
#INFO COLUMN 1: Include your activities in Metro and Heatmap => INCLUDE_YOUR_ACTIVITIES_IN_MET
#INFO COLUMN 2: Who can see that you were part of a group activity => WHO_CAN_SEE_THAT_YOU_WERE_PART
#INFO COLUMN 3: Who can see your activity in Flybys => WHO_CAN_SEE_YOUR_ACTIVITY_IN_F
#INFO COLUMN 4: Who can view your training log => WHO_CAN_VIEW_YOUR_TRAINING_LOG
#INFO COLUMN 5: Profile Visibility => PROFILE_VISIBILITY
#INFO COLUMN 6: Default Activity Visibility => DEFAULT_ACTIVITY_VISIBILITY
```

SQLcl will also print out the full details per file for the load and what settings it used 
```
Create new table and load data into table KLRICE.STRAVA_VISIBILITY_SETTINGS

csv
column_names on
delimiter ,
enclosures ""
encoding UTF8
row_limit off
row_terminator default
skip_rows 0
skip_after_names
batch_rows 50
batches_per_commit 10
clean_names transform
column_size rounded
commit on
date_format
errors 50
map_column_names off
method insert
timestamp_format
timestamptz_format
locale English United States
scan_rows 5000
truncate off
unknown_columns_fail on


```

# SQL For-The-Win

Now that it's loaded into relational tables, have fun. For example, here's my highest elevation things and most are in the virtual world of [Zwift](https://www.zwift.com/) and a trip we did to see the mountain gorrillas in [Bwindi Impenetrable Forest](https://en.wikipedia.org/wiki/Bwindi_Impenetrable_Forest). 

![](/img/strava_sql.png)


Good Luck. Happy SQL-ing