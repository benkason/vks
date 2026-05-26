# External Services Monitoring - Proof of Concept

## Endpoints to monitor

Lekan has provided us the following list of external services that we can
monitor:

- TCP: 10.1.98.165:27018
- TCP: 10.1.76.200:27018
- TCP: Inbound bearer type 10.200.22.176:10000 (SMSC)
- TCP: Inbound bearer type 172.25.32.180:5678 (USSD)
- HTTP: http://10.197.88.66:8913/Air

We have also added the following for testing:

- TCP: github.com:443
- HTTP: https://github.com

## Cluster to monitor from

For testing purposes, we will monitor these endpoints using the Telegraf agents
on the `petclinic` cluster in Sandbox.
These agents run on each worker node in the cluster, so we will get reachability
data from all of them.

## What will be monitored

We will monitor the reachability of all the endpoints, as well as the latency
when those endpoints are reachable.

For HTTP endpoints, we will also monitor the status codes that are received.
