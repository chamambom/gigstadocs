---
title: Python Dependency Conflicts
nav_order: 3
parent: Operations
---

# Python Dependency Conflicts

## Starlette Version Conflict

The following conflict arises when installing certain package combinations:

```text
The conflict is caused by:
    fastapi-mail 1.5.0 depends on starlette<1.0 and >=0.24
    fastapi 0.115.12 depends on starlette<0.47.0 and >=0.40.0
    prometheus-fastapi-instrumentator 8.0.2 depends on starlette<2.0.0 and >=1.0.0
```

## Resolution Options

To fix this you could try to:

1. Loosen the range of package versions you've specified
2. Remove package versions to allow pip to attempt to solve the dependency conflict

```text
ERROR: ResolutionImpossible: for help visit https://pip.pypa.io/en/latest/topics/dependency-resolution/#dealing-with-dependency-conflicts
```