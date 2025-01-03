# Overview
Kaon is a Bitcoin and UTXO native blockchain with cross-chain composability. Our mission is to create a stronger fidelity of on-chain transfers of Bitcoin assets. The result is a decentralized consensus layer that not only offers the first cryptographically secure mirrored BTC, but is also bridge-less and oracle-less to make Bitcoin ecosystem interoperability secure, composable and scalable. 

# Architecture and Terminology

Kaon augments Bitcoin Nodes within the structure of Kaon to enable subprograms, which then introduces synthetic statefulness into Bitcoin. In doing so, we are able to implement on-chain programmability within Bitcoin, and facilitate high fidelity asset transfers of Bitcoin assets cross-chain.

We achieve this by leveraging secure Multi-Party Computation (sMPC) Groups to manage BTC locks and validate transactions, a Cross-Chain Mempool to relay transactions deterministically using UTXO features, and a BFT Consensus Layer to provide validator rotation and ensure Byzantine Fault Tolerance. The Node Augmentation acts as an intermediary, augmenting original nodes to trigger specific logic, receive transactions, and compose mirrored transactions essential for peg-in and peg-out processes, enabling seamless asset mirroring between Bitcoin and Kaon's Consensus Layer.

## Key Components

| Component | Description |
|------|-----|
| Node Augmentation    | Augments original node to trigger logic when receiving transactions and helps compose transactions for signing. |
| Cross Chain Mempool    | Ensures deterministic transaction relaying, by using UTXO features (outputs).|
|  Mirrored Transaction |  Transactions that move in either direction - from Bitcoin to Kaon's Consensus Layer or vice versa - when specific instructions are included in the transaction outputs. |
| Taproot Subprogram    | Defines acceptable Mirrored transaction and controls Mirroring process. |

## Architecture Overview

This diagram is an overview of the architectural framework of Kaon. Definitions for the terminology used in the diagram can be found in the **Terminology** section below.

```mermaid
%% Set default font style to Arial for compatibility
%% Use font-size and color to ensure text displays correctly in PowerPoint

flowchart RL
 subgraph s3[" "]
    direction TB
        BTC["Bitcoin"]
        CL["Consensus<br /> Layer"]
 end
 style BTC fill:transparent,stroke:#F7931A,font-family:Arial,font-size:14px
 style CL fill:transparent,stroke:#000000,font-family:Arial,font-size:14px
 style s3 fill:transparent,stroke-width:0px,font-family:Arial

 subgraph s4[" "]
    direction TB
         NI["Node Augmentation"]
 end
 style NI fill:none,stroke:#000000,font-family:Arial,font-size:14px
 style s4 fill:none,stroke-width:0px,font-family:Arial

 subgraph s1["BFT Consensus:"]
    direction TB
        n4["Crosschain Mempool"]
        s2
        n1["Verifies data accuracy<br> and integrity"]
  end
 style s1 fill:none,stroke:#000000,font-family:Arial,font-size:14px
 style n4 fill:none,stroke:#000000,font-family:Arial,font-size:14px
 style s2 fill:none,stroke:#000000,font-family:Arial,font-size:14px
 style n1 fill:none,stroke:#000000,font-family:Arial,font-size:14px

 subgraph s2["sMPC Groups / TSS Providers"]
    direction TB
        n3["Witness Group"]
        n2["Validator Group"]
 end
 style s2 fill:none,stroke:#000000,font-family:Arial,font-size:14px
 style n3 fill:none,stroke:#000000,font-family:Arial,font-size:14px
 style n2 fill:none,stroke:#000000,font-family:Arial,font-size:14px

    t1["*mirrored transactions can be mirrored again"]
    t2["Validator group: First required signature to send transaction to Kaon or to Bitcoin Network"]
    t3["Witness Group: Second required signature, approve locked BTC to be sent or mark it as corrupt"]
    t4["Transactions are processed in a similar manner as on Bitcoin"]
    
    s3 ~~~ s4 ~~~ s1
    n2 ~~~ n3
    n1 ~~~ s2
    CL -- Kaon Transaction --> NI
    BTC -- Kaon Transaction --> NI
    NI -- Mirrored Transaction --> CL
    NI -- Mirrored Transaction --> BTC
    CL -- Rotate and determine new signers for new epoch --> s1
    NI -- Kaon Transaction --> s1
    s1 -- Aggregates and broadcasts<br /> Mirrored Transactions to Kaon<br /> or Bitcoin --> NI
    n4 -- Pulls in transaction<br /> to be mirrored* --> s2
    s2 -- Pushes partially<br /> signed mirrored<br /> transaction --> n4
    s1 ~~~ t1
    CL ~~~ t2
    CL ~~~ t3
    CL ~~~ t4

    style n1 stroke-width:0px,fill:transparent,font-family:Arial
    style s3 stroke-width:0px,fill:transparent,font-family:Arial

    t1:::Ash
    t2:::Ash
    t3:::Ash
    t4:::Ash
    classDef Ash stroke-width:1px, stroke-dasharray:none, stroke:#999999, fill:#EEEEEE, color:#000000, font-family:Arial, font-size:14px

```
The architecture is designed to facilitate secure and deterministic cross-chain operations between the Bitcoin network and Kaon's Consensus Layer. At its core is the Consensus Layer, an L1 system providing **BFT** (Byzantine Fault Tolerance) by randomly selecting and rotating validators through epochs. Each **Epoch** determines a new set of validators, ensuring a dynamic yet deterministic participant pool.

Validators are organized into **sMPC Groups** (secure Multi-Party Computation groups), which are coordinated sets of nodes responsible for collectively producing threshold signatures and managing cross-chain operations. These groups have specific roles:

- **Validator Groups**: Maintain locks over received BTC and can send it, but only after approval from the corresponding Witness Group.
- **Witness Groups**: Approve the release of locked BTC or mark it as corrupt.

Individual nodes within these groups are called **sMPC Participants**. They hold key shares, participate in collective signing processes, and monitor network transactions for their assigned group.

The **Node Augmentation** serves as an intermediary, augments original nodes to trigger specific logic upon receiving transactions and assisting in composing Mirrored Transactions. These transactions are pivotal for the **Peg In** and **Peg Out** processes, effectively mirroring assets between the Bitcoin Network and Kaon's Consensus Layer.

To ensure deterministic transaction relaying and maintain process integrity, the **Cross Chain Mempool** leverages UTXO features. It acts as a conduit between sMPC Groups, Node Augmentations, and both networks, facilitating the broadcasting and detection of transactions, and acts as a State manager.

To maintain continuity across epochs, the system employs a process called **Sweeping**, which delegates BTC locks from the Validator and Witness Groups of the previous epoch to the sMPC Groups of the current epoch.

Epoch management is a collaborative effort between the Consensus Layer and the BFT Consensus, determining validator selection for each new epoch. In instances of malicious activity, the Consensus Layer has the authority to enforce slashing, penalizing validators by confiscating their stakes to uphold network integrity.

## Consensus Layer

| Term | Definition |
|------|-----|
| Consensus Layer    | L1 system that provides BFT by randomly selecting and rotating participants (validators) that help reach consensus and solve incidents. |
| Delegated Proof of Stake (dPoS)    | Incentivize users to confirm network data and ensure security through a process of collateral staking with an additional use of witnesses (formally called delegates).   |
|  Delegate Stakes  |  A transaction with specific lock script used in Kaon's dPoS to give a user an ability to participate in the chain's management,  can be combined or delegated. |
|  Masternode  |  Node connected to a wallet of Stakeholder.  |
|  Byzantine Fault Tolerance (BFT) Consensus  | Non-optimistic provably safe proof-of-stake binary consensus augmented to offer a proof of settlement and distributed key generation (DKG) through deterministic process where its participants take turns round-robin fashion, its consistency and liveness assuming a number of Byzantine corruptions did not exceeded 1/2, and it has the ability to converge back to safety once synchrony is restored.   |
|  Treshold Signature Scheme (TSS)  |  A method to collectively produce signature where every participant holds only a fragment of a private key, in Kaon's case utilizing Schnorr signature, Shamir Secret Sharing, Simple Coins as a source of deterministic random and Lagrange interpolation.  |
|  Safe Multiparty Computation (sMPC)   |  Secure multiparty computation (MPC / SMPC) is a cryptographic protocol that distributes a computation across multiple parties (in Kaon’s case secret message sequences or SMSs) where no individual party can see the other parties’ data.  |

```mermaid
sequenceDiagram
    Node-->>+Consensus Layer: Discover the network
    Node->>+Consensus Layer: Create stake
    Consensus Layer->>-Node: Announce Masternode
    Consensus Layer-->>-Node: End of Epoch
    Node->>+Consensus Layer: Apply to Validator
    Consensus Layer-->>+Old Validators: Start Election
    Old Validators->>-Consensus Layer: Vote for a Candidate
    Consensus Layer-->>-Node: Selected as<br />Consensus Layer validator
    Cross-chain Validator->>+Node: Add transaction to mempool
    Node->>+Consensus Layer: Propose block
    Consensus Layer-->>-Node: New block
    Node-->>-Cross-chain Validator: Transaction confirmed
```

- Step 1: Node discovers the network by connecting to the Consensus Layer.
- Step 2: Node creates a stake and sends it to the Consensus Layer.
- Step 3: Consensus Layer's nodes announces the node as a Masternode.
- Step 4: Consensus Layer notifies the node about the end of an epoch by reaching certain block height.
- Step 5: Node as a Masternode applies to become a Validator by creating a special transaction.
- Step 6: Consensus Layer starts an election by reaching certain block height.
- Step 7: Old Validators vote for a candidate by sending special transactions.
- Step 8: Consensus Layer informs the node that it has been selected as a Validator.
- Step 9: Cross-chain Validator adds a transaction to the node's mempool.
- Step 10: Node as a Validator proposes a block by sending it to the Consensus Layer.
- Step 11: Nodes of the Consensus Layer accept the block.
- Step 12: Node confirms the transaction with the Cross-chain Validator.

## BFT Consensus Layer

| Term | Definition |
|------|-----|
| BFT Consensus Layer    | Orchestrates data flows between all connected chains by setting up a framework of possible interactions and roles or all participants. |
| sMPC Group    | A coordinated set of nodes that collectively produce threshold signatures and manage cross-chain operations while operating as either Validators (maintaining locks) or Witnesses (approving operations). |
|  sMPC Group   |  A set of participants determined by the BFT consensus for the current epoch from the approved set of validators of the current epoch as determined by dPoS consensus.  |
| sMPC Paritipant    | An individual node that holds key shares and participates in collective signing processes while monitoring network transactions for its assigned group. |
|  Witness Group   |  sMPC Group with a specific role to approve locked BTCs to be sent or to mark it as corrupt.  |
|  Validator Group   |  sMPC Group with a specific role to maintain lock over received BTCs and to send it, which could be executed only after the related Witness Group approval. |

At the start of each cycle, we select a group of validators and randomly divide them into clusters. Any validators that can't form a complete cluster sit out for that cycle. These clusters are then randomly assigned roles as either Validators or Witnesses to keep things balanced. Each Validator Group pairs up with a Witness Group to work together, and any group without a partner doesn't participate this time. Each group participates in a Threshold Signature Scheme (TSS) process, where they collaboratively generate signatures, and a slot leader is determined for each session to coordinate the group's activities. The previous groups help distribute tasks fairly by passing on their roles to the new groups, aiming for an even spread of responsibilities. This whole process happens in two overlapping cycles, so groups can smoothly transition their roles and resources between them, making the network more secure and continuous.

<details>
 
  <summary>See detailed description</summary>
  
At the commencement of each epoch, a set of $N$ validators is selected to participate in the consensus process, where $N$ can be any positive integer. Utilizing a deterministic randomization mechanism — a simple coin flip based on bytes in the available-to-spend outputs — these validators are grouped into clusters of $k$ validators each (e.g., $k = 7$). The total number of complete groups formed is:

$G = \left\lfloor \frac{N}{k} \right\rfloor$

Validators that cannot form a complete group ($N \mod k$ validators) are excluded from the process in this epoch. This grouping ensures randomness while maintaining deterministic reproducibility, which is critical for decentralized consensus.

Next, these $G$ groups are assigned roles as either Validator Groups or Witness Groups using the same deterministic randomization algorithm. To achieve an equal number of each type — a crucial factor for maintaining network equilibrium — the number of Validator Groups $V$ and Witness Groups $W$ are determined as:

$V = \left\lfloor \frac{G}{2} \right\rfloor, \quad W = G - V$

The groups are then randomly assigned roles accordingly, ensuring statistical balance across the network. Subsequently, these groups are paired to form $P = \min(V, W)$ sets, each comprising one Validator Group and one Witness Group. Any group that remains unpaired due to an unequal number of Validator and Witness Groups will not participate in this epoch's process.

The pairing process employs the same underlying simple coin randomization but is modified to enhance security and unpredictability. Specifically, a deterministic permutation function $\pi$ is applied to the group indices to shuffle them before pairing:

$\pi(i) = \text{Permute}(i, s), \quad i = 1, 2, \dots, G$

where $s$ is a seed derived from blockchain data (e.g., previous block hashes), and $\text{Permute}$ is a deterministic permutation function ensuring that the shuffling is both unpredictable and reproducible. The shuffled groups are then paired sequentially.

Finally, the previous sets from the preceding epoch determine, based on the same deterministic basis, to which new groups they should delegate their locks. The goal is to achieve an equitable distribution across all new groups, which can be represented mathematically by minimizing the variance of delegated locks among groups:

$\text{Minimize} \quad \sigma^2 = \frac{1}{G} \sum_{i=1}^{G} (L_i - \bar{L})^2$

where $L_i$ is the total locks delegated to group $i$, and $\bar{L}$ is the average locks per group. This delegation occurs across two parallel epochs, with one operating at a half-phase delay relative to the other. Consequently, groups effectively transfer locks between these two flows, enhancing the continuity and robustness of the consensus mechanism.

</details>

## Actors' Interactions

### Component Roles

| Component | Role |
|------|-----|
| State   | Cross Chain Mempool |
| Observer    | Every sMPC Participant and any other Consensus Layer's node via Node Augmentation  |
| Signer | Validators Group, sMPC Participant, Consensus Layer's validators for emergency preimage |
| Validator | Witness Group |
| Dispute Resolver | Consensus Layer |

### Internal Interactions

| Component | Description |
|------|-----|
| sMPC Participant <-> Node Augmentation | Broadcasts Mirrored Transactions |
| Node Augmentation <-> sMPC Group | Receives transactions and composes Mirrored Transactions |
| Node Augmentation <-> Cross Chain Mempool | Detects new transactions to be mirrored and ensures correctness of the process state |
| sMPC Group <-> Cross Chain Mempool | Pushes prepared and signed Mirrored Transactions to be broadcasted to Bitcoin or Kaon's Consensus Layer. |
| sMPC Participant <-> Cross Chain Mempool | Broadcasts prepared Mirrored Transaction to Bitcoin or Kaon's Consensus Layer. |

### External Interactions

| Component | Description |
|------|-----|
| Node Augmentation <-> Bitcoin Node   | Receives newly confirmed transactions from Bitcoin Network and provides ability to broadcast transactions into Bitcoin Network. |
| Node Augmentation <-> Consensus Layer    | Receives newly confirmed transactions from Consensus Layer and provides ability to broadcast transactions into Consensus Layer. |
| sMPC Group <-> sMPC Participant | Produces an invoice addresses for peg-in transactions and sign Mirrored Transactions |
| Consensus Layer <-> BFT Consensus | Switches epochs and forms a list of participants for the next epoch, also resolves incidents and applies judgement via slashing. |


The interactions between components are as follows:

- The **Node Augmentation** provides to the **Cross Chain Mempool** utility to detect transactions' state.
- **sMPC Participants** broadcast mirrored transactions through the **Node Augmentation**.
- The **Node Augmentation** processes confirmed transactions with specific taproot signature with message "KAON" on its first leaf and interacts through the **Cross Chain Mempool** with sMPC Groups to ensure the correctness of the process state.
- The **Cross Chain Mempool** interacts with sMPC Groups to receive prepared and signed mirrored transactions for broadcasting.
- **sMPC Groups** represents collaboration of its sMPC Participants to produce invoice addresses for peg-in transactions and sign mirrored transactions using PBST standard.
- The **Consensus Layer** works with the BFT Consensus mechanism to switch epochs, form new participant lists, resolve incidents, and apply judgments via slashing.

Overall, the architecture is designed to handle cross-chain operations securely, transparently and uncontrollable. Through the coordinated efforts of sMPC Groups, deterministic transaction management, and a resilient consensus mechanism, the Kaon Network ensures efficient, continuous and secure interactions with the Bitcoin Network.

# Peg-In Process (Inbound)

| Kaon's Terminology | Definition |
|------|-----|
|  Peg In  |  Process of locking BTC in Bitcoin Network to get mirrored BTCs or to perform extended logic.  |

This process ensures secure bridging of BTC to kBTC while maintaining user control through matching private keys.

```mermaid
sequenceDiagram
  participant User as User
  participant Bitcoin Network as Bitcoin Network
  participant sMPC Participant as sMPC Participant
  participant sMPC Group as sMPC Group
  participant Kaon Network as Kaon Network

  User ->> Kaon Network: Request script/PBST(outputs) using his parameters -- number of outputs, amount
  Kaon Network -->> User: Taproot Script for a current epoch with a complex structure
  Note over User, sMPC Group: Taproot Script has multiple branches:<br /> 1) timelock for a few hours with ability to receive BTC back to the sender, in case if the used address is outdated and unused, <br /> 2) branch that allows TSS sMPC Validator Group to transfer BTC, if it was approved by TSS sMPC Witness Group, <br /> 3) gate for presigned image of the transaction to transfer BTC to another existed in a parallel TSS sMPC Validator Group<br /> which can be executed only if TSS sMPC Witness Group signed taproot branch for Rejection.
  User ->>+ Bitcoin Network: Broadcast Transaction with Kaon Sigscript
  Bitcoin Network ->> Bitcoin Network: Validate Transaction and Include Transaction in Block
  Bitcoin Network -->>- User: Transaction Confirmed
  sMPC Participant ->> Bitcoin Network: Monitor for Peg-In Transactions
  Bitcoin Network -->> sMPC Participant: New Block with Transactions
  sMPC Participant ->> sMPC Participant: Detect and Verify Peg-In Transaction
  sMPC Participant ->> sMPC Group: Participate in TSS applying
  sMPC Group ->> sMPC Group: Transfer Ownership of Pegged BTC
  Note right of sMPC Group: Outputs locked to Kaon Consensus via Taproot
  sMPC Group ->> Kaon Network: Create Mirrored Transaction
  Kaon Network ->> Kaon Network: Credit Mirrored BTC (kBTC) to User's Address
  Note left of Kaon Network: User's Address uses same private keys (HD wallets supported)
  Kaon Network -->> User: Mirrored BTC (kBTC) Available
  Note right of User: Access with sender's private keys
```

- Step 1: User broadcasts transaction with Kaon Sigscript.
- Step 2: Bitcoin Network validates and confirms transaction.
- Step 3: sMPC participants monitor for these peg-in transactions.
- Step 4: sMPC participants detect and verify peg-in transactions.
- Step 5: They participate in TSS signing.
- Step 6: Group transfers ownership of pegged BTC.
- Step 7: Outputs are locked to Kaon Consensus via Taproot.
- Step 8: sMPC Group creates mirrored transaction on Kaon Network.
- Step 9: Kaon Network credits mirrored BTC (kBTC) to user's address.
- Step 11: User can access funds using same private keys (HD wallets supported).

# Peg-Out Process (Outbound)

| Kaon's Terminology | Definition |
|------|-----|
|  Peg Out  |  Process of sending BTC out of Kaon Network's locking mechanic to a user within the Bitcoin Network.   |

The peg-out process is a secure method for users to withdraw Bitcoin from Kaon back to the main Bitcoin network.

```mermaid
sequenceDiagram
    %% Participants
    participant User
    participant Consensus Layer
    participant Validator Group
    participant Witness Group
    participant Bitcoin Network

    %% Peg-Out Request by User
    User->>Consensus Layer: Initiate Peg-Out Request
    Consensus Layer->>Validator Group: Notify Peg-Out Request

    %% Peg-Out Transaction Preparation
    Validator Group->>Validator Group: Prepare Peg-Out Transaction
    Validator Group->>Witness Group: Request Witness Signature for Peg-Out
    Witness Group->>Witness Group: Verify and Sign Peg-Out Transaction
    Witness Group->>Validator Group: Provide Witness Signature

    %% Peg-Out Execution
    Validator Group->>Bitcoin Network: Broadcast Peg-Out Transaction
    Bitcoin Network->>Bitcoin Network: Confirm Peg-Out Transaction
    Bitcoin Network->>User: Peg-Out Completed, BTC Transferred

    %% Notes
    note over Validator Group: Validators Must Obtain<br /> Witness Signature for Security
    note over Consensus Layer: Ownership Changes Effective Upon Peg-Out Execution
    note over Witness Group: Provides Verification and Security through Signatures
    note over Bitcoin Network: Transactions Reflect Ownership Changes Only When Executed
```

- Step 1: User sends peg-out request to Consensus Layer by sending kBTC to precompiled smart contract or by sending transaction with a specific taproot script.
- Step 2: Consensus Layer notifies Validator Group.
- Step 3: Validator Group prepares peg-out transaction.
- Step 4: Requests signature from Witness Group.
- Step 5: Witness Group verifies, signs, and returns signature.
- Step 6: Validator Group broadcasts signed transaction to Bitcoin Network.
- Step 7: Bitcoin Network confirms transaction.
- Step 8: User receives BTC.

Key Notes:

- Validator signatures require Witness approval for security.
- Ownership changes finalize upon execution.
- Witness Group provides security through signatures.
- Two validator groups exist: Group A (previous epoch) and Group B (next epoch).

# **Other Important Processes**

# Sweeping

The coordinated transfer of control and funds between validator groups during an epoch transition on Kaon.

```mermaid
sequenceDiagram
    %% Participants
    participant Consensus Layer
    participant Validator Group A
    participant Validator Group B
    participant Witness Group
    participant Bitcoin Network

    %% Epoch Transition Initiation
    Consensus Layer->>Consensus Layer: Initiate Epoch Transition
    Consensus Layer->>Validator Group B: Elect New Validators for Next Epoch
    note over Validator Group A,Validator Group B: Validator Groups Overlap During Transition
    Validator Group B-->>Validator Group A: Next group is announced

    %% Ownership Transfer Preparation
    Validator Group A->>Consensus Layer: Collect additional consensus enforced policies
    Consensus Layer-->>Validator Group A: Gather additional parameters for a transaction <br />i.e. slashing incidents, etc
    Validator Group A->>Validator Group A: Prepare Pre-Signed Ownership Transfer Transaction
    Validator Group A->>Bitcoin Network: Submit signed by validator transaction
    Bitcoin Network->>Witness Group: Wait for Witness Signature
    Witness Group->>Witness Group: Verify Pre-Signed Transaction
    Witness Group->>Bitcoin Network: Sign Transaction as Witness
    Bitcoin Network-->>Validator Group B: Transfer Transaction Confirmation
    Validator Group B->>Consensus Layer: Submit Signed Ownership Transfer Transaction
    Consensus Layer->>Consensus Layer: Update Ownership Records

    %% Commission Distribution
    Consensus Layer->>Validator Group A: Distribute Commission in BTC
    Consensus Layer->>Validator Group B: Distribute Commission in BTC

    %% Notes
    note over Validator Group A,Validator Group B: Validators Must Obtain Witness Signature for Security
    note over Consensus Layer: Ownership Changes Effective Upon Peg-Out Execution
    note over Witness Group: Provides Verification and Security through Signatures

    %% Group Roles Clarification
    rect rgb(230,230,230)
    note left of Validator Group A: Old Validators (Previous Epoch)
    note right of Validator Group B: New Validators (Next Epoch)
    end
```

- Step 1: Consensus Layer initiates epoch transition.
- Step 2: New validators (Group B) elected.
- Step 3: Groups overlap during transition.
- Step 4: Group B announced to Group A.
- Step 5: Group A collects consensus policies.
- Step 6: Kaon provides additional parameters (slashing incidents, etc.).
- Step 7: Group A prepares and submits pre-signed ownership transfer.
- Step 8: Witness Group verifies and signs transaction.
- Step 9: Bitcoin Network processes transfer.
- Step 10: Group B receives confirmation.
- Step 11: Group B submits signed ownership transfer to Kaon.
- Step 12: Kaon updates ownership records.
- Step 13: Both validator groups receive BTC commission.

Key Notes:

- Validator actions require witness signatures.
- Ownership changes finalize upon execution.
- Previous (A) and Next (B) epoch validators operate during transition.

| Kaon's Terminology | Definition |
|------|-----|
|  Sweeping  |  Process of delegating BTC locks maintained by Validator and Witness Groups of the previous Epoch to sMPC Groups of the current Epoch.  |
|  Epoch   |  Iteration of 80 blocks 13.(3) minutes long with a strict lifecircle, where election of validators for the next iteration is happened, allowing the BFT consensus to have non-fixed pool of participants in a determenistic manner. |
| Epoch   | Determines validator selection for the next iteration, allowing BFT consensus to maintain a deterministic yet dynamic participant pool. |
|  Mirrored BTC |  Bitcoin representation in Kaon's Consensus Layer produced by Mirrored Transaction through the Peg In process.  |

# Slashing
A security mechanism in Kaon that activates when unauthorized actions are detected to ensure network continuity and integrity. The Diagram explains the process using extreme situation when all sMPC Participants in a Validator group are malicious.

```mermaid
sequenceDiagram
    participant Validator Group A (Epoch N)
    participant Witness Group W1 (Epoch N)
    participant Validator Group B (Epoch N1)
    participant Witness Group W2 (Epoch N1)
    participant Consensus Layer
    participant Bitcoin Network

    %% Step 1: Malicious Activity by Validator Group A
    Validator Group A (Epoch N)->>Bitcoin Network: Perform Malicious Action (e.g., unauthorized transaction or remain inactive)
    note over Validator Group A (Epoch N): Malicious validator within the group acts improperly
    Bitcoin Network-->>Consensus Layer: Every connected Kaon node reports about a problem automatically

    %% Step 2: Detection by Witness Group W1 (Epoch N)
    Consensus Layer->>Witness Group W1 (Epoch N): Broadcast Validator Actions
    Witness Group W1 (Epoch N)->>Witness Group W1 (Epoch N): Detect Malicious Activity
    note over Witness Group W1 (Epoch N): Validates actions and detects misbehavior

    %% Step 3: Witness Group W1 Signs Rejection Branch
    Witness Group W1 (Epoch N)->>Bitcoin Network: Sign Rejection Branch of Taproot Script
    note right of Bitcoin Network: Rejection branch can now be executed

    %% Step 4: Execute Preimaged Emergency Transfer
    Bitcoin Network-->>Consensus Layer: Emergency Route is Unlocked
    Consensus Layer->>Bitcoin Network: Execute Preimaged Emergency Transaction
    note right of Bitcoin Network: Pre-signed transaction within Taproot script <br />signed by emergency Kaon Validators keys<br />for the current epoch

    %% Step 5: Emergency Transfer of Ownership
    Bitcoin Network->>Validator Group B (Epoch N1): Transfer Ownership of Outputs
    note over Validator Group B (Epoch N1): Receives control of BTC outputs from Group A

    %% Step 6: Consensus Layer Updates Records
    Validator Group B (Epoch N1)->>Consensus Layer: Confirm Emergency Transfer Executed
    Consensus Layer->>Consensus Layer: Update Validator Records
    Consensus Layer->>Validator Group A (Epoch N): Notify of Slashing and Loss of Ownership
    note over Validator Group A (Epoch N): Funds are slashed and penalties applied

    %% Step 7: Epoch Continues with New Validators
    Consensus Layer->>Validator Group B (Epoch N1): Confirm Active Status
    Consensus Layer->>Witness Group W2 (Epoch N1): Confirm Active Status
    note over Consensus Layer: Epoch proceeds with new validator and witness groups

    %% Additional Notes
    rect rgb(230,230,230)
    note over Validator Group A (Epoch N),Witness Group W1 (Epoch N): Participants in Epoch N
    note over Validator Group B (Epoch N1),Witness Group W2 (Epoch N1): Participants in Epoch N+1
    note right of Bitcoin Network: Emergency transfer allowed only between validator groups
    note right of Bitcoin Network: Requires witness signature on rejection branch
    note over Bitcoin Network: Taproot script contains preimaged transactions for such events
    end
```

- Step 1: Validator Group A performs unauthorized action.
- Step 2: Every connected Node Augmentation to Bitcoin Node reports issue to Kaon nodes.
- Step 3: Consensus Layer broadcasts to Witness Group W1.
- Step 4: W1 detects malicious activity.
- Step 5: Signs rejection branch of Taproot Script.
- Step 6: Unlocks emergency route.
- Step 7: Kaon's active dPoS validators executes pre-signed emergency transaction in Bitcoin Network.
- Step 8: Uses pre-signed transactions in Taproot script.
- Step 9: Ownership transfers to Validator Group B.
- Step 10: Group B receives transfer.
- Step 11: Kaon updates records.
- Step 12: Group A notified of slashing/penalties. Stakes of corrupted validators are confiscated.
- Step 13: Group B and W2 confirmed active.
- Step 14: Network continues with new validator/witness groups.

Key Notes:

- Emergency transfers only between validator groups.
- Requires witness signature.
- Taproot script includes pre-signed emergency transactions.
- Emergency transaction has lesser timelock in comparison to pegout transaction which makes possible it's execution before peg-out.
- Two epochs overlap during transition (N and N+1).
- If there are less then 50% of sMPC operators are malicious, emergency transaction is not required.
- Jailing process is applied for less severe situation and potentially can lead to eternal ban of the node by its peers.

| Kaon's Terminology | Definition |
|------|-----|
|  Slashing  |  Execution of validator or group of validators with confiscation of their stakes in case of detected and proven malicious activities, maintained by Kaon's Consensus Layer.  |

# sMPC Group Monitoring and Seeding

The system enables secure, distributed address generation while preserving privacy through consensus-based validation.
```mermaid
sequenceDiagram
    %% Participants
    participant Participant1
    participant Participant2
    participant ParticipantN
    participant sMPC Group
    participant Consensus Layer
    participant Customer

    %% Step 1: Participants create child key fragments
    par Generate Child Key Fragments
        Participant1->>Participant1: Generate Child Key Fragment
        Participant2->>Participant2: Generate Child Key Fragment
        ParticipantN->>ParticipantN: Generate Child Key Fragment
    and
    end

    %% Step 2: Participants send child key fragments to sMPC Group
    Participant1->>sMPC Group: Send Child Key Fragment
    Participant2->>sMPC Group: Send Child Key Fragment
    ParticipantN->>sMPC Group: Send Child Key Fragment

    %% Step 3: sMPC Group combines fragments to create new public address
    sMPC Group->>sMPC Group: Combine Fragments into New Public Address
    note over sMPC Group: New Group Address Created

    %% Step 4: sMPC Group commits new public address to Consensus Layer
    sMPC Group->>Consensus Layer: Commit New Public Address
    Consensus Layer->>Consensus Layer: Record Address, Ensure the Group is Healthy

    %% Step 5: Consensus Layer provides entry points to customers
    loop For Each Customer Request
        Customer->>Consensus Layer: Request Entry Point Address
        Consensus Layer->>Customer: Provide Group Address
        note over Customer: Maintains Bitcoin Network Privacy
    end

    %% Clarifications
    rect rgb(230,230,230)
    note over Participant1,ParticipantN: sMPC Participants use private key fragments<br /> created in a first MPC ceremony as a parent key
    note over sMPC Group: Two-rounds TSS with randomly selected slot leader per each slot,<br /> where random is deterministic and based on Consensus Layer.
    note over Consensus Layer: Uses group address contribution process to monitor health and provide entry points
    end
```
Loop Keys Generation:
- Step 1: Multiple participants (1, 2, ...N) simultaneously generate their own child key fragments.
- Step 2: All participants send their child key fragments to the sMPC Group.
- Step 3: sMPC Group combines received fragments to create a new public address.
- Step 4: sMPC Group commits new public address to Consensus Layer.
- Step 5: Consensus Layer verifies group health and records the address.

Customer Interface:
- Step 1: Customers can request entry point addresses.
- Step 2: Consensus Layer provides group addresses to customers.
