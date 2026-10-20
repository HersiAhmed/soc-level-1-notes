# Elastic Stack (ELK)

ELK started as a tool for storing and searching large amounts of data and monitoring app performance. SOC teams now use it as a SIEM. It's open source, which is a big part of its popularity.

## The four components

| Component | Job | Splunk equivalent |
|---|---|---|
| **Beats** | Lightweight agents that ship data | Forwarder |
| **Logstash** | Parses and normalizes data | Indexer (processing) |
| **Elasticsearch** | Stores and searches data (JSON) | Indexer (storage) |
| **Kibana** | Web interface: search, visualize, dashboards | Search Head |

Flow: **Beats → Logstash → Elasticsearch ⇄ Kibana**

### Beats

Each beat does one job:

| Beat | Ships |
|---|---|
| Filebeat | Log files |
| Winlogbeat | Windows event logs |
| Packetbeat | Network traffic |
| Metricbeat | Performance metrics |
| Auditbeat | Audit data |
| Heartbeat | Uptime |
| Functionbeat | Cloud data |

### Logstash config

Three sections: **Input** (where data comes from) → **Filter** (parse and normalize) → **Output** (where it goes, usually Elasticsearch). I remember it as *in → clean → out*.

## Kibana Discover tab

This is where analysts spend most of their time.

- **Index pattern / data view:** which set of logs you're looking at. Check this first.
- **Search bar:** KQL queries
- **Time filter:** must cover when the logs actually happened. The lab logs were from January 2022, so a default recent range showed nothing.
- **Timeline chart:** event counts over time. **Spikes are worth investigating.** Click a bar to zoom into that window.
- **Fields pane:** click a field for its top 5 values. ➕ filters to that value, ➖ excludes it.
- **Table view:** pick only the fields you care about to cut the noise.

**Investigating a spike:** click the spike bar to zoom in, then click `Source_ip` in the fields pane. If one IP dominates the top values, it's likely the cause. Then check `UserName` the same way.

## KQL (Kibana Query Language)

Much simpler than SPL: no pipes, and Kibana counts hits for you.

**Free text search** matches the term in any field:

```
"United States"
```

KQL matches **whole terms**. `United` alone returns nothing, but the wildcard works:

```
United*
```

**Field-based search** is more precise:

```
Source_ip : 203.0.113.10 AND UserName : alice
```

**Logical operators**

| Query | Effect |
|---|---|
| `"United States" AND "Virginia"` | Both must match (fewer results) |
| `"United States" OR "England"` | Either matches (more results) |
| `"United States" AND NOT ("Florida")` | Excludes Florida |

**Failed VPN attempts**

```
action : failed
```

## SPL vs KQL

| Goal | SPL | KQL |
|---|---|---|
| Events from an IP | `index=x Source_ip="1.2.3.4"` | `Source_ip : 1.2.3.4` |
| Exclude a value | `Source_Country!="France"` | `NOT Source_Country : France` |
| Count | `\| stats count` | Read the hit count |

## Visualizations

From Discover, click a field → **Visualize**. Drag in a second field to **correlate** them (e.g. Source IP vs Source Country) as a pie chart or table.

The useful thing about a correlation table is how fast outliers jump out. In the lab, one IP from a different country had far more events than anything else. That's exactly what you'd dig into next.

**Failed connections table:** filter `action : failed`, then build a table with `UserName` and `Source_ip`. Lots of failures from one user or IP points to brute force.

Always **Save** with a title and description, and **add to library**, or you can't reuse it.

## Dashboards

A dashboard is a board of saved searches and visualizations. **Dashboard → Create new → Add from library →** arrange panels → **Save**.

The full workflow:

> **Discover** (search) → **Visualize** (chart/table) → **Save** → **Dashboard** (one view)

It's the same thinking as Splunk (search → `stats` → visualization → dashboard), just with different buttons.
