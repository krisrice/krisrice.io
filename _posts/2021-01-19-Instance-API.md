---
layout: post
published: true
title: 'Monitoring ORDS Connection Pools'
date: '2021-01-19'

---

# Monitoring ORDS Connection Pools

ORDS can have many connection pools going at any point in time. Typically to see what the state of the connection pool was someone would have to go into the database and do things like `select from v$session` such as:

```
SQL> select count(*) from v$session where username = 'ORDS_PUBLIC_USER';

   COUNT(*)
___________
         61
```

This is can work when it's a small system with 1 or 2 database pools defined in ORDS. However, there are ORDS instances which have 1000s of connection pools servicing 1000s of PDBs spread across many many CDBs. This large of a setup makes it very painful to query every CDB/PDB to find out how the pools are doing in the instance. 

Then with High Availability added to the mix with multiple ords instances things can get more complicated by querying the originating machine for a db connection also.

## UCP Pool Statistics

ORDS levarages Oracle's Universal Connection Pool libraries for it's connection pool. This library has had for a long time [java APIs](https://docs.oracle.com/en/database/oracle/oracle-database/19/jjucp/pool-statistics.html#GUID-E9E1CC73-F1D6-4A0E-9449-07106BC14EED) to enable monitoring the pool and it's health. 

```
PoolDataSource pds = PoolDataSourceFactory.getPoolDataSource();
...
...
int totalConnsCount = pds.getStatistics().getTotalConnectionsCount();
System.out.println("The total connection count in the pool is "+ totalConnsCount +".");

```

## ORDS Instance API

ORDS 20.4 adds a new set of REST APIs defined specifically to monitor the connection pools. This new feature is named the [Instance API](https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/20.4/aelig/installing-REST-data-services.html#GUID-116149DE-01E1-4056-A723-0EFB96737377)


This new API does require a flag to enable it and a user with appropriate role assigned to access. Once enabled, there are a few new REST endpoints defined with an OpenAPI specification.

![](/img/instance_api_open_api.png)


The new APIs can list the over all pool status(es), the list of pools and the status of each pool. 

This is a bare bones html page that uses the API to show the statics of the pool while there is a test of 2k http requests being issued.  There's a lot of detail in this API ranging from the most simple `borrowedConnectionsCount` which is the connections currently being used to things like the wait time to getting a connection. 

These APIs changed in ORDS 21.2 with the following end points.

```
/ords/_/instance-api/stable/database-pools-cache/
/ords/_/instance-api/stable/database-pools-cache/{pool},{type}
/ords/_/instance-api/stable/database-pools-cache/{pool},{type},{service}
/ords/_/instance-api/stable/status
```

## Instance API Setup

The thing to note is this is an ORDS level API across ALL connection pools so there is no database user nor database roles that can be used to gain access to this API. That would not really make security sense to look into a pool on database A from a credential on database B.  This is going to require a webserver/ords level user and role to be assigned. This can be done many ways depending on the webserver being used to host ords. For example, in Tomcat there is a tomcat-users.xml.  

To use the ORDS built in user, issue this to create a user named "instanceAPI"


```
 java  -jar ords.war user instanceAPI "Listener Administrator,System Administrator"

 ```

This can be verified checking the `credentials` file and notice the user is created and the roles are a csv on the last segment

```

➜  more ords/credentials
instanceAPI;{SSHA-512}<...>;Listener Administrator,System Administrator
```

The other step to enabling is to enable the feature in the ords/defaults.xml


```
java -jar ords.war set-property instance.api.enabled true
```



## Trivial HTML for example

![](/img/ords_pools_monitor.gif)

Here is the Gist on Github for this example

<div>
  <script src="https://gist.github.com/krisrice/b35a30e116784f5b9781e06c120d7831.js">
  </script>
</div>
