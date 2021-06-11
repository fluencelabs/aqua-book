# Tour De Aqua

* Why Aqua -- not in order
  * particle model
  * client server model
  * p2p + Aqua model
  * request-response pattern
  * chain-forward pattern
* Note on Marine, Wasm IT

Given an abundance of active and abandoned programming languages, why create another one ? The need for Aqua arises from the desire to maximize the potential afforded by peer-to-peer networks as a distributed hosting environment for services composable into applications and backends.  

Figure x: need one new graphic to illustrate both aspects

That is, Aqua provides the capabilities necessary to implement and execute a "full-stack" peer-to-peer programming model where Aqua is used to program the network as well as compose applications from distributed services providing the following benefits:

* Composition without centralization
* Communication, access and execution security as first class zero trust citizens
* Programmable network requests
* Extensible beyond peer-native services to Web2 resources

At the heart of the peer-to-peer programming model --  is this Fluence or Aquamarine ?

*  _particle_

###  A Taste Of Aqua

or a different example?

```text
service Greeting("service-id"):
    greeting: string, bool -> string

func greeter(name: string, greet: bool, node: string, service_id: string) -> string:
    on node:                                                      
      Greeting service_id
      res <- Greeting.greeting(name, greet)
    <- res
```



