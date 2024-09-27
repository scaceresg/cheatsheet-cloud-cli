# DataDog

**Contents**:

- [DataDog](#datadog)
- [What is DataDog?](#what-is-datadog)
- [Dashboards](#dashboards)
- [Monitors](#monitors)
  - [Configure Monitors](#configure-monitors)
  - [Choose your monitor type](#choose-your-monitor-type)
    - [Host](#host)
    - [Metric](#metric)
    - [Anomaly](#anomaly)
    - [APM](#apm)
    - [Other Monitor Types](#other-monitor-types)
- [Logs](#logs)
  - [Explore Logs](#explore-logs)


**Source**: https://docs.datadoghq.com/getting_started/

# What is DataDog?

DataDog is an **Observability platform that supports every phase
of software development**

DataDog products:

| Category         | Examples                                               |
|------------------|--------------------------------------------------------|
| Development      | * Highlight code vulnerabilities in your text editor or|
|                  | on GitHub with **Code Analysis**                       |
|                  | * Facilitate a remote pair-programming session with    |
|                  | **CoScreen**                                           |
| Testing          | * Block faulty code from deployment to production with |
|                  | **Quality Gates**                                      |
|                  | * Simulate users around the globe to test your web app,|
|                  | API or mobile application with **Synthetic Monitoring**|
| Monitoring       | * Ingest *logs, metrics, events and network traces*    |
|                  | with granular control over processing, aggregation,    |
|                  | and *alerting*                                         |
|                  | * Assess host performance with **Continuous Profiler** |
|                  | * Assess application performance with **Application**  |
|                  | **Performance Monitoring (APM)**                       |
| Troubleshooting  | * Manage *errors and incidents*, summarising issues and|
|                  | suggesting fixes                                       |
|                  | * Measure user churn and detect user frustration with  |
|                  | **Real User Monitoring**                               |
| Security         | * Detect threats and attacks with **DataDog Security** |

# Dashboards

Contain graphs with real-time performance metrics and health of 
systems and applications within an organisation.

# Monitors

Provide alerts and notifications based on metric threshold, 
integration availability, network endpoints, etc.

* Use any metric reporting to DataDog

* Set up multi-alerts by device, host, and more

* Use `@` in alert messages to direct notifications to the right 
people

* Schedule downtimes to suppress notifications for systms 
shutdowns, off-line maintenance and more

## Configure Monitors

* **Define the search query**:
  
  Construct a query to count events, measure metrics, group by 
  dimensiones

* **Set alert conditions**: 
  
  Define alert and warning thresholds, evaluation time frames, 
  and configure advanced alert options

* **Configure notifications and automations**:
  
  Write a custom notification title and message with variables.
  Choose how notifications are sent to your teams (email, Slack, 
  or PagerDuty). Include workflow automations or cases in the 
  alert notification

* **Define permissions and audit notifications**:

  Configure granular access controls and designate specific roles
  and users who can edit a monitor. Enable audit notifications to 
  alert if a monitor is modified.

## Choose your monitor type
  
### Host

Check if one or more hosts are reporting to DataDog

Every DataDog Agent reports a service check called 
`datadog.agent.up` with the status `OK`

* Include/Exclude host name or tag or choose `All Monitored Hosts`
  
  For example:

  - Include all hosts with the tag `env:prod`:
  
    * Include: `env:prod`
    
    * Exclude: Leave empty
  
  - Include all hosts except hosts with the tag `env:test`
    
    * Include: `All Monitored Hosts`
    
    * Exclude: `env:test`

* Set alert conditions:
  
  - Check alert: if a host stops reporting for a given amount 
  of time
  
  - Cluster alert: tracks if some percentage of hosts have 
  stopped reporting for a given amount of time

### Metric
  
Compare values of a metric with a user-defined threshold

* Choose the detection method:
  
  - Threshold
  
  - Change
  
  - Anomaly
  
  - Outliers
  
  - Forecast

* Define the metric: See the **Metrics** section in DataDog
  
  | Step                             | Req | Default | Example      |
  |----------------------------------|-----|---------|--------------|
  | Select a metric                  | Yes | None    | `system.cpu.user`|
  | Define the `from`                | No  | Everywhere| `env:prod` |
  | Specify metric aggregation       | Yes | `avg by`| `sum by`     |
  | Group by                         | No  | Everything| `host`     |
  | Specify monitor query aggregation| No  | `average`| `sum`       |
  | Evaluation window                | No  | `5 minutes`| `1 day`   |

  Definitions:

  - Average: `avg()`
  
  - Max: `max()`
  
  - Min: `min()`
  
  - Sum: `sum()`
  
  - Percentile(pXX): `percentile`
  
  - Evaluation window: `5 minutes`, `15 minutes`, `1 hour` or 
  `custom`

* Set alerts:
  
  - `above`
  
  - `above or equal to`
  
  - `below`
  
  - `below or equal to`
  
  - `equal to`
  
  - `not equal to`
  
### Anomaly

Detect anomalous behaviour for a metric based on historical data.

* After defining the metric, the anomaly detection monitor 
provides two preview graphs:

  - Historical View
  
  - Evaluation Preview

* Set alert conditions:
  
  - Anomaly detection: `above`, `below` or `above or below` a 
  metric

  - Trigger window: Time for the metric to be anomalous
  
  - Recovery window: Time for the metric to no longer be 
  considered anomalous

* Advanced options:
  
  - Deviations: Width of the gray band
  
  - Algorithm: Anomaly detection algorithm: `basic`, `agile` or
  `robust`

  - Seasonality: `hourly`, `daily` or `weekly`
  
  - Daylight savings
  
  - Rollup
  
  - Thresholds: Percentage of points that need to be anomalous
  for alerting, warning and recovery

### APM

Monitor Application Performance M metrics or trace queries

* Select monitor scope: 
  
  - Primary tags, service and resource

* Set alert conditions:
  
  - Threshold alert
  
  - Anomaly alert

### Other Monitor Types

* **Audit Trail**: Alert when a specified type of audit log 
exceeds a user-defined threshold over a given period of time

* **Change Alert**: Alert when the absolute or relative value 
changes against a user-defined threshold over a given period of 
time

* **CI**: Monitor CI pipelines and tests data gathered by DataDog

* **Cloud Cost**: Monitor cost changes associated with cloud 
platforms

* **Composite**: Alert on an expression combining multiple 
monitors

* **Database Monitoring**: Monitor query execution and explain 
plan data gathered by DataDog

* **Error Tracking**: Monitor issues in your applications 
gathered by DataDog

* **Event**: Monitor events gathered by DataDog

* **Forecast**: Alert when a metric is projected to cross a
threshold

* **Integration**: Monitor metric values or health status from
a specific integration

* **Live Process**: Check if one or more processes are running
on a host

* **Logs**: Alert when a specified type of log exceeds a 
user-defined threshold over a given period of time

* **Network**: Check the status of TCP/HTTP endpoints

* **Network Performance**: Set alerts on your network traffic

* **Netflow**: Monitor flow records from your NetFlow-enabled 
devices

* **Outlier**: Alert on members of a group behaving differently
than the others

* **Process Check**: Watch the status produced by the process.up
service check

* **Real User Monitoring**: Monitor real user data gathered by
DataDog

* **Service Check**: Monitor the status of arbitrary custom
checks

* **SLO Alerts**: Monitor your SLO's error budget and burn rate

* **Synthetic Monitoring**: Monitor metric values or test status
from Synthetic test runs

* **Watchdog**: Get notified when Watchdog detects anomalous 
behaviour

# Logs

DataDog Log Management, Logs, is used to collect logs across
multiple logging sources, such as servers, containers, cloud
environments, applications or existing log processors and 
forwarders.

## Explore Logs

* Search and filter:
  
  - Log Search Syntax:

    2 types of terms:

    * Single term: `test` or `hello`
    
    * Sequence: `"hello dolly"`

    Combine multiple terms into a complex query:

    * Operator `AND`: Intersection
    
    * Operator `OR`: Union
    
    * Operator `-`: Exclusion
  
    Full-text search:

    * Single term: `*:hello` (all log attributes for the term
    `hello`), `*:hello*` (all log attributes that start with
    `hello`; e.g., `hello_world`)

    * Multiple terms: ``