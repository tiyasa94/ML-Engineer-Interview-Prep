# Python "Why This, Not That" - MLE Interview Question Bank

## Part A — Data Structures

**Q1. Why use a generator instead of a list when processing a large dataset?**

A generator produces items lazily, one at a time, holding only the current item (plus minimal state) in memory - a list materializes the entire sequence upfront. For a large or unbounded dataset, a generator keeps memory usage `O(1)` instead of `O(n)`. The real tradeoff: a generator is **single-pass** - once consumed, it's exhausted; if you need to iterate the same data multiple times, a list (or a re-creatable generator function/iterable class) is the right choice despite using more memory.

```python
import sys

# List: materializes all 10M items in memory upfront
squares_list = [x**2 for x in range(10_000_000)]
print(sys.getsizeof(squares_list))          # ~85,000,000+ bytes

# Generator: holds only current state, O(1) memory
squares_gen = (x**2 for x in range(10_000_000))
print(sys.getsizeof(squares_gen))           # ~200 bytes, regardless of range size

# The single-pass gotcha:
gen = (x for x in range(3))
print(list(gen))   # [0, 1, 2]
print(list(gen))   # [] -- exhausted, second pass gets nothing

# Fix for re-iteration: a generator FUNCTION (called fresh each time), not a generator object
def make_gen():
    return (x for x in range(3))

g1, g2 = make_gen(), make_gen()
print(list(g1), list(g2))   # [0, 1, 2] [0, 1, 2] -- each is fresh
```

---

**Q2. Why use a tuple instead of a list for function return values or fixed records?**

Tuples are immutable, communicating intent and preventing accidental in-place mutation, and are hashable (usable as dict keys/set members) precisely because they're immutable - a list can't be. Use a list when the collection genuinely needs to grow/change after creation.

```python
def get_bounding_box():
    return (0, 0, 100, 100)   # immutable: caller can't accidentally mutate it

bbox = get_bounding_box()
# bbox[0] = 5   # TypeError: 'tuple' object does not support item assignment -- this is the point

# Tuples are hashable -> usable as dict keys; lists are not
cache = {}
cache[(0, 0, 100, 100)] = "cached_result"     # works

# cache[[0, 0, 100, 100]] = "x"               # TypeError: unhashable type: 'list'

# Use a list instead when contents genuinely change:
batch = []
for i in range(5):
    batch.append(i ** 2)     # naturally growing collection -> list is correct here
```

---

**Q3. Why use a `dict` instead of two parallel lists for key-value lookups?**

A `dict` gives `O(1)` average-case lookup by key; parallel lists require an `O(n)` scan and can silently drift out of sync after an edit to one but not the other.

```python
# Parallel lists: O(n) lookup, and easy to desync
user_ids = ["u1", "u2", "u3"]
user_scores = [0.91, 0.42, 0.77]

def get_score(uid):
    for i, u in enumerate(user_ids):   # O(n) scan
        if u == uid:
            return user_scores[i]

# One dict: O(1) average lookup, single source of truth
scores = {"u1": 0.91, "u2": 0.42, "u3": 0.77}
print(scores["u2"])   # O(1)

# Desync risk with parallel lists (won't happen with a dict):
user_ids.append("u4")
# forgot to append to user_scores -> now indices are misaligned silently
```

---

**Q4. Why use `collections.deque` instead of a list for a queue (FIFO)?**

`list.pop(0)` is `O(n)` (every remaining element shifts left); `deque.popleft()` is `O(1)`. For BFS frontiers, sliding windows, or producer-consumer buffers, a deque avoids quadratic blowup at scale.

```python
from collections import deque
import time

# List as a queue: O(n) per pop(0) -> O(n^2) total for n pops
lst = list(range(100_000))
start = time.perf_counter()
while lst:
    lst.pop(0)          # O(n) each time -- slow at scale
print("list:", time.perf_counter() - start)

# Deque as a queue: O(1) per popleft() -> O(n) total
dq = deque(range(100_000))
start = time.perf_counter()
while dq:
    dq.popleft()         # O(1) each time
print("deque:", time.perf_counter() - start)   # noticeably faster

# Tradeoff: deque is bad at random-access indexing into the middle
dq2 = deque([1, 2, 3, 4, 5])
# dq2[2] works (O(1) at ends, but O(n) toward the middle) -- lists are better for that pattern
```

---

**Q5. Why use a `set` instead of a list for membership testing?**

`x in set` is `O(1)` average case via hashing; `x in list` is `O(n)`. For repeated "have I seen this" checks, a set turns an `O(n²)`-shaped pattern into `O(n)`.

```python
import time

data = list(range(50_000))
lookups = list(range(45_000, 55_000))

# O(n) per lookup with a list -> O(n * m) total
data_list = data
start = time.perf_counter()
found = [x for x in lookups if x in data_list]     # slow
print("list membership:", time.perf_counter() - start)

# O(1) average per lookup with a set -> O(m) total
data_set = set(data)
start = time.perf_counter()
found = [x for x in lookups if x in data_set]      # fast
print("set membership:", time.perf_counter() - start)

# Deduplication, the other classic set use case:
raw = [1, 2, 2, 3, 3, 3, 4]
unique = set(raw)     # {1, 2, 3, 4} -- but order not guaranteed to be preserved
```

---

**Q6. Why use a `namedtuple` or `dataclass` instead of a plain dict for structured records?**

Both give named-attribute access, more self-documenting and IDE/type-checker friendly than string-key dict access. `dataclass` adds defaults, type hints, auto `__repr__`/`__eq__`, and optional immutability.

```python
from dataclasses import dataclass
from collections import namedtuple

# Plain dict: works, but no attribute access, no type safety, easy typo bugs
record_dict = {"user_id": "u1", "score": 0.91}
print(record_dict["scroe"] if "scroe" in record_dict else "KeyError risk on typos")

# namedtuple: lightweight, immutable, tuple-compatible
UserScore = namedtuple("UserScore", ["user_id", "score"])
r1 = UserScore(user_id="u1", score=0.91)
print(r1.score)          # attribute access
# r1.score = 0.5          # AttributeError -- immutable

# dataclass: more flexible, supports defaults, mutability, __post_init__ validation
@dataclass
class UserScoreDC:
    user_id: str
    score: float = 0.0

r2 = UserScoreDC(user_id="u2", score=0.42)
print(r2)                # auto-generated __repr__: UserScoreDC(user_id='u2', score=0.42)
r2.score = 0.5            # mutable by default (use frozen=True to lock it down)
```

---

## Part B — Concurrency & Parallelism

**Q7. Why use `multiprocessing` instead of `threading` for CPU-bound work in Python?**

The GIL allows only one thread to execute Python bytecode at a time, so threads don't give real parallelism for CPU-bound work. `multiprocessing` sidesteps the GIL with separate processes, each with its own interpreter.

```python
import time
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

def cpu_bound(n):
    return sum(i * i for i in range(n))     # pure CPU work, no I/O

N = 5_000_000

# Threads: GIL serializes bytecode execution -> barely faster than sequential
start = time.perf_counter()
with ThreadPoolExecutor(max_workers=4) as ex:
    list(ex.map(cpu_bound, [N] * 4))
print("threads:", time.perf_counter() - start)     # ~same as running 4x sequentially

# Processes: real parallelism across cores, GIL doesn't apply across processes
start = time.perf_counter()
with ProcessPoolExecutor(max_workers=4) as ex:
    list(ex.map(cpu_bound, [N] * 4))
print("processes:", time.perf_counter() - start)   # meaningfully faster on a multi-core machine

# Tradeoff: process overhead + data must be pickled to cross process boundaries
```

---

**Q8. Why use `threading` or `asyncio` instead of `multiprocessing` for I/O-bound work?**

I/O-bound work spends most of its time waiting, not executing bytecode - the GIL is released during blocking I/O, so threads/coroutines can overlap that waiting time cheaply, without the heavier memory/process-spawn cost of `multiprocessing`.

```python
import time
import requests
from concurrent.futures import ThreadPoolExecutor

urls = ["https://httpbin.org/delay/1"] * 5   # each call blocks ~1s waiting on the network

# Sequential: ~5 seconds total (waits for each response before starting the next)
start = time.perf_counter()
for url in urls:
    requests.get(url)
print("sequential:", time.perf_counter() - start)   # ~5s

# Threaded: overlapping waits -> close to ~1s total, not 5s
start = time.perf_counter()
with ThreadPoolExecutor(max_workers=5) as ex:
    list(ex.map(requests.get, urls))
print("threaded:", time.perf_counter() - start)     # ~1s -- GIL released during I/O wait
```

---

**Q9. Why would you use `asyncio` instead of `threading` even though both handle I/O-bound concurrency?**

`asyncio`'s cooperative model hands off control only at explicit `await` points, eliminating most classic race conditions and scaling to far more concurrent units than OS threads can practically support.

```python
import asyncio
import aiohttp
import time

urls = ["https://httpbin.org/delay/1"] * 50   # 50 concurrent waits

async def fetch(session, url):
    async with session.get(url) as resp:
        return await resp.text()

async def main():
    async with aiohttp.ClientSession() as session:
        start = time.perf_counter()
        await asyncio.gather(*[fetch(session, u) for u in urls])
        print("asyncio, 50 concurrent:", time.perf_counter() - start)   # ~1s, not 50s

asyncio.run(main())

# 50 OS threads is *possible* but far more memory/scheduling overhead than 50 coroutines.
# The catch: one accidentally-blocking synchronous call inside an async function
# (e.g. time.sleep(1) instead of await asyncio.sleep(1)) stalls the WHOLE event loop.
```

---

**Q10. In an ML serving context, why might you still use `multiprocessing` (e.g., Gunicorn workers) even with `asyncio`-based FastAPI?**

`asyncio` gives concurrency within one process for I/O, but a heavy synchronous inference call still blocks the whole event loop for every other request. Multiple worker processes give real parallelism across cores for the CPU-heavy inference itself.

```python
# gunicorn command illustrating the combined approach:
#   gunicorn app:app -k uvicorn.workers.UvicornWorker -w 4
#
# -w 4  -> 4 separate PROCESSES (real parallelism across CPU cores for inference)
# each worker runs its own asyncio event loop (efficient concurrent I/O within that process)

# Conceptually, inside one worker:
import asyncio

async def predict_endpoint(request):
    features = await fetch_features(request)      # I/O-bound: asyncio handles this well
    prediction = model.predict(features)           # CPU-bound: blocks THIS worker's event loop
    return prediction

# A single worker's event loop stalls on the model.predict() line for every
# other concurrent request in that process -- this is exactly why running
# multiple worker PROCESSES (not just async concurrency) matters for inference load.
```

---

## Part C — OOP & Language Design Choices

**Q11. Why use a `dataclass` instead of a regular class with a manually written `__init__`?**

Auto-generates `__init__`, `__repr__`, `__eq__` from typed field declarations, eliminating repetitive boilerplate and the bugs that come with it.

```python
# Manual class: repetitive, easy to typo an assignment
class ModelConfigManual:
    def __init__(self, learning_rate, batch_size, epochs=10):
        self.learning_rate = learning_rate
        self.batch_size = batch_size
        self.epochs = epochs
    def __repr__(self):
        return f"ModelConfigManual({self.learning_rate}, {self.batch_size}, {self.epochs})"
    def __eq__(self, other):
        return (self.learning_rate, self.batch_size, self.epochs) == \
               (other.learning_rate, other.batch_size, other.epochs)

# dataclass: same behavior, far less code, less room for error
from dataclasses import dataclass

@dataclass
class ModelConfig:
    learning_rate: float
    batch_size: int
    epochs: int = 10

c1 = ModelConfig(learning_rate=0.01, batch_size=32)
c2 = ModelConfig(learning_rate=0.01, batch_size=32)
print(c1)          # auto __repr__: ModelConfig(learning_rate=0.01, batch_size=32, epochs=10)
print(c1 == c2)    # True, auto __eq__ -- no manual comparison logic written
```

---

**Q12. Why use `@property` instead of plain public attributes with separate getter/setter methods?**

Lets you start with simple attribute access and *later* add validation/computed logic without changing the external calling interface.

```python
class Model:
    def __init__(self, threshold):
        self._threshold = threshold   # underscore: "internal" by convention

    @property
    def threshold(self):
        return self._threshold

    @threshold.setter
    def threshold(self, value):
        if not 0.0 <= value <= 1.0:
            raise ValueError("threshold must be in [0, 1]")
        self._threshold = value

m = Model(0.5)
print(m.threshold)      # reads like plain attribute access: m.threshold, not m.get_threshold()
m.threshold = 0.9       # writes like plain assignment, but validation runs underneath
# m.threshold = 5.0      # raises ValueError -- caught immediately, not silently stored as garbage

# Compare to Java-style getters/setters locked in from day one:
# m.set_threshold(0.9)   # more verbose, and every caller committed to this style upfront
```

---

**Q13. Why use composition instead of inheritance for extending an ML model's behavior?**

Inheritance creates tight coupling to a parent's implementation; composition keeps components independently testable and swappable.

```python
# Inheritance: tightly coupled, hard to swap the preprocessing step independently
class PreprocessedModel(BaseModel):          # "is-a" BaseModel, locked in
    def preprocess(self, x):
        return (x - self.mean) / self.std
    def predict(self, x):
        return super().predict(self.preprocess(x))

# Composition: swappable, independently testable pieces
class Pipeline:
    def __init__(self, preprocessor, model):
        self.preprocessor = preprocessor      # "has-a" preprocessor
        self.model = model                    # "has-a" model

    def predict(self, x):
        return self.model.predict(self.preprocessor.transform(x))

# Swapping preprocessing strategy at runtime, no class hierarchy changes needed:
pipeline_a = Pipeline(preprocessor=StandardScaler(), model=xgb_model)
pipeline_b = Pipeline(preprocessor=MinMaxScaler(), model=xgb_model)   # trivial swap
```

---

**Q14. Why use an abstract base class (`ABC`) instead of relying purely on duck typing?**

Gives an explicit, enforced contract - instantiating a subclass missing a required method fails immediately, loudly, at instantiation time.

```python
from abc import ABC, abstractmethod

class BaseFeatureExtractor(ABC):
    @abstractmethod
    def extract(self, raw_input):
        ...

class TextFeatureExtractor(BaseFeatureExtractor):
    def extract(self, raw_input):
        return {"length": len(raw_input)}

class BrokenExtractor(BaseFeatureExtractor):
    pass   # forgot to implement extract()

# te = TextFeatureExtractor("hello")   # fine
# be = BrokenExtractor()                # TypeError: Can't instantiate abstract class
#                                        # BrokenExtractor with abstract method extract
#                                        # -- fails loudly at instantiation, not later at call time

# Duck typing equivalent: works, but silently fails later, deep in some call path
class DuckExtractor:
    pass
d = DuckExtractor()          # no error here...
# d.extract("x")              # ...AttributeError only surfaces when actually called
```

---

**Q15. Why use `is` instead of `==` when checking against `None`?**

`is` checks identity; `None` is a true singleton, so `is None` is both semantically correct and avoids surprises from an overridden `__eq__`.

```python
class WeirdNumber:
    def __init__(self, value):
        self.value = value
    def __eq__(self, other):
        return True   # deliberately broken __eq__ for illustration

w = WeirdNumber(5)
print(w == None)      # True  -- surprising! __eq__ was overridden to always return True
print(w is None)      # False -- correct: w is genuinely not the None object

# Standard idiom:
x = None
if x is None:          # correct, unambiguous, slightly faster (no __eq__ dispatch)
    print("x is None")
```

---

## Part D — Memory & Performance

**Q16. Why use a NumPy array instead of a Python list for numerical data?**

Contiguous, uniform-type memory layout gives cache locality and lets vectorized ops run as compiled C loops instead of interpreted per-element Python.

```python
import numpy as np
import time

n = 5_000_000
py_list = list(range(n))
np_array = np.arange(n)

# Python list: interpreted loop per element
start = time.perf_counter()
squared_list = [x**2 for x in py_list]
print("list comprehension:", time.perf_counter() - start)

# NumPy: vectorized, compiled C loop under the hood
start = time.perf_counter()
squared_np = np_array ** 2
print("numpy vectorized:", time.perf_counter() - start)   # typically 10-50x faster

# Tradeoff: fixed dtype, and appending is O(n) (full reallocation)
# arr = np.array([1, 2, 3])
# np.append(arr, 4)   # returns a NEW array every time -- O(n), don't do this in a loop
```

---

**Q17. Why use `__slots__` on a class instantiated millions of times?**

Skips creating a per-instance `__dict__`, meaningfully reducing memory for a large number of small object instances.

```python
import sys

class PointDict:
    def __init__(self, x, y):
        self.x = x
        self.y = y

class PointSlots:
    __slots__ = ("x", "y")
    def __init__(self, x, y):
        self.x = x
        self.y = y

p1 = PointDict(1, 2)
p2 = PointSlots(1, 2)

print(sys.getsizeof(p1.__dict__))   # real per-instance dict overhead
print(hasattr(p2, "__dict__"))      # False -- no __dict__ created at all with __slots__

# p2.z = 5   # AttributeError: 'PointSlots' object has no attribute 'z'
#            # -- can't add new attributes dynamically; this rigidity is the tradeoff
```

---

**Q18. Why use a generator expression instead of a list comprehension inside `sum()`/`any()`?**

The generator form streams one value at a time into the consuming function; the list form builds the whole intermediate list first for no benefit.

```python
# List comprehension: builds a full 1,000,000-element list, THEN sums it
total_a = sum([x**2 for x in range(1_000_000)])   # extra O(n) memory for a throwaway list

# Generator expression: streams values directly into sum(), no intermediate list
total_b = sum(x**2 for x in range(1_000_000))     # O(1) extra memory

# Same result, different memory profile -- prefer the generator form (no brackets)
# whenever the result is consumed once and never indexed/reused.
assert total_a == total_b
```

---

**Q19. Why avoid a mutable default argument (e.g., `def f(items=[])`)?**

Default values are evaluated once, at function *definition* time - a mutable default is silently shared and accumulates mutations across unrelated calls.

```python
def add_item_buggy(item, items=[]):     # DANGER: same list reused across calls
    items.append(item)
    return items

print(add_item_buggy("a"))   # ['a']
print(add_item_buggy("b"))   # ['a', 'b']  -- unexpected! previous call's data leaked in

# Correct idiom: use None as the sentinel, create fresh inside the function body
def add_item_fixed(item, items=None):
    items = items if items is not None else []
    items.append(item)
    return items

print(add_item_fixed("a"))   # ['a']
print(add_item_fixed("b"))   # ['b']  -- correct, independent each call
```

---

**Q20. Why use pandas vectorized operations instead of `.apply()`, and `.apply()` instead of an explicit loop?**

Vectorized ops run as compiled loops across the whole column at once; `.apply()` calls back into slow interpreted Python once per row; `.iterrows()` is slower still.

```python
import pandas as pd
import numpy as np
import time

df = pd.DataFrame({"x": np.random.rand(1_000_000)})

# Slowest: explicit loop via iterrows()
start = time.perf_counter()
result = []
for _, row in df.iterrows():
    result.append(row["x"] * 2)
print("iterrows loop:", time.perf_counter() - start)

# Middle: .apply() with a Python function -- still one Python call per row
start = time.perf_counter()
df["y_apply"] = df["x"].apply(lambda v: v * 2)
print(".apply():", time.perf_counter() - start)

# Fastest: vectorized operation -- compiled C loop across the whole column
start = time.perf_counter()
df["y_vectorized"] = df["x"] * 2
print("vectorized:", time.perf_counter() - start)
```

---

## Part E — ML-Serving & Ecosystem Choices

**Q21. Why use Pydantic for request/response validation instead of manually checking types?**

Validates and coerces against type hints automatically with structured error messages for every invalid field, and doubles as self-documenting API schema.

```python
from pydantic import BaseModel, ValidationError

class PredictionRequest(BaseModel):
    user_id: str
    features: list[float]
    threshold: float = 0.5

# Valid input: auto-parsed and type-coerced
req = PredictionRequest(user_id="u1", features=[0.1, 0.2, 0.3])
print(req.threshold)   # 0.5, default applied

# Invalid input: one call surfaces ALL problems clearly, not just the first
try:
    PredictionRequest(user_id="u1", features="not_a_list", threshold="also_wrong")
except ValidationError as e:
    print(e)   # structured, field-by-field error report

# Manual equivalent would need a hand-written if-chain checking each field individually,
# with inconsistent error messages and no free API documentation generated from it.
```

---

**Q22. Why use FastAPI instead of Flask for a new ML model-serving endpoint?**

Native `asyncio` support for concurrent I/O, plus automatic request validation and interactive docs generated directly from type hints.

```python
# FastAPI: async-native, validation + docs generated from type hints automatically
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class PredictRequest(BaseModel):
    features: list[float]

@app.post("/predict")
async def predict(req: PredictRequest):          # async def: non-blocking I/O by default
    result = await run_inference(req.features)    # can await downstream calls cheaply
    return {"prediction": result}
# Visiting /docs gives interactive Swagger UI for free, generated from PredictRequest

# Flask equivalent: synchronous by default, manual validation, no free docs
from flask import Flask, request, jsonify
flask_app = Flask(__name__)

@flask_app.route("/predict", methods=["POST"])
def predict_flask():
    data = request.get_json()
    if "features" not in data or not isinstance(data["features"], list):
        return jsonify({"error": "invalid input"}), 400   # manual validation
    result = run_inference_sync(data["features"])          # blocking call
    return jsonify({"prediction": result})
```

---

**Q23. Why use a context manager (`with open(...) as f`) instead of manual `.open()`/`.close()`?**

Guarantees resource release via `__exit__` even if an exception is raised partway through - a bare open/close with no `try`/`finally` leaks the resource on any exception.

```python
# Risky: leaks the file handle if anything between open and close raises
f = open("data.csv")
data = f.read()
process(data)          # if this raises, f.close() below never runs -> leaked handle
f.close()

# Safe: context manager guarantees cleanup even on exception
with open("data.csv") as f:
    data = f.read()
    process(data)       # if this raises, __exit__ still closes f automatically
# f is guaranteed closed here, exception or not

# Same pattern matters for DB connections in a long-running service:
with get_db_connection() as conn:
    conn.execute(query)
# connection guaranteed released back to the pool, even on error --
# critical when this code path runs millions of times in production
```

---

**Q24. Why use type hints in production ML code even though Python doesn't enforce them at runtime?**

Let static checkers (`mypy`) catch real bugs before deployment, make intent self-documenting, and are what make Pydantic/FastAPI's automatic validation possible at all.

```python
# Without type hints: intent is implicit, mypy can't help
def compute_score(user, weights, threshold=0.5):
    return sum(user[k] * weights[k] for k in weights) > threshold

# With type hints: intent explicit, mypy catches misuse before runtime
def compute_score_typed(
    user: dict[str, float],
    weights: dict[str, float],
    threshold: float = 0.5,
) -> bool:
    return sum(user[k] * weights[k] for k in weights) > threshold

# compute_score_typed(user=[1, 2, 3], weights={}, threshold=0.5)
# mypy flags this BEFORE running: Argument "user" has incompatible type "list[int]";
# expected "dict[str, float]"  -- caught statically, not discovered in production
```

---

**Q25. Why use `logging` instead of `print()` statements in a production ML service?**

Supports severity levels and configurable destinations without touching code, and routes output to a searchable, timestamped, filterable location instead of a busy stdout stream.

```python
import logging

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
logger = logging.getLogger(__name__)

# print(): always fires, no severity, easy to lose in a busy service's stdout
print("model loaded")

# logging: filterable by level, structured, can redirect to file/log-aggregator
logger.debug("feature vector: %s", features)    # silent in production (level=INFO)
logger.info("model loaded, version=%s", model_version)
logger.warning("feature 'x' missing, using default")
logger.error("inference failed for request %s", request_id, exc_info=True)

# Toggle verbosity per environment WITHOUT touching any of the log call sites:
# logging.basicConfig(level=logging.DEBUG)   # verbose in dev
# logging.basicConfig(level=logging.WARNING) # quiet in prod
```
