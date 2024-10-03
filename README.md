# Motion Capture Cleanup Script

This repository contains a Python script designed to delete old motion capture movies from a specified directory after a certain age limit has been met. The script is optimized for readability, security, and flexibility.

## Features

- Deletes files based on their age and file type.
- Uses `os.walk` and `datetime` for precise control over file deletion.
- Easily configurable to add more file types and age limits.

## Requirements

- Python 3.x

## Installation

1. Clone the repository:
    ```bash
    git clone https://github.com/sigmaenigma/motion-utilities.git
    ```
2. Navigate to the repository directory:
    ```bash
    cd motion-utilities
    ```

## Usage

1. Open the script file and configure the `file_types` dictionary with the desired file types and age limits.
2. Set the `default_motion_directory` to the directory where your motion capture files are stored.
3. Run the script:
    ```bash
    python motion_file_rotate.py
    ```

## Running the Script via Cron

To run this script automatically at regular intervals using cron:

1. Open the cron table for editing:
    ```bash
    crontab -e
    ```
2. Add the following line to schedule the script to run daily at midnight:
    ```bash
    0 0 * * * /usr/bin/python3 /path/to/motion-utilities/motion_file_rotate.py
    ```
   Replace `/usr/bin/python3` with the path to your Python interpreter and `/path/to/motion-utilities/motion_file_rotate.py` with the path to your script.

## Containerizing the Script

To containerize the script and have it run perpetually:

1. Create a `Dockerfile` in your repository directory:
    ```Dockerfile
    FROM python:3.9-slim

    WORKDIR /app

    COPY motion_file_rotate.py .

    CMD ["python", "motion_file_rotate.py"]
    ```
2. Build the Docker image:
    ```bash
    docker build -t motion-cleanup .
    ```
3. Run the Docker container:
    ```bash
    docker run -d --name motion-cleanup motion-cleanup
    ```

This will run the script inside a Docker container. You can set up a cron job inside the container or use Docker's scheduling capabilities to run the script at regular intervals.

## Script Details

```python
import os
import subprocess
from datetime import datetime, timedelta

# Define file types and their respective age limits
file_types = {
    '*.mkv': 7,
    '*.mpg': 7
}

default_motion_directory = '/var/lib/motion/'

def delete_old_files(directory, file_type, age_limit):
    now = datetime.now()
    cutoff = now - timedelta(days=age_limit)
    
    for root, dirs, files in os.walk(directory):
        for file in files:
            if file.endswith(file_type.split('.')[-1]):
                file_path = os.path.join(root, file)
                file_mtime = datetime.fromtimestamp(os.path.getmtime(file_path))
                if file_mtime < cutoff:
                    try:
                        os.remove(file_path)
                        print(f'Deleted {file_path}')
                    except Exception as e:
                        print(f'Error deleting {file_path}: {e}')

def run_delete():
    print('Running...\n')
    for file_type, age_limit in file_types.items():
        delete_old_files(default_motion_directory, file_type, age_limit)
    print('Complete...\n')

if __name__ == "__main__":
    run_delete()
```
