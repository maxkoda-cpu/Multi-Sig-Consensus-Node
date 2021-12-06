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

**Deposits (XMR --> sXMR)**

Secret Monero Bridge deposits are made directly by users with the wallet of their choice. A deposit is made by sending an amount of Monero (XMR) to the Secret Monero Bridge Monero multi-signature wallet (46KTmvCDx862ijymLsCDVaCZ5UNc2A6yNhYEBt4t6AkrLf5CpF7XuB8HjUffdAfcZRTnZD1f3JyeTixqSsdMW7Sd9x1odvN).

Once Monero is sent to the Secret Monero Bridge Monero multi-signature wallet, a deposit can be processed. To start the processing of a deposit, the user must send an email message to secretmonero@i2pmail.org (an I2P email destination). The body of the email message must be an RSA 4096-bit encrypted string describing the details of the deposit which includes the Monero txid and txkey for the deposit transaction which we refer to as the "Monero Proof-of-Payment", along with the Secret Network address to receive the sXMR. The RSA 4096-bit encrypted message is created through our client applications (Web Dapp or Secret Monero Bridge CLI). The details of the deposit are encrypted to keep the information private in case the email is transferred over the clearnet (we advise sending emails through anonymous email accounts over end-to-end encrypted communications such as I2P mail so as to not leave metadata trails).

Once a deposit email is received, the Secret Monero Bridge decrypts the email message and then verifies the Monero Proof-of-Payment. If the Monero Proof-of-Payment is valid, and the specified deposit has not been previously processed, the deposit transaction moves to the next step in processing. If the Monero Proof-of-Payment is not valid, the email message is deleted. If a valid Monero Proof-of-Payment deposit transaction has been previously processed, the email is deleted and processing for that deposit is terminated (We will not process any valid deposit more than once).

Once the Monero Proof-of-Payment is verified as valid, and checked to make certain that the deposit transaction has not already been processed, the Secret Monero Bridge then submits the deposit transaction to the Multi-Sig Consensus Node api. The endpoint for the Multi-Sig Consensus Node api is an I2P destination (a base64 representation of the public I2P key of the destination). Each Multi-Sig Consensus Node has it's own i2p router configured with the Multi-Sig Consensus Node api configured as a hidden service.

Each Multi-Sig Consensus Node can accept deposit transactions through the decentralized Multi-Sig Consensus Node api. Once received, the receiving Multi-Sig Consensus Node claims ownership of the deposit transaction. The *owner* role, is provided authorization privileges for the transaction. The other Multi-Sig Consensus Nodes (without ownership) role, simply verify and sign multi-signature transactions for the deposit.

The *owner* Multi-Sig Consensus Node, verifies the Monero Proof-of-Payment, then creates a multi-signature *mint* sXMR transaction, and then signs the transaction that will eventually be executed on the Secret Network, once the required number of signatures (from additional Multi-Sig Consenus Nodes) are received. 

The *owner* Multi-Sig Consensus Node then records the deposit transaction (with signature) in its local database, and then replicates that transaction to all the other Multi-Sig Consensus Nodes via the Multi-Sig Consenus Node api.

When a Multi-Sig Consensus Node receives a replicated transaction, it verifies the Monero Proof-of-Payment (notice multiple confirmations), signs the transaction, updates its copy of the database, and then replicates its signature to all the other Multi-Sig Consensus Nodes (again via the Multi-Sig Consensus Node api).

Once the required number of signatures have been obtained for a given deposit transaction, the *owner* Multi-Sig Consensus Node then broadcasts the signed transaction on the Secret Network and the sXMR is minted. The receiving Secret Network address (which is included in encrypted email message for the deposit and a required data element for a deposit transaction), obtains the newly minted sXMR.

The *owner* Multi-Sig Consensus node then marks the transaction as complete and replicates the completed status to other Multi-Sig Consensus nodes via the api.

**Withdrawals (sXMR --> XMR)**

Secret Monero Bridge withdrawals are initiated directly by users, transfering a specified amount of sXMR to a Secret Monero Bridge multi-signature Secret Network account address. A withdrawal is processed once a withdrawal email message is sent by the user to the Secret Monero Bridge email address. Similar to deposits, withdrawal email messages include an RSA 4096-bit encrypted string containing the details of the withdrawal.

Once received by the Secret Monero Bridge backend, the email message is decrypted, and the withdrawal transfer information is verified. If the withdrawal transfer can't be verified, the email is deleted and processing is terminated. If the withdrawal transfer is verified, the withdrawal information is submitted to the Multi-Sig Consensus Node api.

Each Multi-Sig Consensus Node can accept withdrawal transactions through the decentralized Multi-Sig Consensus Node api. Once received, the receiving Multi-Sig Consensus Node claims ownership of the withdrawal transaction. The *owner* role, is provided authorization privileges for the transaction. The other Multi-Sig Consensus Nodes (without ownership) role, simply verify and sign multi-signature transactions for the withdrawal.

The *owner* Multi-Sig Consensus Node, verifies the withdrawal transfer of sXMR to the Secret Monero Bridge multi-signature Secret Network account, then creates a multi-signature *burn* sXMR transaction, and then signs the transaction that will eventually be executed on the Secret Network, once the required number of signatures (from additional Multi-Sig Consenus Nodes) are received. 

The *owner* Multi-Sig Consensus Node then records the withdrawal transaction (with signature) in its local database, and then replicates that transaction to all the other Multi-Sig Consensus Nodes via the Multi-Sig Consenus Node api.

When a Multi-Sig Consensus Node receives a replicated transaction, it verifies the the withdrawal transfer (notice multiple confirmations), signs the transaction, updates its copy of the database, and then replicates its signature to all the other Multi-Sig Consensus Nodes (again via the Multi-Sig Consensus Node api).

Once the required number of signatures have been obtained for a given withdrawal transaction, the *owner* Multi-Sig Consensus Node then broadcasts the signed transaction on the Secret Network and the sXMR is burned. 

The *owner* Multi-Sig Consensus node then prepares a Monero transfer transaction with the *amount* of sXMR, less the bridge fee. The *owner* Multi-Sig Consensus Node then signs the Monero transaction and requests the other Multi-Signature Consensus Nodes to refresh, synchronize their Monero wallet, and sign the transaction via the api. Each Multi-Sig Consensus Node being asked to sign, performs its own verification of the underlying withdraw transaction information before providing its signature. If for any reason a Multi-Sig Consensus Node is unable to verify a withdrawal transaction, it witholds its signature.

*Each Multi-Sig Consensus Node records all actions taken in its local database and replicates the transaction state to the other Multi-Sig Consensus Nodes via the api.*

Once the required number of signatures have been obtained, the *owner* Multi-Sig Consensus Node broadcasts the signed Monero transaction on the Monero network. 

The receiving Monero wallet address (which is included in encrypted email message for the withdrawal and a required data element for a withdraw transaction), receives the XMR. *The Monero Proof-of-Payment of the transfer is retained as evidence that the withdrawal payment was made.*

The *owner* Multi-Sig Consensus node then marks the transaction as complete and replicates the completed status to other Multi-Sig Consensus nodes via the api.


*More information to follow...*
