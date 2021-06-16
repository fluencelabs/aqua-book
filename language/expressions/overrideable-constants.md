---
description: Static configuration pieces that affect compilation
---

# Overrideable constants



`const`

Constant definition.

Constants can be used all across the functions, exported and imported. If a constant is defined using `?=` , it can be overriden by-value via compiler flags or imported values.

```text
-- This can be overriten with -const "target_peer_id = \"other peer id\""
const target_peer_id ?= "this is a target peer id"

-- This constant cannot be overriden
const service_id = "service id"
```

You can assign only literals to constants. Constant type is the same as literal type. You can override only with a subtype of that literal type.

