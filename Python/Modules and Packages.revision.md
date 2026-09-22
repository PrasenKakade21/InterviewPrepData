# Modules and Packages — Revision Notes (Intermediate)
> One-line mental model: `.py file = module`, `folder of modules = package`, `import = load it via sys.path`.
## Core recall
- `import math` → access via `math.sqrt()`; `import math as m` alias; `from math import sqrt, pi` selective.
- Avoid `from x import *` → namespace pollution, shadowing, unclear origin.
- Module = single `.py` file (e.g. `calculator.py` with `add/subtract`); Package = directory of modules (`utils/calculator.py`).
- `__init__.py` marks package, can re-export (`from .calculator import add`); Py3 supports namespace packages without it but still used for init/exports.
- `__name__ == "__main__"` when run directly, `== module_name` when imported; gate demo code with `if __name__ == "__main__": main()`.
- Absolute import: `from utils.validators import validate_email` (from top-level); Relative: `.` current, `..` parent (e.g. `from ..models.product import Product` in `services/`).
- `sys.path` = project dir + stdlib + site-packages search order; `__pycache__/*.pyc` bytecode, gitignore it.
- Run package module as `python -m app.main`, not `python app/main.py` (fixes relative-import error).
- Never name files `random.py/os.py/json.py` — shadows stdlib.
## Must-remember code
- ```python
  import math as m
  print(m.sqrt(25), m.pi)
  from math import sqrt as square_root
  print(square_root(25))
  ```
- ```python
  # calculator.py
  def add(a, b): return a + b
  if __name__ == "__main__":
      print(add(10, 20))
  ```
- ```python
  # absolute vs relative
  from app.services.product_service import create_product  # absolute
  from ..models.product import Product  # relative inside services/
  ```
- ```python
  import sys; print(sys.path)  # debug ImportError
  ```
## Interview traps
- `from utils import *` then `sqrt` collision → use explicit `import math; math.sqrt()`.
- `numbers[0] = x` style top-level `print()` in module runs on import → guard with `if __name__ == "__main__"`.
- `python app/main.py` with `from .utils import x` → ImportError; correct: `python -m app.main`.
- Relative `from ..routes import x` only works inside package context, not as standalone script.
- File named `random.py` + `import random` imports itself → rename file.
## 3 things to remember
1. `@ Absolute = from top`, relative = from where I am (`.`/ `..`).
2. Always guard runnable code with `if __name__ == "__main__"`.
3. Debug imports with `sys.path`; run packages with `python -m`.
