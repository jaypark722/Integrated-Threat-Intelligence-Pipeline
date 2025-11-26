# Threat Intelligence Pipeline: OpenCTI ➜ Splunk ➜ Zeek/Snort

## Overview

This project demonstrates a **threat intelligence pipeline** leveraging **OpenCTI** to feed **Indicators of Compromise (IOCs)** into detection platforms such as **Splunk**, **Zeek**, and **Snort**. 

It walks through the process of:
- Creating fake IOCs in OpenCTI
- Streaming those IOCs into Splunk
- Detecting simulated malicious activity using Snort and Zeek
- Visualizing and alerting on IOC matches within Splunk
- Sending sightings back to OpenCTI

The entire setup is containerized using **Docker** to maintain portability and reproducibility.

---

## Installation & Setup

### Docker-Based Lab Environment

This lab uses **Docker** containers to host each of the following components:
- **Splunk** (with OpenCTI Add-on)
- **OpenCTI** (Threat Intelligence Platform)
- **Zeek** (Network Traffic Analyzer)
- **Snort** (Intrusion Detection System)
- Two Linux VMs:
  - **Attacker VM** – Simulates malicious traffic.
  - **Target VM** – Monitored by Zeek/Snort.

---

### 1. Splunk Installation

To enable integration with OpenCTI, Splunk must run in its **own container**.

> In this repository, refer to the `splunk/` folder for the correct `docker-compose.yml`.

> **Important Note:** The latest Splunk images may cause **KV store errors** which break the OpenCTI indicator dashboard.  
> **Solution:** Use **Splunk version `9.0.5`**, which mitigates this issue.

---

### 2. OpenCTI Installation

OpenCTI also runs in a Docker container.

> Refer to the `OpenCTI/` folder for:
> - `docker-compose.yml`
> - `.env` (must be configured accordingly)

Once the containers are up and running, OpenCTI will be accessible via `http://<your-local-ip>:8080`.

---

### 3. OpenCTI Add-on for Splunk

After Splunk is up and running:

1. Install the **OpenCTI Add-on** from the Splunk UI.
2. **Issue:** The add-on requires an HTTPS URL, but OpenCTI is typically served over HTTP.
3. **Workaround:** Manually configure the add-on inside the container:

```bash
docker exec -it splunk /bin/bash
sudo vi etc/apps/TA-opencti-add-on/local/ta_opencti_add_on_settings.conf
```

Add the following manually:

```ini
[proxy]
proxy_password = ******

[additional_parameters]
opencti_api_key = <OPENCTI API KEY>
opencti_url = http://<LOCAL SERVER IP>:8080
```

This establishes the integration between Splunk and OpenCTI.

### 4. Create Indicators in OpenCTI
From the OpenCTI UI:

Navigate to "Observables" > "Indicators"
Add fake IOCs such as:
IPv4 addresses
Domains
URLs

These will serve as the malicious artifacts for detection.

### 5. Create a Live Stream in OpenCTI
To enable data flow to Splunk:

Go to Settings > Data Sharing > Live Streams

Create a new stream filtering on:
ipv4
domain
url
indicator

Start the stream and copy the Stream ID

### 6. Configure Splunk Input for OpenCTI Stream
In the Splunk UI:

Create a new Input under the OpenCTI Add-on

Enter the Stream ID obtained earlier

You should now see the OpenCTI Dashboard in Splunk, reflecting the indicators you've created.

### 7. Install Snort & Zeek
On your Target VM:

Install Snort and Zeek
Configure rules to detect IOCs (e.g., fake C2 IPs/domains)

On the Attacker VM:
Simulate traffic using:
curl to fake C2 IP
nslookup to fake domains
nmap to simulate port scans

### 8. Configure Splunk Forwarding
Ensure Snort and Zeek logs are forwarded to Splunk:

Set up Universal Forwarder or syslog ingestion.
Make sure to create appropriate indexes in Splunk before ingestion.

## Demonstration of Pipeline
Here's the order in which the lab demonstrates IOC detection and correlation:

### 1. OpenCTI Threat Intelligence with Fake IOCs
![alt text](Screenshots/image-14.png)
![alt text](Screenshots/image.png)

### 2. Splunk Query Results Matching IOCs
Matches on fake C2 IPs
![alt text](Screenshots/image-3.png)

DNS queries to fake domains
![alt text](Screenshots/image-1.png)
![alt text](Screenshots/image-13.png)


### 3. Snort or Zeek Log Samples with IOC Hits
![alt text](Screenshots/image-4.png)

### 4. OpenCTI Sighting Created from Splunk
__Creating Alerts__


![alt text](Screenshots/image-15.png)
![alt text](Screenshots/image-16.png)

__OpenCTI Sightings__
![alt text](Screenshots/image-10.png)
![alt text](Screenshots/image-11.png)
![alt text](Screenshots/image-12.png)

## Conclusion
This lab demonstrates a working end-to-end threat intelligence pipeline:

OpenCTI serves as the central IOC management platform

Splunk ingests threat intelligence and logs from detection tools

Zeek and Snort provide low-level detection of malicious traffic

Sightings are pushed back into OpenCTI for threat context

This approach provides real-time detection, correlation, and threat enrichment across multiple platforms in a modular, containerized environment.

***BONUS***

# Caldera

## Overview

**Caldera** is an open-source cybersecurity platform developed by MITRE and designed to automate adversary emulation, red teaming, and defensive testing. It provides a flexible framework where security teams can simulate real-world attack techniques—based on the MITRE ATT&CK framework—against their environments to evaluate detection and response capabilities. By using plug-ins and customizable agents, Caldera enables users to test specific tactics, techniques, and procedures (TTPs) in a controlled way, helping organizations identify gaps, strengthen defenses, and continuously improve their security posture without requiring manual execution of every test.

## Installation

```bash
git clone https://github.com/mitre/caldera.git --recursive
cd caldera
docker build --build-arg WIN_BUILD=false . -t caldera:server
docker run -p 8888:8888 caldera:server --insecure
```

**Credentials**: `admin` / `admin`

## The Custom Ability Stack

### Initial Discovery (T1057, T1082, T1016)

- **T1057** | Process Discovery via `ps >> /tmp/loot.txt; ps aux >> /tmp/loot.txt`
- **T1082** | System Discovery (custom) via `uname -a ; lsblk ;`
- **T1016** | Network Discovery via `ip addr ; ip neigh ; netstat ;`

### Credential Access (T1003, T1552)

- **T1003** | OS Credentials Dumping via `/etc/passwd`, `/etc/shadow`
- **T1552** | Discover Private SSH keys via `find / -name id_rsa`

### Exfiltration (T1048)

- **T1048** | Exfiltrate Data HTTP via `curl`

## The Caldera Components

### Agent

A lightweight software component that runs on a target system and executes commands sent by the Caldera server. It acts like a simulated attacker's foothold, allowing the platform to carry out various attack techniques safely for testing purposes.

![alt text](Screenshots/Caldera/image.png)
![alt text](Screenshots/Caldera/image-1.png)

*An agent and its related script running on the target*

### Ability

A specific action or tactic that an agent can perform, such as collecting files, creating processes, or escalating privileges. Each ability is mapped to a MITRE ATT&CK technique, making it easy to simulate realistic adversary behavior.

### Adversary

A predefined or custom profile that combines multiple abilities to mimic the behavior of a real-world attacker. Adversaries allow teams to simulate complex attack campaigns without manually triggering each ability.

![alt text](Screenshots/Caldera/image-2.png)
![alt text](Screenshots/Caldera/image-3.png)

*The custom Ability stack and the detailed Discover Private SSH Key ability*

### Operation

The execution of an adversary plan against one or more agents. It orchestrates the selected adversary's abilities in a controlled sequence, enabling security teams to assess how well their defenses detect and respond to attacks.

![alt text](Screenshots/Caldera/image-4.png)

*The running Operation and its Ability stack*