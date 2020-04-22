# Dynatrace-HikariCP-Plugin
Hikari Connection Pool JMX Plugin for Dynatrace


**Metrics**

| Name | Description |
| --- | --- |
| **Active Connections** | Connections currently in use |
| **Idle Connections** | Idle connection in the pool |
| **Total Connections** | Total connections in the pool |
| **Threads Awaiting Connection** | Self-described |

**Aggregation**
For all Metrics the Aggregation is SUM. This can be changed in the Json file. 


**Splitting**
In this plugin the spilt is happening per *type* for all metrics. This usually displays a `Pool` and a `PoolConfig` per Pool.

👉 In order to get these metrics, you must set the pool property `registerMbeans=true`. You can find more information at the [HikariCP](https://github.com/brettwooldridge/HikariCP/wiki/MBean-(JMX)-Monitoring-and-Management) repository.

Cloned from: https://github.com/chulderman/Dynatrace-HikariCP-Plugin
