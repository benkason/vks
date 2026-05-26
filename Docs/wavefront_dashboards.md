# Wavefront Dashboards

## Getting access to Wavefront

To request access to MTN Nigeria's TO instance, contact Ifeanyi Otudoh
<Ifeanyi.Otudoh@mtn.com>

## Cluster health dashboards

We created two dashboards:
+ [Cluster Health](https://binz.wavefront.com/dashboards/Cluster-Health)
+ [Cluster Health Drilldown](https://binz.wavefront.com/dashboards/Cluster-Health-Drilldown)

## Platform Overall Health - "North Star" metrics

We did an exercise as a team to determine metrics that could be overall
indicators of platform health.
The results were documented in Miro here: https://miro.com/app/board/uXjVMXgrVm0=/

We are monitoring the following metrics as part of our "North Star" for platform
health:
+ Percentage of nodes in the `Ready` state over the course of the dashboard's
    time window

## Cluster capacity

We are monitoring the following three metrics:
+ Memory utilization
+ CPU utilization
+ Storage utilization

These are displayed on the Cluster Health dashboard with different colors based
on their usage thresholds.
+ Green, up to 70% usage
+ Yellow, from 70%-90% usage
+ Red, from 90% usage
