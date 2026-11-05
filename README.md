# PrivacyNexus-Agent 🛡️

[![Python](https://img.shields.io/badge/Python-3.x-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Linux-purple.svg)]()

**PrivacyNexus-Agent** is a modular, AI-driven Host-based Intrusion Detection System (HIDS) & Data Loss Prevention (DLP) endpoint security agent optimized for lightweight Linux environments. 

Developed by **Ovinda Sakun** as part of a BSc (Hons) Computer Networks Dissertation, this agent operates locally as a unified Security Information and Event Management (SIEM) and Endpoint Detection and Response (EDR) suite. It features a non-blocking CustomTkinter GUI and multi-threaded architecture to perform real-time security correlations.

---

## 🧠 Core Architecture & Code Mechanics

The agent is driven by a central **Security Decision Engine** that aggregates and correlates telemetry from two parallel machine learning pipelines.

### 1. Module A: Data Loss Prevention (DLP)
The DLP module is responsible for protecting sensitive documents on the filesystem.
* **Event Monitoring:** Utilizes `watchdog` to monitor user directories for file creation and modification events.
* **Extraction & Debouncing:** Extracts raw text from `.txt`, `.pdf` (via `PyPDF2`), and `.docx` (via `python-docx`) files. It implements a 2.0-second debounce logic to filter out redundant OS-level filesystem events and prevent excessive rescanning.
* **AI Classification:** Processes the extracted text using a pre-trained TF-IDF vectorizer and a Random Forest Classifier (`.joblib` models). Documents are scored and categorized (e.g., Financial, Credentials, Personal) based on content sensitivity.

### 2. Module B: Host-based Intrusion Detection (HIDS)
The HIDS module actively hunts for malicious behavioral patterns in running processes.
* **Process Scanning:** Continuously samples active system processes using `psutil`, intentionally bypassing root processes and known benign daemons (e.g., `dbus-daemon`, `Xorg`, `pipewire`) to reduce noise.
* **Dynamic Syscall Hooking:** Dynamically attaches to non-whitelisted processes using `strace` to capture a live, 0.5-second sequence of system calls.
* **ADFA-LD Alignment:** Maps common string-based system calls to numeric IDs (e.g., `read: 3`, `write: 4`) to align with the ADFA-LD benchmark standard.
* **Anomaly Detection:** Evaluates the captured sequence using a bigram (2-gram) TF-IDF vectorizer and a Random Forest Classifier (`.pkl` models) to determine the probability of a malware-like anomaly.

### 3. Security Decision Engine & GUI Dashboard
The decision engine coordinates the modules and surfaces insights to the user.
* **Multi-threaded Architecture:** Uses a Python `queue.Queue()` to safely pass DLP and HIDS events from background worker threads to the central engine.
* **Sliding-Window Correlation:** The engine maintains a 60-second sliding window of recently accessed sensitive files. If the HIDS module detects anomalous behavior (threat confidence > 0.7) *while* sensitive data is actively being accessed, it triggers a `CRITICAL_ESCALATION` alert (simulating process termination and admin alerting).
* **Audit Logging:** Persistently logs all security scans, threats, and escalations into a local SQLite database (`security_events.db`).
* **Interactive GUI:** Built with `CustomTkinter`, the dashboard features three tabs displaying real-time data classifications, live AI anomaly scores, active threats, hardware telemetry (CPU/RAM usage), and historical audit logs.

---

## 🧪 Testing & Validation Suite

The repository includes a comprehensive testing suite to verify the ML models and simulate attacks against the agent's monitoring pipeline.

* **`test_dlp_model.py`:** Unit test script that loads the `model1/` vectorizers and classifiers to verify inference capabilities on sample benign and sensitive text inputs.
* **`test_hids_model.py`:** Unit test script that loads the `model2/` artifacts to validate threat predictions against raw syscall sequences (e.g., normal desktop idle vs. aggressive anomaly).
* **`trigger_dlp.py`:** Simulates an extended 60-second malicious workload (spawning processes, unauthorized reads of `/etc/shadow`, and TCP network beaconing). Designed to provoke the HIDS module while giving the DLP module a sufficiently long window to trigger a correlation escalation.
* **`trigger_hids.py`:** Runs a rapid, condensed 15-second malicious syscall workload to explicitly trigger a positive threat classification within the HIDS module for demonstration purposes.

---

## ⚙️ Setup & Privileges

Because the HIDS module dynamically hooks into live processes using `strace`, **the agent requires elevated root privileges** to function securely and effectively in Linux environments.

### 1. System Requirements
Ensure your Linux distribution has Python 3, Tkinter, and strace installed:
```bash
sudo apt update
sudo apt install python3-tk strace
```

### 2. Python Dependencies
Install the required packages within your environment:
```bash
pip install -r requirements.txt
```

### 3. Running the Agent
Launch the agent with `sudo` to enable full HIDS capabilities and retain the environment variables for the GUI:
```bash
sudo -E python3 security_agent.py
```

---
*Developed for a BSc (Hons) Computer Networks Dissertation*
