**Level:** 🔴 Hard

**โจทย์:**
มีพนักงาน 9 คน ต้องการแบ่งออกเป็น 3 ทีม ทีมละ 3 คนพอดี โดยมีเงื่อนไขความขัดแย้งดังนี้:
- นาย 1 ห้ามอยู่ทีมเดียวกับนาย 2
- นาย 3 ต้องอยู่ทีมเดียวกับนาย 4
- นาย 5 ห้ามอยู่ทีมเดียวกับนาย 6 และ 7
ถ้าคุณเขียน Model ตรงๆ ไปเลย คุณอาจจะพบว่า Solver ใช้เวลาหานานมาก หรือให้คำตอบที่ซ้ำซาก (เช่น สลับชื่อทีม 1 กับทีม 2) 

**จงเพิ่มเงื่อนไข "การทำลายความสมมาตร (Symmetry Breaking)"** 1-2 บรรทัด เพื่อบังคับไม่ให้ Solver คิดซ้ำซ้อน และคายคำตอบออกมาภายในเสี้ยววินาที

---

เราจะทำการตั้งชื่อกลุ่มพร้อมกับลดเงื่อนไขขอโจทย์โดยการตั้งหัวหน้ากลุ่มให้นาย 1 อยู่ในกลุ่มที่ 1 และนาย 2 อยู่ในกลุ่มที่ 2 และกลุ่มที่ 3 ไม่มีหัวหน้ากลุ่ม เนื่องจากนาย 1 และนาย 2 ไม่ได้อยู่กลุ่มเดียวกันอย่างแน่นอจึงสามารถตัดเงื่อนไขนี้ตอนสร้าง model ได้
แสดงว่าตัวแปรที่เราต้องพิจารณาเหลือแค่ นาย 3 ถึงนาย 9 แสดงว่า model ของเราจะเป็นดังนี้

**Step 1: Import and Construct Tools**

```python
from ortools.sat.python import cp_model
model = cp_model.CpModel()
```

**Step 2: Create Decision Varaibles** 

เราต้องการให้ทุกคนเลือกกลุ่มแสดงว่าเราสามารถตั้งตัวแปรเป็น Integer ได้ซึ่งค่าที่เป็นไปได้คือ 1,2 หรือ 3 เท่านั้น

$$
\forall i \leq 9, p_i \in \lbrace 1,2,3 \rbrace
$$

```python
p = {}
p[0] = model.NewConstant(1)
p[1] = model.NewConstant(2)
for i in range(2,9):
  p[i] = model.NewTntVar(1,3,f'Mr.{i + 1}')
```

**Step 3: Add Constraints**

- นาย 3 ต้ออยู่ทีมเดียวกับนาย 4 

$$
p_3 = p_4
$$

```python
model.Add(p[2] == p[4])
```

- นาย 5 ห้ามอยู่ทีมเดียวกับนาย 6 และ 7

$$
p_5 \neq p_6 \wedge p_5 \neq p_7
$$

```python
model.Add(p[4] != p[5])
model.Add(p[4] != p[6])
```

- เงื่อนไขสำคัญ: ทุกทีมต้องมีสมาชิก 3 คนพอดี

```python
for team in range(1, 4):
    model.Add(sum(p[i] == team for i in range(9)) == 3)
```

**Step 4: Add Objective**

```python
solver = cp_model.CpSolver()
status = solver.Solve(model)
```

**Step 5: Print Result**

```python
if status == cp_model.FEASIBLE or status == cp_model.OPTIMAL:
    for i in range(9):
        print(f"นาย {i + 1} อยู่ทีมที่ {solver.Value(p[i])}")
else:
    print("This problem has no feasible answer")
```
