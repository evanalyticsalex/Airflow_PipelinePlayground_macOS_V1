
# 🎧 Project: Podcast Pipeline with Apache Airflow (macOS Setup + DAG Example)

## 🧠 Purpose

In this project, we're building a **four-step data pipeline** using **Apache Airflow**, a powerful, Python-based workflow orchestration tool. The pipeline will **automatically download podcast episodes**, store them in a **SQLite database**, and optionally **transcribe audio to text**.

You can proudly include this as a real-world project in your portfolio.

---

## 🧰 Tools Used

- macOS Terminal
- Python 3.9.6
- Virtual Environment
- Apache Airflow 2.8.2
- SQLite
- Python Libraries: `requests`, `xmltodict`, `pandas`, `pydub`, `vosk`

---

## ✅ Why Airflow?

Airflow helps by:
- Scheduling the podcast check to run daily
- Logging errors per task
- Managing tasks independently
- Allowing extensions like speech recognition

---

## 💻 Setup Instructions

### 1. Create Virtual Environment

```bash
python3 -m venv airflow_env
source airflow_env/bin/activate
```

---

### 2. Install Airflow (2.8.2 with Python 3.9)

```bash
AIRFLOW_VERSION=2.8.2
PYTHON_VERSION=3.9
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"

pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"
```

---

### 3. Initialize Airflow DB

```bash
rm -rf ~/airflow
airflow db init
```

---

### 4. Start Airflow

```bash
airflow standalone
```

Visit `http://localhost:8080` and use the credentials printed in the terminal.

---

### 5. Install Additional Packages

```bash
pip install pandas xmltodict requests pydub vosk
```

---

## 📦 Understanding Constraints

Airflow relies on many dependencies. The **constraints file** ensures that all packages are installed in versions that are tested and compatible. Without it, Airflow might break due to version mismatches.

Airflow provides official constraint files for every version + Python combo, like:
```
https://raw.githubusercontent.com/apache/airflow/constraints-2.8.2/constraints-3.9.txt
```

---

## 🧩 The Podcast DAG (workflow)

```python
import os
import json
import requests
import xmltodict

from airflow.decorators import dag, task
import pendulum
from airflow.providers.sqlite.operators.sqlite import SqliteOperator
from airflow.providers.sqlite.hooks.sqlite import SqliteHook
from vosk import Model, KaldiRecognizer
from pydub import AudioSegment

PODCAST_URL = "https://www.marketplace.org/feed/podcast/marketplace/"
EPISODE_FOLDER = "episodes"
FRAME_RATE = 16000

@dag(
    dag_id='podcast_summary',
    schedule_interval="@daily",
    start_date=pendulum.datetime(2022, 5, 30),
    catchup=False,
)
def podcast_summary():
    create_database = SqliteOperator(
        task_id='create_table_sqlite',
        sql="""
        CREATE TABLE IF NOT EXISTS episodes (
            link TEXT PRIMARY KEY,
            title TEXT,
            filename TEXT,
            published TEXT,
            description TEXT,
            transcript TEXT
        );
        """,
        sqlite_conn_id="podcasts"
    )

    @task()
    def get_episodes():
        data = requests.get(PODCAST_URL)
        feed = xmltodict.parse(data.text)
        episodes = feed["rss"]["channel"]["item"]
        print(f"Found {len(episodes)} episodes.")
        return episodes

    podcast_episodes = get_episodes()
    create_database.set_downstream(podcast_episodes)

    @task()
    def load_episodes(episodes):
        hook = SqliteHook(sqlite_conn_id="podcasts")
        stored_episodes = hook.get_pandas_df("SELECT * from episodes;")
        new_episodes = []
        for episode in episodes:
            if episode["link"] not in stored_episodes["link"].values:
                filename = f"{episode['link'].split('/')[-1]}.mp3"
                new_episodes.append([episode["link"], episode["title"], episode["pubDate"], episode["description"], filename])
        hook.insert_rows(table='episodes', rows=new_episodes, target_fields=["link", "title", "published", "description", "filename"])
        return new_episodes

    new_episodes = load_episodes(podcast_episodes)

    @task()
    def download_episodes(episodes):
        audio_files = []
        for episode in episodes:
            name_end = episode["link"].split('/')[-1]
            filename = f"{name_end}.mp3"
            audio_path = os.path.join(EPISODE_FOLDER, filename)
            if not os.path.exists(audio_path):
                print(f"Downloading {filename}")
                audio = requests.get(episode["enclosure"]["@url"])
                with open(audio_path, "wb+") as f:
                    f.write(audio.content)
            audio_files.append({
                "link": episode["link"],
                "filename": filename
            })
        return audio_files

    audio_files = download_episodes(podcast_episodes)

    @task()
    def speech_to_text(audio_files, new_episodes):
        hook = SqliteHook(sqlite_conn_id="podcasts")
        untranscribed_episodes = hook.get_pandas_df("SELECT * from episodes WHERE transcript IS NULL;")

        model = Model(model_name="vosk-model-en-us-0.22-lgraph")
        rec = KaldiRecognizer(model, FRAME_RATE)
        rec.SetWords(True)

        for index, row in untranscribed_episodes.iterrows():
            print(f"Transcribing {row['filename']}")
            filepath = os.path.join(EPISODE_FOLDER, row["filename"])
            mp3 = AudioSegment.from_mp3(filepath)
            mp3 = mp3.set_channels(1)
            mp3 = mp3.set_frame_rate(FRAME_RATE)

            step = 20000
            transcript = ""
            for i in range(0, len(mp3), step):
                print(f"Progress: {i/len(mp3)}")
                segment = mp3[i:i+step]
                rec.AcceptWaveform(segment.raw_data)
                result = rec.Result()
                text = json.loads(result)["text"]
                transcript += text
            hook.insert_rows(table='episodes', rows=[[row["link"], transcript]], target_fields=["link", "transcript"], replace=True)

    # Uncomment the next line to enable transcription
    # speech_to_text(audio_files, new_episodes)

summary = podcast_summary()
```

---

## 🏁 Result

By the end of this project, you will have:
- A daily podcast data pipeline
- Data stored in a local database
- Optional transcription support
- A scalable Airflow DAG for your portfolio
