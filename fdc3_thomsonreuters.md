# FDC3 - A Thomson Reuters View

This document was disussed on March 28th during the App-Directory Working Group. 

## About this document

This document is intended to describe some general observations/concerns that Thomson Reuters has with regards to the standard being worked on. 

We acknowledge that the FDC3 standard is work in progress and nothing has yet been signed off as the industry standard. For this reason, the **goal of this document** is to get the members' views on the points detailed below - and to agree what should or should not be included in the FDC3 standard.

Lastly, for full disclosure and transparency - we do have an interop API (`Side by Side`) and the below points are some challenges that we either faced during our initial implementation, or that are experiencing now and we are looking for solutions. 

All of the below items are of major importance for us Thomson Reuters. 


## Definitions 
Below are some definitions that are **only relevant in the context of this document**. It doesn't imply the same terminology needs to be applied across the standard.  

|Term|Definition|
|---|---|
|`product`| Software that has unique identity in the app directory, able to establish a handshake and **authenticate** as such. Responsible for **launching** other instances or applications. It can have product related `intents`. This is analogous to a `launcher` in `Plexus` land. Examples: _Symphony, Excel, Eikon, FactSet, OpenFin_|
|`application`| An `application` is instantiated by a `product`. It has specific `intents` and `context` that it can send/receive. The `type` could be explicitly stated (_ie charting_) or implied by its `intents`. |
|`component`| `Components` can be instantiated by a an `application`. These can be logical items within an app that have specific logic, and or can be individually targeted by another `application`. For example: _a leg in a swap, a line in a chart, an `application` in a Layout (multi-app) container._  The components can establish a _parent-child_ relationship across these instances. |
|`bus`| Without too much technical detail, this is the channel that enables the communication across _connected_ `applications` in the desktop.  |
|`app directory`| Holds the unique identity and metadata of the universe of `products`, `applications` and `components`, along with its relevant `intents` and `context` each of them can accept. Allows for discoverability. |
|`app manifest`| This is the _runtime manifest_ that holds the information of all the `product` and `applications` that are authenticated/connected to the bus. Perhaps a copy of its `intents` which has been fetched from the `app directory`. This `app manifest` could also hold the information about `application` instances (state), linked `applications`, as well as information regarding `application` subscriptions to a given channel. |
|`broker`| This is a `product` that is capable of instantiating a `bus` and responsible for providing authentication workflows, fetching information from an `app directory` as well as holding the `app manifest`. Not all **FDC3** compliant `applications` should be expected to be `brokers`.  |
|`consumer apps`| `Products` (`applications`) that know how to look for an already established `bus`, send authentication credentials (if needed), subscribe to events and send/receive `intents` and relevant `context`. These `consumer apps` are not responsible for establishing a `bus` or to hold an `app manifest` (although they should be able to query the `broker` for it). |

Other definitions 

|Term|Definition|
|---|---|
|`FDC3`| In the context of this document, `FDC3` refers to the standard proposal in its **current** state (as of March 28th 2018). It **does not** mean or reflect what `FDC3` can be in the future.  |
|`Plexus`| [Plexus](https://github.com/symphonyoss/plexus-interop) is an interoperability project hosted by the `FINOS` foundation (ex-Symphony Foundation) and contributed by Deutsche Bank. Main differences vs `FDC3` are that `Plexus` ~~is an interop implementation rather than a standard (although a standard could be based on it) and that it implies a single independent software (interop broker) will be instantiated (`broker`) and every other `application` becomes a `consumer app`.~~ **CORRECTION:** Plexus **is** as standard and it can support multiple concurrent brokers. |


## Interop Challenges

### App Hierarchy
We acknowledge that the majority of the applications serve a defined and encapsulated purpose _(ie charting app, chat app etc)_. However, there are other Desktop Software `products` that can host multiple applications with different purposes _(ie Thomson Reuters Eikon, REDI, Finsemble, FactSet etc)_.

* It is not expected that every underlying application hosted by these products (can be thousands) require to go through a handshake/authentication workflow. 
    * These workflows should be managed by the `product`.
    * The `product` is responsible of instantiating these `applications`. `Plexus` refers to these as `launchers`. 
* It is however expected that the `intents` and relevant `context` each of these `applications` accept, to be declared in the `app directory` for discoverability purposes.
    * The `application` identity could reflect this relationship through namespacing ie _eikon.chart_, _redi.spreadtrader_. 

### Brokers
Already defied above. There are multiple scenarios on regards to `brokers`. 

1. **Single** central interop broker (separate software). Every app becomes a  `consumer app`.  
    * **Pros:**   
        * Massively simplifies the management of buses
        * Better utilisation of physical resources (memory)
        * There are no `broker` compatibility issues. 
        * Simplifies configuration (`app directory` addresses, IDPs etc)
    * **Cons:** 
        * It may require to cover multiple authentication/entitlement scenarios to satisfy every `product's` needs. 
        * If the broker crashes, then all connections/subscriptions may get lost
2. **Multiple** concurrent `brokers`. 
    Consider the scenario where `OpenFin`, `Eikon` and `FactSet` are installed on the same Desktop. 
    * All of the above can be `brokers` in terms that they could've adopted the `FDC3` standard and instantiate a `bus` and manage authentication as well as an `app manifest`. 
    * If only one of them is running, the interop workflows are easy to understand. The same _cons_ and _pros_ of the **Single** model apply. 
    * However, if there is more than one running, then should there be:
        * A single **active** `broker` running (the first one) and the second one will act as a `consumer app`?
            * **Pros**
                * Possibility of a **active/passive** model: When the active `broker` is closed/crashes, one of the passive `brokers` is elected Active.  
            * **Cons**
                * `App manifest` information may get lost (unless passive `brokers` are aware of it at all times)
                * Need to re-establish subscriptions 
        * Multiple `brokers` where each `broker` instantiates its own `bus` and interoperate amongst them? 
            * `Consumer apps` may connect to either `broker`. 
            * A _bridge_ is established across `brokers`
            * Every `brokers` holds a copy of the entire `app manifest`.
            * **Pros:**
                * If any of the `brokers` closes down (or crashes), then the `app manifest` is knows by the rest of other `brokers` and connections can be re-established. 
                * There can be a fall-back mechanism for `consumer apps`. 
            * **Cons:**
                * Complicates the architecture of `brokers`
                * Resource (memory) utilisation
                * Version compatibility challenges
        * Both (or more) `broker` instantiate a `bus` but they act independently? 
            * **Pros:**
                * Simplifies the architecture
            * **Cons:**
                * Creates complexity for the `consumer` apps and challenges for the user in having to understand which `product` to connect to.
                * Resource (memory) utilisation
                * Discoverability becomes a challenge
3. **Peer to Peer** Every `product` is a `broker`. 
    * There is a point to point connection across every participant
    * Every app is aware of the app manifest at all times
        * **Pros:**
            * Every app can manage their own entitlements
            * The `app manifest` is known by all participants, therefore if any of the `applications` closes or crashes, the `app manifest` would automatically reflect this. All other connections/subscriptions are kept alive. 
        * **Cons:**
            * Creates massive complexity (overkill) for every `consumer application` having to handle all of this. 
            * All `applications` would need to fetch information from the `app directory`
            * Resource (memory) utilisation
    
### App (Instance) State
Consider the following workflow:
```
redi: tell eikon to launch a chart and set IBM as the context
eikon: ok. ChartID:abc123
redi: tell eikon to update the context of ID:abc123 to VOD
```

For the above scenario it is indispensible to keep an `app manifest` and know that there is an instance `id:abc123` of type `eikon.chart`. Only this way we can keep interacting (interoperating) with these objects. 

We **cannot** expect that we need to launch a new instance everytime a interaction is needed (fire and forget). 

### App Definition Inheritance 
Consider the following scenario:

* ChartIQ registers its `intents` and `context` in the `app directory`. 
* ChartIQ can be accessed as a standalone `application`. Or it can be accessed through Symphony's Marketplace as well as from Eikon's App Studio. 

Is it expected to re-register the whole set of `intents` and `context` for each scenario (3) in the `app registry`, ie to have _chartiq, symphony.chartiq and eikon.chartiq_ with their full definition? Or is their a way that we can refer to a base definition and only the `product` (launcher) changes? 

### Discoverability | Multiple App-Registries

An `app registry` can live at the Desktop level, at premise or hosted in the cloud. 

* Multiple `app registries` should be accessible on a given session. E.g. _a bank may have its own `app registry` in premise, and fall-back to a cloud based one._
* Configuration files should be centralised and accessible by all brokers. These files should state the multiple `app registry` addresses, either local to the desktop, local to the customer or cloud based. 

### Security | Authentication

It is not expected that the `FDC3` will act as an IDP or work with a single IDP. However it is important to acknowledge that security and authentication are **critical** to interoperate across `brokers`. 

* How are going to manage multiple IDPs? 
* What about encryption across `brokers`? 
* Where to define _trusted_ `broker`? 

### Security | Entitlements 

This **does not** relates to app specific entitlements (ie content, features permissioning etc). 
It is more about:

*  is `product a` allowed to talk to `product b`? 
*  is `app a` allowed to talk to `app b`? 
*  can `app a` subscribe to a given channel? 
*  can `product a` request a copy of the `app manifest` for `instances` that already were loaded previous to `product a` joining the `bus`? 
* Can the the user manage these entitlements? 
    * If so, where is this held?


