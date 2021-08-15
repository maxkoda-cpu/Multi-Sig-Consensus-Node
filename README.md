# Multi-Sig-Consensus-Node
Secret Monero Bridge Multi-Signature Consensus Node

The Multi-Signature Consensus Nodes (MSCNs) are a decentralized collection of p2p nodes running on the I2P network. They process deposit and withdraw messages and perform multi-signature concensus operations. Implementing both Monero and Secret Network multi-signature technology.

The MSCNs do **not** utilize the Monero Multisig Messaging system (MMS) and do **not** utilize PyBitmessage. We use a simple messaging system that does not require a third party package for Monero multi-signature communications. 

Each MSCN maintains it's own copy of system data. Deposit and withdraw messages are replicated across all active MSCNs to ensure each MSCN has current data.
