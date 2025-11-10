# Chapter 4 – Workflow Automation for Optimization Using SA (3D Printing Job Scheduling)

### Objective
Build an Airflow pipeline that automates Simulated Annealing for minimizing 3D printing makespan.

### Concept
Each job has a processing time. The algorithm distributes jobs across machines to minimize completion time.

![Placeholder: Job Scheduling Diagram](../assets/fig4_job_scheduling.png)
*Figure 4. 3D Printing job assignment illustration.*

### Step 1 – Write Algorithm Script
Follow the stepwise code blocks to create `job_scheduling_sa.py` in `$MODULE11_SCRATCH`.

#### Block 1 – Imports
```bash
target="$MODULE11_SCRATCH/job_scheduling_sa.py"
mkdir -p "$MODULE11_SCRATCH"
: > "$target"
cat >> "$target" <<'PY'
#!/usr/bin/env python3
import argparse, json, math, os, random
from typing import List, Tuple
PY
```

#### Block 2 – Makespan Evaluation
```bash
cat >> "$target" <<'PY'
def eval_makespan(permutation: List[int], proc: List[int], m: int):
    loads = [0]*m
    for job in permutation:
        j = min(range(m), key=lambda i: loads[i])
        loads[j] += proc[job]
    return max(loads), loads
PY
```

#### Block 3 – Simulated Annealing Loop
```bash
cat >> "$target" <<'PY'
def sa_schedule(proc: List[int], m: int, iters=5000, seed=42, init_temp=1.0, cooling=0.999):
    random.seed(seed)
    n = len(proc)
    cur = list(range(n)); random.shuffle(cur)
    cur_cost, _ = eval_makespan(cur, proc, m)
    best, best_cost = cur[:], cur_cost; T = init_temp
    for _ in range(iters):
        i, j = random.sample(range(n), 2)
        nxt = cur[:]; nxt[i], nxt[j] = nxt[j], nxt[i]
        nxt_cost, _ = eval_makespan(nxt, proc, m)
        d = nxt_cost - cur_cost
        if d <= 0 or random.random() < math.exp(-d/max(T,1e-12)):
            cur, cur_cost = nxt, nxt_cost
            if cur_cost < best_cost: best, best_cost = cur[:], cur_cost
        T *= cooling
    _, loads = eval_makespan(best, proc, m)
    return {"best_perm": best, "best_makespan": best_cost, "loads": loads}
PY
```

#### Block 4 – Main Function
```bash
cat >> "$target" <<'PY'
def main():
    ap = argparse.ArgumentParser(description="SA for 3D Printing Job Scheduling")
    ap.add_argument("--jobs", type=int, default=20)
    ap.add_argument("--machines", type=int, default=3)
    ap.add_argument("--iters", type=int, default=5000)
    ap.add_argument("--seed", type=int, default=42)
    args = ap.parse_args()
    random.seed(args.seed); proc = [random.randint(5, 60) for _ in range(args.jobs)]
    res = sa_schedule(proc, args.machines, args.iters)
    res["processing_times"] = proc
    with open("job_result.json","w") as f: json.dump(res,f,indent=2)
    print(json.dumps(res,indent=2))
if __name__=="__main__": main()
PY
chmod +x "$target"
```

### Step 2 – Verification
```bash
python "$MODULE11_SCRATCH/job_scheduling_sa.py" --jobs 10 --machines 3 --iters 300
```

### Step 3 – Create Airflow DAG
```bash
dagfile="$AIRFLOW_HOME/dags/job_scheduling_sa.py"
cat > "$dagfile" <<'PY'
from datetime import datetime
import os, subprocess, json
from airflow import DAG
from airflow.operators.python import PythonOperator

AIRFLOW_HOME=os.environ.get("AIRFLOW_HOME",os.path.expanduser("~/airflow"))
OUT_DIR=os.path.join(AIRFLOW_HOME,"outputs")
os.makedirs(OUT_DIR,exist_ok=True)
SCRIPT=os.path.join(os.environ.get("MODULE11_SCRATCH","/opt/mcd/work"),"job_scheduling_sa.py")

def run_sa(**_):
    out=os.path.join(OUT_DIR,"job_result.json")
    cmd=["python3",SCRIPT,"--jobs","20","--machines","3","--iters","3000","--save",out]
    subprocess.check_call(cmd)
    with open(out) as f: res=json.load(f)
    print("Best makespan:",res["best_makespan"])

with DAG(dag_id="job_scheduling_sa",start_date=datetime(2024,1,1),
         schedule=None,catchup=False,tags=["optimization"]) as dag:
    solve=PythonOperator(task_id="run_sa_job_scheduling",python_callable=run_sa)
PY
```

### Step 4 – Verification in Airflow
Trigger the DAG and view logs to confirm output.

![Placeholder: Airflow DAG Screenshot](../assets/fig5_job_dag.png)
*Figure 5. Job Scheduling DAG execution log.*
