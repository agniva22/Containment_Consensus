# Trajectory & Consensus Error 


## Trajectory & Consensus Error (X–axis)

This README explains the two plots you provided and how to regenerate them from logs.  
It also clarifies how **consensus** is computed and how to interpret the figures.

---

### Files in this folder

- `all_drones_x_vs_time.eps` – X-position trajectories of all drones over time  
  ![All drones: x vs time](./all_drones_x_vs_time.eps)

- `x_consensus_error_vs_time.eps` – Per-drone absolute error to the **x-consensus** value  
  ![Consensus error in x](./x_consensus_error_vs_time.eps)

> Tip: If your viewer doesn’t render EPS inline, export as `--fmt png` in the script.

---

## What the plots show

### 1) All-drones overlay: `x` vs `time`
- Each curve is a drone’s **x(t)**.
- Labels use your custom mapping: `A1…A5` are agents; `T1/T2` are **anchors**.
- Colors/linestyles come from the script’s `STYLE_MAP`.
- Convergence (lines clustering together) hints at consensus along the x-axis.

### 2) X–consensus error: \(|x_i(t) - \bar{x}|\)
- For each **non-anchor** agent, we plot the absolute error to the **consensus**:

  \[
  e_i(t) = \big|x_i(t) - x^*\big|,\quad
  x^* = \operatorname{median}\big(\{x_k(T_{end})\}_{k\in\text{all drones}}\big)
  \]

- **Anchors** (`T1`, `T2`) are **excluded** from the error plot; they bound/guide the network rather than converge.
- As consensus is reached, each error curve should tend toward **0** (subject to noise and controller dynamics).

> Why **median** of final samples? It’s robust to outliers (e.g., one agent drifting), making the consensus estimate stable.

---

## How to read the plots

- **Good consensus:** overlay shows traces approaching a common x-level; error curves decay to near-zero.
- **Slow convergence:** shallow error decay or long plateaus.
- **Oscillation/overshoot:** ripples or ringing in trajectories and error.
- **Bias or steady-state error:** error curves flatten above zero → controller or measurement bias.
- **Outlier/failed agent:** one error curve remaining high while others drop.

---

## About the time axis

If your error plot appears to “start” around **120 s**, that’s because the CSV’s `t_sec` column already starts near 120 (e.g., logging began mid-run).  
Two options:

**Global re-zero (all drones share t=0):**
```python
t0 = float(df['t_sec'].min())
df['t'] = df['t_sec'] - t0
```

**Per-drone re-zero (each drone’s first sample is t=0):**
```python
df['t'] = df.groupby('drone')['t_sec'].transform(lambda s: s - s.min())
```

Then use `t` instead of `t_sec` in plotting.

---

## Reproducing the plots

Assuming your upgraded plotting script (with `CUSTOM_LABELS`, `STYLE_MAP`, and consensus-error support):

```bash
# All-drones overlay for x
python plot_odom.py --dir /path/to/logdir --combined all_odom.csv     --component x --fmt eps --overlay-xlim 150

# X–consensus error (anchors skipped by default)
python plot_odom.py --dir /path/to/logdir --combined all_odom.csv     --error-comp x --tmax 150 --fmt eps
```

### Required CSV columns
- `drone` (string name, e.g., `crazyflie1`)
- `t_sec` (float seconds since some epoch)
- `x`, `y`, `z` (float, meters)

### Styling & labels
- Edit `CUSTOM_LABELS` to map **numeric IDs → labels** (e.g., `1 → A1`, `2 → T1`).
- Edit `STYLE_MAP` to change line color/linestyle per label.
- Legends are generated automatically.

---

## Common pitfalls

- **Units:** ensure `x`, `y`, `z` are in **meters**.
- **NaNs:** rows with missing `x/y/z/t_sec/drone` are dropped—check your logs.
- **Time crop:** the error plot uses `--tmax` to cap the time horizon.
- **Anchors in error plot:** make sure they’re **excluded** (default) when analyzing consensus of agents.

---

### At a glance

- **Goal:** show how agents move in x and how fast they agree on a common x.  
- **Success condition:** error curves → 0 and stay there (within tolerance).  
- **Diagnostics:** look for the slowest-decaying curve; that agent or its neighbors may limit convergence.

---

If you want a PDF/PNG version, re-run with `--fmt pdf` or `--fmt png`.
