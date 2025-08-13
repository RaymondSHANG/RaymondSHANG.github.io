---
layout: post
title: "tricks_running_notebooks_in_cloud_clusters"
subtitle: "env_long run_etc"
date: 2025-08-13 14:23:41
header-style: text
catalog: true
author: "Yuan"
tags: ['cloud','cluster','notebook','long run','env','sql','line magics','cell magics','autosave','autoload','gcloud storage','Bucket','gcloud']
---
{% include linksref.html %}
>Cloud clusters hum, notebooks run, and you just wait the work get done.

## Run notebook in the background
Build a new notebook in the same directly of (**NOTEBOOK_TO_RUN**), and run it!

Click  Cell -> Run All, then you could close this notebook tab and wait until finished.

Below are the codes in the new notebook that execute the notebook (**NOTEBOOK_TO_RUN**), upload it to GCS, and also capture all stdout/stderr logs into a .log file that gets uploaded too

### Setup
```python
import os
import time
import nbformat
import gcsfs
import sys
import io
import shutil
from nbconvert.preprocessors import ExecutePreprocessor, CellExecutionError
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'  # or '3' to suppress both INFO and WARNING logs

# ==============================
# Configuration
# ==============================
NOTEBOOK_TO_RUN = "your_notebook.ipynb"
KERNEL_NAME = "python3"
DATESTAMP = time.strftime("%Y%m%d")
TIMESTAMP_SUFFIX = time.strftime("_%Y%m%d_%H%M%S.ipynb")
OUTPUT_NOTEBOOK = NOTEBOOK_TO_RUN.replace(".ipynb", TIMESTAMP_SUFFIX)
LOG_FILE = OUTPUT_NOTEBOOK.replace(".ipynb", ".log")

WORKSPACE_BUCKET = os.getenv("WORKSPACE_BUCKET")  # e.g., "gs://my-bucket"

print(f'Executed notebook will be saved as "{OUTPUT_NOTEBOOK}" locally and in the workspace bucket.')
print(f'Execution logs will be saved as "{LOG_FILE}" locally and in the workspace bucket.')

# Initialize GCS file system
fs = gcsfs.GCSFileSystem()
```

### Run your notebook in background
```python
# ==============================
# Helpers
# ==============================
def run_notebook(input_nb, output_nb, kernel_name):
    """Execute a notebook and write the executed version."""
    with open(input_nb) as f_in:
        nb = nbformat.read(f_in, as_version=4)

    ep = ExecutePreprocessor(timeout=-1, kernel_name=kernel_name)

    try:
        ep.preprocess(nb, {"metadata": {"path": ""}})
    except CellExecutionError:
        print(f'Error executing the notebook "{input_nb}". '
              f'See "{output_nb}" for traceback.')
    finally:
        with open(output_nb, "w", encoding="utf-8") as f_out:
            nbformat.write(nb, f_out)
        print(f"Notebook execution completed: {output_nb}")

def save_to_bucket(local_path, bucket_subdir):
    """Copy a local file to the workspace bucket."""
    dest_path = os.path.join(WORKSPACE_BUCKET, bucket_subdir, os.path.basename(local_path))
    with open(local_path, "rb") as src, fs.open(dest_path, "wb") as dst:
        shutil.copyfileobj(src, dst)
    print(f"Wrote file to {dest_path}")

def run_notebook_with_logging(input_nb, output_nb, log_file, kernel_name):
    """Execute a notebook and capture all stdout/stderr logs."""
    # Redirect stdout and stderr
    log_buffer = io.StringIO()
    sys_stdout_orig, sys_stderr_orig = sys.stdout, sys.stderr
    sys.stdout = sys.stderr = log_buffer

    try:
        with open(input_nb) as f_in:
            nb = nbformat.read(f_in, as_version=4)

        ep = ExecutePreprocessor(timeout=-1, kernel_name=kernel_name)
        ep.preprocess(nb, {"metadata": {"path": ""}})

    except CellExecutionError as e:
        print(f'Error executing the notebook "{input_nb}". See "{output_nb}" for traceback.')
        print(str(e))

    finally:
        # Restore stdout/stderr
        sys.stdout, sys.stderr = sys_stdout_orig, sys_stderr_orig

        # Save executed notebook
        with open(output_nb, "w", encoding="utf-8") as f_out:
            nbformat.write(nb, f_out)

        # Save logs
        with open(log_file, "w", encoding="utf-8") as f_log:
            f_log.write(log_buffer.getvalue())

        print(f"Notebook execution completed: {output_nb}")
        print(f"Logs saved to: {log_file}")

# ==============================
# Execution
# ==============================
run_notebook_with_logging(NOTEBOOK_TO_RUN, OUTPUT_NOTEBOOK, LOG_FILE, KERNEL_NAME)

# Upload to bucket
save_to_bucket(OUTPUT_NOTEBOOK, "notebooks")
save_to_bucket(LOG_FILE, "notebooks")

```

## Basic, execute bash commands in a Notebook
{{note}} <p>You could use <code>%lsmagic</code> in a cell to list all <b>"line magic" commands"</b> and <b>"cell magic" commands</b></p>
{{end}}

### Exclamation Mark `!`

The exclamation mark is used to execute **shell commands** directly from a notebook cell. This is useful for running system commands, such as installing a package or checking a file's content, without leaving the notebook environment.

**Example:**
* `!pip install numpy` will install the numpy library.
* `!ls -l` will list the files in the current directory.
* `!ls -d */` will list all directories or `ls -d */ | sed 's:/$::'`

---

### Dollar Sign `$`

In some interactive environments, particularly those with a bash or shell kernel, the dollar sign is used to reference a **variable's value**. This allows you to pass a variable from your notebook's code to a shell command.

**Example:**
* If `my_file = "data.csv"`, then `!ls -l $my_file` will list the details of the `data.csv` file.

---

### Percent Sign `%` (Line Magics)

The single percent sign is a **"line magic" command**. It applies a specific function to a **single line of code** in a notebook cell. These commands are built into the IPython kernel and are used for a variety of tasks, like timing code execution or loading external files.

**Example:**
* `%timeit sum(range(100))` will measure the execution time of summing numbers from 0 to 99.
* `%pwd` will print the current working directory.
* `%lsmagic` will list all **"line magic" commands** and **"cell magic" commands**

---

### Double Percent Sign `%%` (Cell Magics)

The double percent sign is a **"cell magic" command**. Unlike line magics, it applies a command to the **entire cell**. This is commonly used to change the language interpreter for a cell or to write the cell's content to a file.

**Example:**
* `%%bash` at the top of a cell will tell the notebook to interpret the entire cell's content as bash commands.
* `%%writefile my_script.py` will save the contents of the cell to a file named `my_script.py`.

Here is the information in the requested format.

-----

### The `%autoreload` and `%autosave` Magic Commands

#### `%autoreload`

**Explanation:**
The `%autoreload` command is a powerful "magic" command used in IPython and Jupyter notebooks to automatically reload modules before executing code. This is particularly useful during active development when you are making changes to external Python files (`.py`) and want those changes to be immediately reflected in your notebook without needing to restart the kernel. By default, Python imports modules only once, so `%autoreload` bypasses this behavior. To use it, you must first enable it with `%load_ext autoreload`.
- `%autoreload 0`: Disables auto-reloading (the default setting).

- `%autoreload 1`: Only reloads modules that were imported with %aimport.

- `%autoreload 2`: Reloads all modules (except those you've specifically excluded) every time a cell is run. This is the most common setting for active development.

**Usage Example:**

```python
# Enable the autoreload extension
%load_ext autoreload

# Set autoreload to mode 2 to reload all modules before execution
%autoreload 2
```

Once this is set, any changes you make to imported Python files will be automatically loaded the next time you run a cell.

#### `%autosave`

**Explanation:**
The `%autosave` command is a magic command that allows you to control the frequency of your notebook's autosave function. By default, Jupyter notebooks autosave every 120 seconds (2 minutes). Using `%autosave`, you can customize this interval to save more or less frequently, or even turn autosaving off completely. This can be helpful to prevent data loss with a shorter interval or to avoid unnecessary disk writes with a longer one. The command takes a numerical argument representing the desired interval in seconds.

**Usage Example:**

```python
# Set the autosave interval to 60 seconds (1 minute)
%autosave 60

# Turn autosaving off completely
%autosave 0
```

## Communications between virtual machine (VM) and Google buckets

This guide will show you how to use the **gcloud storage** command to interact with Google Cloud Storage buckets directly from your VM. 

### Creating a Folder in a Google Bucket

Google Cloud Storage doesn't use a traditional file system with folders. Instead, it uses a flat hierarchy where objects have names that can include a prefix that acts like a directory path. You can create this "folder" structure by simply copying an object into a path that includes the desired folder name.

To create a folder named `data` in your bucket, you'll first create a dummy file on your VM and then copy it to the desired location.

```
# First, create a dummy file on your VM
touch empty_file.txt

# Then, copy it to the 'data' directory within your bucket
gcloud storage cp empty_file.txt gs://<YOUR_BUCKET_NAME>/data/

```

This command will create a `data/` "folder" and place `empty_file.txt` inside it. You can now delete the dummy file from your VM if you no longer need it.

### Copying a File from a VM to a Bucket

To copy a local file from your VM to the bucket, you'll use the `gcloud storage cp` command. This is the command to use for the specific task you requested.

```
# Create a sample file on your VM named test1.csv
echo "header1,header2" > test1.csv
echo "value1,value2" >> test1.csv

# Now, copy the test1.csv file into the 'data' folder in your bucket
gcloud storage cp test1.csv gs://<YOUR_BUCKET_NAME>/data/test1.csv

```

### Moving a File within a Google Bucket

The `gcloud storage mv` command is used to move files. This command also works for renaming a file, as it simply moves the file to a new location with a new name.

```
# Move a file from the bucket root to the 'data' folder
gcloud storage mv gs://<YOUR_BUCKET_NAME>/old_file.txt gs://<YOUR_BUCKET_NAME>/data/new_file.txt

```

### Renaming a File in a Google Bucket

To rename a file, you use the same `gcloud storage mv` command, but you keep the file in the same "directory" and just change its name.

```
# Rename 'test1.csv' to 'final_data.csv' within the 'data' folder
gcloud storage mv gs://<YOUR_BUCKET_NAME>/data/test1.csv gs://<YOUR_BUCKET_NAME>/data/final_data.csv

```

### Remove a file in a Google Bucket
Using `gcloud storage rm` command
```
gcloud storage rm gs://<YOUR_BUCKET_NAME>/data/final_data.csv
```
---
