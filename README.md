# 安裝需要套件
!pip install graphviz matplotlib pandas

from collections import defaultdict, deque
import pandas as pd
import matplotlib.pyplot as plt
from graphviz import Digraph

# ------------------------------------------------------------
# 1. 定義任務 (ID, 名稱, 工期, 前置任務)
# ------------------------------------------------------------
tasks = [
    ("1", "研擬計畫", 1, []),            
    ("2", "任務分配", 4, ["1"]),
    ("3", "取得硬體", 17, ["1"]),
    ("4", "程式開發", 70, ["2"]),
    ("5", "安裝硬體", 10, ["3"]),
    ("6", "程式測試", 30, ["4"]),
    ("7", "撰寫使用手冊", 25, ["5"]),
    ("8", "轉換檔案", 20, ["5"]),
    ("9", "系統測試", 25, ["6"]),
    ("10", "使用者訓練", 20, ["7","8"]),
    ("11", "使用者測試", 25, ["9","10"]),
]

# ------------------------------------------------------------
# 2. 建立圖結構
# ------------------------------------------------------------
graph = defaultdict(list)
durations = {}
predecessors = defaultdict(list)
for tid, name, duration, preds in tasks:
    durations[tid] = duration
    for p in preds:
        graph[p].append(tid)
    predecessors[tid] = preds

# ------------------------------------------------------------
# 3. 拓樸排序 + 最早開始/完成時間
# ------------------------------------------------------------
indegree = {tid: 0 for tid, _, _, _ in tasks}
for tid in graph:
    for nxt in graph[tid]:
        indegree[nxt] += 1

queue = deque([tid for tid in indegree if indegree[tid] == 0])
ES, EF = {}, {}
while queue:
    cur = queue.popleft()
    ES[cur] = max([EF[p] for p in predecessors[cur]] or [0])
    EF[cur] = ES[cur] + durations[cur]
    for nxt in graph[cur]:
        indegree[nxt] -= 1
        if indegree[nxt] == 0:
            queue.append(nxt)

project_duration = max(EF.values())

# ------------------------------------------------------------
# 4. 計算最晚開始/完成時間 (LS, LF)
# ------------------------------------------------------------
LS, LF = {}, {}
for tid in reversed(list(EF.keys())):
    if not graph[tid]:  # 沒有後繼
        LF[tid] = project_duration
    else:
        LF[tid] = min([LS[nxt] for nxt in graph[tid]])
    LS[tid] = LF[tid] - durations[tid]

# ------------------------------------------------------------
# 5. 找出關鍵路徑 (slack=0)
# ------------------------------------------------------------
slack = {tid: LS[tid] - ES[tid] for tid in ES}
critical_path = [tid for tid in slack if slack[tid] == 0]

print("專案總工期:", project_duration)
print("關鍵路徑:", " → ".join(critical_path))

# ------------------------------------------------------------
# 6. 畫 PERT/CPM 圖
# ------------------------------------------------------------
dot = Digraph(format="png")
dot.attr(rankdir="LR")
for tid, name, duration, preds in tasks:
    label = f"{tid}. {name}\\n工期:{duration}天\\nES:{ES[tid]}, EF:{EF[tid]}\\nLS:{LS[tid]}, LF:{LF[tid]}"
    if tid in critical_path:
        dot.node(tid, label, shape="box", style="filled", color="lightcoral")
    else:
        dot.node(tid, label, shape="box")
    for p in preds:
        dot.edge(p, tid)
dot.render("pert_chart", cleanup=True)
print("PERT/CPM 圖已輸出: pert_chart.png")

# ------------------------------------------------------------
# 7. 畫甘特圖
# ------------------------------------------------------------
df = pd.DataFrame([
    {"任務": f"{tid}.{name}", "開始": ES[tid], "結束": EF[tid], "工期": durations[tid], "是否關鍵": tid in critical_path}
    for tid, name, _, _ in tasks
])

fig, ax = plt.subplots(figsize=(10,6))
for i, row in df.iterrows():
    color = "red" if row["是否關鍵"] else "skyblue"
    ax.barh(row["任務"], row["工期"], left=row["開始"], color=color, edgecolor="black")
    ax.text(row["開始"]+row["工期"]/2, i, f"{row['工期']}天", va="center", ha="center", color="black")
ax.set_xlabel("時間（天）")
ax.set_ylabel("任務")
ax.set_title("專案甘特圖")
plt.gca().invert_yaxis()
plt.show()
