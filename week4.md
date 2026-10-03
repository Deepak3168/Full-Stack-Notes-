# WEEK 4 — Object-Oriented Programming (OOP) + Design Thinking
### Student Guide

> **Read this like a story, not a dictionary.** Every topic follows the same path:
> **Real-life idea → Why we need it → Code → Run it → Common mistakes → Try it yourself.**
> Type every example yourself. Do not copy-paste. Your fingers learn what your eyes skip.

---

## 📖 What You Will Learn This Week

| Part | Topics |
|---|---|
| **Part 1 — OOP** | Class, Object, `__init__`, Instance & Class attributes, Instance / Class / Static methods, Encapsulation, Inheritance, Multiple inheritance, Polymorphism, Abstraction, Abstract classes & methods, Composition, Aggregation |
| **Part 2 — Design Thinking** | How to decide which classes to create |
| **Part 3 — Design Patterns** | Factory, Strategy, Observer, Singleton, Adapter |
| **Part 4 — Mini Project** | Banking System (build it step by step) |

**Time needed:** about 7 days, ~2–3 hours a day.

---

# PART 1 — OOP

## 0. Why do we even need OOP?

Imagine you are writing a program for a college. You store a student like this:

```python
student1_name = "Ravi"
student1_marks = 85
student2_name = "Sita"
student2_marks = 92
# 500 students → 1000 variables. Chaos!
```

Better idea: use dictionaries.

```python
student1 = {"name": "Ravi", "marks": 85}
student1["mraks"] = 90      # typo! Python silently creates a NEW key. No error.
```

Still not great: no rules, and the data and the functions that work on it live in different places.

**OOP idea:** put the **data** and the **actions** that belong together into **one package** called a **class**.

> 🧠 **One-line summary of OOP:** *Model real things in your program, the way they exist in real life — each thing has its own information and its own abilities.*

---

## 1. Class and Object

### Real-life idea
- A **class** is a **blueprint** (like the design of a mobile phone).
- An **object** is a **real thing made from that blueprint** (your phone, my phone, a friend's phone).

One blueprint → many objects. Each object is separate.

### Code

```python
class Student:
    pass                      # empty class for now

s1 = Student()                # create an object
s2 = Student()                # create another one

print(type(s1))               # <class '__main__.Student'>
print(s1 is s2)               # False  → two different objects
```

- `class Student:` → define the blueprint. **Class names use `CapitalWords`.**
- `Student()` → build an object. This is called **instantiation**.

### 🧪 Try it
Create a class `Phone` and make 3 objects from it. Print `type()` of each.

---

## 2. `__init__` and `self`

### The problem
Our `Student` objects are empty. We want each student to start with a **name** and **marks**.

### `__init__`: the "setup" method
Python automatically runs `__init__` **the moment you create an object**.

```python
class Student:
    def __init__(self, name, marks):
        self.name = name
        self.marks = marks

s1 = Student("Ravi", 85)
s2 = Student("Sita", 92)

print(s1.name, s1.marks)      # Ravi 85
print(s2.name, s2.marks)      # Sita 92
```

### What is `self`?
`self` means **"this particular object."**

When you write `Student("Ravi", 85)`, Python does this:
1. Creates an empty object.
2. Calls `__init__(that_object, "Ravi", 85)`.
3. Inside, `self` **is** that object — so `self.name = name` means *"store the name inside this object."*

```
 name  ──────────►  the value you passed in ("Ravi")
 self.name  ─────►  the value stored inside the object
```

### Common mistakes

```python
class Student:
    def __init__(name, marks):      # ❌ forgot self → TypeError
        ...

class Student:
    def __init__(self, name, marks):
        name = name                  # ❌ stores nothing! Needs self.name = name
```

### Default values

```python
class Student:
    def __init__(self, name, marks=0):     # marks is optional
        self.name = name
        self.marks = marks

print(Student("Anil").marks)               # 0
```

### 🧪 Try it
Create a `Book` class with `title`, `author`, `pages`. Make 2 books and print their details.

---

## 3. Instance Attributes vs Class Attributes

| | Instance attribute | Class attribute |
|---|---|---|
| Belongs to | **one object** | **the whole class** (shared) |
| Defined | inside `__init__` using `self.` | directly inside the class |
| Example | each student's name | the college name (same for all) |

```python
class Student:
    college = "Sunrise College"        # class attribute: same for everyone
    total_students = 0                 # class attribute used as counter

    def __init__(self, name):
        self.name = name               # instance attribute: different per student
        Student.total_students += 1

s1 = Student("Ravi")
s2 = Student("Sita")

print(s1.name, s2.name)                # Ravi Sita
print(s1.college, s2.college)          # Sunrise College Sunrise College
print(Student.total_students)          # 2
```

If the college changes its name, change it **once**:

```python
Student.college = "Moonlight College"
print(s1.college)                      # Moonlight College  (all objects see the change)
```

### ⚠️ Biggest trap: a list as a class attribute

```python
class Team:
    members = []                       # ❌ ONE list shared by ALL teams

    def add(self, name):
        self.members.append(name)

t1 = Team()
t2 = Team()
t1.add("Ravi")
print(t2.members)                      # ['Ravi']  ← t2 never added anyone!
```

**Fix:** anything that should be separate for every object (lists, dicts) goes in `__init__`:

```python
class Team:
    def __init__(self):
        self.members = []              # ✅ every team gets its own list
```

> 📌 **Rule:** Same for everyone → class attribute. Different per object → instance attribute.

### 🧪 Try it
Make a `Employee` class with a class attribute `company = "TechNova"` and an instance attribute `name`. Add a counter that tracks how many employees were created.

---

## 4. Instance Methods

Functions written **inside a class** are called **methods**. An **instance method** works on one specific object, so it takes `self`.

```python
class Student:
    def __init__(self, name):
        self.name = name
        self.marks = []

    def add_marks(self, mark):
        self.marks.append(mark)

    def average(self):
        if not self.marks:
            return 0
        return sum(self.marks) / len(self.marks)

    def __str__(self):                      # what print(student) shows
        return f"{self.name} (avg: {self.average():.1f})"


s = Student("Ravi")
s.add_marks(80)
s.add_marks(90)
print(s.average())     # 85.0
print(s)               # Ravi (avg: 85.0)
```

### Handy "magic" methods (also called dunder methods)

| Method | Runs when… | Example |
|---|---|---|
| `__init__` | object is created | `Student("Ravi")` |
| `__str__` | `print(obj)` / `str(obj)` | friendly text for users |
| `__repr__` | you inspect an object / debugging | `Student('Ravi')` |
| `__len__` | `len(obj)` | number of marks |
| `__eq__` | `obj1 == obj2` | compare two objects |

```python
class Student:
    def __init__(self, name):
        self.name = name
        self.marks = []
    def __len__(self):
        return len(self.marks)
    def __repr__(self):
        return f"Student({self.name!r})"

s = Student("Ravi")
s.marks = [70, 80]
print(len(s))          # 2
print(repr(s))         # Student('Ravi')
```

### 🧪 Try it
Make a `ShoppingCart` class with `add_item(name, price)`, `total()`, and `__len__` (number of items).

---

## 5. Class Methods (`@classmethod`)

### Problem
Sometimes data arrives in a different shape, like a text line `"Ravi,85"` from a file. We want an **alternative way to create an object**.

### Solution

```python
class Student:
    def __init__(self, name, marks):
        self.name = name
        self.marks = marks

    @classmethod
    def from_string(cls, text):            # cls = the class itself (Student)
        name, marks = text.split(",")
        return cls(name.strip(), int(marks))


s = Student.from_string("Ravi, 85")
print(s.name, s.marks)                      # Ravi 85
```

- `self` → the **object**. `cls` → the **class**.
- A class method doesn't need an existing object; you call it on the class.
- Perfect for **alternative constructors**: `from_string`, `from_dict`, `from_file`.

### 🧪 Try it
Add `from_dict({"name": "Sita", "marks": 92})` to `Student`.

---

## 6. Static Methods (`@staticmethod`)

A **static method** is just a normal function that lives inside the class because it **belongs to that topic**. It gets **neither `self` nor `cls`**.

```python
class Student:
    PASS_MARK = 35

    def __init__(self, name, marks):
        self.name = name
        self.marks = marks

    @staticmethod
    def is_valid_mark(mark):               # no self, no cls
        return 0 <= mark <= 100


print(Student.is_valid_mark(105))          # False
print(Student.is_valid_mark(80))           # True
```

### Which method type should I use?

```
Does it need the specific object's data?      → instance method   (self)
Does it need the class, e.g. to build objects? → class method      (cls)
Needs neither, just a helper?                  → static method
```

| Type | Decorator | First parameter | Typical use |
|---|---|---|---|
| Instance | none | `self` | normal actions: `add_marks()` |
| Class | `@classmethod` | `cls` | alternative creation: `from_string()` |
| Static | `@staticmethod` | nothing | helpers: `is_valid_mark()` |

### 🧪 Try it
In a `Temperature` class, add a static method `celsius_to_fahrenheit(c)`.

---

## 7. Encapsulation

### Real-life idea
At an ATM you can't walk into the bank's vault and change your balance. You use **buttons** (deposit, withdraw) and the machine **checks the rules**.

### The problem

```python
class BankAccount:
    def __init__(self, balance):
        self.balance = balance

acc = BankAccount(1000)
acc.balance = -50000          # anybody can set anything! No rules.
```

### Encapsulation = hide the data, allow changes only through safe methods

Python uses naming conventions:

| Name | Meaning |
|---|---|
| `name` | public: anyone can use |
| `_name` | "protected": *please don't touch from outside* (just a polite request) |
| `__name` | "private": Python renames it to `_ClassName__name` to make accidental access harder |

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.__balance = balance                  # private

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Deposit must be positive")
        self.__balance += amount

    def withdraw(self, amount):
        if amount > self.__balance:
            raise ValueError("Not enough money")
        self.__balance -= amount

    def get_balance(self):
        return self.__balance


acc = BankAccount("Ravi", 1000)
acc.deposit(500)
print(acc.get_balance())       # 1500
# print(acc.__balance)         # ❌ AttributeError
```

> 💡 Python never *truly* hides things (`acc._BankAccount__balance` still works). Encapsulation in Python is about **good manners and clear intent**.

### The Pythonic way: `@property`

Instead of `get_balance()`, use a **property** so it looks like a normal attribute:

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.__balance = balance

    @property
    def balance(self):                 # reading: acc.balance
        return self.__balance

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Deposit must be positive")
        self.__balance += amount


acc = BankAccount("Ravi", 1000)
acc.deposit(200)
print(acc.balance)             # 1200   (looks like an attribute!)
# acc.balance = 99999          # ❌ AttributeError: no setter → protected ✅
```

### Property with a setter (validation on assignment)

```python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age = age                 # this calls the setter below!

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, value):
        if not (3 <= value <= 100):
            raise ValueError("Age must be between 3 and 100")
        self._age = value


s = Student("Ravi", 20)
s.age = 21                  # fine
# s.age = -5                # ❌ ValueError: setter blocked it
```

### 🧪 Try it
Create a `Player` class in a game with a private `__health` (0–100). Add `take_damage(n)` and `heal(n)` so health never goes below 0 or above 100.

---

## 8. Inheritance

### Real-life idea
A **child inherits** features from parents. In code, a **child class inherits** attributes and methods from a **parent class**. This avoids rewriting the same code.

### The problem
A company has Developers and Managers. Both have a name and salary, and both need `show_info()`. Writing it twice is wasteful.

### Solution

```python
class Employee:                              # parent / base class
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

    def show_info(self):
        print(f"{self.name} earns ₹{self.salary}")


class Developer(Employee):                   # child / derived class
    def code(self):
        print(f"{self.name} is writing code")


class Manager(Employee):
    def hold_meeting(self):
        print(f"{self.name} is in a meeting")


d = Developer("Ravi", 60000)
m = Manager("Sita", 90000)

d.show_info()          # inherited from Employee
d.code()               # Developer's own
m.show_info()
m.hold_meeting()
```

`Developer(Employee)` reads as **"Developer is an Employee."**

### Overriding: child changes the parent's behaviour

```python
class Manager(Employee):
    def show_info(self):                     # same name → overrides parent's
        print(f"Manager {self.name} earns ₹{self.salary}")
```

### `super()`: reuse the parent's code and add more

```python
class Developer(Employee):
    def __init__(self, name, salary, language):
        super().__init__(name, salary)       # let Employee set name & salary
        self.language = language             # add our own

d = Developer("Ravi", 60000, "Python")
print(d.name, d.language)                    # Ravi Python
```

### Useful checks

```python
print(isinstance(d, Developer))     # True
print(isinstance(d, Employee))      # True  (a Developer IS an Employee)
print(issubclass(Developer, Employee))  # True
```

### ✅ The "is-a" test
Before inheriting, say the sentence out loud:
- "Developer **is an** Employee" ✅ inherit
- "Car **is an** Engine" ❌ wrong. A car **has an** engine → use *composition* (Section 13).

### 🧪 Try it
Make a `Vehicle` class (brand, speed). Create `Car` and `Bike` that inherit from it and add something unique to each.

---

## 9. Multiple Inheritance

A class can inherit from **more than one** parent.

### Real-life idea
A **smartphone** is a phone *and* a camera *and* a music player.

```python
class Phone:
    def call(self, number):
        print(f"Calling {number}...")

class Camera:
    def take_photo(self):
        print("Click! 📸")

class MusicPlayer:
    def play(self, song):
        print(f"Playing {song} 🎵")


class SmartPhone(Phone, Camera, MusicPlayer):
    pass


sp = SmartPhone()
sp.call("98765")
sp.take_photo()
sp.play("Naatu Naatu")
```

### What if two parents have the same method name?

Python follows an order called **MRO (Method Resolution Order)**: left to right in the class line.

```python
class A:
    def hello(self): print("Hello from A")

class B:
    def hello(self): print("Hello from B")

class C(A, B):
    pass

C().hello()                 # Hello from A  (A is listed first)
print(C.__mro__)            # (C, A, B, object)
```

### Mixins: the practical use

A **mixin** is a small class that adds *one* extra ability. You "mix" it into other classes.

```python
import json

class JSONMixin:
    def to_json(self):
        return json.dumps(self.__dict__)

class PrintMixin:
    def show(self):
        print(f"{type(self).__name__}: {self.__dict__}")


class Product(JSONMixin, PrintMixin):
    def __init__(self, name, price):
        self.name = name
        self.price = price

p = Product("Keyboard", 1200)
print(p.to_json())          # {"name": "Keyboard", "price": 1200}
p.show()                    # Product: {'name': 'Keyboard', 'price': 1200}
```

Now `User`, `Order`, `Invoice` can reuse the same two mixins for free.

> ⚠️ Keep it simple. If you can't explain the MRO of your class easily, redesign.

### 🧪 Try it
Build `LoggerMixin` (a `log(msg)` method) and `TimestampMixin` (`created_at` attribute), then make a `Task` class using both.

---

## 10. Polymorphism

*Poly = many, morph = forms → "many forms."*

> **Same method name, different behaviour depending on the object.**

### Real-life idea
Pressing the **"Pay"** button on an app: if you chose UPI it opens UPI, if you chose Card it asks for card details. The button is the same; the behaviour differs.

### Without polymorphism (painful)

```python
def pay(method, amount):
    if method == "upi":
        print(f"Paid ₹{amount} via UPI")
    elif method == "card":
        print(f"Paid ₹{amount} via Card")
    elif method == "cash":
        print(f"Paid ₹{amount} in Cash")
    # New method? Edit this function again and again.
```

### With polymorphism

```python
class UPI:
    def pay(self, amount):
        print(f"Paid ₹{amount} via UPI")

class Card:
    def pay(self, amount):
        print(f"Paid ₹{amount} via Card")

class Cash:
    def pay(self, amount):
        print(f"Paid ₹{amount} in Cash")


def checkout(payment_method, amount):
    payment_method.pay(amount)       # checkout doesn't care WHICH one it is


for method in (UPI(), Card(), Cash()):
    checkout(method, 499)
```

Output:
```
Paid ₹499 via UPI
Paid ₹499 via Card
Paid ₹499 in Cash
```

Add `NetBanking` tomorrow → write one new class. `checkout()` stays unchanged. 🎉

### Three flavours of polymorphism in Python

**1. Method overriding** (via inheritance, see Section 8)

```python
class Report:
    def export(self):
        print("Exporting generic report")

class PDFReport(Report):
    def export(self):
        print("Exporting as PDF")
```

**2. Duck typing:** *"If it walks like a duck and quacks like a duck, it's a duck."* Python doesn't check the class; it only checks that the **method exists**. No inheritance needed (the `UPI/Card/Cash` example above is duck typing).

**3. Operator & built-in polymorphism**

```python
print(5 + 3)            # 8       (numbers add)
print("5" + "3")        # 53      (strings join)
print([1] + [2])        # [1, 2]  (lists merge)

print(len("hello"))     # 5
print(len([1, 2, 3]))   # 3
```

You can give your own class this power with dunder methods:

```python
class Money:
    def __init__(self, amount):
        self.amount = amount

    def __add__(self, other):
        return Money(self.amount + other.amount)

    def __repr__(self):
        return f"Money({self.amount})"


print(Money(100) + Money(50))     # Money(150)
```

### 🧪 Try it
Create `EmailNotifier`, `SMSNotifier`, `WhatsAppNotifier`, each with `send(message)`. Write one function `alert_all(notifiers, message)` that loops through them.

---

## 11. Abstraction (Abstract Classes & Abstract Methods)

### Real-life idea
When you drive a car you use the **steering, brake and accelerator**. You don't need to know *how* the engine burns fuel. The complicated details are **hidden**; only a simple, required interface is shown.

> **Abstraction = define WHAT must be done; leave HOW to the child classes.**

### The problem
You're making a game. Every character **must** be able to `attack()`. But a Warrior attacks differently from a Mage. How do you *force* every new character to define `attack()`?

### Solution: Abstract Base Class (ABC)

```python
from abc import ABC, abstractmethod

class GameCharacter(ABC):               # abstract class
    def __init__(self, name):
        self.name = name

    @abstractmethod
    def attack(self):                   # abstract method: no body, just a promise
        pass

    def introduce(self):                # normal method: shared by everyone
        print(f"I am {self.name}")


class Warrior(GameCharacter):
    def attack(self):
        print(f"{self.name} swings a sword ⚔️")

class Mage(GameCharacter):
    def attack(self):
        print(f"{self.name} casts a fireball 🔥")


w = Warrior("Arjun")
m = Mage("Meera")
w.introduce()
w.attack()
m.attack()
```

### What does the abstract class protect us from?

```python
# g = GameCharacter("Test")      # ❌ TypeError: can't create object of an abstract class

class Archer(GameCharacter):
    pass                         # forgot attack()!

# a = Archer("Ravi")             # ❌ TypeError right here, immediately
```

You find the mistake **instantly**, not in the middle of a live game.

### Key points
- An **abstract class** can't be instantiated; it's a template.
- An **abstract method** has no real code; every child **must** implement it.
- An abstract class **can also have normal methods** (like `introduce`).
- Needs `from abc import ABC, abstractmethod`.

### Abstraction vs Encapsulation (common confusion!)

| | Abstraction | Encapsulation |
|---|---|---|
| Question it answers | *"What should this thing do?"* | *"How do I protect its data?"* |
| Tool | abstract classes, interfaces | private/protected, properties |
| Example | every character must `attack()` | `__health` can't be set to -50 |

### 🧪 Try it
Create an abstract class `FileReader` with abstract method `read(path)`. Make `TextReader` and `CSVReader` that implement it.

---

## 12. Composition ("HAS-A")

### Real-life idea
A **computer has a CPU, RAM and a hard disk.** A **house has rooms.** The computer is *not* a CPU, so we don't inherit; we **include** the parts inside it.

> **Composition = a class contains objects of other classes as its parts. The parts cannot live without the whole.**

```python
class CPU:
    def __init__(self, model):
        self.model = model

    def run(self):
        print(f"{self.model} is processing...")


class RAM:
    def __init__(self, size_gb):
        self.size_gb = size_gb


class Computer:
    def __init__(self, cpu_model, ram_gb):
        self.cpu = CPU(cpu_model)         # Computer CREATES its own parts
        self.ram = RAM(ram_gb)

    def start(self):
        print(f"Starting with {self.ram.size_gb}GB RAM")
        self.cpu.run()


pc = Computer("Ryzen 5", 16)
pc.start()
```

Destroy the computer → its CPU and RAM objects go with it.

### Another example: Order and its items

```python
class OrderItem:
    def __init__(self, name, price, qty):
        self.name, self.price, self.qty = name, price, qty

    def total(self):
        return self.price * self.qty


class Order:
    def __init__(self):
        self.items = []

    def add_item(self, name, price, qty=1):
        self.items.append(OrderItem(name, price, qty))     # Order creates items

    def total(self):
        return sum(item.total() for item in self.items)


o = Order()
o.add_item("Pen", 10, 5)
o.add_item("Notebook", 50, 2)
print(o.total())          # 150
```

### Why prefer composition over inheritance?
Inheriting makes the child tightly attached to the parent. Composition is flexible: you can swap parts without breaking the whole.

```python
# ❌ Bad: Stack inherits from list, so it also gets insert(), sort(), remove()...
class BadStack(list):
    pass

# ✅ Good: Stack HAS a list and exposes only the stack actions
class Stack:
    def __init__(self):
        self._items = []
    def push(self, x):
        self._items.append(x)
    def pop(self):
        return self._items.pop()
    def is_empty(self):
        return len(self._items) == 0
```

### 🧪 Try it
Build a `Car` that **has** an `Engine` (with `start()`) and 4 `Wheel` objects. Print a message from each part when `car.drive()` is called.

---

## 13. Aggregation ("HAS-A, but parts live independently")

Like composition, but the parts **exist on their own** and are only **passed in** / referenced.

### Real-life idea
A **football team has players**, but if the team is dissolved, the players still exist and can join another team. A **classroom has students**, but students exist without that classroom.

```python
class Player:
    def __init__(self, name):
        self.name = name


class Team:
    def __init__(self, team_name):
        self.team_name = team_name
        self.players = []

    def add_player(self, player):             # player is created OUTSIDE and passed in
        self.players.append(player)

    def show(self):
        names = ", ".join(p.name for p in self.players)
        print(f"{self.team_name}: {names}")


ravi = Player("Ravi")
sita = Player("Sita")

team_a = Team("Lions")
team_a.add_player(ravi)
team_a.add_player(sita)
team_a.show()                # Lions: Ravi, Sita

del team_a                   # team is gone...
print(ravi.name)             # ...but Ravi still exists ✅
```

### Composition vs Aggregation (the one-minute difference)

| | Composition | Aggregation |
|---|---|---|
| Who **creates** the part? | the whole (inside it) | someone else (passed in) |
| Can the part live alone? | ❌ no | ✅ yes |
| Example | House → Rooms, Order → Items | Team → Players, Classroom → Students |
| In code | `self.cpu = CPU()` | `def __init__(self, cpu): self.cpu = cpu` |

**Quick test:** *"If I delete the whole, does the part still make sense?"*
No → composition. Yes → aggregation.

### 🧪 Try it
`Library` has `Book` objects (books can be moved to another library). `Book` has `Page`s that only exist inside a book. Which is composition? Which is aggregation? Write the code.

---

## 14. Quick Decision Guide: Inheritance vs Composition vs Aggregation

```
Is it a kind of that thing?          ("Developer IS an Employee")   → Inheritance
Does it own parts that die with it?  ("Computer HAS a CPU")         → Composition
Does it just use things that exist
independently?                       ("Team HAS Players")           → Aggregation
```

---

# PART 2 — DESIGN THINKING

Programming is not only typing code. It is also **deciding what classes to create**. Use this 5-step method before you code:

**Step 1 — Read the problem. Underline the nouns. These are candidate classes.**
> "A *customer* opens an *account* and makes *transactions*." → `Customer`, `Account`, `Transaction`

**Step 2 — Underline the verbs. These are candidate methods.**
> open, deposit, withdraw, transfer, show history

**Step 3 — Ask: "Who should own this data?"**
Balance belongs to `Account`, not to `Bank`.

**Step 4 — Check relationships**
- *is-a* → inheritance (`SavingsAccount` is an `Account`)
- *has-a (owned)* → composition (`Account` has its `Transaction` list)
- *has-a (shared)* → aggregation (`Customer` has accounts)

**Step 5 — Ask "What might change later?"**
Hide that part behind a parent class or abstract class. (Tomorrow someone asks for a new account type — can you add one *without editing old code*?)

### Three golden rules
1. **One class, one job.** (`Account` handles money; it should *not* send emails.)
2. **Add new features by adding new classes**, not by editing old working code.
3. **Depend on the general idea, not the specific one.** (A function that accepts "any payment method" is better than one that accepts only "UPI".)

### Warning signs your design needs improvement

| Warning sign | Likely fix |
|---|---|
| A long `if type == "x" ... elif type == "y"` chain | Use polymorphism |
| Same code copied in 3 classes | Move it to a parent class |
| One class with 20 unrelated methods | Split into smaller classes |
| Child class ignores everything from its parent | You chose the wrong relationship |

---

# PART 3 — DESIGN PATTERNS

> **A design pattern is a proven solution to a problem that programmers meet again and again.**
> Don't memorise the code. Memorise the **problem**. When you feel that pain, you'll remember the pattern.

For each pattern we ask: **What's the problem? → What's the idea? → Code → Where is it used?**

---

## Pattern 1 — Factory

### 🔥 The problem
Your food app lets users choose how to pay. Everywhere in the app, you have this:

```python
if choice == "upi":
    method = UPI()
elif choice == "card":
    method = Card()
elif choice == "cash":
    method = Cash()
```

Copied in 10 places. Adding "NetBanking" means finding and editing all 10. 😩

### 💡 The idea
Put the "which class should I create?" decision in **one function** (a *factory*). Everyone else just asks the factory.

### Code

```python
class UPI:
    def pay(self, amount): print(f"Paid ₹{amount} via UPI")

class Card:
    def pay(self, amount): print(f"Paid ₹{amount} via Card")

class Cash:
    def pay(self, amount): print(f"Paid ₹{amount} in Cash")


def create_payment(choice):                 # ← the FACTORY
    methods = {
        "upi": UPI,
        "card": Card,
        "cash": Cash,
    }
    if choice not in methods:
        raise ValueError(f"Unknown payment method: {choice}")
    return methods[choice]()                # build and return the object


payment = create_payment("upi")
payment.pay(300)                            # Paid ₹300 via UPI
```

Add NetBanking: write the class + add **one line** to the dictionary. Nothing else changes.

### Where is it used?
Loading different file types (CSV/JSON/XML), creating the right database connection, picking the right enemy type in a game.

**Use it when:** *the exact class to create depends on user input or settings.*

---

## Pattern 2 — Strategy

### 🔥 The problem
A delivery app calculates delivery fees with different rules:

```python
def delivery_fee(kind, distance):
    if kind == "normal":
        return distance * 5
    elif kind == "express":
        return distance * 10 + 20
    elif kind == "premium_member":
        return 0
    # A new rule every week from the business team...
```

The function keeps growing, and every change risks breaking the other rules.

### 💡 The idea
Make **each rule its own small piece** (a strategy). The main code just says *"use this strategy."* You can swap strategies anytime.

### Code

```python
class NormalDelivery:
    def fee(self, distance):
        return distance * 5

class ExpressDelivery:
    def fee(self, distance):
        return distance * 10 + 20

class PremiumDelivery:
    def fee(self, distance):
        return 0


class Order:
    def __init__(self, amount, delivery_strategy):
        self.amount = amount
        self.delivery = delivery_strategy      # we HAVE a strategy (aggregation!)

    def total(self, distance):
        return self.amount + self.delivery.fee(distance)


print(Order(500, NormalDelivery()).total(4))    # 520
print(Order(500, ExpressDelivery()).total(4))   # 560
print(Order(500, PremiumDelivery()).total(4))   # 500
```

### 🐍 Python shortcut: a strategy can just be a function

```python
def normal(distance):  return distance * 5
def express(distance): return distance * 10 + 20

def total(amount, distance, fee_rule):
    return amount + fee_rule(distance)

print(total(500, 4, normal))                   # 520
print(total(500, 4, lambda d: d * 2))          # 508  (a one-off rule!)
```

### Factory vs Strategy: don't mix them up

| Pattern | Question it answers |
|---|---|
| **Factory** | *Which object should I create?* |
| **Strategy** | *Which way of doing the job should I use?* |

**Use it when:** *you have several ways of doing the same job and want to swap them easily* (sorting rules, discount rules, retry timing).

---

## Pattern 3 — Observer

### 🔥 The problem
When a YouTuber uploads a video, many things must happen: notify subscribers, update the website, send a message to the editor... If the upload function does all this directly, it must **know about everyone**:

```python
def upload_video(video):
    save(video)
    notify_subscribers(video)
    update_website(video)
    message_editor(video)
    # every new requirement → edit this function again
```

### 💡 The idea
Like **subscribing to a channel**. The channel doesn't know who you are; it just announces *"new video!"* and everyone subscribed gets notified. People can subscribe/unsubscribe anytime.

### Code

```python
class Channel:
    def __init__(self, name):
        self.name = name
        self.subscribers = []                  # list of functions to call

    def subscribe(self, callback):
        self.subscribers.append(callback)

    def unsubscribe(self, callback):
        self.subscribers.remove(callback)

    def upload(self, title):
        print(f"📹 {self.name} uploaded: {title}")
        for notify in self.subscribers:        # tell everyone
            notify(title)


def ravi_phone(title):  print(f"  📱 Ravi got a notification: {title}")
def sita_email(title):  print(f"  📧 Sita got an email: {title}")


channel = Channel("CodeWithFun")
channel.subscribe(ravi_phone)
channel.subscribe(sita_email)

channel.upload("OOP in 10 minutes")
channel.unsubscribe(ravi_phone)
channel.upload("Design Patterns made easy")
```

Output:
```
📹 CodeWithFun uploaded: OOP in 10 minutes
  📱 Ravi got a notification: OOP in 10 minutes
  📧 Sita got an email: OOP in 10 minutes
📹 CodeWithFun uploaded: Design Patterns made easy
  📧 Sita got an email: Design Patterns made easy
```

### Where is it used?
Button clicks in apps, notifications, stock price alerts, chat apps, "when an order is placed → send email + update stock + log analytics."

**Use it when:** *when one thing happens, many other things need to react, and you don't want the main code to know about them all.*

---

## Pattern 4 — Singleton

### 🔥 The problem
Some things should exist **only once**. Think of your game's **settings** (volume, difficulty). If two settings objects existed, one screen could show volume 80 while another shows 20 — confusing bugs.

### 💡 The idea
Make sure that no matter how many times someone asks for the object, they always get **the same one**.

### Code

```python
class GameSettings:
    _instance = None                           # remembers the one and only object

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.volume = 50          # set up only the first time
        return cls._instance


a = GameSettings()
b = GameSettings()

a.volume = 90
print(b.volume)        # 90  → same object!
print(a is b)          # True
```

### 🐍 Python's easier way: use a module
A Python file is imported **only once**, so it is already a singleton.

```python
# settings.py
VOLUME = 50
DIFFICULTY = "easy"

# any other file
import settings
print(settings.VOLUME)     # same values everywhere
```

### ⚠️ Be careful
Singletons are like global variables: any part of the program can change them, which makes bugs hard to find and testing harder. Use only for things that truly must be single (settings, logger, connection pool). In many cases it's better to create one object and **pass it in** where needed.

---

## Pattern 5 — Adapter

### 🔥 The problem
You travel abroad. Your phone charger has a 2-pin plug, but the wall socket is 3-pin. You don't rebuild your charger or the wall. You use a **travel adapter**. 🔌

Same in code. Your app expects every payment gateway to have a `pay(amount)` method. But two third-party libraries (that you **cannot modify**) look different:

```python
class FastPayLibrary:
    def make_payment(self, rupees):
        print(f"FastPay: received ₹{rupees}")

class SafeCashLibrary:
    def charge(self, paise):                 # takes paise, not rupees!
        print(f"SafeCash: received {paise} paise")
```

### 💡 The idea
Write a small **adapter class** that translates the library's interface into the one your app wants.

### Code

```python
class FastPayAdapter:
    def __init__(self, library):
        self.library = library

    def pay(self, amount):
        self.library.make_payment(amount)


class SafeCashAdapter:
    def __init__(self, library):
        self.library = library

    def pay(self, amount):
        self.library.charge(amount * 100)    # convert rupees → paise


def checkout(gateway, amount):               # our app only knows .pay()
    gateway.pay(amount)


checkout(FastPayAdapter(FastPayLibrary()), 250)
checkout(SafeCashAdapter(SafeCashLibrary()), 250)
```

Output:
```
FastPay: received ₹250
SafeCash: received 25000 paise
```

Our `checkout()` never changed. Tomorrow a third gateway arrives → write one more adapter.

**Use it when:** *you need to use code you can't change (libraries, old systems) but its interface doesn't match yours.*

---

## 🧭 Pattern Summary Card

| Pattern | Problem in one line | Real-life picture |
|---|---|---|
| **Factory** | Too many `if/elif` just to *create* the right object | A restaurant kitchen: you order "pizza", the kitchen decides how to make it |
| **Strategy** | Too many `if/elif` to choose *how* to do a job | Google Maps: walking, bike, car routes, same goal, different method |
| **Observer** | One event, many reactions | Subscribing to a YouTube channel |
| **Singleton** | Exactly one instance wanted | The Principal of a school |
| **Adapter** | Interfaces don't match | Travel plug adapter |

---

# PART 4 — MINI PROJECT: 🏦 Banking System

## What you'll build
A small bank that supports customers, different account types, deposit, withdraw, transfer, and transaction history.

## Classes

| Class | Job |
|---|---|
| `Transaction` | One record: type, amount, balance after, time |
| `Account` *(abstract)* | Common behaviour for all accounts |
| `SavingsAccount` | Account with minimum balance + interest |
| `CurrentAccount` | Account with overdraft + monthly fee |
| `Customer` | Person who owns accounts |
| `Bank` | Manages customers & accounts, does transfers |

## You must show these OOP ideas

| Concept | How you'll show it |
|---|---|
| **Inheritance** | `SavingsAccount` and `CurrentAccount` extend `Account` |
| **Polymorphism** | `bank.month_end()` calls `account.month_end()` for every account, no `if/elif` |
| **Encapsulation** | balance is private; only changes via deposit/withdraw |
| **Abstraction** | `Account` is an abstract class with abstract methods |
| **Composition** | `Account` owns its list of `Transaction`s |

## Rules of the bank
- Amount must be **greater than 0**.
- **Savings:** must always keep at least **₹500**. Gets **4% yearly interest**, added monthly.
- **Current:** may go down to **-₹1000** (overdraft). Pays **₹50 monthly fee**.
- Can't transfer to the same account. Account must exist.

## Picture of the design

```
Bank ──has──► Customers
 │
 └─has──► Accounts (abstract Account)
              ├── SavingsAccount
              └── CurrentAccount
Each Account ──owns──► many Transactions
```

---

## Build it step by step

### ✅ Step 1: `Transaction` (a simple record)

```python
from datetime import datetime

class Transaction:
    def __init__(self, kind, amount, balance_after, note=""):
        self.kind = kind                   # "DEPOSIT", "WITHDRAW", ...
        self.amount = amount
        self.balance_after = balance_after
        self.note = note
        self.time = datetime.now()

    def __str__(self):
        when = self.time.strftime("%H:%M:%S")
        return f"{when} | {self.kind:<12} | {self.amount:>8} | balance {self.balance_after:>8} | {self.note}"
```

### ✅ Step 2: Our own error types (clear messages!)

```python
class BankError(Exception):
    pass

class InvalidAmount(BankError):
    pass

class InsufficientFunds(BankError):
    pass

class AccountNotFound(BankError):
    pass
```

### ✅ Step 3: The abstract `Account`

```python
from abc import ABC, abstractmethod

class Account(ABC):
    def __init__(self, account_no, owner_name, opening_balance=0):
        self.account_no = account_no
        self.owner_name = owner_name
        self.__balance = 0                     # private (encapsulation)
        self._history = []                     # owns its transactions (composition)
        if opening_balance > 0:
            self._credit(opening_balance, "DEPOSIT", "opening deposit")

    @property
    def balance(self):
        return self.__balance

    # ----- public actions -----
    def deposit(self, amount):
        self._check_amount(amount)
        self._credit(amount, "DEPOSIT")

    def withdraw(self, amount):
        self._check_amount(amount)
        self._debit(amount, "WITHDRAW")

    # ----- helpers used by deposit/withdraw/transfer -----
    def _check_amount(self, amount):
        if amount <= 0:
            raise InvalidAmount("Amount must be greater than zero")

    def _credit(self, amount, kind, note=""):
        self.__balance += amount
        self._history.append(Transaction(kind, amount, self.__balance, note))

    def _debit(self, amount, kind, note=""):
        self._check_can_debit(amount)          # each account type has its own rule
        self.__balance -= amount
        self._history.append(Transaction(kind, amount, self.__balance, note))

    # ----- abstract: children MUST define these -----
    @abstractmethod
    def _check_can_debit(self, amount):
        pass

    @abstractmethod
    def month_end(self):
        pass

    # ----- reporting -----
    def statement(self):
        print(f"--- Statement: {self.account_no} ({type(self).__name__}) - {self.owner_name} ---")
        for t in self._history:
            print(t)
        print(f"Closing balance: {self.balance}")
```

### ✅ Step 4: The two account types (inheritance + polymorphism)

```python
class SavingsAccount(Account):
    MIN_BALANCE = 500
    YEARLY_RATE = 0.04

    def _check_can_debit(self, amount):
        if self.balance - amount < self.MIN_BALANCE:
            raise InsufficientFunds(
                f"Savings account must keep at least ₹{self.MIN_BALANCE}")

    def month_end(self):
        interest = round(self.balance * self.YEARLY_RATE / 12, 2)
        if interest > 0:
            self._credit(interest, "INTEREST", "monthly interest")


class CurrentAccount(Account):
    OVERDRAFT_LIMIT = 1000
    MONTHLY_FEE = 50

    def _check_can_debit(self, amount):
        if self.balance - amount < -self.OVERDRAFT_LIMIT:
            raise InsufficientFunds(
                f"Overdraft limit of ₹{self.OVERDRAFT_LIMIT} exceeded")

    def month_end(self):
        try:
            self._debit(self.MONTHLY_FEE, "FEE", "monthly maintenance fee")
        except InsufficientFunds:
            pass                                # skip if the fee would break the limit
```

Notice: `Account` doesn't know *how* each type decides what is allowed. The children decide. That's **polymorphism**.

### ✅ Step 5: `Customer`

```python
class Customer:
    def __init__(self, customer_id, name, email):
        if len(name.strip()) < 2:
            raise ValueError("Name is too short")
        if "@" not in email:
            raise ValueError("Invalid email")
        self.customer_id = customer_id
        self.name = name.strip()
        self.email = email
        self.accounts = []                      # the accounts this person has
```

### ✅ Step 6: `Bank` (brings everything together)

```python
class Bank:
    ACCOUNT_TYPES = {                           # ← a mini Factory
        "savings": SavingsAccount,
        "current": CurrentAccount,
    }

    def __init__(self, name):
        self.name = name
        self.customers = {}
        self.accounts = {}
        self._next_customer = 1
        self._next_account = 1001

    def register_customer(self, name, email):
        cid = f"C{self._next_customer:03d}"
        self._next_customer += 1
        customer = Customer(cid, name, email)
        self.customers[cid] = customer
        return customer

    def open_account(self, customer, kind, opening_balance=0):
        if kind not in self.ACCOUNT_TYPES:
            raise ValueError(f"Unknown account type: {kind}")
        account_no = f"ACC{self._next_account}"
        self._next_account += 1
        account_class = self.ACCOUNT_TYPES[kind]
        account = account_class(account_no, customer.name, opening_balance)
        self.accounts[account_no] = account
        customer.accounts.append(account)
        return account

    def get_account(self, account_no):
        if account_no not in self.accounts:
            raise AccountNotFound(f"No account {account_no}")
        return self.accounts[account_no]

    def transfer(self, from_no, to_no, amount):
        if from_no == to_no:
            raise BankError("Cannot transfer to the same account")
        source = self.get_account(from_no)       # check both exist FIRST
        target = self.get_account(to_no)
        source._check_amount(amount)
        source._debit(amount, "TRANSFER_OUT", f"to {to_no}")   # may fail → nothing changed
        target._credit(amount, "TRANSFER_IN", f"from {from_no}")

    def month_end(self):
        for account in self.accounts.values():
            account.month_end()                  # polymorphism, no if/elif!
```

### ✅ Step 7: Run it

```python
bank = Bank("PyBank")

ravi = bank.register_customer("Ravi Kumar", "ravi@example.com")
sita = bank.register_customer("Sita Rao", "sita@example.com")

savings = bank.open_account(ravi, "savings", 10000)
current = bank.open_account(sita, "current", 2000)

savings.deposit(2500)
bank.transfer(savings.account_no, current.account_no, 3000)
current.withdraw(6000)                     # goes into overdraft: allowed

try:
    savings.withdraw(50000)
except InsufficientFunds as e:
    print("Blocked:", e)

bank.month_end()

savings.statement()
print()
current.statement()
```

**Expected result (times will differ):** Ravi's savings ends at about ₹9,531.67 and Sita's current account sits at -₹1,000. The ₹50 fee is skipped because the overdraft limit has been reached.

---

## 🧩 Testing checklist (try each one!)

| Test | Expected |
|---|---|
| `savings.deposit(-10)` | `InvalidAmount` |
| Withdraw from savings leaving < ₹500 | `InsufficientFunds` |
| Current account to -₹1000 | allowed |
| Current account below -₹1000 | `InsufficientFunds` |
| Transfer to same account | `BankError` |
| Transfer to a non-existing account | `AccountNotFound` |
| Failed transfer | Both balances unchanged |
| `savings.balance = 5` | `AttributeError` (balance protected) |
| `bank.open_account(ravi, "gold")` | `ValueError` |

## 🚀 Level-up challenges
1. Add **`FixedDepositAccount`**: no withdrawal allowed. You should only **add a class and one line in `ACCOUNT_TYPES`**; if you edit old classes, rethink your design.
2. Add an **Observer**: when a withdrawal is over ₹50,000, print "⚠️ Large withdrawal alert".
3. Add a **Strategy** for interest (`FlatRate`, `TieredRate`).
4. Add a **daily withdrawal limit** to `SavingsAccount`.
5. Save all data to a JSON file and load it back.

> 💬 **A note on money in real software:** we used normal numbers to keep things simple. Real banking software uses Python's `Decimal` type, because floating-point math can give results like `0.1 + 0.2 = 0.30000000000000004`.

---

# 📌 REVISION SECTION

## Master cheat sheet

| Concept | In one sentence | Mini example |
|---|---|---|
| Class | Blueprint | `class Student:` |
| Object | Real thing from the blueprint | `s = Student()` |
| `__init__` | Setup that runs on creation | `def __init__(self, name):` |
| `self` | "This object" | `self.name = name` |
| Instance attribute | Different per object | `self.marks` |
| Class attribute | Shared by all | `college = "Sunrise"` |
| Instance method | Works on one object | `def average(self)` |
| Class method | Works on the class; alternative constructors | `@classmethod from_string(cls, ...)` |
| Static method | Helper inside class | `@staticmethod is_valid(...)` |
| Encapsulation | Protect data; change only through safe methods | `__balance`, `@property` |
| Inheritance | Child reuses parent ("is-a") | `class Developer(Employee)` |
| `super()` | Call the parent's version | `super().__init__(...)` |
| Multiple inheritance | Many parents | `class SmartPhone(Phone, Camera)` |
| MRO | Order Python searches parents | `Cls.__mro__` |
| Polymorphism | Same call, different behaviour | `payment.pay(100)` |
| Abstraction | Define WHAT, children define HOW | `@abstractmethod` |
| Composition | Owns parts that die with it | `self.cpu = CPU()` |
| Aggregation | Uses independent parts | `team.add_player(p)` |

## Top 10 mistakes beginners make

1. Forgetting `self` in method definitions or when using attributes.
2. Writing `name = name` instead of `self.name = name`.
3. Putting a list/dict as a **class** attribute by accident.
4. Calling a method without brackets: `obj.show` instead of `obj.show()`.
5. Using inheritance when it should be "has-a" (composition).
6. Forgetting `super().__init__()` in the child's `__init__`.
7. Trying to create an object of an abstract class.
8. Forgetting to implement all abstract methods in a child.
9. Long `if/elif` chains checking object types, a sign you need polymorphism.
10. Making everything public and letting any code change anything.

---

# 📝 SELF-TEST (check yourself)

### Part A: Predict the output

**A1.**
```python
class Counter:
    count = 0
    def __init__(self):
        Counter.count += 1

a = Counter()
b = Counter()
c = Counter()
print(Counter.count)
```

**A2.**
```python
class Basket:
    items = []
    def add(self, x):
        self.items.append(x)

b1 = Basket()
b2 = Basket()
b1.add("apple")
print(b2.items)
```

**A3.**
```python
class A:
    def show(self): print("A")
class B(A):
    def show(self):
        print("B")
        super().show()

B().show()
```

**A4.**
```python
class Temp:
    def __init__(self, c):
        self._c = c
    @property
    def fahrenheit(self):
        return self._c * 9 / 5 + 32

t = Temp(100)
print(t.fahrenheit)
t.fahrenheit = 50
```

### Part B: Explain in your own words
1. Difference between a class and an object?
2. What does `self` mean?
3. When do you use `@classmethod` instead of a normal method?
4. Difference between composition and aggregation (give your own example).
5. Why can't you create an object of an abstract class?
6. Which pattern would you use for: (a) "creating the correct type of object from user input", (b) "swappable discount rules", (c) "notify many things when an order is placed", (d) "using a library whose method names don't match mine"?

### Part C: Code
1. Create an abstract class `Employee` with abstract method `calculate_salary()`. Make `FullTimeEmployee` (fixed monthly salary) and `Freelancer` (hourly rate × hours). Put both in a list and print each salary in a loop.
2. Write a `Library` class that **aggregates** `Book` objects, with `add_book`, `find_by_author`, and `__len__`.
3. Write a small Observer: a `Thermometer` that notifies subscribers when the temperature goes above 40°C.

---

### ✅ Answers to Part A
| | Answer | Reason |
|---|---|---|
| A1 | `3` | one shared class attribute, increased 3 times |
| A2 | `['apple']` | `items` is one list shared by all objects |
| A3 | `B` then `A` | B prints, then `super().show()` runs A's version |
| A4 | prints `212.0`, then `AttributeError` | a property without a setter is read-only |

---

## 🎯 End-of-week goals

By Sunday you should be able to:
- [ ] Explain every topic in this guide **without looking** at it
- [ ] Spot when to use inheritance, composition or aggregation
- [ ] Replace an `if/elif` chain with polymorphism
- [ ] Write an abstract class and explain why it helps
- [ ] Tell the 5 patterns apart by the **problem** each solves
- [ ] Finish the Banking project and explain each class's role
- [ ] Score 70%+ on the weekly test

> 💪 **Remember:** OOP feels confusing at first for everyone. It becomes natural when you build things. Don't just read, **run the code, break it on purpose, and fix it**.