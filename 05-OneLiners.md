# Basics and quick expressions
- **Print formatted string (f-string)**  
```python
name, n = "Anurag", 3
print(f"user {name} has {n} items")
```
- **Ternary conditional**  
```python
res = "OK" if x>0 else "NOT OK"
```
- **Swap two variables**  
```python
a, b = b, a
```
- **Inline loop with index**  
```python
for i, v in enumerate(["a","b","c"]): print(i, v)
```
- **Create a list of squares**  
```python
squares = [i*i for i in range(10)]
```
- **Generator (lazy) and next with default**  
```python
g = (x*x for x in range(5)); first = next(g, None)
```
- **Repeat a string**  
```python
dash = "-" * 40
```
- **Check membership**  
```python
found = item in container
```
- **Truthy test compact**  
```python
if items: print("has items")
```

# Lists, tuples, sets, dicts (data structures)
- **Reverse a list**  
```python
rev = L[::-1]
```
- **Flatten one-level nested list**  
```python
flat = [y for x in nested for y in x]
```
- **Unique preserving order**  
```python
seen=set(); uniq=[x for x in L if not (x in seen or seen.add(x))]
```
- **Chunk list into n-sized pieces**  
```python
chunks = [L[i:i+n] for i in range(0,len(L),n)]
```
- **Top N with heapq**  
```python
import heapq; top3 = heapq.nlargest(3, L)
```
- **Dict from two lists**  
```python
d = dict(zip(keys, values))
```
- **Invert dict (values unique)**  
```python
inv = {v:k for k,v in d.items()}
```
- **Merge dicts (Python 3.9+)**  
```python
merged = d1 | d2
```
- **Count frequencies (Counter)**  
```python
from collections import Counter; cnt = Counter(L)
```
- **Defaultdict accumulate**  
```python
from collections import defaultdict; dd=defaultdict(list); [dd[k].append(v) for k,v in pairs]
```

# Strings, text and regex
- **Reverse a string**  
```python
s[::-1]
```
- **Split/strip into words**  
```python
words = [w.strip() for w in s.split(",")]
```
- **Join list into string**  
```python
out = " ".join(words)
```
- **Replace multiple whitespace**  
```python
import re; clean = re.sub(r"\s+"," ", text).strip()
```
- **Find all words with regex**  
```python
import re; words = re.findall(r"\w+", text)
```
- **Extract first capture safely**  
```python
m = re.search(r"id:(\d+)", s); id = m.group(1) if m else None
```
- **Check palindrome (ignore nonletters)**  
```python
import re; t="".join(re.findall(r"[a-z]", s.lower())); t==t[::-1]
```
- **Format with padding**  
```python
f"{value:08d}"  # zero-pad integer to width 8
```

# File, OS, shell and quick scripting
- **Read entire file**  
```python
text = open("file.txt").read()
```
- **Read lines stripped**  
```python
lines = [l.rstrip("\n") for l in open("file.txt")]
```
- **Write to file**  
```python
open("out.txt","w").write(data)
```
- **Count lines in file**  
```python
n_lines = sum(1 for _ in open("file.txt"))
```
- **Tail last N lines**  
```python
from collections import deque; last10 = deque(open("file.txt"), maxlen=10)
```
- **Execute simple shell command and capture output**  
```python
import subprocess; out = subprocess.check_output(["ls","-1"]).decode()
```
- **One-liner HTTP server (shell)**  
```bash
python -m http.server 8000
```
- **Read stdin ints and sum (shell pipe)**  
```bash
cat file | python -c "import sys; print(sum(int(x) for x in sys.stdin))"
```

# Networking, web requests, JSON, CSV
- **Simple urllib GET**  
```python
import urllib.request; data = urllib.request.urlopen(url).read().decode()
```
- **Requests GET (if installed)**  
```python
import requests; text = requests.get(url, params={"q":"x"}).text
```
- **Load JSON from file**  
```python
import json; obj = json.load(open("data.json"))
```
- **Dump JSON pretty**  
```python
json.dumps(obj, indent=2, ensure_ascii=False)
```
- **Read CSV as dicts**  
```python
import csv; rows = list(csv.DictReader(open("file.csv")))
```
- **Post JSON with requests**  
```python
requests.post(url, json={"k":"v"}).status_code
```



# Data science, numeric, and small algorithms
- **Sum, min, max, mean**  
```python
import statistics; total=sum(L); mn=min(L); mx=max(L); mean=statistics.mean(L)
```
- **Product of elements**  
```python
import math; math.prod(L)
```
- **Running totals**  
```python
import itertools; list(itertools.accumulate(L))
```
- **Transpose matrix**  
```python
list(map(list, zip(*matrix)))
```
- **GCD of list**  
```python
import math; from functools import reduce; reduce(math.gcd, L)
```
- **Simple prime check**  
```python
is_prime = lambda n: n>1 and all(n%i for i in range(2,int(n**0.5)+1))
```
- **Apply function column-wise to list of dicts**  
```python
vals = [d["col"]*2 for d in rows]
```
- **Pandas quick load and head**  
```python
import pandas as pd; df = pd.read_csv("file.csv"); df.head()
```
- **Normalize list to 0-1**  
```python
mx=max(L); mn=min(L); norm=[(x-mn)/(mx-mn) for x in L]
```

# Concurrency, timing, debugging, introspection
- **Run function in thread**  
```python
import threading; threading.Thread(target=fn, args=(a,)).start()
```
- **Simple async call (Python 3.7+)**  
```python
import asyncio; asyncio.run(main())
```
- **Measure elapsed time**  
```python
import time; t=time.perf_counter(); f(); print(time.perf_counter()-t)
```
- **Pretty-print structure**  
```python
import pprint; pprint.pprint(obj)
```
- **Get callable members of object**  
```python
[k for k in dir(obj) if callable(getattr(obj,k))]
```
- **Dynamic attribute set**  
```python
setattr(obj, "name", value)
```
- **Instantiate with kwargs dict**  
```python
obj = MyClass(**kwargs)
```
- produce a printable one-page cheat sheet with a curated subset, or  
- filter and expand any single domain above (web, data science, scripting, text processing) with short, explained examples and best practices. Which domain should I expand next?
