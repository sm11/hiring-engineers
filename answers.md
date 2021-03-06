# Solutions Engineering Technical Exercise
#### - Suzie Mae

-----
## Environment Setup
-----
_For this technical exercise, I chose to install my agent on Docker rather than vagrant because Docker is lighter weight and would have used less of my system's (rather limited) resources._

**Installation:**
I created my docker container (with the datadog agent) by running the command below. <br/>
> **Side note:** ``` -p 127.0.0.1:8126:8126/tcp```  is particularly important to bind the host port to the docker port; thereby enabling communication between the two machines.

```
docker run -d --name <container_name>
              -v /var/run/docker.sock:/var/run/docker.sock:ro \
              -v /proc/:/host/proc/:ro \
              -v /sys/fs/cgroup/:/host/sys/fs/cgroup:ro \
              -e DD_API_KEY=<YOUR_API_KEY> \
              -e DD_APM_ENABLED=true \
              -p 127.0.0.1:8126:8126/tcp \
              datadog/agent:latest

```
Following this, I ran:
 ``` 
 docker exec -it <container_name> /opt/datadog-agent/bin/agent/agent status 
 ``` 
 to get information on my installation as well as to ensure my container was running without errors. An excerpt of the output is shown below:

```
==============
Agent (v6.6.0)
==============

  Status date: 2018-11-21 17:08:00.027401 UTC
  Pid: 341
  Python Version: 2.7.15
  Logs: /var/log/datadog/agent.log
  Check Runners: 4
  Log Level: debug

```

> **Side note:** Some important docker commands to understand the state of the cointainers can be [found here](https://docs.docker.com/engine/reference/commandline/docker/#child-commands).

-----
## Collecting Metrics
-----
#### Tagging

To edit the files within docker (which include the datadog-agent configuration file), there are two potential ways: <br/>

**Method 1:** 
- Copy the file to the host, edit on the host and copy back.
```
  docker cp <container_name>:<container filepath> <local filepath>
  vim <local filepath>
  docker cp <local file path> <container_name>:<Container file path>

```
**OR**

**Method 2:** <br/> 
- Log into the container ```docker exec -it <container_name> bash ``` 
- Install the neccessary editor. 

I chose to install ```vim-tiny``` over ```vim``` since it satisfied my requirements for a basic editor and occupies less space. Running the commands below enabled this installation.


```
apt-get update
apt-get install -y vim-tiny

``` 
> **Side note 1:** The command when using ```vim-tiny``` to work with files is ```vim.tiny``` <br/>
>
> **Side note 2:** Although I did most of my work form within the container (as this was more convient for me), all the subcommands run within the container can also be run from outside the container by using:<br/>
``` docker exec -it <container_name> <path to bin> <command>```
>
> For example:
> - To check the agent status from within the container, run: 
>``` 
>/opt/datadog-agent/bin/agent/agent status 
>```
> - To check the status of the agent from the host, run: 
>``` 
>docker exec -it <container_name> /opt/datadog-agent/bin/agent/agent status 
>```
>
> _Other useful agent commands can be [found here](https://docs.datadoghq.com/agent/faq/agent-commands/?tab=agentv6)_

> **_Fun tip_**: If you mess up your configuration file (like I did), and the container refuses to start with a  corrupted configuration file, all isn't lost and you don't have to spin up a new container. Create a new  configuration file (```datadog.yaml```) on the host and copy it, from the host into the right folder of the container (```/etc/datadog-agent/```) using **Method 1** above.


I added my tags to the agent configuration file by executing:
```
vim.tiny /etc/datadog-agent/datadog.yaml

```
(within the container). Then, I added the following lines to my datadog.yaml file:

```
tags:
  - host_tag
  - env:prod
  - role:database
```
#### Navigating to host map

![alt text][img1a]

[img1a]: ./images/host_map_navigation.png "Navigating to host map"

#### Host map with added tags

![alt text][img1b]

[img1b]: ./images/host_map_1.png "Host map"

#### Installing a database
**Chosen integration:** Postgres SQL <br/><br/>
For this exercise, I installed Postgres SQL using Postgres.app (since it is specifically packaged for my development OS). Instructions on downloading and installing Postgress.app can be [found here](https://postgresapp.com). After this, following the [instructions here](https://docs.datadoghq.com/integrations/postgres/) for integrating Postgres SQL had my integration up and running.

> **Side note 1**: To ensure the integration is running as expected, check the status of the agent (as explained above)
>
> The output should be along the following lines (if working properly):
```
    postgres (2.2.3)
    ----------------
        Instance ID: postgres:8e6dded31c225e88 [OK]
        Total Runs: 1,510
        Metric Samples: 43, Total: 64,930
        Events: 0, Total: 0
        Service Checks: 1, Total: 1,510
        Average Execution Time : 210ms
        

```

> **Side note 2**: Custom integrations can be found with integration instructions after logging into Datadog and going to integrations (as shown in the image below): <br/>
> #### Navigating to integrations after logging into Datadog
> ![alt text][img_a]
>
>[img_a]: ./images/integrations_navigation.png "Navigating to integrations after logging into Datadog"


#### Creating a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000.

- [Custom check python file](./my_check.py) <br/>
- [Custom check configuration file](./my_check.yaml)

> **Side note**: To ensure the custom check is running as expected, run: <br/>
>```/opt/datadog-agent/bin/agent/agent check <check_name>```
>
> The output should be along the following lines:
 ```
  Running Checks
  ==============
    
    my_check (1.0.0)
    ----------------
        Instance ID: my_check:5ba864f3937b5bad [OK]
        Total Runs: 1
        Metric Samples: 1, Total: 1
        Events: 0, Total: 0
        Service Checks: 0, Total: 0
        Average Execution Time : 0s
 ```
>
> where ``` my_check ``` is the name of your custom check.

#### Changing the collection interval without modifying the Python check file.

In programattically creating a custom check, two files are involved:
- a python file (ending in ```.py```) and 
- a configuration file (ending in ```.yaml```) 
<br/>
Both files must have the same name and be placed in the following folders:

```
Config file: /etc/datadog-agent/conf.d/
Python file: /etc/datadog-agent/checks.d/
```

Within the configuration file, a dictionary of instances can be declared stating the `min_collection_interval`. In this case, 45 seconds. The syntax for configuration file content stating 45 seconds collection interval is shown below:

```
init_config:

instances:
    - min_collection_interval: 45
```

More information can be found [here on custom metrics and their configuration](https://docs.datadoghq.com/developers/write_agent_check/?tab=agentv6). 

-----
## Visualizing Data
-----
#### Timeboard Scripts:
- [Script for creating a single graph timeboard](./create_timeboard_single.py)
- [Script for creating a multiple graph timeboard](./create_timeboard_multiple.py)

#### Timeboard - Snapshot of single graph over 5 minutes
![alt text][img2a]

[img2a]: ./images/timeboard_over_5_minutes.png "Timeboard - Snapshot of single graph over 5 minutes"


> **Note:** 
> I also created a timeboard with multiple graphs in order to extract clearer details about the rollup sum function. <br/><br/>
> **Reason**: On the single graph, the rollup sum has values so high they cannot be accomodated comfortably with the other graphs. This meant that either:
> -  the details of the rollup sum would be clear and the other plots (```my_metric average``` and ```postgresql rows returned```) will be too small to give useful information **or** 
> - the details of the my_metric average and postgresql plots will be clear and the rollup sum plot will be virtually non-existent <br/>
>
> In additon, the hourly buckets of the rollup sum contribute to the "adequate information" issue. This is because the rollup sum plot has much more points that are spread out than the ```average my_metric``` and ```postgresql rows returned``` plots --- only one data point per hour, as opposed to the multiple data points for the other plots. (See the graphs below).

#### Timeboard - single graph (mulitple plots)
![alt text][img2b]

[img2b]: ./images/timeboard_over_x_hours.png "Single graph over 4 hours - to show the difference in values"

#### Timeboard - multiple graphs

![alt text][img2c]

[img2c]: ./images/timeboard_for_multiple_boards.png "Timeboard - multiple graphs"

<br/>
As can be seen, the "multiple graphs" timeboard above facilitates easy information extraction from all the plots. 


### Anomaly Graph

The anomaly graph displays metrics/situations where the query returns values outside the historical/established norm. It highlights these areas in red (as seen in the image below). For my choice of metric, it includes extreme and/or outlier values as well as unexpected behaviour (the area of minimal change shown by a relatively straight line). 

#### Timeboard - Highlighting the anomalies

![alt text][img2d]

[img2d]: ./images/anomalies_1.png "Timeboard - Highlighting the anomalies"

## Monitoring Data


#### Metric Monitor Configuration
**Steps to configure the monitor:** <br/> Choose **New Monitor** (under Monitors) ---> **Metric** (on the Select a monitor type page) and **Threshold Alert**. <br/>

#### Navigating to New Monitor

![alt text][img2e]

[img2e]: ./images/monitor_new_monitor.png "Navigating to New Monitor"

For a warning threshold of 500 and alert of 800 (over the last 5 minutes) as well as 'No data' (over 10 minutes), the configuration is shown in the image below.

#### Metric monitor configuration snapshot

![alt text][img3]

[img3]: ./images/metric_monitor_config.png "Metric monitor configuration"

#### Alert message configuration ####
```
{{#is_warning}}

@xidornx@aol.com 

Warning has been triggered on:

- **Host Name**: {{host.name}} 
- **IP Address**: {{host.ip}} 
- **Trigger Value**: {{value}}

{{/is_warning}}



{{#is_alert}}

@xidornx@aol.com 

Alert: Has been triggered on:

- **Host Name**: {{host.name}} 
- **IP Address**: {{host.ip}} 
- **Trigger Value**: {{value}}

{{/is_alert}} 



{{#is_no_data}}

@xidornx@aol.com 

No data: There has been no data on my_metric on:
  
- **Host Name**: {{host.name}} 
- **IP Address**: {{host.ip}} 
- **Trigger Value**: {{value}}

{{/is_no_data}}

```

#### Output
- #### Monitor Notification 1 (Warning)
![alt text][img3a]

[img3a]: ./images/monitor_alert_log_1.png "Monitor Notification 1 (Warning)"


- #### Monitor Notification 2 (Alert)

![alt text][img3b]

[img3b]: ./images/monitor_alert_log_2.png "Monitor Notification 2 (Alert)"

- #### Monitor Notification 3 (No Data)

![alt text][img3c]

[img3c]: ./images/no_data_alert.png "Monitor Notification 2 (No Data)"

#### Scheduling Downtimes

**Steps to schedule downtimes:**<br/>
Monitors --> Manage Downtime --> Schedule Downtime. Select monitor, configuration and create message. <br/>
I have included snapshots of the various required settings below:

#### Navigating to Manage Downtime

![alt text][img3d]

[img3d]: ./images/monitor_manage_downtime.png "Navigating to Manage Downtime"

#### Sat - Sun scheduled configuration notification (recurring)
![alt text][img4a]

[img4a]: ./images/weekly_downtime_schedule(recurrent).png "Sat - Sun scheduled downtime configuration (recurring)"


#### Sat - Sun scheduled configuration notification (one-off)

![alt text][img4b]

[img4b]: ./images/weekly_downtime_schedule(one_off).png "Sat - Sun scheduled downtime configuration (one-off)"


#### Sat - Sun scheduled downtime notification
**Note:** I configured a one-off notification for this exercise)<br/>
![alt text][img4c]

[img4c]: ./images/scheduled_downtime_1.png "Sat - Sun scheduled downtime notification"


#### Expiration notification for  Sat - Sun scheduled downtime notification
![alt text][img4d]

[img4d]: ./images/downtime_expiry_1.png "Expiration notification for Sat - Sun scheduled downtime notification"


#### 7pm to 9am, Monday - Friday,  scheduled downtime configuration (recurring)

![alt text][img4e]

[img4e]: ./images/daily_downtime_schedule.png "7pm to 9am, Monday - Friday, scheduled downtime configuration (recurring)"

#### 7pm to 9am, Monday - Friday,  scheduled downtime notification

![alt text][img4f]

[img4f]: ./images/scheduled_downtime_2.png "7pm to 9am, Monday - Friday,  scheduled downtime notification"

#### Expiration notification for 7pm to 9am, Monday - Friday,  scheduled downtime notification

![alt text][img4g]

[img4g]: ./images/downtime_expiry_2.png "Expiration notification for 7pm 7pm to 9am, Monday - Friday,  scheduled downtime notification"

> **Side note:** After scheduling a downtime, review the dashboard to ensure it has been scheduled as necessary, as well as to check the status of the scheduled downtime. 
>
> #### Schedule Downtime Dashboard
> ![alt text][img4h]
>
> [img4h]: ./images/schedule_downtime_dashboard.png "Expiration notification for 7pm 7pm to 9am, Monday - Friday,  scheduled downtime notification"



-----
## Collecting APM Data:
-----

#### APM Dashboard
![alt text][img7a]

[img7a]: ./images/apm_board_1.png "Snapshot of APM Board"

#### Snapshot of APM Dashboard-Services"

![alt text][img7b]

[img7b]: ./images/apm_dashboard_1.png "Snapshot of APM Dashboard-Services"


#### Snapshot of APM Resource Stats
![alt text][img7c]

[img7c]: ./images/apm_resource_stats.png "Snapshot of APM Resource Stats"

#### Snapshot of APM - Individual Traces with Flame

![alt text][img7d]

[img7d]: ./images/individual_traces1.png "Snapshot of APM - Individual Traces with Flame"


#### Service vs. Resource <br/>
A **Service** is a **set of processes** that perform the same job (for example a web application, or a database). An example of a Service, in this technical exercise, is my flask web application (shown as **flask-app** in the image below). <br/>

A **Resource**, on the other hand, is a particular **action** for a service. In the case of a database, this could be a query. For example: ```SELECT * FROM users WHERE id = ?```. In web applications, resources could be canonical URLs (/users/status/) or handler functions/routes. On the APM interface, these resources can be found after clicking a particular service. <br/>

> _NB: More information can be found in the reference section_

#### Infrastructure and APM Metrics


![alt text][img8a]

[img8a]: ./images/apm_infra_1.png "Infrastructure and APM Metrics"

#### Snapshot of APM - Individual Traces with Flame

![alt text][img8b]

[img8b]: ./images/apm_infr_2.png "Snapshot of APM - Individual Traces"


[Link to APM and Infrastructure Metrics Dashboard](https://app.datadoghq.com/apm/search?cols=%5B%22core_service%22%2C%22log_duration%22%2C%22log_http.method%22%2C%22log_http.status_code%22%5D&from_ts=1542642466428&graphType=span_list&index=trace-search&live=true&query=env%3Add_docker&saved_view=6953&spanID=15764508887163998640&stream_sort=desc&to_ts=1542646066428&trace)

#### The Flask App
[Fully instrumented app](./app.py)<br/><br/>
**Note:** Instrumentation could be done by either:
- Installing ```ddtace``` and running the application as follows::

``` 
    pip install ddtrace
    ddtrace-run python <app_name>.py
```
**OR**
- Including a middleware in the script. <br/>

For my instrumentation, I used the Flask middleware and instrumented it as follows:
``` 
    from ddtrace.contrib.flask import TraceMiddleware
    instrumented_app = TraceMiddleware(app, tracer, service="flask-app", distributed_tracing=False) 
```


### Datadog - Creative Use:
I am presently working om a performance measurement application, and I can clearly see myself using Datadog to montior application usage as well as bottle necks in the system (when it launches). Also, remembering how long I had to wait in line at a restaurant on a Thanksgiving weekend, it would be interesting to use Datadog to find out about traffic at my favorite restaurants. Two extra benefits will be the predicitive aspect of Datadog that would analyze the pattern of traffic flow to make predictions that would:
- help clients in organising their schedules to get into the restaurant during a 'good' window
- help the restaurant owners decide when they need to assign more workers/get extra hands on deck.

###

------
### References
------
1. [Agent Commands](https://docs.datadoghq.com/agent/faq/agent-commands/?tab=agentv6)
2. [Building a custom check agent(Hands on instructions)](https://datadog.github.io/summit-training-session/handson/customagentcheck/)
3. [Creating and configuring custom metrics](https://docs.datadoghq.com/developers/write_agent_check/?tab=agentv6)
4. [Creating time boards](https://docs.datadoghq.com/api/?lang=python#timeboards)
5. [Distributed Tracing](https://docs.datadoghq.com/tracing/faq/distributed-tracing/)
6. [Downtimes](https://docs.datadoghq.com/monitors/downtimes/)
7. [Getting started with APM](https://docs.datadoghq.com/tracing/visualization/)
8. [Graphing](https://docs.datadoghq.com/graphing/)
9. [JSON Graphing Primer](https://docs.datadoghq.com/graphing/graphing_json/)
10. [Monitoring Docker - Datadog Training Site](https://datadog.github.io/summit-training-session/handson/monitordocker/)
11. [Monitoring Flask apps with Datadog](https://www.datadoghq.com/blog/monitoring-flask-apps-with-datadog/)
12. [Postgres SQL](https://docs.datadoghq.com/integrations/postgres/)
13. [Timeboards](https://docs.datadoghq.com/api/?lang=python#timeboards)
14. [Tracing Python Applications](https://docs.datadoghq.com/tracing/setup/python/)
15. [Tracing Docker Applications](https://docs.datadoghq.com/tracing/setup/docker/?tab=java#tracing-from-the-host)
16. [Rollup](https://docs.datadoghq.com/graphing/functions/rollup/)
17. [What is the Difference Between "Type", "Service", "Resource", and "Name"?](https://help.datadoghq.com/hc/en-us/articles/115000702546-What-is-the-Difference-Between-Type-Service-Resource-and-Name-)

