# Chapter 5 – Extension: Newsvendor Optimization Using SA

### Objective
Use Simulated Annealing to find the optimal order quantity (Q) that maximizes expected profit under stochastic demand.

![Placeholder: Newsvendor Diagram](../assets/fig6_newsvendor.png)
*Figure 6. Newsvendor problem model.*

### Step 1 – Write Algorithm Script
Follow the stepwise code blocks to create `newsvendor_sa.py`.

#### Block 1 – Imports
```bash
target="$MODULE11_SCRATCH/newsvendor_sa.py"
: > "$target"
cat >> "$target" <<'PY'
#!/usr/bin/env python3
import argparse, json, math, os, random, statistics
PY
```

#### Block 2 – Profit Simulation
```bash
cat >> "$target" <<'PY'
def simulate_profit(Q, mu, sigma, price, cost, salvage, penalty, samples, seed):
    random.seed(seed); prof=[]
    for _ in range(samples):
        d=max(0.0,random.gauss(mu,sigma))
        sales=min(Q,d); leftover=max(0.0,Q-d); lost=max(0.0,d-Q)
        revenue=sales*price+leftover*salvage
        cost_val=Q*cost; penalty_cost=lost*penalty
        prof.append(revenue-cost_val-penalty_cost)
    return statistics.mean(prof)
PY
```

#### Block 3 – SA Optimizer
```bash
cat >> "$target" <<'PY'
def sa_newsvendor(mu,sigma,price,cost,salvage,penalty,qmax,iters=3000,seed=42,init_temp=5.0,cooling=0.9995,samples=200):
    random.seed(seed); Q=random.randint(0,qmax)
    bestQ=Q; bestVal=simulate_profit(Q,mu,sigma,price,cost,salvage,penalty,samples,seed+1)
    curQ,curVal=bestQ,bestVal; T=init_temp
    for _ in range(iters):
        step=max(1,int(T))
        cand=max(0,min(qmax,curQ+random.choice([-step,step])))
        val=simulate_profit(cand,mu,sigma,price,cost,salvage,penalty,samples,seed+1)
        d=val-curVal
        if d>=0 or random.random()<math.exp(d/max(T,1e-12)):
            curQ,curVal=cand,val
            if curVal>bestVal: bestQ,bestVal=curQ,curVal
        T*=cooling
    return {"best_Q":bestQ,"expected_profit":bestVal}
PY
```

#### Block 4 – Main Function
```bash
cat >> "$target" <<'PY'
def main():
    ap=argparse.ArgumentParser(description="Newsvendor via SA")
    ap.add_argument("--mu",type=float,default=120.0)
    ap.add_argument("--sigma",type=float,default=35.0)
    ap.add_argument("--price",type=float,default=20.0)
    ap.add_argument("--cost",type=float,default=8.0)
    ap.add_argument("--salvage",type=float,default=2.0)
    ap.add_argument("--penalty",type=float,default=0.0)
    ap.add_argument("--qmax",type=int,default=400)
    ap.add_argument("--iters",type=int,default=3000)
    ap.add_argument("--seed",type=int,default=42)
    ap.add_argument("--samples",type=int,default=200)
    ap.add_argument("--save",type=str,default="newsvendor_result.json")
    args=ap.parse_args()
    res=sa_newsvendor(args.mu,args.sigma,args.price,args.cost,args.salvage,
                      args.penalty,args.qmax,args.iters,args.seed,
                      samples=args.samples)
    with open(args.save,"w") as f: json.dump(res,f,indent=2)
    print(json.dumps(res,indent=2))
if __name__=="__main__": main()
PY
chmod +x "$target"
```

### Step 2 – Verification
```bash
python "$MODULE11_SCRATCH/newsvendor_sa.py" --iters 300
```

### Step 3 – Create Airflow DAG
```bash
dagfile="$AIRFLOW_HOME/dags/newsvendor_sa.py"
cat > "$dagfile" <<'PY'
from datetime import datetime
import os,subprocess,json
from airflow import DAG
from airflow.operators.python import PythonOperator

AIRFLOW_HOME=os.environ.get("AIRFLOW_HOME",os.path.expanduser("~/airflow"))
OUT_DIR=os.path.join(AIRFLOW_HOME,"outputs")
os.makedirs(OUT_DIR,exist_ok=True)
SCRIPT=os.path.join(os.environ.get("MODULE11_SCRATCH","/opt/mcd/work"),"newsvendor_sa.py")

def run_newsvendor(**_):
    out=os.path.join(OUT_DIR,"newsvendor_result.json")
    cmd=["python3",SCRIPT,"--mu","120","--sigma","35","--price","20","--cost","8","--salvage","2","--penalty","0","--qmax","400","--iters","3000","--save",out]
    subprocess.check_call(cmd)
    with open(out) as f: res=json.load(f)
    print("Best Q:",res["best_Q"],"Expected Profit:",res["expected_profit"])

with DAG(dag_id="newsvendor_sa",start_date=datetime(2024,1,1),schedule=None,catchup=False,tags=["optimization"] ) as dag:
    solve=PythonOperator(task_id="run_sa_newsvendor",python_callable=run_newsvendor)
PY
```

### Step 4 – Verification in Airflow
Trigger the DAG, monitor the logs, and observe printed best_Q and expected_profit.

![Placeholder: Newsvendor DAG Screenshot](../assets/fig7_newsvendor_dag.png)
*Figure 7. Newsvendor DAG execution log.*
