# SSH Cracker Project

---

## Overview

The **SSH Cracker** project is an educational exercise where students implement Python scripts that test SSH credentials against a target host. Two versions are provided for learning progression:

* `ssh_brute.py` — a basic, sequential brute-force implementation aimed at teaching SSH connection handling and error management.
* `advanced_ssh_brute.py` — an advanced implementation that adds multithreading, configurable password generation, retry/backoff logic, and a job queue for controlled concurrency.

This repository emphasizes defensive learning: understanding how credential-guessing attacks work so students can design better defenses, logging, and rate-limiting in real systems.

> **Critical:** Only run these tools against hosts and accounts you own or have written authorization to test. Unauthorized access attempts are illegal and unethical.

---

## Learning Objectives

Students who complete this project will:

* Learn how the SSH protocol can be used to authenticate and how authentication failures are handled.
* Gain experience with the `paramiko` SSH client library for establishing and closing SSH sessions in Python.
* Understand socket-level network errors and how to handle timeouts, connection refusals, and transient network failures.
* Learn to design safe concurrency with `threading` and `queue` (producer/consumer pattern) to avoid overwhelming a target.
* Implement retry/backoff logic and logging for robust tooling.
* Appreciate ethical and legal constraints and how to design defensive mitigations (rate limiting, account lockout, multifactor auth).

---

## Features (Suggested)

* Sequential brute-force script (`ssh_brute.py`) that reads username(s) and password list and attempts SSH auth one-by-one.
* Advanced multithreaded scanner (`advanced_ssh_brute.py`) with:

  * Worker thread pool
  * Job queue to feed username-password pairs
  * Configurable retry/backoff and connection timeout
  * Optional password generator (small scoped) for classroom experiments
  * Graceful shutdown and detailed result reporting
* Colored console output for readability (via `colorama`) and an option to save results to JSON/CSV.

---

## Requirements

* Python 3.8+
* `paramiko` — SSH client
* `colorama` — colored CLI output
* Optional: `tqdm` for progress display

Example `requirements.txt`:

```
paramiko
colorama
tqdm
```

---

## Safety & Legal Notice

This project is strictly for educational and authorized security testing only. Performing credential-guessing attacks against systems, services, or accounts without explicit written permission is illegal in many jurisdictions and may result in criminal prosecution.

Before running any tests, obtain explicit authorization and preferably perform testing in an isolated lab network or using intentionally vulnerable VMs. Keep experiments small in scope (short password lists, conservative thread counts) to avoid causing service disruption.

---

## Installation

1. Clone the repository:

```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
```

2. Create and activate a virtual environment (recommended):

```bash
python3 -m venv venv
source venv/bin/activate  # Linux / macOS
venv\\Scripts\\activate  # Windows (PowerShell)
```

3. Install dependencies:

```bash
pip install -r requirements.txt
```

---

## Usage (Examples)

**Sequential mode** (basic):

```bash
python3 ssh_brute.py --host 192.168.1.10 --user testuser --wordlist small-words.txt --timeout 5
```

**Advanced mode** (multithreaded):

```bash
python3 advanced_ssh_brute.py --host 192.168.1.10 --users users.txt --wordlist passwords.txt --threads 10 --timeout 4 --retries 2 --backoff 2
```

Recommended conservative defaults for classroom/lab usage:

* `--threads`: 4–10
* `--timeout`: 3–5 seconds
* `--retries`: 1–3
* `--backoff`: exponential or linear small delays to avoid flooding

---

## High-level Design (Safe Pseudocode)

This project intentionally provides **design-level pseudocode** rather than line-by-line brute-force loops to keep the exercise educational and reduce misuse risk.

```
# Producer: read usernames and password candidates and enqueue (user, password) jobs
open username_source
open password_source
for each user in usernames:
    for each password in passwords:
        job_queue.put((user, password))

# Consumer workers: run in threads
while not job_queue.empty():
    user, password = job_queue.get()
    try:
        attempt SSH connection with paramiko using timeout
        if auth succeeds:
            record success and optionally stop further attempts for that user
    except transient network error:
        if retries left: re-enqueue job with backoff
    finally:
        job_queue.task_done()
```

Notes:

* Limit retries and worker count to avoid DoS-like behavior.
* Use explicit timeouts and handle `paramiko.ssh_exception` classes carefully.

---

## Defensive Topics & Extensions

Use this project to teach secure practices and mitigations:

* Demonstrate how enabling account lockout, rate limiting, or MFA stops simple credential guessing.
* Show how logging and intrusion detection can surface brute-force behavior.
* Replace password storage demos with strong KDFs (e.g., Argon2) and salted hashes.
* Implement a safe lab harness (local SSH server in a container or VM) for repeatable classroom tests.

---

## Troubleshooting

* **Connection refused / no route to host:** Verify the host is reachable and that the SSH daemon is running.
* **Paramiko errors:** Ensure correct authentication method (password vs key) and check `paramiko` version.
* **Too many failures / target blocks you:** Reduce thread count and throttle attempts; use a controlled lab environment.

---

## Ethics & Responsible Use

Only use this tool on systems you own or have explicit written permission to test. The pedagogical goal is improving security — include clear rules of engagement and written authorization when using these techniques in any real-world testing.

---

## Contributing

Contributions that improve safety, add defensive demos, clarify documentation, or add lab-friendly examples are welcome. Please open issues or pull requests with tests and documentation.

---

## License

MIT License — see `LICENSE` for details.

---
