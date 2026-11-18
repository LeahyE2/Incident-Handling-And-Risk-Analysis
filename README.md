# 🔍 Log Analyzer

An interactive Python tool that scans **SSH/auth logs** and **Apache-style HTTP access logs** for signs of attack activity. It detects brute-force bursts and scanner-tool probing, looks up the country of origin for each offending IP, and produces a colour-coded terminal report, a bar chart, and a saved text report.

![Python](https://img.shields.io/badge/python-3.8%2B-blue?logo=python&logoColor=white)
![Interface](https://img.shields.io/badge/interface-interactive%20CLI-informational)
![Input](https://img.shields.io/badge/input-SSH%20%2B%20Apache%20logs-orange)

![Top attacker IPs chart](top_attackers.png)

---

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [How Detection Works](#how-detection-works)
- [Output](#output)
- [Sample Data](#sample-data)
- [Project Structure](#project-structure)
- [Configuration](#configuration)
- [Limitations](#limitations)

---

## Features

- **Brute-force detection**: flags any IP with 5 or more failed attempts inside a 10-minute sliding window, across both SSH and HTTP logs.
- **Scanner-tool detection**: flags IPs that request a sensitive path while using a known scanning tool.
- **GeoIP lookup**: reports the country for public IPs; private/reserved addresses are labelled `Private Address`.
- **Bar chart**: saves and displays a chart of failed attempts per incident (`top_attackers.png`).
- **Text report**: exports all findings to a file of your choosing.
- **Interactive menu**: arrow-key navigation with colour-coded terminal output.

---

## Requirements

- Python **3.8+**
- Third-party packages:
  - [`matplotlib`](https://matplotlib.org/): chart generation
  - [`geocoder`](https://geocoder.readthedocs.io/): GeoIP lookups (needs internet access)
  - [`questionary`](https://github.com/tmbo/questionary): interactive menu
- A display or GUI backend if you want the chart window to open (the PNG is saved either way)

---

## Installation

```bash
# 1. Clone the repository
git clone https://github.com/notyounotyouu/Log-Analyzer.git
cd Log-Analyzer

# 2. (Optional) create a virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install matplotlib geocoder questionary
```

---

## Usage

1. Put your log file in the project folder and name it `log_file.log` (or change `LOGFILE` at the top of the script).
2. Run the analyzer:

   ```bash
   python CA1.py
   ```

3. Choose an option from the menu:

| # | Option | What it does |
|:-:|--------|--------------|
| 1 | **Analyze Logs** | Parses the log file and detects incidents. **Run this first.** |
| 2 | **Show Brute Force Incidents** | Prints incidents grouped by IP with count, first/last timestamp and country |
| 3 | **Show Suspicious Tool Usage** | Lists IPs that used scanner tools against sensitive paths, with country |
| 4 | **Generate Bar Chart of Top Attackers** | Saves `top_attackers.png` and opens the chart |
| 5 | **Save Results to File** | Writes a text report (default `security_report.txt`, filename is prompted) |
| 6 | **Exit** | Quits the program |

> Options 2-5 use the results of option 1 and will ask you to analyze first if nothing has been loaded.

---

## How Detection Works

### Supported log formats

| Type | Looks like |
|------|-----------|
| SSH / auth log | `Mar 10 13:45:10 host1 sshd[1933]: Failed password for invalid user ftpuser from 198.51.100.66 port 7722 ssh2` |
| Apache-style access log | `198.51.100.66 - - [10/Mar/2025:13:45:12 +0000] "POST /images/logo.png HTTP/1.1" 200 1382 "https://example.com/" "Googlebot/2.1"` |

### What counts as a "failed" event

| Source | Counted as failed when… |
|--------|-------------------------|
| SSH | the line contains `Failed password` |
| HTTP | the status code is `4xx` or `5xx`, **or** the request contains a sensitive keyword, **or** the user agent contains a scanner tool |

### Brute-force incidents

Failed events are grouped per IP and sorted by time. A window opens at the first failure and extends while later failures fall within **10 minutes** of it. If the window holds **5 or more** failures, it is recorded as one incident (IP, count, first/last timestamp, country) and the next window starts after it, so incidents never overlap. One IP can therefore produce several incidents.

### Suspicious tool usage

An IP is flagged when a single HTTP request matches **both**:

| Condition | Values (case-insensitive substring match) |
|-----------|--------------------------------------------|
| Sensitive path | `admin`, `login`, `phpinfo`, `secret`, `passwd` |
| Scanning user agent | `sqlmap`, `curl`, `wget` |

---

## Output

| File | Description |
|------|-------------|
| `top_attackers.png` | Bar chart with one bar per brute-force incident (x: IP, y: failed attempts) |
| `security_report.txt` | Text report listing every brute-force incident and every suspicious-tool IP |

Example report layout:

```text
SECURITY ANALYSIS REPORT
============================================================

BRUTE FORCE INCIDENTS (300 found)
----------------------------------------
IP: 198.51.100.66
  Count: 6
  First: 2025-03-10T13:45:10
  Last: 2025-03-10T13:53:45
  Country: Private Address
...
SUSPICIOUS TOOL USAGE (n IPs)
----------------------------------------
IP: ...
```

---

## Sample Data

`log_file.log` is a 5,000-line sample containing both SSH and Apache entries from 10 March 2025. It uses reserved documentation IP ranges (such as `198.51.100.x` and `203.0.113.x`), which Python treats as private, so GeoIP results for the sample show `Private Address`. Use real logs with public IPs to see country lookups in action.

---

## Project Structure

```
Log-Analyzer/
├── CA1.py                          # Main script (parsing, detection, menu)
├── log_file.log                    # Sample input log (5,000 lines)
├── security_report.txt             # Example generated report
├── top_attackers.png               # Example generated chart
├── CA1 - Log File Analysis.docx    # Accompanying write-up
└── README.md
```

---

## Configuration

These values are set directly in `CA1.py`:

| Setting | Where | Default |
|---------|-------|---------|
| Input file | `LOGFILE` (top of file) | `log_file.log` |
| Auth-log year | `parse_auth_line()` | `2025` |
| Brute-force window / threshold | `analyze_logs()` | 10 minutes / 5 failures |
| Sensitive keywords and scanner tools | `parse_http_line()` and `analyze_logs()` | see [above](#suspicious-tool-usage) |

> SSH/auth logs don't include a year, so the script assumes **2025**. Change it in `parse_auth_line()` if your logs are from another year.

---

## Limitations

- **Slow on large logs with public IPs**: GeoIP lookups go through an external service and are made while parsing, so big logs can take a while.
- **GeoIP can fail**: results may read `Unknown` or `Lookup Failed` if the service is unreachable or rate-limited.
- **IPv4 only** for SSH/auth lines.
- **Single input file** and hard-coded thresholds; there are no command-line arguments.
- **Heuristic detection**: matching is by simple substrings, so legitimate use of `curl`/`wget` on an admin path will also be flagged.

---

