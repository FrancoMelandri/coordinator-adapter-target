# Coordinator Adapter Target

## abstract
A bunch of micro services cooperating each other to reach third party integration scalability.

A connector is a single microservice able to connect to a defined **target** in order to implement the related integration logic.

We can add connectors to the domain just following the steps:
- creating a new micro service to handle the integration
- including the new target in the domain changing the adapter configuration
- configuring the target's `flavor` in the repository

<br/>

## architecture

<br/>

![CAT](./assets/cat.png)

<br/>

## flow

<br/>

```mermaid
sequenceDiagram
    participant Client
    participant Coordinator
    participant Repository
    participant Adapter
    participant Target

    Client->>+Coordinator: /action/{application1}
    Coordinator->>+Repository: /config/{application1}
    Repository->>-Coordinator: target and flavor1
    Coordinator->>+Adapter: /action/target/flavor1
    Adapter->Adapter: resolve target
    Adapter->>Target: /action/flavor1
    activate Target
    Adapter-->>Coordinator: OnAdapted
    Coordinator->>+Repository: Add event
    Target->>Adapter: ok
    Adapter->>-Coordinator: ok
    Coordinator->>-Client: ok
    Target->>Target: Process async integration
    Target-->>Coordinator: OnProcessed
    deactivate Target
    Coordinator->>+Repository: Add event


```
<br />

## cat-coordinator
<br />

The coordinator micro service is the "orchestrator" for the entire logic in the CAT domain. The coordinator is similar to the __SAGA__ pattern orchestrator.
<br />

It acts as entry point for the exposed RESTful api and it is responsible to retrieve information from repository to handle the incoming request.
<br />

Using the parameter `applicationId` we can retrieve __target__ and __flavor__ data to call the adapter.
- __target__ identify a kind of integration
- __flavor__ are the settings allowed by the selected target

<br />

> Usually the coordinator subscribes to events in order to handle notifications coming form all the other microservices.
<br />
The coordinator should be able to store data in the repository using the `event sourcing` pattern.

<br />

## cat-repository
<br />

The repository micro service is a basic CRUD service.

<br />

## cat-adapter
<br />

The main purpose of the adapter micro service is to resolve the right target to perform the action.
<br />
Once the target was resolved the adapter will call it using the __flavor__ parameters.
<br />
For each target we can have different flavor representing different target options.
<br />

## cat-target
<br />

The target micro service is responsible to implement the right third party integration.


<br />

## use cases

[cat-pay](./cat-pay.md)
<br />

[cat-export](./cat-export.md)
<br />

[cat-email](./cat-email.md)