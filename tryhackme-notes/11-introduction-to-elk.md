# Introduction to ELK

## Overview

The Elastic Stack, also known as ELK, is used to collect, process, search, and visualize large volumes of data. Although it is not a traditional SIEM, many SOC teams use it for log monitoring and security investigations.

## Core Components

### Elasticsearch

Elasticsearch stores JSON-formatted data and provides fast searching, analytics, and correlation capabilities.

### Logstash

Logstash receives data from different sources, processes or normalizes it, and sends it to another destination.

A Logstash configuration contains:

- **Input:** Defines where the data comes from.
- **Filter:** Parses or normalizes the data.
- **Output:** Defines where the processed data is sent.

### Beats

Beats are lightweight agents that collect and transfer specific types of data.

Examples:

- **Winlogbeat:** Windows event logs
- **Packetbeat:** Network traffic
- **Filebeat:** Log files

### Kibana

Kibana is the web interface used to search, investigate, visualize, and present data stored in Elasticsearch.

## ELK Data Flow

```text
Endpoints → Beats → Logstash → Elasticsearch → Kibana
```

Beats collect the data, Logstash processes it, Elasticsearch stores it, and Kibana displays it.

## Kibana Discover

The Discover tab is the main workspace for log analysis. SOC analysts use it to:

- Review individual events
- Select a data view
- Search logs
- Apply field-based filters
- Adjust the time range
- Identify unusual activity spikes
- Create tables using relevant fields

The `vpn_connections` data view was used to investigate VPN activity.

## Kibana Query Language

KQL is used to search data in Kibana.

### Free-Text Search

```text
"United States"
```

A wildcard can search for partial terms:

```text
United*
```

### Logical Operators

```text
"United States" AND "Virginia"
```

```text
"United States" OR "England"
```

```text
"United States" AND NOT "Florida"
```

### Field-Based Search

```text
Source_ip: 238.163.231.224 AND UserName: Suleman
```

Field-based searches are more precise because they target specific normalized fields.

## Visualizations

Kibana can present data as:

- Tables
- Pie charts
- Bar charts
- Time-based charts

For failed VPN connections, the data was filtered using:

```text
action: failed
```

The `UserName` and `Source_ip` fields were used to identify the users and IP addresses involved in failed connection attempts.

## Dashboards

Dashboards combine saved searches and visualizations into a single monitoring view.

Basic process:

1. Create and save searches or visualizations.
2. Open the Dashboard section.
3. Select **Add from Library**.
4. Add the required objects.
5. Arrange and save the dashboard.

## SOC Relevance

This room demonstrated how a SOC analyst can use ELK to:

- Search VPN logs
- Filter suspicious activity
- Investigate failed connections
- Correlate usernames and source IP addresses
- Identify spikes in event activity
- Build visualizations and dashboards

## Key Takeaway

ELK provides the main capabilities required for log investigation: collection, normalization, storage, searching, visualization, and dashboard creation. For a SOC analyst, the most important skills are using Kibana Discover, writing accurate KQL queries, applying filters, and interpreting the resulting events.
