# Chapter 2 – Environment Setup on AWS EC2 (Ubuntu 22.04 LTS)

## Objective
Set up Apache Airflow on an EC2 instance launched from the pre-configured **MCD-Module11-Ubuntu 22.04 AMI**.  
You will perform all tasks directly in the **EC2 Instance Connect web terminal**—no SSH keys or local terminal setup are required.

---

## 2.1 Launch the Instance

1. Sign in to the AWS Management Console.  
2. Navigate to **AMIs** and select  
   **ApacheAirflowAMI**.  
3. Click **Launch instance from AMI**.  
4. Choose **t2.medium**.  
5. Under **Network settings**, ensure inbound rules include:  
   - **Port 22 (SSH)** – required for EC2 Instance Connect  
   - **Port 8080 (Custom TCP)** – for Airflow Web UI  
6. Leave storage and IAM role as default, then click **Launch instance**.

When the instance is running:
1. Go to **EC2 → Instances** and select your new instance.  
2. Click **Connect** → choose **EC2 Instance Connect** (tab).  
3. Click **Connect** again to open a browser-based terminal.  

You are now inside your Ubuntu environment.  
A welcome message should appear:

```
Mizzou Cloud DevOps — Module 11 (Workflow Automation for Optimization with Airflow)
```

This confirms that your instance was built from the correct AMI.

---

## 2.2 Activate Environment Variables

The AMI already includes a workspace and environment variable.  
Verify it by running:

```bash
echo $MODULE11_SCRATCH
```
Expected output:
```
/opt/mcd/work
```

If it matches, your environment is configured correctly.

---

## 2.3 Create and Activate a Python Virtual Environment

```bash
python3 -m venv ~/mcd-env
source ~/mcd-env/bin/activate
```

*(You should see `(mcd-env)` appear in your prompt.)*

---

## 2.4 Install Apache Airflow

```bash
pip install --upgrade pip wheel
pip install "apache-airflow==2.9.3" --constraint   "https://raw.githubusercontent.com/apache/airflow/constraints-2.9.3/constraints-3.10.txt"
```

This installs Airflow inside your virtual environment.

---

## 2.5 Initialize Airflow and Create an Admin User

```bash
export AIRFLOW_HOME=$HOME/airflow
airflow db init
airflow users create --username admin --firstname Admin --lastname User   --role Admin --email admin@example.com --password admin
```

When the initialization completes, the Airflow database and default folders are created under `~/airflow`.

---

## 2.6 Start Airflow Services

Use the pre-installed helper script to start both the webserver and scheduler.

```bash
module11-start-airflow.sh
```

This command launches:
- **Airflow webserver** (on port 8080)  
- **Airflow scheduler** (in background)  

Wait ~30 seconds, then open the Airflow UI:

```
http://<EC2-Public-IP>:8080
```

Login credentials:
```
Username: admin
Password: admin
```

> *Tip:* If you close or disconnect the web terminal, Airflow will continue running in the background.  
> To stop services later, use:
> ```bash
> module11-stop-airflow.sh
> ```

---

## 2.7 Verify Installation

Inside the web UI, confirm:
- The **DAGs dashboard** loads without errors.  
- The **“example_dags”** list appears (Airflow default examples).  

If visible, your setup is complete.

---

## 2.8 Proceed to the Next Chapter

You now have a working Airflow environment.  
Continue to **Chapter 3 – Verifying Airflow Installation** to create and test your first DAG.
