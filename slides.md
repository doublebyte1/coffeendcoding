# Location Analytics
## Understanding the coverage of mobile antennas
## <i class="fa fa-podcast" aria-hidden="true"></i>
#### Joana Simoes

--

## Processing Large Geospatial Datasets on the Cloud
## <i class="fa fa-rocket" aria-hidden="true"></i>

--
## About Me

<small>
<div id="container">
  <div id="navbar" class="column1">
    <ul>
       <li >Software Engineer & Data Scientist</li>
       <li >CTO & instructor at CodeOp</li>
       <li >Founder EarthPulse</li>
       <li >FOSS enthusiast, Charter member OSGeo</li>
    </ul>
  </div>
    <div id="content" class="column2">
    <img src="images/me_boston.jpg" >
  </div>
</div>

</small>


--
## CodeOp

<small>
Code school which supports women and other groups who traditionally have been in the margins of tech, to transition to tech careers.
</small>

![corona](/images/codeop.jpg)<!-- .element  width="65%" -->

https://codeop.tech/

---
## Bootcamps

- Full-stack development
- Data analytics
- Product management

![corona](/images/bootcamp.jpeg)<!-- .element  width="65%" -->


Note:
- a Bootcamp is halfway between university education and the hacker self-learn approach.
- You learn by yourself, but we are their to support you.
- It's a practical, hands-on approach, combining short, interactive lectures, activities and projects.
- It is also industry oriented: we provide the skills that are on demand in industry.
- 
--
![corona](/images/earthpulse_trans.gif)<!-- .element  width="25%" -->

Company specialized in Big Data and geospatial analytics.

https://earthpulse.pt

---
## Goal of this Project

- Understand the specific coverage areas of all cells/antennas on mobile network.
- Coverage across technology generations.

![corona](/images/mobile-users.jpeg)<!-- .element  width="50%" -->

Note:
- The use cases for this insight:
- The recipients for this insights
- location-based services (e.g. the cell in essence becomes a geo-f- ence), IOT (e.g. low-grade IOT devices which cannot use GPS, but still need some location indication) and  mobile network operations (operators do not know the exact coverage of their cells, instead they rely on calculation/projections/estimations)
- 2G, 3G, 4G

---
## Assets

- Crowd-sourced data about mobile usage (e.g.: technology type, technology generation, location).

![corona](/images/cell_tower_original.png)<!-- .element  width="50%" -->

Note: 
- explain beterhow this is obtained 

---
## How <i class="fa fa-cogs" aria-hidden="true"></i>

- Processing and analysing these large and noisy datasets.

Note:
- The total number of points for the UK for the O2 operator was around the 100 million.
- The idea was to crunch these data
- Turn raw data into useful information

--

## Specific Problem
- Transform a point cloud into a set of discrete surfaces (e.g.: polygons).

![corona](/images/few_clusters_trans.png)<!-- .element  width="55%" -->

Note:
- which follow a certain criteria.
- It basically comes down to delimitate the area covering all the points with the same
cid (A GSM Cell ID) base transceiver station )and lac)

---
## Points to polygons
- Convex Hull

![corona](/images/convex_hull6.png)<!-- .element  width="60%" -->


Note:
-  CH of a set of points X is the smallest convex set that contains X.

---
### Dealing with outliers

- Some points are outliers, but not all outliers are invalid. <!-- .element: class="fragment fade-in-then-semi-out"-->
- Valid points appear on clusters.<!-- .element: class="fragment fade-in-then-semi-out"-->

![corona](/images/img.png)<!-- .element  width="60%" -->

Note:
- Measurements can show up in odd locations, due to the behaviour of signals around water, mountains, etc.
- In this example the top measuraments sit on a slope of a mountain with a clear line of sight towards the centre of the polygon: they are valid, but there are a bunch of them.

---
## Clustering

- In density-based clustering, clusters are defined as areas of higher density than the remainder of the data set.
- DBSCAN is a cluster algorithm which does not require a pre-defined number of clusters.<!-- .element: class="fragment"-->

![corona](/images/dbscan.png)<!-- .element  width="25%" -->

Note:
- Objects in sparse areas - that are required to separate clusters - are usually considered to be noise and border points. 
- Our challenge was to tune DBSCAN to exclude the invalid outliers, but include the valid outliers (minpts).

---
## Putting it all together

- Some small-scale experiments were run in QGIS.
- Tune algorithm parameters.

![corona](/images/qgis.png)<!-- .element  width="60%" --> 


Note:
- minpts and radius from dbscan
- The next step was to run at this at scale.

--
## Tech Approach <i class="fa fa-database" aria-hidden="true"></i>

- We are dealing with a large volume of data, so it is not really feasible to hold it in memory.
- Databases are capable of storing and managing large amounts of data.<!-- .element: class="fragment"-->

---

PostGIS supports both, the **convex hull** and **DBSCAN** algorithms.

```SQL
SELECT * FROM            
            (SELECT ST_ClusterDBSCAN(geom_, %s, %s)
            OVER()
            AS
            cluster_id, latitude, longitude, geom_, techtype, cid,
            lac 
            FROM
            sample where cid=%s) sq
            WHERE cluster_id IS NOT NULL;
            
    SELECT d.techtype, d.cid, d.lac,
    ST_AsText(ST_ConvexHull(ST_Collect(d.geom_))) As geom_
    FROM clusters As d
    GROUP BY d.techtype, d.cid, d.lac;
```

---
## Architecture <i class="fa fa-sitemap" aria-hidden="true"></i>

- Python application to automate the interaction with the database.
- Wrapped the database and Python app in docker containers and orchestrated them using docker-compose.

![corona](/images/docker-compose_trans.png)<!-- .element  width="20%" --> 

---
## Technology Stack <i class="fa fa-cubes" aria-hidden="true"></i>

<table style="border-collapse: collapse; color:black; border: 0px yellow none; font-size: 15px; vertical-align: bottom;">
    <tr>
        <td><img src="/images/python.png" height="150px" style="background:none; border:none; box-shadow:none;"></td> 
        <td><img src="/images/postgis.png" height="150px" style="background:none; border:none; box-shadow:none;"></td>
        <td><img src="/images/docker.png" height="150px" style="background:none; border:none; box-shadow:none;"></td> 
    </tr>
</table>

Note:
- FOSS, FOSS4G

---

- The application supports batch processing of csv files.

![corona](/images/sausage_factory.jpeg)

Note:
- bulk processing

---

- For each file:

![corona](/images/sausage_factory.png) 

Note:
- OGC WKT format

---

## Deployment <i class="fa fa-cloud" aria-hidden="true"></i>

- To run this at scale, the container orchestration was deployed on AWS.
- The use of cloud servers, enabled us to run jobs in parallel.
- Docker made it really easy to deploy the application in different servers.

![corona](/images/aws_logo_179x109.gif)<!-- .element  width="30%" --> 

Note:
- We used 3 EC2 machines with 32 GB RAM machines
- The last run, we used large machines with 8 cores
- With this setup it took less than 2 hours to run.


- TODO: speak about which machines were created, and how long it took them to run.
- We split the batch jobs for each area code

- Job structure: process a bulk of files which contains one file, per area code

- When you need to process a large volume of data in a short period of time, cloud is a must
- The hardware was not so much an issue, but the number of servers was.

--
## Results <i class="fas fa-magic"></i>

<small>
- Run for the O2 network
- Processed around 100 million points.
- LACs: 4G: 550, 3G: 1212, 2G: 450.
- CIDs per LAC: ~ 500/700

</small>
![corona](/images/minpts_25b.png)<!-- .element  width="60%" --> 



Note:
- Use case case was to support location-based services for an MVNO (Mobile Virtual Operator) running over O2.
- The average/median LAC is 500/700
- Large LAcs can contain as much as 2000 CIDs and small lacs can contain 20-50

--
## Next Steps

- An API is now operational.
- Expanding into network planning and optimisation.


![corona](/images/api.png)<!-- .element  width="60%" --> 

Note:
- supporting location-based use cases where people do not want to use GPS, but still need a trustworthy location indication.

--
## Lessons Learned  <i class="fas fa-graduation-cap" aria-hidden="true"></i>
- When you process at scale, memory bugs are taken to an unprecedent level.<!-- .element: class="fragment fade-in-then-semi-out"-->
- Small tweaks have a great impact on the performance.<!-- .element: class="fragment fade-in-then-semi-out"-->
- Bulk processing: the art of split and merge.<!-- .element: class="fragment fade-in-then-semi-out"-->
- Monitor everything.<!-- .element: class="fragment fade-in-then-semi-out"-->
- Check the outputs in a GIS.<!-- .element: class="fragment fade-in-then-semi-out"-->
- Try to tune the parameters in small scale experiments.<!-- .element: class="fragment fade-in-then-semi-out"-->

Note:
- processing large amounts of data: it is hard to debug
- Let's say that we have a bug, that at each iteration is eating a small amount of memory. This can grow to a point that will eat all the RAM memory of the machine and cause it fail.
- If we can avoid clustering sets with less than x points
- There is a delicate balance here: we want to run batch jobs to decrease the amount of human intervention, but this also decreases our chance of intervention in case of errors.
- Generate detail logs through the application, in order to understand what happened in case of failure. This allows you to go back to a GIS and check what happened. Also monitor the hardware.
- It is not really easy to understand what is going on here, withour any visual queues.
- It is not always possible to tune the paramaeters in small scale experiments, because you dont have the scale and diversity of situations that you do with the real dataset.
- The use of spatial indexes in the tables is a must.

--

- PostGIS performed really well for this use case, even considering an iterative method such as DBSCAN.

![corona](/images/postgis.png)<!-- .element  width="25%" --> 


--
## Thank you!
### I would love to <i class="fa fa-bullhorn" aria-hidden="true"></i> from you


<table style="border-collapse: collapse; color:black; border: 0px yellow none; font-size: 25px; vertical-align: middle;">
    <tr>
        <td><i class="fa fa-envelope" aria-hidden="true"></i></td> 
        <td> joana@earthpulse.pt</td>

    <tr>
        <td><i class="fa fa-twitter" aria-hidden="true"></i></td> 
        <td>@doublebyte</td>
    </tr>    

 
</table>


--

## Questions?

![corona](/images/lisa.png) <!-- .element  width="30%" --> 
--

This presentation was created using [Reveal.js](https://revealjs.com/#/), the HTML presentation framework. Fork it at:
[https://github.com/doublebyte1/foss4guk.online](https://github.com/doublebyte1/foss4guk.online)

```
npx reveal.js-online
```


