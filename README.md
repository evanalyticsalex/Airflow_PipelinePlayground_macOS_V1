# 🎵 Project: Podcast Pipeline with Apache Airflow (macOS Setup + DAG Example)

## 🧠 Purpose

This project builds a **four-step data pipeline** using **Apache Airflow**, a powerful, Python-based workflow orchestration tool. The pipeline will:

- Automatically **download podcast episodes**
- Store episode data in a **SQLite database**
- Optionally **transcribe audio to text** using Vosk

A real-world project you can proudly showcase in your portfolio.

---

## 🧰 Tools Used

- macOS Terminal
- Python 3.9.6
- Virtual Environment
- Apache Airflow 2.8.2
- SQLite
- Python Libraries: `requests`, `xmltodict`, `pandas`, `pydub`, `vosk`

---

## ✅ Why Use Airflow?

Airflow enables:
- Scheduled, automated daily runs
- Independent task execution with retry and logging
- Easy debugging with logs
- Seamless pipeline expansion (e.g., transcription, summaries)

---

## ⚙️ One-Time Setup Instructions (First-Time Only)

### 1. Create Virtual Environment

```bash
python3 -m venv airflow_env
source airflow_env/bin/activate
```

### 2. Install Airflow (2.8.2 + Python 3.9)

```bash
AIRFLOW_VERSION=2.8.2
PYTHON_VERSION=3.9
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"

pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"
```

### 3. Initialize Airflow DB

```bash
rm -rf ~/airflow  # optional cleanup
airflow db init
```

### 4. Start Airflow (once)

```bash
airflow standalone
```

Visit `http://localhost:8080` and use the credentials shown in the terminal.

### 5. Install Required Python Libraries

```bash
pip install pandas xmltodict requests pydub vosk
```

---

## 🔄 How to Resume the Project After Reboot

Every time you restart your Mac:

### 1. Open Terminal

### 2. Run your startup script:

```bash
start_airflow
```

#### If not created yet, do this once:

Create the script:

```bash
nano ~/start_airflow.sh
```

Paste this:

```bash
#!/bin/zsh
cd ~/Documents/airflow_project   # Replace with your actual project folder
source airflow_env/bin/activate
airflow webserver -p 8080 &
airflow scheduler &
```

Make it executable:

```bash
chmod +x ~/start_airflow.sh
```

Add an alias to simplify command (one time):

```bash
echo "alias start_airflow='~/start_airflow.sh'" >> ~/.zshrc
source ~/.zshrc
```

Now, just type `start_airflow` after every reboot to run Airflow again.

---

## 📦 Airflow Dependency Constraints

Airflow uses many interdependent libraries. Installing with a **constraints file** ensures compatibility:

```
https://raw.githubusercontent.com/apache/airflow/constraints-2.8.2/constraints-3.9.txt
```

Always install with constraints to avoid version conflicts.

---

## 🧩 DAG: `podcast_summary`

Below is the DAG for the podcast pipeline. It:

- Creates a table in SQLite
- Downloads and stores new podcast episodes
- Saves metadata and optionally transcribes the episode to text

```python
<-- Insert Full DAG Code Here (Omitted for brevity) -->
```

---

## 🧼 Clean Shutdown (Optional)

Before shutting down your Mac, you can stop all Airflow services:

```bash
pkill -f "airflow webserver"
pkill -f "airflow scheduler"
deactivate
```

This step is optional but helps avoid zombie processes.

---

## 🏁 Project Outcomes

By the end of this project, you’ll have:

- A daily podcast data pipeline
- Metadata stored in SQLite
- Optional speech-to-text functionality
- A clean, portfolio-ready Airflow project
