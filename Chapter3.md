# Chapter 3 – Verifying Airflow Installation

### Objective
Confirm that Airflow is operational by creating and running a simple DAG.

### Step 1 – Create Hello World DAG
```bash
cat > "$AIRFLOW_HOME/dags/hello_world.py" <<'PY'
from datetime import datetime
from airflow import DAG
from airflow.operators.bash import BashOperator

with DAG(dag_id="hello_world",
         start_date=datetime(2024,1,1),
         schedule=None,
         catchup=False) as dag:
    hello = BashOperator(task_id="hello", bash_command="echo 'Hello, Airflow!'")
PY
```

### Step 2 – Verify
1. Refresh the Airflow UI.
2. Locate `hello_world` DAG.
3. Trigger the DAG and inspect logs.

![Placeholder: Hello World DAG Screenshot](../assets/fig3_hello_dag.png)
*Figure 3. Successful execution of Hello World DAG.*
