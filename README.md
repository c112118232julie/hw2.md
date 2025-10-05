# 安裝必要套件 (Colab 用)
!pip install graphviz matplotlib pandas

from collections import defaultdict, deque
import pandas as pd
import matplotlib.pyplot as plt
from graphviz import Digraph

# -----------------------------
# 1. 定義任務 (ID, 名稱, 工期, 前置任務)
# ⚠️ 這裡只是範例，請改成你圖3-27 的真實 11 項任務
# -----------------------------
tasks = [
    ("A", "Project Approval", 5, []),
    ("B", "Hire Analyst", 10, ["A"]),
    ("C", "Requirements Gathering", 8, ["A"]),
    ("D", "System Design", 12, ["B","C"]),
    ("E", "Procure Equipment", 14, ["A"]),
    ("F", "Setup Environment", 6, ["E"]),
    ("G", "Training Plan", 5, ["D"]),
    ("H", "Logistics Arrangement", 10, ["E"]),
    ("I", "Conduct Training", 15, ["G","H","F"]),
    ("J", "Pilot Test", 7, ["F","D"]),
    ("K", "Final Deployment", 4, ["I","J"]),
]

# -----------------------------
# 2. 建立關聯 (前置 & 後繼)
# -----------------------------
duration = {t[0]: t[2] for t in tasks}
name = {t[0]: t[1] for t in tasks}
pred = {t[0]: list(t[3]) for t in tasks}
succ = defaultdict(list)
for t in tasks:
    for p in t[3]:
        succ[p].append(t[0])

# -----------------------------
# 3. 拓樸排序 (找任務順序)
# -----------------------------
in_deg = {t[0]: 0 for t in tasks}
for t in tasks:
    for p in pred[t[0]]:
        in_deg[t[0]] += 1

q = deque([n for n,d in in_deg.items() if d==0])
topo = []
while q:
    n = q.popleft()
    topo.append(n)
    for s in succ[n]:
        in_deg[s] -= 1
        if in_deg[s]==0:
            q.append(s)

# -----------------------------
# 4. Forward pass (最早開始/完成)
# -----------------------------
ES, EF = {}, {}
for n in topo:
    ES[n] = max([EF[p] for p in pred[n]] or [0])
    EF[n] = ES[n] + duration[n]

project_duration = max(EF.values())

# -----------------------------
# 5. Backward pass (最晚開始/完成)
# -----------------------------
LF, LS = {}, {}
for n in reversed(topo):
    LF[n] = min([LS[s] for s in succ[n]] or [project_duration])
    LS[n] = LF[n] - duration[n]

# -----------------------------
# 6. Slack 與關鍵路徑
# -----------------------------
slack = {n: LS[n] - ES[n] for n in topo}
critical = [n for n in topo if slack[n]==0]

# -----------------------------
# 7. 輸出 DataFrame (排程表)
# -----------------------------
df = pd.DataFrame([{
    "ID": n,
    "Task": name[n],
    "Duration": duration[n],
    "Predecessors": ",".join(pred[n]) if pred[n] else "-",
    "ES": ES[n],
    "EF": EF[n],
    "LS": LS[n],
    "LF": LF[n],
    "Slack": slack[n],
    "Critical": ("Yes" if slack[n]==0 else "No")
} for n in topo])

print("=== 專案排程表 ===")
print(df)
print("\n專案總工期:", project_duration, "天")
print("關鍵路徑:", " -> ".join(critical))

# -----------------------------
# 8. 畫 Gantt 圖
# -----------------------------
plt.figure(figsize=(10,6))
y_positions = range(len(topo))
plt.barh(y_positions, [duration[n] for n in topo], left=[ES[n] for n in topo], color="skyblue")
plt.yticks(y_positions, [f"{n}: {name[n]}" for n in topo])
plt.xlabel("Day")
plt.title("Gantt Chart")
plt.grid(axis='x', linestyle='--', linewidth=0.5)
plt.show()

# -----------------------------
# 9. 畫 PERT/CPM 圖 (Graphviz)
# -----------------------------
dot = Digraph(comment="PERT/CPM")
dot.attr(rankdir="LR", shape="record")

for n in topo:
    lbl = f"{name[n]}\\nID:{n}\\nDur:{duration[n]}d\\nES:{ES[n]} EF:{EF[n]}"
    if n in critical:
        dot.node(n, lbl, style="filled", fillcolor="lightcoral")  # 關鍵路徑用紅色
    else:
        dot.node(n, lbl, style="filled", fillcolor="lightyellow")

for n in topo:
    for s in succ[n]:
        dot.edge(n, s)

dot.render("PERT_CPM", format="png", cleanup=True)  # 會輸出 PERT_CPM.png

