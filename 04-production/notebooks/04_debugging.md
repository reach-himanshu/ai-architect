# Production 04: Debugging & Profiling

Master the tools that separate junior from senior developers.

## 1. `pdb` - The Python Debugger

```python
import pdb

def buggy_function(x):
    pdb.set_trace()  # Breakpoint
    result = x * 2
    return result
```

### Key Commands
| Command | Action |
|:---|:---|
| `n` | Next line |
| `s` | Step into function |
| `c` | Continue execution |
| `p var` | Print variable |
| `l` | List code around current line |
| `q` | Quit debugger |

## 2. `breakpoint()` (Python 3.7+)
Built-in, no import needed.

```python
def my_func():
    breakpoint()  # Same as pdb.set_trace()
```

## 3. Profiling with `cProfile`

```bash
python -m cProfile -s cumtime my_script.py
```

## 4. Line Profiler (Install: `pip install line_profiler`)

```python
@profile
def slow_function():
    ...
```

Run: `kernprof -l -v my_script.py`
