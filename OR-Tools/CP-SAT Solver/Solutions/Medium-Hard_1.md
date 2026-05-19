**Level:** 🟠 Medium-Hard

**โจทย์:**
คุณรับงานฟรีแลนซ์เขียนโปรแกรม โดยมีโปรเจกต์ให้เลือกทำ 5 โปรเจกต์ (A, B, C, D, E) แต่ละโปรเจกต์ใช้เวลาทำไม่เท่ากัน (หน่วยเป็นวัน) แต่ละโปรเจกต์ให้ผลตอบแทนดังนี้:
- A: กำไร 120.50 บาท (ใช้เวลา 2 วัน)
- B: กำไร 340.75 บาท (ใช้เวลา 5 วัน)
- C: กำไร 85.00 บาท (ใช้เวลา 1 วัน)
- D: กำไร 210.25 บาท (ใช้เวลา 3 วัน)
- E: กำไร 450.50 บาท (ใช้เวลา 6 วัน)

คุณมีเวลาว่างรวมแค่ 10 วัน จงหาว่าควรรับงานไหนบ้างเพื่อให้ได้กำไรสูงสุด?

---

**Step 1: import tools and create model**

```python
from ortools.sat.python import cp_model
model = cp_model.CpModel()
```

**Step 2: construct decision variables**

เราทราบว่าปัญหาที่เราทำนั้นคือการเลือกว่าเราจะทำงานไหนบ้างแสดงว่าเราสามารถสร้างตัวแปรได้เป็น 

$$
\forall i \leq 5, a_i \in \lbrace 0, 1 \rbrace
$$

```python
a = {}
for i in range(5):
  a[i] = model.NewBoolVar(f'job {i}')
```

**Step 3: add constraints**

เรามีเงื่อนไขเพียงเวลาที่จำกัดที่ 10 วันนั้นคือ 

$$
2a_1 + 5a_2 + a_3 + 3a_4 + 6a_5 \leq 10
$$

```python
model.Add(2*a[0] + 5*a[1] + a[2] + 3*a[3] + 6*a[4] <= 10)
```

**Step 4: add objectives**

เราต้องการให้กำไรมากที่สุดแสดงว่าเราต้อง maximiza (เครื่องมือนี้เราสาารถป้อนข้อมูลท่เป็นจำนวนเต็มได้อย่างเดียว)

$$
12050a_1 + 34075a_2 + 8500a_3 + 21025a_4 + 45050a_5
$$

```python
model.Maximize(12050 * a[0] + 34075 * a[1] + 8500 * a[2] + 21025 * a[3] + 45050 * a[4])
```

**Step 5: solve and print results**

```python
solver = cp_model.CpSolver()
status = solver.Solve(model)

if status == cp_model.OPTIMAL or status == cp_model.FEASIBLE:
  print(f"Profit: {solver.ObjectiveValue() / 100} บาท")
  name = ['A', 'B', 'C', 'D', 'E']
  for i in range(5):
    if solver.BooleanValue(a[i]):
      print(name[i])
else:
  print("This problem has no feasible answer")
```
