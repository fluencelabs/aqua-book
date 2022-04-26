---
description: Scheduling Script
---

# Scheduling Scripts

Using Scheduled scripts it is possible to decouple service execution from the client and instead rely on a cron-like scheduler running on a node to trigger the service calls.&#x20;

To schedule an Aqua function, all argument values must be literal (strings, numbers, or bools). There is no error handling in these scripts as there is no service to send errors to, so, you need to handle errors on your own.&#x20;

#### Add Script

You can add script as follows:

```
aqua script add --sk secret-key -i path/to/aqua/file --func 'someFunc(arg1, arg2, "literal string")' --addr relay/multiadd --data-path path/to/json/with/args --interval 100
```

`--sk` is a secret key. You cannot delete script with different secret key.

`-i` is a path to an aqua file with your function

`--func` function with arguments that will be scheduled

`--addr` your relay

Use `--on` if you want to schedule a script on another node (not on the relay)

`--data-path` path to a JSON file with arguments

`--data` JSON string with arguments

`--interval` how often your script will be called, in seconds. If the option is not specified, then the script will be run only once

Output:

```
Script was scheduled
"dfc9cb4f-46be-48cb-a742-d3e23d03b6cf"
```

Where the last string is a script id. It could be used to remove your script.

#### Remove script

```
aqua script remove --sk secret_key --addr /relay/multiadds --script-id script_id
```

Output:

```
Script was removed
```

#### List scheduled scripts

You can get info aboud all scheduled scripts on node:

```
aqua script list --addr /relay/addr
```

Output:

```
[
  {
    "failures": 0,
    "id": "d1683d7e-cd9f-4c02-802e-250d800177d4",
    "interval": "1h",
    "owner": "12D3KooWDp7qmZDh83GUvSnfb43W5vqogwBrgMhEHA7nqdtgJJ3w",
    "src": "air-code-here"
  },
  {
    "failures": 0,
    "id": "3890e3d6-ae4a-45bb-9ab4-229cfee2893c",
    "interval": "1day",
    "owner": "12D3KooWDp7qmZDh83GUvSnfb43W5vqogwBrgMhEHA7nqdtgJJ3w",
    "src": "air-code-here"
  }
]
```
