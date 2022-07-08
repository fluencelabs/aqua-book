# Service Management

#### Deploy A Service

The `aqua remote deploy_service` command allows you to deploy Wasm modules and associate service configuration specifications to some peer, where the command line structure looks like this:

```
aqua remote deploy_service \
--addr multiaddr  \
--config-path configs/service_config.json \
--service service_name \
--sk your_secret_key
```

where `--addr` is the multiaddr of some relay to connect to the network of choice. Use the  `--on peerId` option to deploy the service to a peer other than the connection relay.

`Note that --config-path` is a path to a service configuration file that includes the path to the Wasm modules.&#x20;

Consider the following service configuration JSON template:

```
{
  "service_name": {
    "modules": [
      {
        "name": "module_name",
        "path": "/path/to/wasm",
        "mounted_binaries": [["command_line_tool", "/path/to/command_line_tool"]],
        "logger_enabled": true
      },
      {
        "name": "second_module_name",
        "path": "path/to/second_module.wasm",
        "logger_enabled": true,
        "mapped_dirs": [["/path/to/dir"]],
        "preopened_files": ["/path/to/file"],
        "envs": [["env1", "env2"]],
        "max_heap_size": "100Mb"
      }
    ]
  },
  "another_service_name": {
    "modules": [
      {
	"name": "third_module",      
        "path": "./artifacts/mean_service.wasm",
        "logger_enabled": true
      }
    ]
  }
}

```

It is important to note that in a single configuration file, you can specify multiple service configurations, which are called from Aqua with `--service-name` flag referencing the respective config name, e.g., `service_name` and `another_service_name` respectively. For more information see the builtin [aqua-lib reference](https://github.com/fluencelabs/aqua-lib/blob/main/builtin.aqua#L206).

``

`--sk secret_key` a secret key that you can get with `aqua create_keypair`. It is a required field, cause you cannot manage or delete your service without this key.

Command output:

```
Your peerId: 12D3KooWCNDmmX9wGsbUuhP99A1V7tc2GseVq3Wx6KvRWNFck1Yx
"Going to upload a module..."
2022.02.17 15:55:12 [INFO] created ipfs client to /ip4/164.90.164.229/tcp/5001
2022.02.17 15:55:12 [INFO] connected to ipfs
2022.02.17 15:55:14 [INFO] file uploaded
"Now time to make a blueprint..."
"Blueprint id:"
"6f438ff43b9d5e7f980992e339a3eeef7da4d0e2fefca8f6229e440b509d454d"
"And your service id is:"
"a1076a0f-091b-4c80-84f3-1582cb02ecd9"
```

Here we see that modules are uploading to the node first. Then creating a blueprint. The last line is the most important. Service id will be used for service discovery, management and removal.

#### Remove A Service

```
aqua remote remove_service \
     --addr multiaddr \
     --sk your_secret_key \
     --id service_id
```

To remove service you need to know the host peer, service id and secret key. If the service location is different from your relay, use the  `--on peerId` option to provide the required peer location. Since only a service owner is authorized to delete a service, the secret key for the service needs to be provided.

#### Manual deploy

Deploying a service is a step-by-step process where the script will upload WASM modules, create blueprints and then will create services out of blueprints. You can do it manually to have more control and write your own scripts.

Upload a module to IPFS:

```
aqua ipfs upload --path /path/to/file \
     --addr /peer/multiaddress \
     --sk secret_key (if needed)
```

You will get CID of your module. Then you need to add a module from a vault. Right now you can do it with aqua service:

```
res <- Ipfs.get(cid)
conf <- Dist.make_module_config(...)
hash <- Dist.add_module_from_vault(res.path, conf)
```

`hash` - is a module hash that can be used in blueprint creation.

Create a blueprint:

```
aqua remote add_blueprint --name blueprint_name \
     --addr /peer/multiaddress \
     --dependency hash:ipfs_hash1
     --dependency hash:ipfs_hash2
```

Blueprint id will be as a result.

Create a service from a blueprint:

```
aqua remote create_service --name blueprint_name \
     --addr /peer/multiaddress \
     --id blueprint_id
```

Service id will be as a result.

Also, you can create services manually from Aqua language. You can have a close look at these examples and services:

{% embed url="https://github.com/fluencelabs/aqua-lib/blob/main/builtin.aqua#L189" %}

{% embed url="https://github.com/fluencelabs/aqua/blob/main/npm/aqua/dist.aqua#L36" %}
