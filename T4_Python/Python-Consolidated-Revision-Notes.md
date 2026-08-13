# Python — Consolidated Revision Notes (01–13)

One-sitting cram sheet across all 13 Python notebooks. Dense by design — this is retrieval practice, not re-teaching. If a line doesn't immediately make sense, that notebook needs a real revisit, not just this summary.

---

## 01 — Fundamentals: Syntax & Control Flow

- Dynamically typed, **strongly** typed — `"2" + 2` raises `TypeError`, no silent coercion.
- `/` always float division; `//` floors **toward negative infinity** (`-7 // 2 == -4`, not `-3`); `%` sign follows the **divisor** (`-7 % 2 == 1`).
- `**` binds tighter than unary minus: `-2 ** 2 == -4`.
- Strings are immutable. `s += x` in a loop is *in principle* O(n²), but CPython optimizes the common case (refcount==1) into an in-place resize — that optimization silently breaks the moment another reference to the string exists (e.g. stored in a list). `''.join(...)` is O(n) unconditionally regardless of this CPython detail.
- Falsy: `0, 0.0, "", [], {}, set(), (), None`. Truthy (commonly missed): `"0"`, `[0]`, `-1`.
- `or`/`and` return **operands**, not booleans, and short-circuit: `0 or "default"` → `"default"`.
- `match`/`case` (3.10+) — structural pattern matching, `case [a, b]:`, `case {"type": t}:`.
- `enumerate(seq, start=1)`, `zip(a, b)` (stops at the shortest) — default over manual index counters.
- `for...else` / `while...else` — `else` runs only if the loop completed **without** `break`.

```python
seen = set()
for x in nums:
    if target - x in seen:
        break
    seen.add(x)
else:
    print("no pair found")   # only runs if the loop never broke
```

---

## 02 — Native Data Structures

| Op | Complexity | Note |
|---|---|---|
| `lst[i]` | O(1) | |
| `lst.append(x)` | O(1) amortized | |
| `lst.pop(0)` / `lst.insert(0,x)` | O(n) | shifts everything |
| `d[key]`, `x in dict`/`set` | O(1) avg | |
| `lst.sort()` | O(n log n) | |

- **Shallow-copy trap:** `[[0]*3]*3` — same inner list repeated by *reference*; mutating one row mutates all. Fix: `[[0]*3 for _ in range(3)]`.
- Tuples are hashable (usable as dict keys/set members); lists are not.
- `collections.Counter` — frequency counting; `.most_common(k)`.
- `collections.defaultdict(list)` — no manual "if key not in dict" boilerplate.
- `namedtuple` / `@dataclass(frozen=True)` for lightweight immutable records.
- **The single highest-leverage DSA habit**: "have I seen this before?" / membership checks → reach for a `set`/`dict` for O(1), not a `list`'s O(n) scan. Measured: ~5,600x faster on a 100k-element membership check.

---

## 03 — Functions & OOP for ML

- `*args`/`**kwargs` collect extra positional/keyword args — the mechanism that lets decorators forward *any* signature.
- Positional-only (`/`) and keyword-only (`*`) params: `def f(a, b, /, *, c):`.
- **Type hints are not enforced at runtime** — `def double(x: int): return x*2; double("ab") → "abab"`, no error. They're for tooling (mypy, FastAPI/Pydantic).
- Closures capture the **variable**, not its value at creation time — classic loop gotcha:
```python
funcs = [lambda: i for i in range(3)]
print([f() for f in funcs])          # [2, 2, 2], not [0, 1, 2]
funcs = [lambda i=i: i for i in range(3)]   # fix: default-arg forces early binding
```
- `functools.partial` (pre-fill args), `reduce` (fold), `lru_cache`/`cache` (memoization).
- **Decorators**: `@deco` = `func = deco(func)`. Always use `*args, **kwargs` in the wrapper + `@functools.wraps(fn)` (preserves `__name__`/`__doc__`; without it the wrapped function looks like `"wrapper"`).
- A decorator that takes its own arguments (`@retry(max_attempts=5)`) is a **decorator factory** — one extra layer of nesting.
- **Magic methods that matter for ML**:
  - `__repr__`/`__str__` — define `__repr__` at minimum; `print()` falls back to it.
  - `__len__` + `__getitem__` → makes a class work with `len()`, `obj[i]`, `for x in obj` — literally PyTorch's `Dataset` interface.
  - `__call__` → `obj(x)` instead of `obj.method(x)` — why `model(x)` works instead of `model.forward(x)` (PyTorch's `nn.Module.__call__` does bookkeeping then calls `forward`).
- `@staticmethod` (no self/cls access, namespacing only) vs `@classmethod` (`cls`, alternate constructors) vs `@property` (attribute-like access with validation behind it).
- Inheritance = "is-a" (`super().__init__()`); composition = "has-a". Prefer composition unless polymorphism is genuinely needed.
- `ABC` + `@abstractmethod` — instantiating an incomplete subclass raises `TypeError` **immediately**, not later when the missing method is finally called. This is scikit-learn's `fit`/`predict` and PyTorch's `forward` pattern.
- `@dataclass` — auto `__init__`/`__repr__`/`__eq__` from type-annotated fields; mutable defaults need `field(default_factory=...)` — a plain mutable default is rejected outright at class-definition time.

---

## 04 — Iterators, Generators & Context Managers

- **Iterable** has `__iter__`; **iterator** has `__next__` and raises `StopIteration` when done. A `for` loop = `iter()` once + `next()` repeatedly, catching `StopIteration`.
- A `list` yields a **fresh** iterator each `iter()` call (two independent loops don't interfere); a hand-written iterator that `return self` from `__iter__` gets **exhausted** after one full loop.
- `yield` → a generator function; calling it doesn't run the body, returns a generator object. Vastly less boilerplate than hand-writing `__iter__`/`__next__`.
- **List comprehension = eager** (built fully, now); **generator expression = lazy** (`(x for x in ...)`, computed on demand) — near-constant memory regardless of range size; this is *the* reason ML data pipelines stream instead of loading everything.
- `itertools.islice` takes a finite slice of an infinite generator. `yield from` delegates to a sub-generator (flattening nested structures).
- `with obj as x:` → `obj.__enter__()` / `obj.__exit__()`, **guaranteed** even on exception — this guarantee is the entire reason `with` exists over manual `try/finally`.
- `@contextlib.contextmanager` — a generator function; code before `yield` is `__enter__`, the `finally` block after `yield` is `__exit__`.
- `contextlib.ExitStack` — a variable number of context managers, all guaranteed closed. `contextlib.suppress(ExceptionType)` — one-line ignore.

---

## 05 — Threading, the GIL & Multiprocessing

- **GIL**: only one thread executes Python bytecode at a time in CPython (reference-counting needs atomic updates). Threads give **concurrency**, not **parallelism**, for CPU-bound work.
- GIL **is released** during I/O waits (`time.sleep`, network, disk) and many C-extension ops (NumPy) — measured: CPU-bound 4 threads → **1.04x** (no speedup); I/O-bound 4 threads → **3.99x** (near-linear).
- Race condition: `counter += 1` is read-modify-write, not atomic — interleaving corrupts it. Fix with `threading.Lock()` + `with lock:`.
- `Semaphore(n)` caps concurrent access to n at once. Deadlock = circular lock-waiting; prevent by always acquiring multiple locks in the same global order.
- `concurrent.futures.ThreadPoolExecutor` — `.map()` (ordered results) or `.submit()`+`as_completed()` (as-they-finish order).
- `ProcessPoolExecutor`/`multiprocessing.Pool` — **real** parallelism (separate GIL per process), but data must be pickled across the process boundary (real overhead), and process spawning itself costs time.
- Portability: Linux defaults to `fork` (routes added/code run after start "just work"); Windows/macOS default to `spawn` (re-imports the script) — **always** guard multiprocessing code with `if __name__ == "__main__":` for cross-platform code.
- **Decision rule**: I/O-bound → threading (or asyncio); CPU-bound → multiprocessing; CPU-bound-but-NumPy-heavy → threading often still works (releases the GIL internally).

---

## 06 — Async/Await & asyncio

- `async def` → calling it returns a coroutine **object**, doesn't run the body. `await` suspends the current coroutine, handing control back to the event loop.
- Forgetting `await` → silent no-op (coroutine created, never run); Python warns `RuntimeWarning: coroutine '...' was never awaited` only once garbage-collected.
- **`time.sleep()` inside async code blocks the entire event loop** — defeats the whole point. `asyncio.sleep()` yields control properly. Measured: 3 tasks with `time.sleep(0.2)` → ~0.6s (blocked/sequential); with `asyncio.sleep(0.2)` → ~0.2s (concurrent).
- Sequential `await` in a loop is still sequential. `asyncio.gather(*coros)` runs them concurrently — ~5x speedup measured on 5 fake API calls.
- `asyncio.create_task(coro)` — starts running **immediately** in the background; `await` it later to collect the result.
- **Why asyncio outscales thread-per-task at high concurrency**: an asyncio Task is a lightweight Python object, not an OS thread (~MBs of stack each) — measured 300 concurrent tasks: asyncio ~1.4x faster than 300 real threads, with the gap widening further at real production scale.
- Async generators (`async def` + `yield`, consumed with `async for`); async context managers (`@contextlib.asynccontextmanager`, `async with`).
- `asyncio.Semaphore(n)` — caps concurrent in-flight requests, essential for any real "call many APIs" pipeline (an unlimited `gather` over thousands of requests can exhaust file descriptors / hit rate limits).
- `asyncio.gather(..., return_exceptions=True)` — collects exceptions as values instead of cancelling everything on first failure.
- **Notebook-specific gotcha**: `asyncio.run()` fails inside Jupyter (`RuntimeError: cannot be called from a running event loop`, since the kernel itself runs one) — notebooks use top-level `await`; a real `.py` script uses `asyncio.run(main())` as the single entry point.

---

## 07–09 — APIs: HTTP, FastAPI, Pydantic, Serving

- HTTP verbs: `GET` (read, idempotent), `POST` (create, **not** idempotent), `PUT` (replace, idempotent), `PATCH` (partial update), `DELETE` (idempotent). Idempotent = safe to blindly retry.
- Status codes: `2xx` success, `4xx` **client** error (bad request — retrying is pointless), `5xx` **server** error (retrying can be reasonable). `422` = FastAPI/Pydantic validation failure.
- `requests`: `.status_code`, `.json()`, `.headers`, `.text`; **always set `timeout=`** — an unbounded request can hang forever; `.raise_for_status()` turns a bad status into a catchable `HTTPError`.
- FastAPI: a path-matching function parameter (`{item_id}`) → path param; a plain param → query param; a `BaseModel`-typed param → the JSON request body, validated automatically before the handler body runs.
- **A missing required `Header(...)` returns `422` from FastAPI's own validation — not whatever custom logic you wrote for a present-but-wrong header (that's a separate, later check, e.g. `401`).**
- Pydantic: `Field(gt=0, le=100)` numeric bounds, `min_length`/`max_length`, `Optional[X]` (defaults to `None`), nested `BaseModel`s parsed from nested dicts, `Field(alias="clientId")` for camelCase↔snake_case, `@field_validator` for custom logic, `pydantic-settings` for typed env-var config.
- `response_model=` filters the returned object down to exactly the declared fields (extra fields silently dropped) and powers the auto-generated `/docs`.
- Sync `def` route → runs in FastAPI's thread pool automatically (a slow sync call doesn't block others). `async def` route → runs on the event loop directly — must be genuinely non-blocking inside (no `time.sleep`, same rule as notebook 06).
- Auth: API key via a `Depends()` function reading a `Header`; JWT via `pyjwt` — `header.payload.signature`, `exp` claim checked automatically by `jwt.decode(..., algorithms=[...])`, raises `ExpiredSignatureError`/`InvalidTokenError`.
- Middleware (logging, `CORSMiddleware`) **must be registered before the app's first request** — unlike routes, which can be added even after the server is already running.
- CORS is enforced by the **browser**, not the server — a non-browser client (`requests`, server-to-server) ignores it entirely; not a security boundary by itself.
- Production model serving: load the model **once**, at startup (FastAPI `lifespan`), never inside the route function; batch requests where possible; keep the service stateless; add a health/readiness endpoint.

---

## 10 — Exceptions, Memory & Performance

- Custom exception hierarchy (`AppError` → `ValidationError`, `NotFoundError`) lets callers catch broadly or specifically.
- `raise X from Y` sets `__cause__`, preserving the original traceback when re-raising a different exception type — critical for real debugging.
- `except*` (3.11+) — handles multiple independent exception types from an `ExceptionGroup` (relevant with `asyncio.gather`-style concurrent failures).
- CPython frees objects the instant refcount hits zero — deterministic, no GC pause needed for the common case. **Cycles** (`A.other = B; B.other = A`) can't be freed by refcounting alone; the generational `gc` module finds and collects them.
- `weakref.ref(obj)` references without bumping refcount — doesn't keep the object alive, avoids creating cycles (caches, observer patterns).
- `__slots__` skips the per-instance `__dict__`, real memory savings at scale, but disables ad-hoc attributes (`AttributeError` on typos — a feature) and complicates multi-inheritance with differently-slotted classes.
- `cProfile` for time (where is it actually spent — profile before optimizing, never guess); `tracemalloc` (stdlib, no install) for memory, showing which line allocates the most.
- **A classic optimization now largely obsolete**: binding `_len = len` before a hot loop used to meaningfully help; on Python 3.11+'s adaptive interpreter the gap has mostly closed — measure before trusting old advice like this.

---

## 11 — Packaging & Project Structure

- `pyproject.toml` is the modern standard (replacing `setup.py`/`setup.cfg`): `[build-system]` (which tool builds it), `[project]` (name/version/`dependencies`), `[project.optional-dependencies]` (extras, e.g. `pip install pkg[dev]`), `[project.scripts]` (registers a real CLI command).
- `src/` layout (package under `src/pkgname/`) over flat layout for anything published/installed — forces tests to run against the **installed** package, catching "forgot to include this file" bugs a flat layout hides.
- `pip install -e .` (editable) — links back to source, so edits are visible immediately without reinstalling; `pip install .` copies, edits afterward have no effect. Real gotcha: a package installed **while a process is already running** isn't automatically importable in that same process (`site.addsitedir()` + `importlib.invalidate_caches()` fixes it) — normally invisible since you install before starting the process that uses it.
- `pyproject.toml` dependencies use loose ranges (installable alongside others); `requirements.txt` / a lockfile (`poetry.lock`, `uv.lock`) pins exact versions for one reproducible environment. Libraries ship the former; deployed apps add the latter on top.
- `venv`: `python3 -m venv .venv && source .venv/bin/activate` — isolates dependencies per project.

---

## 12 — Git & Version Control

- Three states: working directory → staging area (`git add`) → repository (`git commit`). `git diff` = working vs. staged; `git diff --staged` = staged vs. last commit.
- **`merge`** creates a join commit, preserving exact history — safe for shared branches. **`rebase`** replays commits on top of another branch, linear history but **rewrites hashes** — never rebase a branch others have already pulled.
- **`revert` vs `reset`** (the most-asked git interview question): `revert` makes a **new** commit undoing an old one (history intact, safe on shared branches). `reset` moves the branch pointer backward, **rewriting** history (`--soft` keeps changes staged, `--mixed` unstages, `--hard` discards) — never reset a commit others already pulled.
- `cherry-pick <commit>` — applies one specific commit onto the current branch without merging everything else from its source branch (backporting a fix).
- A bare remote created without `-b main` leaves `HEAD` pointing at a nonexistent ref (`master`) — clones silently come back **empty**. Fix: `git init --bare -b main`.
- `.gitignore` excludes matching files from `git add -A`. `git stash` / `git stash pop` — shelve uncommitted work, restore it later.
- `pull` = `fetch` + `merge` in one step; `fetch` alone is the safer default when you just want to see what changed without touching your files.
- Detached HEAD: checking out a specific commit (not a branch) — commits made here aren't on any branch and can be lost unless a branch is created before switching away.
- `.git` holds the **entire** history locally — every clone is a full backup, which is what "distributed" means here.

---

## 13 — Rapid-Fire Interview Gotchas

```python
# 1. Mutable default argument — evaluated ONCE at definition time, shared across calls
def f(x, bucket=[]):
    bucket.append(x); return bucket
f(1); f(2)   # [1, 2], not [2] — fix: default None, create inside

# 2. is vs == — value vs identity. Small ints (-5..256) are cached, "accidentally" making `is` work
256 is 256   # True (cached)      257 is 257   # often False (not cached — don't rely on it)

# 3. Shallow vs deep copy — copy.copy() shares NESTED mutables; copy.deepcopy() doesn't
original = [[1,2]]; shallow = copy.copy(original); original[0].append(9)
# shallow sees the change; copy.deepcopy(original) would not

# 4. Late-binding closures — captures the VARIABLE, not its value at creation
[lambda: i for i in range(3)]   # all return 2 — fix: lambda i=i: i

# 5. Exception variable scope — `as e` is deleted when the except block ends (breaks a ref cycle)
except ZeroDivisionError as e: print(e)   # fine inside
print(e)   # NameError outside the block

# 6. Float equality — binary floats can't represent 0.1/0.2 exactly
0.1 + 0.2 == 0.3   # False — use math.isclose() instead

# 7. Chained comparisons — a < b < c means (a<b) AND (b<c), both share the middle term
1 < 3 < 2   # False, not "translated" from (1<3)<2

# 8. Tuple of mutables — the tuple blocks REBINDING a slot, not mutating what's in it
t = (1, [2,3]); t[1].append(4)   # allowed; t[1] = [99] would raise TypeError

# 9. Class vs instance attribute shadowing — assignment always creates an INSTANCE attr
class C: count = 0
def inc(self): self.count += 1   # reads class var, creates a NEW instance var — other instances unaffected

# 10. global vs nonlocal — only needed to REASSIGN an outer-scope name, never to just read it
```

---

## Master Cross-Reference Table

| If asked about... | It's covered in |
|---|---|
| Big-O of list/dict/set ops | 02 |
| Decorators, `functools.wraps` | 03 |
| Why `model(x)` works in PyTorch | 03 (`__call__`) |
| Generators vs lists (memory) | 04 |
| GIL, threading vs multiprocessing | 05 |
| `asyncio.gather`, event loop | 06 |
| REST status codes, idempotency | 07 |
| Pydantic validation, 422 | 07–08 |
| API-key/JWT auth | 08–09 |
| `revert` vs `reset` | 12 |
| Mutable default arguments | 13 |

---

## Self-Check Before Calling This "Revised"

- [ ] Can state the amortized-O(1) reasoning behind `list.append()` and why front-inserts are O(n)
- [ ] Can write a decorator with `*args, **kwargs` and `functools.wraps` from memory
- [ ] Can explain the GIL's effect on CPU-bound vs I/O-bound threading, with the measured numbers
- [ ] Can explain why `time.sleep()` inside `async def` code is a real bug, not a style nit
- [ ] Can state the `revert` vs `reset` distinction without hesitating
- [ ] Can predict the output of at least 8 of the 10 gotchas in the rapid-fire section, cold, no peeking

If any box is unchecked, that notebook — not this summary — is the next stop.
