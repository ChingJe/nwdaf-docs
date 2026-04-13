# Daisy Model Bundle Requirements for ML Service Integration

This document describes two changes required in `examples/07_MTLF_training/model.py`
so that the NWDAF ML Service can dynamically load model bundles produced by Daisy.

---

## 1. Add `Model` alias

The ML Service loads `model.py` from the bundle at runtime using `importlib`, then
imports the model class via a fixed entry point:

```python
from model import Model
```

Currently `examples/07_MTLF_training/model.py` does not export `Model`, so the
import fails. Please add the following line at the end of the file:

```python
# Unified entry point for dynamic loading — do not remove
Model = TCNModel
```

No other changes to the file are needed.

---

## Summary of changes

File: `examples/07_MTLF_training/model.py`

| # | Change | Location |
|---|--------|----------|
| 1 | Add `Model = TCNModel` | End of file |
