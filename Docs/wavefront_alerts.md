# Wavefront Alerts

## Creating Alerts

Wavefront has detailed documentation that may be found in [Create and manage alerts](https://docs.wavefront.com/alerts_manage.html)

In order to create alerts, it is important you
* Know the query you'll be using (eg get a chart or dashboard that visualizes the respective data)
* Know the distribution list that will be receiving alerts (Wavefront calls it **Alert Targets**, and there is a ["Cloud Team" alert target set](https://binz.wavefront.com/notificants/FykC4wwxnUBaYMB2) up as of May 2023

## What to alert on

Before you introduce new alert, consider answering what do I want to improve?

* will alert improve platform reliability (uptime/MTTR/MTBF)?
* will alert improve efficiency of the platform team?
* will alert help with inter-team communication?
* will alert help to control operational efficiency?
* ...

There are many events that can be looked at, so having a **goal** before setting up alert(s) helps to align the team and stakeholders on a reason to invest time in alert creation.

If you are not sure where to start with platform-related metrics, you may use below materials for inspiration

* [K8s monitoring checklist from Tanzu](https://tanzu.vmware.com/developer/guides/observability-k8s-monitoring-checklist/)
* [Platform team brainstorming on platform metrics](https://miro.com/app/board/uXjVMXgrVm0=/?moveToWidget=3458764553219423884&cot=14)
* [Platform team brainstorming on KPIs](https://miro.com/app/board/uXjVMXgrVm0=/?moveToWidget=3458764550886061148&cot=14)

## Alerting on missing data

Sometimes you might want to get notified in case metrics stop coming into the system. There is a Wavefront guide on this in [this article](https://docs.wavefront.com/alerts_missing_data.html) which roughly distinguishes between 2 cases: being notified when specific metric is no longer coming in (completely) and being notified when [individual time series stop sending (enough) data](https://docs.wavefront.com/alerts_missing_data.html#alerting-on-missing-data-in-individual-time-series)

To address [case when collection stopped after cert rotation](https://github.com/MTNNigeria/ng-tkgi-platform/issues/6#issuecomment-1550014690) and make sure the team is notified in similar cases, the following alerts have been created to alert if cluster-level metrics stop coming in for selected clusters 

- Production: [Too few metrics collected for wfproxysandbox](https://binz.wavefront.com/alerts/1684333535341)
- Sandbox: [Too few metrics collected for wfproxy](https://binz.wavefront.com/alerts/1684333366903)

For both alerts the following severity levels are set to notify team if number of metrics received in the clusters over a minute is less than expected
  * WARN < 3 (# of nodes in cluster e.g. _some_ nodes are not reporting metrics as expected)
  * SEVERE < 1 (eg _all_ nodes of a cluster are not reporting metrics as expected)
 

## How to create alerts

In order to create the alerts, 

1. Go to respective dashboard / edit the chart to see the query and chart layout

1. Go to Wavefront / Alerts / Create alert and use the same query / chart

1. Select the thesholds for Critical, Warn and (if applicable) Smoke

1. Select the "Cloud Team" as alert recipient

1. Select dashboard that would help to diagnose the alert (normally the same that you took for reference when creating a chart) and Runbook URL (if exists) and save it

![image](https://github.com/MTNNigeria/ng-tkgi-platform/assets/95610225/b08aa056-7d04-4d2a-97ea-c548da425dda)

## Alerts configured

Currently all alerts go to the "Cloud Team" target, which is the platform team.

We have created alerts to notify us in case cluster capacity reaches the
thresholds we defined in
[Cluster Health dashboard](https://binz.wavefront.com/dashboards/Cluster-Health):

* [High cluster CPU utilization alert](https://binz.wavefront.com/alerts/1684223761268)
* [High cluster memory utilization alert](https://binz.wavefront.com/alerts/1684223466462)

For both alerts, thresholds are
* SEVERE > 90% (corresponds to "red" color on dashboard)
* WARN > 70%  (corresponds to "orange" color on dashboard)
* SMOKE > 65%

We created this alert to tell us when an application endpoint's availability
dips below a certain threshold:

* [Endpoint Reachability is low](https://binz.wavefront.com/alerts/1684411274688)
  * WARN < 85%
  * SEVERE < 70%

**Notes and tips**

- When creating a query, query insights (lightbulb icon) might prompt you to consider using explicit `align` vs using `raw*` aggregation functions (eg `sum` vs `rasum` vs `align(sum(...`)
  - Use `rawsum` to see the smaller spikes and make use of individual recorded timeseries values
  - Aggregation (via `align`) might provide better performance if you are not interested in individual spikes that happen within the align time window
  - If you are using aggregation functions like `sum`, the results might often be aligned automatically, so consider explicit `align` to have more control (or be aware of [side effects of automated alignment](https://docs.wavefront.com/query_language_align_function.html#pre-alignment-side-effects))
- Consider using tags to group the alerts (the above alerts are tagged with `tkgi`)
- Consider using thresholds that match dashboard (for table charts, go to Color Mapping to see the thresholds)
![image](https://github.com/MTNNigeria/ng-tkgi-platform/assets/95610225/c833cf03-7d99-4526-9305-2f3346f307a3)

- I used 65% for `Smoke` level of alert
- You may need to refresh browser window (F5 / Ctrl-R) after saving a new alert, otherwise it does not always show up in Wavefront
