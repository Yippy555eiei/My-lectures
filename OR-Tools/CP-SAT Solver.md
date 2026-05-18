CP-SAT Solver เป็นเครื่องมือที่ใช้ในการแก้ปัญหา Optimization และ การหาความเป็นไปได้ โดยใช้ Constraints Programming (การแจกแจงปัญหาเป็นเงื่อนไข) กับตัวแปรที่เป็น Integer, Boolean, Interval
**วิธีการใช้ CP-SAT Solver กับ python**
- **เลือกเครื่องมือที่ต้องการ** ในหัวข้อนี้เราเลือก CP-SAT Solver

```python
from ortools.sat.python import cp_model
model = cp_model.CpModel()
```

- **การกำหนดตัวแปร** ที่นิยม้กันมีดังนี้
  - Integer Variable สามารถกำหนดขอบเขตของค่าที่เป็นไปได้ได้

```python
# สร้างตัวแปรใน model ชื่อ name โดยมีค่าที่เป็นไปได้คือ -100 ถึง 100 และสามารถเรียกตัวแปรดังกล่าไปสร้างเงื่อนไขอื่นได้โดยเรียก var
var = model.NewIntVar(-100,100,'name')
```

  - Boolean Variable

```python
# สร้างตัวแปรใน model ชื่อ name โดยมีค่าที่เป็นไปได้คือ True ถึง False และสามารถเรียกตัวแปรดังกล่าไปสร้างเงื่อนไขอื่นได้โดยเรียก var
var = model.NewBoolVar('name')
```

  - Interval Variable เป็นตัวแปรที่จะล็อกความยาวช่วงได้

```python
# สร้างจุดเริ่มและจุดสิ้นสุดของเวลา
start = model.NewIntVar(0, 24, 'start')
end = model.NewIntVar(0, 24, 'end')

# สร้าง Interval ผูก start และ end เข้าด้วยกันด้วยระยะเวลา 5
var = model.NewIntervalVar(start_time, 5, end_time, 'job_1')
```

  - Optional Interval Variable เป็น Interval Variable + Boolaan (เลือกทำหรือไม่ก็ได้)

```python
# ถ้า is_performed = 0 (ไม่ทำ) ตัวแปร Interval นี้จะล่องหนไปเลย ไม่เอามาคิดกติกาชนกัน
var = model.NewOptionalIntervalVar(start, duration, end, is_performed, 'opt_job')
```

- **การกำหนดเงื่อนไข**
  - Linear Constraints

```python
# ผลรวมของ x และ y ต้องไม่เกิน 10
model.Add(x + y <= 10)

# x ต้องไม่เท่ากับ y
model.Add(x != y)

# สามารถเขียนแบบช่วง (Range) ได้เลย! (x อยู่ระหว่าง 5 ถึง 10)
model.AddLinearConstraint(x, 5, 10)
```

  - Global Constraints

```python
# 1. AllDifferent: ทุกตัวแปรในลิสต์นี้ "ห้ามมีค่าซ้ำกันเลย" (เหมาะกับ Sudoku, จัดตารางเวร)
model.AddAllDifferent([x, y, z])

# 2. NoOverlap: ห้ามให้ช่วงเวลา (Interval) ทับซ้อนกัน (หัวใจของการจัดตารางงาน)
model.AddNoOverlap([job1_interval, job2_interval, job3_interval])

# 3. MaxEquality / MinEquality: หาค่าสูงสุด/ต่ำสุดจากกลุ่มตัวแปร
# บังคับให้ตัวแปร max_val มีค่าเท่ากับค่าที่มากที่สุดในกลุ่ม [x, y, z]
model.AddMaxEquality(max_val, [x, y, z])
```

  - Logical Constraints

```python
# 1. Implication (ถ้า A เป็นจริง แล้ว B ต้องเป็นจริงด้วย)
# เช่น ถ้าพนักงาน A มาทำงาน (a=1) เครื่องจักร B ต้องเปิด (b=1)
model.AddImplication(a, b)

# 2. BoolOr (อย่างน้อย 1 ตัวต้องเป็นจริง)
# ใน 3 คนนี้ ต้องมีอย่างน้อย 1 คนที่ได้เข้ากะดึก
model.AddBoolOr([is_a_night, is_b_night, is_c_night])

# 3. BoolXOr (ต้องเป็นจริงแค่ตัวเดียวเท่านั้น! ห้ามขาด ห้ามเกิน)
model.AddBoolXOr([is_a_manager, is_b_manager])
```

  - OnlyEnforceIf

```python
b = model.NewBoolVar('b') # สมมติ b คือ "กดปุ่มเปิดเครื่อง"

# ถ้ากดปุ่ม (b=1) เครื่องจักรต้องผลิตสินค้าได้มากกว่า 100 ชิ้น
model.Add(production >= 100).OnlyEnforceIf(b)

# ถ้าไม่กดปุ่ม (b=0 หรือ b.Not()) เครื่องจักรต้องผลิตสินค้า = 0
model.Add(production == 0).OnlyEnforceIf(b.Not())
```

- **การกำหนดจุดประสงค์**

```python
# หาคำตอบที่เป็นไปได้
solver.Solve(model)
# หาค่า minimize
model.minimize(x)
# หาค่า maximize
model.maximize(x)
```

- **การแสดงคำตอบ**
  - เช็ค status คำตอบก่อนเสมอ
    - `cp_model.OPTIMAL`: เจอคำตอบที่ดีที่สุดแล้ว! (เพอร์เฟกต์)
    - `cp_model.FEASIBLE`: เจอคำตอบที่ถูกกฎ แต่ยังไม่รับประกันว่าดีที่สุด (มักจะเกิดตอนเราตั้งเวลาจำกัด Time Limit ไว้แล้วมันหมดเวลาก่อน)
    - `cp_model.INFEASIBLE`: กฎขัดแย้งกันเอง ไม่มีทางเป็นไปได้ในจักรวาลนี้ (UNSAT)
  - Value ของคำตอบ
  หาคำตอบเดียว

```python
# จะได้ตัวเลขผลกำไรสูงสุด หรือ เวลาที่น้อยที่สุดออกมา
print(f"กำไรสูงสุดที่ทำได้คือ: {solver.ObjectiveValue()} บาท")

# 1. ตัวแปร Integer (ได้เลขจำนวนเต็ม)
print(f"จำนวนไก่: {solver.Value(chickens)} ตัว")

# 2. ตัวแปร Boolean (จะได้ค่า 0 หรือ 1)
print(f"เปิดเครื่องจักรหรือไม่?: {solver.Value(machine_on)}")

# 3. ตัวแปร Boolean แบบ True/False (ใช้ใน If-Else ง่ายกว่า)
if solver.BooleanValue(machine_on):
    print("เครื่องจักรทำงานอยู่")
```

  หาความเป็นไปได้ทุกคำตอบ

```python
class SolutionPrinter(cp_model.CpSolverSolutionCallback):
    def __init__(self, variables):
        cp_model.CpSolverSolutionCallback.__init__(self)
        self.variables = variables
        self.solution_count = 0

    # ฟังก์ชันนี้จะถูกเรียกอัตโนมัติ ทุกครั้งที่ Solver เจอคำตอบใหม่!
    def on_solution_callback(self):
        self.solution_count += 1
        print(f"คำตอบรูปแบบที่ {self.solution_count}:")
        for v in self.variables:
            print(f"  {v.Name()} = {self.Value(v)}")

# วิธีใช้งาน:
solver = cp_model.CpSolver()
# บังคับให้ Solver วิ่งหาทุกคำตอบจนกว่าจะหมดจักรวาลความน่าจะเป็น
solver.parameters.enumerate_all_solutions = True 

# โยน SolutionPrinter เข้าไปจับตาดูผลลัพธ์
printer = SolutionPrinter([var_a, var_b, var_c])
solver.Solve(model, printer)

print(f"ค้นพบคำตอบทั้งหมด: {printer.solution_count} รูปแบบ")
```
