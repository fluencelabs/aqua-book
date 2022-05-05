---
description: Static configuration pieces that affect compilation
---

# Overrideable constants

### `const`

Constant definition.

Constants can be used all across functions, exported and imported. If a constant is defined using `?=` , it can be overridden by value via compiler flags or imported values.

```haskell
-- This can be overridden with -const "TARGET_PEER_ID = \"other peer id\""
const TARGET_PEER_ID ?= "this is a target peer id"

-- This constant cannot be overridden
const SERVICE_ID = "service id"
```

You can assign only literals to constants. Constant type is the same as literal type. You can override only with a subtype of that literal type.
