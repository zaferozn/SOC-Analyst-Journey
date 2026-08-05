# Introduction to Splunk

Splunk is a SIEM platform used to collect, index, search, and analyze logs from endpoints, network devices, applications, databases, cloud services, and security tools.

## Core Components

- **Forwarder:** Collects logs and sends them to Splunk.
- **Indexer:** Processes and stores logs as searchable events.
- **Search Head:** Allows analysts to search data using SPL and create dashboards or visualizations.

## Splunk Interface

The home screen includes the **Splunk Bar**, **Apps Panel**, data-upload shortcuts, documentation links, and customizable dashboards. The **Search & Reporting** app is used for log analysis.

## VPN Log Ingestion

A newline-delimited JSON VPN log file was uploaded through:

**Add Data → Upload → Select Source → Source Type → Input Settings → Review → Done**

The logs were stored in the `VPN_Logs` index, and the time range was set to **All time**.

## Basic SPL Searches

```spl
index=VPN_Logs
| stats count
```

```spl
index=VPN_Logs
| spath
| search UserName="Maleena"
| stats count
```

```spl
index=VPN_Logs
| spath
| search Source_ip="107.14.182.38"
| stats values(UserName) as UserName count
```

The `spath` command extracts fields from JSON events when they are not automatically displayed.

## Conclusion

This room introduced Splunk architecture, interface navigation, JSON log ingestion, field extraction, and basic VPN log analysis using SPL.
