**Overview**

*This document is a work in progress and will be written here over time -- maxkoda (2021-11-10)*

This document describes the Secret Monero Bridge backend which consists of a collection of decentralized nodes with the same software configuration.

Each node is assigned a unique key, which is a participant key, in a multi-signature scheme. The **Multi-Sig Consensus Nodes** participate as signers for 
Secret Network multisignature transactions that mint and burn sXMR tokens. Additionally, each Multi-Sig Consensus Node is assigned another unique key which
participates as a signatory for the Secret Monero Bridge, multi-signature Monero wallet.

Each Multi-Sig Consensus Node maintains a database. Collectively Multi-Sig Consensus Nodes replicate data amongst themselves to arrive at an eventually
consistent data set.

Deposit (XMR --> sXMR) and withdrawal (sXMR--> XMR) transaction flows will be described here to provide a comprehensive description of what happens on the
backend. This document will be structured with the intent of placing the various software components into context to provide a comprehensive overview of the 
Secret Monero Bridge backend.

Every Multi-Sig Consensus Node can initiate deposit and withdrawal transactions via an api call. The specific node that receives a deposit or withdrawal api call
is called the **owner** of the transaction. Every other node participates as a signatory to the **owner** for the transaction.

Multi-Sig Consensus Nodes are available through the api via the I2P network. The utilization of the I2P network provides not only anonymous end-to-end encrypted
network communication, it also decentralizes the api. This will be better explained in the walkthrough provided below.


*More information to follow...*
