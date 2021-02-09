# High-availability validator setup

* **Status:**
* **Proposer:** mmagician
* **Projects you think this work could be useful for:** Polkadot & Kusama Validators

## Project Description :page_facing_up: 

### Summary

Validator leader selection via Raft consensus for submitting blocks. Inspired by internal discussions & [certus.one active-active validator setup](https://kb.certus.one/validator_ha.html#active-active-validator).

### Present state

Currently, the recommended setup is to have one active node per validator. The main advantage of this approach is that it removes the danger of double-signing, thus preventing slashing. The key drawback is the lack of a ready backup machine to takeover the responsibility of producing blocks in case the primary one fails. 

The drawback can be somewhat remedied by having a backup node in sync, but without access to the session keys necessary for authoring blocks. The process of replacing the keys, however, is manual. Furthermore, the session keys cannot be replaced mid-session and this introduces a time delay for when the validator is active again.

### Solution

Rather than relying on one validator node to author blocks, what if we had multiple nodes equally capable of signing, yet still avoiding the risk of equivocation? The proposed approach relies on recognising a distinction between **authoring** a block and **submitting** it. 

Since all our validator nodes are trusted and we don't worry about censorship resistance, we can leverage a leader-follower model to ensure high availibility. [Raft](https://raft.github.io/) is a good candidate - it offers a simple consensus mechanism, fault-tolerance and performance. It ensures only one leader is ever in charge of interacting with the blockchain, while the followers simply receive the state updates and automatically replace the leader in case of a failure.


## Deliverables :nut_and_bolt:

* **Total Estimated Duration:** 3 months
* **Full-time equivalent (FTE):** 1
* **Total Costs:** 30,000 DAI

### Milestone 1 - Block authorship PoC

The main goal of the PoC is to add a switch to the substrate node which allows it to decide whether it's a valid block producer or not. 
The concept could be analogous to the [remote signing](https://github.com/paritytech/substrate/pull/7628) feature, in that the node could reach out to an endpoint (local or remote) which tells it whether the node is allowed to submit new blocks.

The service should contain only basic logic (e.g. return `true` for node0 and return `false` for node1 & node2).

* **Estimated Duration:** 1 month
* **FTE:** 1
* **Costs:** 10,000 DAI


| Number | Deliverable | Specification | 
| ------------- | ------------- | ------------- |
| 1. | Basic service | Create a microservice (to run locally or remotely) that accepts connections from the substrate node. | 
| 2. | Microservice logic | Hardcoded logic for deciding which node is the leader
| 3. | Update substrate client | Modify the substrate code to reach out to a remote service for receiving permission (or not) for submitting blocks |  
| 4. | Allow as optional | The choice of using an outside decision making agent for block submission should be configurable in the cli |
| 5. | Integration test | A dockerised setup that allows to test the developed PoC | 

### Milestone 2 - Raft consensus

Replace the dummy microservice with a Raft consensus mechanism, responsible for leader selection. 

Each node should integrate a Raft client in its code. A good candidate is tikv-client. It should be able to push/receive information on what the latest authored block is (persistent data storage) & act accordingly. 


* **Estimated Duration:** 1 month
* **FTE:** 1
* **Costs:** 10,000 DAI


| Number | Deliverable | Specification | 
| ------------- | ------------- | ------------- |
| 1. | Run the necessary Raft services | Add a key-value (or other) store binaries that follows a Raft consensus. They should run alongside the validator code |
| 2. | Integrate a Raft client into the node | Extend the block submission logic to allow only Raft-selected leader as a valid block submitter | 
| 3. | Integration test | A dockerised setup that allows to test the Raft consensus mechanism | 



### Milestone 3 - Production readiness

Take the previous developments to a state where it's ready to be deployed in production as part of a Polkadot/Kusama validator setup.


* **Estimated Duration:** 1 month
* **FTE:** 1
* **Costs:** 10,000 DAI


| Number | Deliverable | Specification | 
| ------------- | ------------- | ------------- |
| 1. | Security | Make sure that a firewall and proper networking is in place |
| 2. | Monitoring | Feed the Raft consensus data into Prometheus and display basic info in Grafana.
| 3. | Validator setup integration | Integrate the above into one of the validator-setup repositories (e.g. https://github.com/w3f/polkadot-secure-validator)
