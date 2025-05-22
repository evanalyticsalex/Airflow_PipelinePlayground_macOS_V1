
# 📊 Project Title: Apache Airflow Local Setup on macOS with Python 3.9.6

## 🧠 Purpose

This project documents the step-by-step process of setting up **Apache Airflow**, a workflow orchestration tool, on a **Mac** using **Python 3.9.6**. It provides a clean, isolated development environment using virtual environments and ensures compatibility through constraint files.

---

## 🔧 Tools Used

- **macOS Terminal**
- **Python 3.9.6**
- **Virtual Environment**
- **Pip (Python package manager)**
- **Apache Airflow 2.8.2**
- **SQLite (default Airflow DB)**
- **Web browser (for UI)**

---

## ✅ Prerequisites

Ensure Python 3.9.6 is installed:

```bash
python3 --version
```

If not, install it using [Homebrew](https://brew.sh/).

---

## 🚀 Step-by-Step Installation

### 1. Create a Virtual Environment

```bash
python3 -m venv airflow_env
source airflow_env/bin/activate
```

This creates a clean sandbox so Airflow doesn’t interfere with system-wide packages.

---

### 2. Set Airflow and Python Versions

```bash
AIRFLOW_VERSION=2.8.2
PYTHON_VERSION=3.9
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
```

These variables define the Airflow version and the Python version to use for installing compatible dependencies.

---

### 3. Install Apache Airflow with Constraints

```bash
pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"
```

This ensures all dependencies are installed at compatible versions to avoid errors.

---

### 4. Initialize the Airflow Metadata Database

```bash
airflow db init
```

This sets up the SQLite database where Airflow stores task history, DAGs, users, etc.

---

### 5. Launch Airflow in Standalone Mode

```bash
airflow standalone
```

This command:
- Starts the scheduler
- Starts the webserver (UI on [http://localhost:8080](http://localhost:8080))
- Creates a default admin user
- Prints login credentials in Terminal

---

### 6. Log in to the Airflow Web UI

Open your browser and visit:

```
http://localhost:8080
```

Use the default credentials printed in the terminal to log in.

---

## 📦 Optional: Install Common Python Libraries

These are often used in Airflow DAGs:

```bash
pip install pandas xmltodict requests
```

---

## 🧠 What Are Constraint Files?

Airflow depends on many Python libraries. Constraint files list **exact, tested versions** of all these dependencies. They ensure that:
- All packages work together
- Your setup is stable and error-free

Airflow provides official constraint files for every version + Python combo, like:

```
https://raw.githubusercontent.com/apache/airflow/constraints-2.8.2/constraints-3.9.txt
```

---

## ✅ Summary

You’ve now set up a local instance of Apache Airflow in a clean, reproducible way on macOS using Python 3.9.6. You can now build, schedule, and monitor workflows (DAGs) using the web UI and command line.
