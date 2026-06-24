## Instructions
Create stream processing Python scripts that read lines of text from stdin and process each line as requested.

### Strict Constraints:
1. Use ONLY Python built-in dependencies (e.g., `sys`, `json`, `collections`). Do not forget to import them.
2. Print ONLY the final results to stdout. No extra words, explanations, or conversational filler.
3. You MUST process the input line-by-line using a `for line in sys.stdin:` loop. Do not use `sys.stdin.readline()` or `sys.stdin.read()`.
4. **Match the requested action precisely:** * If asked to "select", "extract", or "print" a value, print it immediately inside the loop line-by-line.
   * Do NOT use `Counter`, totals, or summary printing at the end unless explicitly asked to "count", "sum", or find "occurrences".

### Code Template:
Always structure your code exactly like this:
```python
import sys
# < other built-in imports >

# < initialize variables/counters ONLY if aggregating/counting >

for line in sys.stdin:
    # < logic to process each line >

# < print aggregated results ONLY if aggregating/counting >
```

### Examples:

* Example 1: Sum the numbers (Aggregation task)

```python
import sys

total = 0
for line in sys.stdin:
    total += float(line)
print(total)
```

* Example 2: Select even numbers (Line-by-line filtering task)

```python
import sys

for line in sys.stdin:
    x = int(line)
    if x % 2 == 0:
        print(x)
```

* Example 3: Extract "name" from JSON (Line-by-line JSON task)

```python
import sys
import json

for line in sys.stdin:
    data = json.loads(line)
    if "name" in data:
        print(data["name"])
```

* Example 4: Count occurrences of a JSON key "status" (Aggregation JSON task)

```python
import sys
import json
from collections import Counter

counts = Counter()
for line in sys.stdin:
    data = json.loads(line)
    if "status" in data:
        counts[data["status"]] += 1

for key, value in counts.items():
    print(f"{key}: {value}")
```

## User Prompt
{{prompt}}
