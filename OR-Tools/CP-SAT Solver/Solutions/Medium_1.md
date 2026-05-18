# **Level:** 🟡 Medium 1

**โจทย์:**
คุณกำลังออกแบบด่านให้เกมดนตรีที่มีเลนกด 4 เลน (Lane 1-4) ใน 1 วินาทีมีจุดที่สามารถวางตัวโน้ตได้ 10 จังหวะ (t=0 ถึง t=9) 
จงวางตัวโน้ตให้ได้ **จำนวนมากที่สุด** โดยมีกติกาความยากดังนี้:
1. ห้ามมีตัวโน้ตเกิน 2 เลน ในจังหวะเวลาเดียวกัน (เดี๋ยวนิ้วผู้เล่นพันกัน)
2. ถ้าเลนที่ 1 มีตัวโน้ตในจังหวะที่ $t$ แล้ว เลนที่ 1 ต้องว่างในจังหวะที่ $t+1$ เสมอ (ป้องกันการกดรัวเกินไป)
3. ต้องมีตัวโน้ตอย่างน้อย 1 ตัวเสมอในทุกๆ 3 จังหวะติดกัน (ห้ามปล่อยให้เกมเงียบเกินไป)

---

**Step 1: import and construct tools**

```python
from ortools.sat.python import cp_model
model = cp_model.CpModel()
```

**Step 2: create decision variables** 

เนื่องจากเรามี 4 เลน และ 10 จังหวะ เราสามารถจิตนาการเป็นตาราง 4*10 ซึ่งในแต่ละช่องผลลัพท์ที่เป็นไปได้จะกลายเป็นใส่หรือไม่ใส่เท่านั้น
ถ้าเรามองในมุมมองของการกำหนดตัวแปรในคณิตศาสตร์ก็จะเป็น

$$
(x_{i,j})_{10 \times 4} \text{ โดยที่ } \forall i,j, \left( x_{i,j} \in \lbrace \text{True}, \text{False} \rbrace \right)
$$

เขียนลง python เป็น

```python
x = {}
for i in range(10):
  for j in range(4):
    x[i,j] = model.NewBoolVar(f'Position ({i}, {j})')
```

**Step 3: add constraints**

- ห้ามมีตัวโน้ตเกิน 2 เลน ในจังหวะเวลาเดียวกัน นั้นคือทุกจังหวะถ้ามี 2 ตัวที่เป็น True แล้วที่เหลือจะเป็น False ซึ่งตัวแปรที่เรกำหนดสามารถ represent เป็น 0, 1 ได้เช่นกัน

$$
\forall i \sum_{j} x_{i, j} \leq 2
$$

```python
for i in range(10):
  model.Add(sum([x[i,j] for j in range(4)]) <= 2)
```

- ถ้าเลนที่ 1 มีตัวโน้ตในจังหวะที่ $t$ แล้ว เลนที่ 1 ต้องว่างในจังหวะที่ $t+1$ เสมอ

$$
\forall i < 10,\forall j \left( x_{i,j} \rightarrow \neg x_{i + 1,j} \right)
$$

```python
for i in range(9):
  for j in range(4):
    model.AddImplication(x[i, j],x[i + 1, j].Not())
```

- ต้องมีตัวโน้ตอย่างน้อย 1 ตัวเสมอในทุกๆ 3 จังหวะติดกัน

$$
\forall 1 \leq i \leq 8,\forall j\left( x_{i,j} \vee x_{i + 1,j} \vee x_{i + 2, j}  \right)
$$

```python
for i in range(8):
  model.Add(sum([x[t, j] for t in range(i, i+3) for j in range(4)]) >= 1)
```

**Step 4:  create objective**

เราต้องการให้จำนวนโน้ตเยอะที่สุดแสดงว่าเราจะเลือก Maximize

$$
\sum_i \sum_j x_{i,j}
$$

```python
model.Maximize(sum([x[i,j] for i in range(10) for j in range(4)]))
```

**Step 5: find result and show result**

```python
solver = cp_model.CpSolver()
status = solver.Solve(model)

if status == cp_model.OPTIMAL or status == cp_model.FEASIBLE:
    for i in range(10):
      # เก็บตัวอักษรของแต่ละเลนในจังหวะที่ i
      row_display = []
      for j in range(4):
        if solver.BooleanValue(x[i,j]):
          row_display.append("O") # ใส่โน้ต
        else:
          row_display.append("_") # เลนว่าง
      
      # ปริ้นท์ทั้ง 4 เลน ติดกัน (เช่น _ O O _)
      print(" ".join(row_display))
else:
    print('This model dont have a feasible result')
```
