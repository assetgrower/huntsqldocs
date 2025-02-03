# Hunt.io SQL Documentation

Welcome to the Hunt.io SQL Documentation for the web portal. This guide explains how to use the read-only SQL-like interface in the Hunt App at [https://app.hunt.io](https://app.hunt.io) for threat hunting and security event analysis. (For API-specific details, see [Hunt API Documentation](https://apidocs.hunt.io).)

---

## Table of Contents
1. [Introduction](#introduction)
2. [Hunt SQL Concepts](#hunt-sql-concepts)
3. [Using the Web Interface](#using-the-web-interface)
4. [SQL Query Syntax Overview](#sql-query-syntax-overview)
5. [Field Properties and Allowed Operators](#field-properties-and-allowed-operators)
6. [Datasets Overview](#datasets-overview)
7. [Time Stamps and Relative Time Analysis](#time-stamps-and-relative-time-analysis)
8. [Example Queries](#example-queries)
9. [Querying by IP, ASN, and Hosting Company](#querying-by-ip-asn-and-hosting-company)
10. [Advanced Query Examples and Nuances](#advanced-query-examples-and-nuances)
11. [Additional Resources](#additional-resources)
12. [Change Log](#change-log)
13. [Internal ToDo](#internal-todo)

---

## 1. Introduction
The Hunt.io SQL interface empowers threat hunters and analysts to explore a massive repository of security events in a way that best suits their investigative needs. Using this read-only SQL-like language you can select fields, filter results, group data, and leverage functions (such as `uniq()`, `min()`, and `max()`) to analyze events—from malware probes and certificate anomalies to open directory listings and phishing alerts.

---

## 2. Hunt SQL Concepts
- **Perform Threat Hunting:** Query and analyze events quickly to detect patterns or anomalies.

- **Analyze Historical Data:** Use relative time expressions to filter events and aggregation functions to determine activity periods.

- **Monitor Specific Indicators:** Focus on key indicators such as phishing URLs, certificate issues, or unusual file names.

- **Aggregate Data:** Group and summarize results to reveal trends or count unique events.

- **Query by IP, ASN, and Hosting Company:** Filter events based on specific IP addresses, IP ranges, ASNs, or hosting company domains.

---

## 3. Using the Web Interface
- **Accessing SQL:** Click the **SQL** option in the left navigation bar.

- **Samples Dropdown:** Select sample queries from the dropdown (top right of the query box) to get started.

- **Run Query:** Click **Run Query** to execute your query. A spinner indicates processing, and results appear under the **Results** tab.

- **Format Query:** Use the **Format Query** button to automatically reformat your query for readability.

- **Copy and Clear:** Use the copy icon to quickly copy your query, or click the trash can icon to clear the query box.

- **Autocomplete:** Autocomplete assists in finding specific field names as you type.

- **API Access:** For further details, refer to [Hunt API Documentation](https://apidocs.hunt.io).

---

## 4. SQL Query Syntax Overview
The interface supports a read-only SQL-like subset:

- **SELECT Clause:**  
  Specify the fields to return.
  ```sql
  SELECT ip, port
  FROM http
  ```
- **FROM Clause:**  
  Specify the dataset (e.g., `http`, `malware`, `certificates`).
- **WHERE Clause:**  
  Filter records using supported operators. Use `==` for equality and `gt` for "greater than":
  ```sql
  SELECT ip, port
  FROM http
  WHERE timestamp.day gt '2024-08-01'
  ```
- **GROUP BY Clause:**  
  Group results by one or more fields.
- **Aggregate Functions:**  
  Use functions such as `uniq()`, `min()`, and `max()` for aggregations.

> **Note:** Only `SELECT` queries are permitted. Data modifications (INSERT/UPDATE/DELETE) are not allowed.

---

## 5. Field Properties and Allowed Operators
Each field in a dataset includes metadata that indicates:
- **Sortable:** Fields that can be used in `ORDER BY` clauses.
- **Comparable:** Fields that support operators such as `==` and `gt`.
- **Regex Support:**  
  Many string fields (e.g., `hostname` in the `http` dataset or `header.raw`) support regex via the `RLIKE` operator, while simpler pattern matching can be performed with `LIKE`.

---

## 6. Datasets Overview

| **Dataset**                 | **Description**                                                                                                            | **Key Fields / Notes**                                                                                                     |
|-----------------------------|----------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------|
| **http**                    | Web scanning data for HTTP/HTTPS services.                                                                               | Banners, status codes, headers, timestamps.                                                                              |
| **protocol**                | Raw protocol scanning data from internet probes.                                                                         | IP, port, TTL, fingerprint.                                                                                                |
| **ssh**                     | Scan data from hosts with exposed SSH services.                                                                          | Host, port, banner, versions, key values.                                                                                |
| **malware**                 | Information on Command and Control (C2) servers tracked by Hunt.                                                         | Malware name, ASN information, IP, hostname, scan details.                                                               |
| **certificates**            | TLS/SSL certificate events with full issuer and subject details.                                                         | Subject common name, issuer details, timestamps. *(JA4X fields are defined here but are currently not accessible via SQL.)* |
| **open_directories**        | Data on exposed open directories, including hosts, file names, and directory structures.                                  | Hosts, file names, directory structures, file URLs.                                                                     |
| **open_directory_filenames**| File data from open directories that were not downloaded via AttackCapture.                                               | Full file names and URLs for pivoting.                                                                                   |
| **phishing**                | Records of phishing sites and URLs tracked by proprietary scanners.                                                      | URLs, timestamps, verdicts.                                                                                                |
| **honeypot**                | Data from devices scanning Hunt honeypots, including probe details.                                                      | Probe details, IP, HTTP requests.                                                                                        |
| **crawler**                 | Data obtained by Hunt URL crawlers; useful for discovering URLs not yet linked to phishing domains.                        | Discovered URLs, status, errors.                                                                                           |
| **urlx**                    | An experimental dataset—a collection of URLs for reconnaissance.                                                         | Experimental; used for recon; subject to change.                                                                         |

---

## 7. Time Stamps and Relative Time Analysis
Time stamps are essential for analyzing event data. Every dataset includes time stamp fields (e.g., `timestamp`, `timestamp.day`, `timestamp.hour`, `timestamp.week`) that allow both absolute and relative filtering.

**Default Time Range Settings:**  
- For most tables, if no explicit time filter is specified, the system defaults to the past **30 days**.
- For the **certificates** and **http** tables, the default is the past **7 days**.

You can override these defaults by specifying explicit time conditions. Querying data beyond these ranges is subject to your data quota; please contact support if you require extended historical data or special permissions.

**Relative Time Expressions:**  
Use `NOW` with allowed units (SECOND, MINUTE, HOUR, DAY, WEEK, MONTH, YEAR) to dynamically filter recent events:
```sql
SELECT ip, port, timestamp
FROM http
WHERE timestamp > NOW - 1 DAY
```

**Custom Time Ranges:**  
Combine conditions to set a custom range:
```sql
SELECT ip, port, timestamp
FROM malware
WHERE timestamp > NOW - 7 DAY
  AND timestamp <= NOW - 1 DAY
```

**Time Aggregation:**  
Aggregate time data to determine the first and last occurrence of events:
```sql
SELECT ip, min(timestamp) AS first_active, max(timestamp) AS last_active
FROM malware
GROUP BY ip
ORDER BY first_active
LIMIT 10
```

---

## 8. Example Queries
### Basic Examples
- **Select All Malware Probes:**
  ```sql
  SELECT *
  FROM malware
  ```
- **Query by Individual IP:**
  ```sql
  SELECT *
  FROM malware
  WHERE ip == '192.168.1.1'
  ```
- **Query by IP Range:**
  ```sql
  SELECT *
  FROM http
  WHERE ip LIKE '1.1.1.%'
  LIMIT 10
  ```

---

## 9. Querying by IP, ASN, and Hosting Company
- **By Individual IP:**
  ```sql
  SELECT *
  FROM malware
  WHERE ip == '192.168.1.1'
  ```
- **By IP Range:**
  ```sql
  SELECT *
  FROM malware
  WHERE ip LIKE '192.168.1.%'
  ```
- **By ASN:** *(Using a numeric ASN as an example)*
  ```sql
  SELECT *
  FROM malware
  WHERE asn.number == 12345
  ```
- **By Hosting Company:**  
  If available, filter events by hosting company domains.

---

## 10. Advanced Query Examples and Nuances

### Aggregation and Grouping
- **Query A: Malware Distribution by Name**
  ```sql
  SELECT malware.name, count(*) AS events
  FROM malware
  WHERE timestamp.day gt '2024-07-01'
  GROUP BY malware.name
  ORDER BY events DESC
  LIMIT 10
  ```
- **Query B: Aggregate Malware by ASN (Unique Malware Names)**
  ```sql
  SELECT asn.name, uniq(malware.name)
  FROM malware
  WHERE timestamp.day gt '2024-08-01'
  GROUP BY asn.name
  ```
- **Query C: Grouping by Certificate Common Name**
  ```sql
  SELECT subject.common_name, min(timestamp) AS first_seen, max(timestamp) AS last_seen, count(*) AS cert_count
  FROM certificates
  GROUP BY subject.common_name
  ORDER BY cert_count DESC
  LIMIT 10
  ```

### Regex (RLIKE) Examples
Demonstrate advanced pattern matching:
- **URL Pattern Matching in urlx Dataset:**
  ```sql
  SELECT *
  FROM urlx
  WHERE url RLIKE '^https:\/\/[0-9\.]+\/[a-z0-9]{16}\.php'
    AND timestamp.day gt '2024-11-01'
  ```
  **Explanation:**  
  - `^https:\/\/` ensures the URL starts with "https://".  
  - `[0-9\.]+` matches one or more digits or dots (an IP-like pattern).  
  - `\/[a-z0-9]{16}\.php` matches a slash followed by exactly 16 lowercase alphanumeric characters, ending with ".php".

- **Certificate Common Name Regex Matching:**
  ```sql
  SELECT ip, port, subject.common_name
  FROM certificates
  WHERE subject.common_name RLIKE 'Server$'
  LIMIT 10
  ```
  **Explanation:**  
  - The pattern `'Server$'` matches any certificate common name ending with "Server", useful for identifying server certificates.

- **Open Directories Example: Filtering PDF Files**
  ```sql
  SELECT file_name, file_url
  FROM open_directories
  WHERE file_name RLIKE '.*\.pdf$'
  LIMIT 10
  ```
  **Explanation:**  
  - `.*` matches any characters preceding the file extension.  
  - `\.pdf` matches the literal ".pdf" (with the dot escaped).  
  - `$` asserts that the match occurs at the end of the string.

### Testing Unsupported Operations
- **Arithmetic Expressions Not Supported:**  
  The following query attempts to add 100 to the `port` field, which is unsupported:
  ```sql
  SELECT *
  FROM certificates
  WHERE port + 100 > 500
  ```
  This query returns an error. Use standard comparison operators (`==`, `gt`) instead.

---

## 11. Additional Resources
- **Hunt SQL Blog Post:** [https://hunt.io/blog/sql](https://hunt.io/blog/sql)
- **Hunt SQL Features:** [https://hunt.io/features/hunt-sql](https://hunt.io/features/hunt-sql)
- **SQL Search API Documentation:** [https://apidocs.hunt.io](https://apidocs.hunt.io)
- **Support:** support@hunt.io

---

## 12. Change Log
- **11/01/2024:** Initial SQL interface launch in the Hunt App.
- **12/15/2024:** API documentation released.
- **01/10/2025:** Updated SQL documentation with advanced query examples, relative time syntax, and URLx query enhancements.
- **[Current Date]:** Refined documentation with additional queries and improved user experience details.

---

## 13. Internal ToDo
- Add table names to self-writing documentation.
- Get new files from Courtney.
- Add IOC hunter data.
- Test IP ranges and include discussions on querying by ASN and hosting company domains.
- Add a section on how to use the internal documentation.
- Adjust header order as needed.
- Enhance instructions on querying by IP ranges, ASN, and hosting company domains.
