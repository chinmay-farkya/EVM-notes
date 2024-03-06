# EVM
This is an accumulated list of pointers about the Ethereum network and the low-level EVM execution. Note that some parts of this information might be outdated/ wrong : these are based on my understanding of the concepts. Also, the pointers are not too structured.

Feel free to add or correct me on anything. Please spare the numbering and formatting, I moved it from my Notion page.



1. The world state (state), is a mapping between addresses (160-bit identifiers) and account
states. Though not stored on the blockchain, it is assumed that
the implementation will maintain this mapping in a modified Merkle Patricia tree (trie). The trie
requires a simple database backend that maintains a mapping of byte arrays to byte arrays; we name this underlying database the state database. As a whole, the state is the sum total of database relationships in the state database.
2. The account state comprises the following four fields:
nonce: the number of transactions sent from this address or, in the case of accounts with associated code, the number of contract-creations made by this account.
balance  : the number of Wei owned by this address.
storageRoot: A 256-bit hash of the root node of a Merkle Patricia tree that encodes the storage contents of the account (a mapping between 256-bit integer values), encoded into the trie as a mapping from the Keccak 256-bit hash of the 256-bit integer keys to the RLP-encoded 256-bit integer values.
codeHash: The hash of the EVM code of this account—this is the code that gets executed should this address receive a message call. All such code fragments are contained in the state database under their corresponding hashes for later retrieval. If the codeHash field is the Keccak-256 hash of the empty string, i.e. σ[a]c = KEC(), then the node represents a simple account
3. An account is empty when it has no code, zero nonce and zero balance. Even callable precompiled contracts can have an empty account state. This is because their account states do not usually contain the code describing its behavior
4. A transaction (formally, T) is a single cryptographically-signed instruction constructed by an actor externally to the scope of Ethereum. The sender of a transaction cannot be a contract. While it is assumed that the ultimate external actor will be human in nature, software tools will be used in its construction and dissemination
5. EIP-2718 introduced the notion of different txn types. As of the Berlin version of
the protocol, there are two transaction types: 0 (legacy) and 1 (EIP-2930). Further, there are two subtypes of transactions: those which result in message calls and those which result in the creation of new accounts with associated code (known informally as contract creation)
6. All transaction types specify a number of common fields:
type: EIP-2718 transaction type; formally Tx.
nonce: Tn.
gasPrice: A value equal to the number of Wei to be paid per unit of gas for all computation costs incurred as a result of the execution of this transaction; formally Tp.
gasLimit: A value equal to the maximum amount of gas that should be used in executing this transaction. This is paid up-front, before any computation is done and may not be increased
later; formally Tg.
to: The 160-bit address of the message call’s recipient or, for a contract creation transaction, ∅ 
value: A value equal to the number of Wei to be transferred to the message call’s recipient or,
in the case of contract creation, as an endowment to the newly created account; 
r, s: Values corresponding to the signature of the transaction and used to determine the sender of the transaction
7. EIP-2930 (type 1) transactions also have:
accessList: List of access entries to warm up; formally TA. Each access list entry E is a tuple
of an account address and a list of storage keys: E ≡ (Ea, Es).
chainId: Chain ID; Must be equal to the network chain ID β.
yParity: Signature Y parity
8. Additionally, a contract creation transaction (regardless whether legacy or EIP-2930) contains:
init: An unlimited size byte array specifying the EVM-code for the account initialization procedure, In contrast, a message call transaction contains:
data: An unlimited size byte array specifying the input data of the message call,
9. The block in Ethereum is the collection of relevant pieces of information (known as the block
header ), together with information corresponding to the comprised transactions, T, and a set of other block headers U(ommers)
10. The block header contains ::

parentHash: The Keccak 256-bit hash of the parent
block’s header, in its entirety
ommersHash: The Keccak 256-bit hash of the ommers list portion of this block

beneficiary: The 160-bit address to which all fees collected from the successful mining of this block
be transferred

stateRoot: The Keccak 256-bit hash of the root node of the state trie, after all transactions are
executed and finalisations applied

transactionsRoot: The Keccak 256-bit hash of the root node of the trie structure populated with each transaction in the transactions list portion of the block

receiptsRoot: The Keccak 256-bit hash of the root node of the trie structure populated with the receipts of each transaction in the transactions list portion of the block

logsBloom: The Bloom filter composed from indexable information (logger address and log topics)
contained in each log entry from the receipt of each transaction in the transactions list

difficulty: A value corresponding to the difficulty level of this block. This can be calculated
from the previous block’s difficulty level and the timestamp

number: A value equal to the number of ancestor blocks. The genesis block has a number of zero

gasLimit: A value equal to the current limit of gas expenditure per block

gasUsed: A value equal to the total gas used in transactions in this block

timestamp: A value equal to the reasonable output of Unix’s time() at this block’s inception;

extraData: An arbitrary byte array containing data relevant to this block. This must be 32 bytes or
fewer

mixHash: A 256-bit hash which, combined with the nonce, proves that a sufficient amount of computation has been carried out on this block

nonce: A 64-bit value which, combined with the mixhash, proves that a sufficient amount of computation has been carried out on this block;

1. The other two components in the block are simply a list of ommer block headers (of the same format as above) and a series of the transactions
2. The transaction receipt has encoded information about its execution.  The transaction receipt, R, is a tuple of five items comprising: the type of the transaction, the status code of the transaction, the cumulative gas used in the block containing the transaction receipt as of immediately after the transaction has happened, the set of logs created through execution of the transaction and the Bloom filter composed from information in those logs. The logs bloom is a hash of size 256 bytes
3. We can assert a block’s validity if and only if it satisfies several conditions: it must be internally consistent with the ommer and transaction block hashes and the given transactions, when executed in order on the base state σ (derived from the final state of the parent block)
4. The mechanism enforces a homeostasis in terms of the time between blocks; a smaller period between the last two blocks results in an increase in the difficulty level and thus additional computation required, lengthening the likely next period. Conversely, if the period is too large, the difficulty, and expected time to the next block, is reduced
5. Every transaction has a specific amount of gas associated with it: gasLimit. This is the amount of gas which is implicitly purchased from the sender’s account balance. The purchase happens at the according gasPrice, also specified in the transaction. The transaction is considered invalid if the account balance cannot support such a purchase. It is named gasLimit since any unused gas at the end of the transaction is refunded (at the same rate of purchase) to the sender’s account
6. Gas does not exist outside of the execution of a transaction. Thus for accounts with trusted code associated, a relatively high gas limit may be set and left alone
7. Miners, in general, will choose to advertise the minimum gas price for which they will execute transactions and transactors will be free to canvas these prices in determining what gas price to
offer (for a timely and likely inclusion)
8. any transactions executed must first pass the initial tests of intrinsic validity. These include:
(1) The transaction is well-formed RLP, with no additional trailing bytes;
(2) the transaction signature is valid;
(3) the transaction nonce is valid (equivalent to the sender account’s current nonce);
(4) the sender account has no contract code deployed (see EIP-3607 by Feist)
(5) the gas limit is no smaller than the intrinsic gas, g0, used by the transaction; and
(6) the sender account balance contains at least the cost, v0, required in up-front payment.
9. Throughout transaction execution, we accrue certain information that is acted upon immediately following the transaction. We call this the accrued transaction substate, or accrued substate. The execution of a valid transaction begins with an irrevocable change made to the state: the nonce of the account of the sender is incremented by one and the balance is reduced by part of the up-front cost
10. There are a number of intrinsic parameters used when creating an account: sender, original transactor, available gas, gas price, endowment(wei sent) together with an arbitrary length byte array, i, the initialization EVM code, the present depth of the message-call/contract creation stack, the salt for new account’s address (ζ) and finally the permission to make modifications to the state. The salt ζ might be missing (ζ = ∅), but not if create2 is used
11. The address of the new account is defined as being the rightmost 160 bits of the Keccak-256 hash of the RLP encoding of the structure containing only the sender and the account nonce. For CREATE2 the rule is different and is described in EIP-1014. The account’s nonce is initially defined as one(for a contract that gets created), the balance as the value passed, the storage as empty and the code hash as the Keccak 256-bit hash of the empty string; WTF howcan storage and code be empty for a new contract
12. During contract creation, the value of the transaction is not transferred to the created account, if an Out-of-gas exception occurs and the contract code is not stored.  while the initialisation code is executing, the newly created address exists but with no intrinsic body code (so extcodesize is not safe when contract is being constructed).
13. If the initialization execution ends with a SELFDESTRUCT instruction, the matter is moot since the account will be deleted before the transaction is completed. For a normal STOP code, or if the code returned is otherwise empty, then the state is left with a zombie account, and any remaining balance will be locked into the account forever. WTF why in STOP opcode ?
14. A message call has all parameters from point 20 plus a recipient and the account whose code is to be executed. Aside from evaluating to a new state and accrued transaction substate, message calls also have an extra component—the output data denoted by a byte array(this is return data from a call). This is ignored when executing transactions, however message calls can be initiated due to VM-code execution and in this case this information is used.
15. there are nine exceptions to the usage of the general execution framework Ξ for evaluation of the message call: these are so-called ‘precompiled’ contracts, meant as a preliminary piece of architecture that may later become native extensions. The contracts in addresses 1 to 9 execute the elliptic curve public key recovery function, the SHA2 256-bit hash scheme, the RIPEMD 160-bit hash scheme, the identity function, arbitrary precision modular exponentiation, elliptic curve addition, elliptic curve scalar multiplication, an elliptic curve pairing check, and the BLAKE2 compression function F respectively
16. The EVM is a simple stack-based architecture. The word size of the machine (and thus size of stack items) is 256-bit. This was chosen to facilitate the Keccak256 hash scheme and elliptic-curve computations. 
17. Gas fees are charged in 3 circumstances : computation, Secondly, gas may be deducted in order to form the payment for a subordinate message call or contract creation; this forms part of the payment for CREATE, CREATE2, CALL and CALLCODE. Finally, gas may be paid due to an increase in the usage of the memory.
18. the execution fee for an operation that clears an entry in the storage is not only waived, a qualified refund is given; in fact, this refund is effectively paid up-front since the initial usage of a storage location costs substantially more than normal usage OMG Who gets this refund ? The caller of delete operation ? 
19. In addition to the system state σ, the remaining gas for computation g, and the accrued substate A(within the block till now), there are several pieces of important information used in the execution environment that the execution agent must provide : sender, originator, code account, gas price, value in wei, call stack depth, permission to make state modifications,+ block header of present block, input data byte array, machine code byte array
20. the execution environment specifies the output data of the instruction if and only if the present state is a normal halting state of the machine. the machine may reach an exceptional halting state in cases like stack overflow, OOG, invalid instruction, invalid jump destination, insufficient stack items or state modifications attempted within a static call. (Usually the compiler will not allow you to reach an exceptional halting state)
21. machine state : The machine state µ is defined as the tuple (g, pc, m, i, s) which are the gas available, the program counter pc ∈ N256 , the memory contents, the active number of words in memory (counting continuously from position 0), and the stack contents. (within a transaction/message call execution)
22. any position in the code occupied by a JUMPDEST instruction. All such positions must be on valid instruction boundaries, rather than sitting in the data portion of PUSH operations and must appear within the explicitly defined portion of the code (rather than in the implicitly defined STOP operations that trail it).
23. Normal halting state : data returning halting operations : return, revert, : and STOP
24. Execution cycle : Stack items are added or removed, the gas is reduced by the instruction gas cost, the program counter increments for each cycle. In general, we assume the memory, accrued substate and system state do not change (However, instructions do typically alter one or several components of these values) (this is machine state within a particular message call execution, accrued substate is between transactions and system state is after finalization of complete block.)
25. The canonical blockchain is a path from root to leaf through the entire block tree. In order to have consensus over which path it is, conceptually we identify the path that has had the most computation done upon it, or, the heaviest path. The longer the path, the greater the total mining effort that must have been done in order to arrive at the leaf . hence block number is considered. Since a block header includes the difficulty, the header alone is enough to validate the computation done. Any block contributes toward the total computation or total difficulty of a chain
26. The process of finalising a block involves four stages:
(1) Validate (or, if mining, determine) ommers;
(2) validate (or, if mining, determine) transactions;
(3) apply rewards;
(4) verify (or, if mining, compute a valid) state and block nonce.
27. The validation of ommer headers means nothing more than verifying that each ommer header is both a valid header and satisfies the relation of Nth-generation ommer to the present block where N ≤ 6. The maximum of ommer headers is two.
28. Transaction validation : The given gasUsed of block must correspond faithfully to the transactions listed: ie. the total gas used in the block must be equal to the accumulated gas used according to the final transaction(it is added to accrued substate after each txn)
29. The application of rewards to a block involves raising the balance of the accounts of
the beneficiary address of the block and each ommer by a certain amount. We raise the block’s beneficiary account by Rblock; for each ommer, we raise the block’s beneficiary by an additional 1/32 of the block reward and the beneficiary of the ommer gets rewarded depending on the block number. If there are collisions of the beneficiary addresses between ommers and the block (i.e. two ommers with the same beneficiary address or an ommer with the same beneficiary address as the present block), additions are applied cumulatively
30. 
![the application of rewards](https://github.com/chinmay-farkya/EVM-notes/assets/93861625/6e1cedc6-2b7a-4fe1-bd1f-c3e33443df93)


1. A block is constructed as follows : the next transaction is applied to the resultant state of the previous txn(or the block’s initial state in case of first txn). The block reward function is applied to the end state of final txn
2. Ethash is the PoW algorithm of Ethereum
3. Merkle-Patricia Trees are modified Merkle trees where nodes represent individual characters from hashes rather than each node representing an entire hash. Since state database is stored on node operator’s computers, the tree can be traversed speedily and without network delay
4. A block is made up of 17 different elements. The first 15 elements are part of what is called the block header.
5. Each transaction applies the execution changes to the machine state( OMG maybe accrued substate), a temporary state which consists of all the required changes in computation that must be made before a block is finalized and added to the world state.
6. The machine is actually an instantial runtime that executes several substates, as EVM computation instances, before adding the finished result, all calculations having been completed, to the final state via the finalization function.
7. The Ethereum Runtime Environment is the environment under which Autonomous Objects execute in the EVM: the EVM runs as a part of this environment
8. Ethereum clients are actually the implementations of ethereum protocol (like geth, parity etc.)
9. **the blockchain does not actually store Ethereum’s state**, e.g. what your account balance is. It is up to the computers that implement the Ethereum protocol to store the state as a Merkle Patricia tree. When we use metamask it indirectly routes our txn request to an actual physical ethereum node via the RPC URL.
10. blocks form a chain. That’s actually not entirely true—anyone has the opportunity to create a new block on some older pre-existing block, which means a tree of blocks get formed. Since there’s a tree of blocks, we need some way to obtain consensus about which path of the tree is the “right” one
11. In order to be more space efficient, nodes don’t store a separate state tree for each block. Instead, for each block, nodes store only the updated values, and refer to the previous immutable trees for the rest of the state.
12. The root hash of the state tree is stored on-chain in a block header. The state tree maps addresses to accounts. Each account has a storageRoot value, which refers to yet another Merkle Patricia tree. Each ethereum node is responsible for storing 3 different merkle patricia trees : state tree , transactions tree, receipt tree (each is a key value architecture) For txn and receipt trees, keys are txn indices while values are transactions and receipts respectively. 
13. Old block strcuture (PoW)
![Old block structure](https://github.com/chinmay-farkya/EVM-notes/assets/93861625/63c217d3-4f74-492c-a0ab-7061dbd50913)

For nodes/validators other than the miner :
![For nodes validators](https://github.com/chinmay-farkya/EVM-notes/assets/93861625/66f209be-ac03-4df2-860c-45a6c381584a)

1. In reality, the block is sent to validators after being constructed by the miner, and its properties get validated at execution environment of the validator. Recall that each block contains **stateRoot**, the root hash of the world state Merkle Patricia tree. In order to determine if that hash is valid, you yourself will execute all the transactions in the block, setting the initial state to the state at the end of the previous block. By doing this, you’ll end up with a new Merkle Patricia tree, and you can compare the root hash of that tree with stateRoot. If they are the same, then stateRoot is valid. Something similar must also be done for the other Merkle Patricia trees, represented by **transactionsRoot** and **receiptsRoot**
2. Transaction execution steps :
- Check if the transaction is valid. If not, return an error. An example of an invalid transaction is a transaction with an invalid signature.
- Subtract gas fees from the sender’s account, and increment their nonce. If there’s not enough gas, return an error.
- Transfer the transaction’s value from the sender’s account to the receiver’s account.
- If receiving account is a contract, run the code.
- If the value transfer failed because the sender didn’t have enough money, or code execution ran out of gas, revert all state changes except the payment of gas fees to the miner’s account.
- Otherwise, if successful, refund any remaining gas to sender, and send gas fees to the miner.
1. There are 3 merkle patricia trees(also called trie individually) : transactions, receipts and state trie. These are stored in leveldb database etc. on geth client on nodes and we can lookup information based on path stored in key and then operate on it. Only the updated values are changed for a new block and for the rest the previous block’s state trie is referenced
2. There is one global state trie, and it is updated every time a client processes a block. In it, a `path`is always: `keccak256(ethereumAddress)`and a `value`is always: `rlp(ethereumAccount)`. More specifically an ethereum `account`is a 4 item array of `[nonce,balance,storageRoot,codeHash]`. At this point, it's worth noting that this `storageRoot` is the root of another patricia trie. storageRoot is where all contract data lives (storage slots ⇒ values). The storage position in the trie can be derived by hashing from the contract address + integer position of actual storage(slot)
3. There is a separate transactions trie for every block, again storing `(key, value)`
 pairs. A path here is: `rlp(transactionIndex)`. Every block has its own Receipts trie. A `path`
 here is: `rlp(transactionIndex)` .`transactionIndex`is its index within the block it's mined. The receipts trie is never updated
4. Information from merkle trees can be verified (use case of merkle proofs) by providing a merkle branch that takes you to the given information ) and verifying that it hashes back to the actual block root hash : https://blog.ethereum.org/2015/11/15/merkling-in-ethereum
5. A blockchain has three layers - Networking, consensus and application

## PoS

1. While Proof-of-Work and Proof-of-Stake are often called "consensus algorithms," this is technically inaccurate. They are actually **Sybil resistance mechanisms**, which are an important part of consensus algorithms but not the whole thing. Another very important part is the fork-choice rule , and for PoS this algorithm (block rejection and acceptance rules, + validator rewards and punishments) is called gasper.
2. Proof-of-Work uses compute power as a Sybil resistance mechanism, while proof of stake uses money as a backing. of votes. 
3. Bitcoin uses the "**longest chain**" rule, which means that whichever blockchain is the longest will be the one the rest of the nodes accept as the valid chain. The longest chain is determined by the cumulative Proof-of-Work difficulty on that chain.
4. With the transition to Proof-of-Stake, Ethereum now looks at the “**weight**” of the chain, which is determined by the accumulated sum of validator votes (attestations)
5. With PoW, penalty is money lost in computing hashes that are found to be incorrect, in PoS validators "vote" for what they believe the next valid block will be. if a validator is wrong, there need to be explicit penalties
6. A validator needs to run two separate pieces of software : execution client and consensus client
7. There is an activation queue for becoming a validator, this rate limiting is done to limit number of validators joining the network at a given time to reduce communication overhead of the network(to ensure votes are propagated on time) and prevent malicious validators from entering and leaving the network when they feel like(and escape the penalties)
8. For Gasper, the algorithm divides time into "**slots**" and "**epochs**." **Slots are 12 seconds long**
, and **each epoch consists of 32 slots** (thus 6.4 minutes)
9. For every slot, one validator is randomly selected to be a "**block proposer.**" The block proposer validator is responsible for constructing a new block out of pending transactions. It then sends the block out to other validators in the network, which verify the txns and vote on whether that block is valid.
10. Not every validator votes for every block, though. Instead, for each epoch, a validator is assigned to a "**committee**." Each committee is randomly assigned one slot where they need to attest to determine whether the proposed block is valid. Committees are spread among the 32 slots in an epoch, which means 1/32 of the total validator set attests to the validity of each slot or block, with the maximum allowed validators per committee being 2048. In other words, **each validator only attests one block per epoch for the slot their committee was assigned to**.
11. ‍*Note that a small set of validators are also randomly chosen to join sync committees, which are different from the committees mentioned above. Being in a sync committee requires validators to help light clients sync up and determine the head of the chain, which they earn additional rewards for. However, as a validator, you are only part of a sync committee once every ~22 months, so it’s not a responsibility carried on a daily basis.*
12. Validators don't only attest to the current head block. They also attest to two others called "**checkpoint**" blocks. Every epoch has one checkpoint block that identifies the latest block at the start of that epoch. so validators attest to their view of source and target checkpoint blocks.
13. 
![instead of having every validator](https://github.com/chinmay-farkya/EVM-notes/assets/93861625/0ac96793-590d-46ea-a1c3-42d02f0eca69)


1. instead of having every validator listen to every other validator (his attestation broadcasted) , attestations from individual validators are aggregated within "**subnets**" before they’re broadcasted.
2. Each of those committees is further divided into **64 subnets**. This lets aggregation happen. In each epoch, a validator in each subnet is selected to be the aggregator. The aggregator collects all the attestations that have equivalent data to its own. The aggregator then broadcasts the aggregate attestations to the broader network.
3. If a block reaches "finality," it cannot be reverted unless there has been a critical consensus failure. (6 block confirmations for bitcoin, negligible possibility of reorg)
4. In Gasper,

- If ⅔ of the total staked ether votes in favor of that block, then it becomes "**justified**." Justified blocks are unlikely to be reverted, but they can be under certain conditions.
- When another block is justified on top of a justified block, the latter gets upgraded to "**finalized**." Finalizing a block is a commitment to include the block in the canonical chain. It can't be reverted unless an attacker destroys millions of ether.

Blocks do not reach the "justified" and "finalized" states in every slot. Instead, they happen to the first block in every epoch, which is called a "checkpoint" block. When a checkpoint block gets upgraded to justified, it must have a link to the previous checkpoint. As a result, the previous checkpoint block becomes finalized, and the more recent checkpoint block is justified.

1. There are certain checkpoints where every node in every network agrees that a block (and each that came before it) belongs in the canonical chain. This is called a "**weak subjectivity checkpoint.**" If a node sees a block that conflicts with a weak subjectivity checkpoint, then it rejects that block. New nodes that are trying to sync to the canonical head of the Beacon chain must use the latest weak subjectivity checkpoint as the starting point of their sync.
2. the only way the network can be subverted is if a majority of the nodes waste millions of ether trying to perform malicious activity.
3. Mining is just finding a random nonce number and hashing it with block data to find the correct hash. Typically, miners use a trial-and-error approach to estimate the nonce and start with fresh blockchain nonce values with each calculation. Miners must determine the correct nonce and add it to the current header’s hash before rehashing the value and comparing it to the target hash (with 30 leading zeros for example)
4. The Beacon chain maintains two separate records of each validator's balance:
- **Actual balance**: This is the actual balance of each validator. This includes the deposit they made to the deposit contract, plus rewards they earned for participating in the protocol, minus penalties for inactivity. The actual balance starts at 32 ETH, but it changes frequently based on rewards and penalties incurred by the validator.
- **Effective balance:** This is a number derived from the actual balance, but it's designed to only change once per epoch and is denominated in whole ether (no decimals), which makes it more efficient while calculating rewards within an epoch. It's also capped at 32 ETH, so a validator's actual balance could be 100 ETH, but his rewards and penalties are a function of his effective balance, which is capped at 32 ETH.
5. Effective balance is also used to determine the probability of being selected to propose blocks or participate in sync committees. The higher your effective balance, the more likely you are to be selected to propose blocks or participate in sync committees
6. An attestation actually has 3 votes :
![an attestation actually has](https://github.com/chinmay-farkya/EVM-notes/assets/93861625/6b749fcf-cb20-4986-a7bb-473040c3d80b)


1. the base reward is proportional to the validator's effective balance and inversely proportional to the number of validators on the network. In other words, as more validators enter the network, more rewards are issued at lower values per validator. Moreover, validators with an effective balance below 32 ETH (due to penalties for going offline or being slashed for malicious behavior) will have their rewards scaled downward versus validators with a max effective balance of 32 ETH.
2. The final reward is calculated by multiplying the base reward by the sum of the weights applicable to that validator(from the table above) and then dividing by 64. Validators only receive rewards for "correct" attestations — meaning their votes agree with the current block proposer and the reward gets divided by inclusion delay (time validator takes to attest after block is proposed)
3. For example, if the validator makes the attestation within one slot of the block being proposed, the attestor receives base reward * 1/1. If the attestation arrives in the next slot, the attestor receives base reward * ½, and so on.
4. It's important to note that attestation rewards are scaled in proportion to participation. So, for each source, target, or head vote, the validator's reward is scaled by the proportion of the total stake that made the same vote. This is done to incentivize one validator to help other validators by forwarding gossip messages and aggregating votes. A block proposer gets all other rewards (vote and sync etc.) for “including” others’ votes. That means as more validators attest to the block, more rewards are given to the block proposer. Except the reward for including a vote is a fraction of the reward for making the vote
5. Once every 256 epochs (27.3 hours), 512 validators are selected to participate in a sync committee and selected validators get rewards for it

### Penalties :

1. Validators are penalized for missing, late, or incorrect attestations. The math is easy here; their penalty is the same as the reward would have been if they participated.
2. What happens if most validators dont participate in making attestations ? If the Beacon chain has gone more than four epochs without finalizing, then it enters an "**inactivity leak**" mode. Recall that we need ⅔ of validators to finalize blocks, so if too many validators are absent, then we can't finalize blocks.
3. By removing all rewards for active attestors and penalizing inactive validators. The penalties for being inactive increase quadratically over time. The idea is that this would gradually reduce the stake that inactive validators have in the network until they control less than ⅓ of the total stake. This allows remaining active validators to finalize the chain. The ultimate aim is to recover finality.
4. Proposer penalties : no penalties. If a block proposer fails to propose a block, we just don't have a block in that slot, and we move on to the next slot.
5. Sync committee validators are rewarded in each slot they perform their duty. If they sign the wrong head or don't participate, they're penalized the same amount as they would have been rewarded if they were active and correct.
6. all penalties are subtracted from a validator's balances(actual balance) on the Beacon chain and effectively burned, reducing the net issuance of ether. Secondly, penalties don't scale with participation as rewards do. OMG How does effective balance change ?
7. If a validator engages in malicious behavior, they're heavily penalized through slashing — that is, by forcing them to leave the network and dealing out a steep penalty. There are three ways a validator can be slashed:
- Being a proposer and signing two different beacon blocks for the same slot
- Being an attester and signing an attestation that "surrounds" another one
- Being an attester and signing two different attestations having the same target
1. To catch the malicious validator, the whistleblower would need to create a special message and propagate it to the network, and the proposer would need to include it in a block. If the accusation is correct, then the proposer and the whistleblower are entitled to a reward. If a block proposer includes evidence that results in a slashing, they will be rewarded with the slashed validator's effective balance divided by 512. The whistleblower rewards are not yet incorporated into the protocol, so they don't receive any reward.
2. As for the validator, once slashed, it gets an initial penalty, and then it's queued to exit the protocol in ~36 days (8192 epochs). The validator continues to receive attestation penalties until it's due to exit the protocol.(but they cant participate in attestations ⇒ very dangerous) and are included in inactivity leak penalties. Just like penalties, slashed amounts are effectively burned, reducing the overall net issuance of ether.
3. Anti-correlation penalties : if many validators perform the same offense at the same time, then they get penalized more for the same offense. There are two types of anti-correlation penalties:
- **Inactivity Leak**: If you fail to produce an attestation, you normally get a small penalty. But if you fail during an inactivity leak, the penalties become much larger(because of timing correlation with other inactive validators that lead to inactivity leak)
- **Slashing**: Halfway through their withdrawal period, a validator gets penalized a second time. This second penalty is based on the total amount of stake slashed during the 18 days before and after the validator was slashed. This scales the punishment such that if the validator was slashed for a one-off event, they would only be lightly punished, but if a lot of validators were slashed during that period, they all lose a lot of money.
1. This prevents validators from collectively trying to subvert the network, and to try to de-correlate from other failed validators. for example, not running the same client, not being part of the same staking pool, or not running on the same cloud service.
2. Validators earn rewards until they exit the protocol by either:
- Voluntarily choosing to exit
- Being forced to exit due to slashing
- Having a balance below 16 ETH due to penalties
1. validators can leave whenever they want. To voluntarily exit, all they have to do is sign a "VoluntaryExit" message included on-chain. The exit queue is processed similarly to the activation queue with a four-epoch delay. If they exit without being slashed, they can withdraw their ether balance after one day of exiting. But if they were slashed, they would need to wait four eeks (ie eek = 2048 epochs ie. 9.1 days) this is the same period as after being slashed first time.(8192 epochs, 36 days)
2. The Beacon chain was launched in Dec 2020, running alongside the mainchain the whole time as it produced and validated empty blocks. There were several upgrades before transitioning to PoS. For eg. difficulty bomb upgrades and others such as requiring node operators to update their client software and merging various test networks once they've been tested.
3. 20 shadow forks were also run. A shadow fork is a fork that uses historical blockchain data (unlike testnets which use test data) to simulate what the shift from Proof-of-Work to Proof-of-Stake would look like in a controlled environment. In other words, the goal was to run simulations and account for issues that might arise when the Ethereum mainchain merged with the Beacon chain.
4. the Merge was officially triggered when the chain reached the pre-specified terminal total difficulty (TTD). Once the chain reached the TTD, no additional proof-of-work blocks were added to the chain from that point on

What changed after the merge ?

- Rinkeby and ropsten testnets were replaced by Goerli and sepolia
- now node operators have to run 2 clients - execution and consensus The **Execution Layer Client**
 listens to new transactions and executes them, while the **Consensus Layer Client** implements the PoS algorithm (Gasper)
- blocks that were generated in the Proof-of-Work chain will no longer exist. Instead, the contents of those blocks(represented by executionpayloads in the block) become a component of blocks created on the Beacon chain.
- some block fields like ommerhash, ommers list, nonce and difficulty are irrelevant to the new chain, but were not removed instead set to zero values. mixhash is also one of these, but set to RANDAO value instead of zero (mixhash and difficulty are combinedly replaced by prevrandao)
- opcode changes : blockhash now has weaker randomness value, difficulty renamed to prevrandao (stronger source of randomness)

## EVM low-level

1. Compilation of a smart contract creates two things : Bytecode and ABI. Bytecode, in very simpler terms, is a collection (long sequence) of instructions or opcodes that defines how a smart contract should be executed by the EVM. The details provided by ABI are then used to encode contract calls that are made to the EVM so that the virtual machine can read, understand and execute these instructions. Opcodes are 1 byte in length leading to 256 different possible opcodes. The EVM only uses 140 unique opcodes currently. 
2. Creation code contains constructor logic, parameters, free memory pointers etc., plus the logic to generate and return the runtime code and store it on-chain
3. the common init code opcodes basically instruct the EVM to do 4 main tasks:
- Assigning a **Free-Memory Pointer**
- Validating **Non-Payable Constructor** Check
- Retrieve any constructor parameters from end of code
- Initializing state variables as per the **constructor**
- Returning and storing the **Runtime Bytecode**
1. Whenever there is a need to store any given data in memory, the EVM performs this action in two steps:
- ***Fetch the Free Memory*** (*using the free memory pointer*)In order to store the data, the EVM fetches the location of the free memory first. It's easy to do as the EVM knows that the position for free memory is stored at location 0x40 (*the free memory pointer*).
- ***Update the Free Memory Pointer to a new position.*** However, after it uses the free memory, it very cleverly *updates the free memory pointer to the next position in memory that is free and ready to use.*
1. After a jump, the execution reaches jumpdest and all instructions in between are skipped.
- **JUMPDEST**: This opcode simply represents a valid location to *jump to.* This means, although JUMP and JUMPI opcodes can shift the execution flow to any location, the target location for both of them should always contain JUMPDEST. Otherwise, it won't be considered a valid jump, and execution shall revert. OMG revert or exceptional halt due to invalid jump destination?
- **JUMP:** This opcode simply takes the topmost value from the stack and moves the execution to that particular location.(location is defined by program counter)
- **JUMPI:** This is exactly similar to JUMP, however, this is more of a conditional jump. It only jumps when a condition is met.

**JUMPI** only jumps:

a. If the 2nd Position of the Stack is a **NON-ZERO** value, then **JUMP**

b. If the 2nd Position of the STACK is **ZERO** value, then **DO NOT JUMP**

1. The opcodes are an exact simulation of what we code in Solidity. For example, payable checks are present for a constructor only when constructor is not marked payable.
2. The bytecode after verifying a contract on etherscan is actually the complete creation code (init + runtime) plus the constructor arguments appended at the end, while for unverified contracts it is just the runtime code displayed.
3. The state trie contains a key and value pair for every account which exists on the Ethereum network. The “key” is a single 160 bit identifier (the address of an Ethereum account). The “value” in the global state trie is created by encoding the following account details of an Ethereum account (using the Recursive-Length Prefix encoding (RLP) method):- nonce- balance- storageRoot- codeHash
4. The state trie’s root node’s hash ( a hash of the entire state trie at a given point in time) is used as a secure and unique identifier for the state trie; the state trie’s root node (stateRoot in Block header) is cryptographically dependent on all internal state trie data. storageRoot is a 256-bit hash of the trie that stores inside the account state data. So, storageRoot is account state and stateRoot is world state
5. Transaction trie :: 
![Transaction trie](https://github.com/chinmay-farkya/EVM-notes/assets/93861625/83f15e98-9c0f-4f3d-8cf2-ca4c0e3f9fb2)


1. *accounts in Ethereum are only added to the state trie once a transaction has taken place associated to that account (involving gas and included in a mined block). So the state only has accounts that have been initialized (nonce/code)*
2. In Yul, any opcode written has supplied parameters in the order of left to right == top of the stack to second top of stack and so on . for ex. mstore(0x40, 0x80) *consumes elements from the stack from left to right, always consuming what’s on the top of the stack first*
3. Push instructions are the only instructions that are two or more bytes. So when you write PUSH1 0x80 then push is first instruction and 0x80 is second instruction (these are skipped by remix in the debug display). Program counter and bytecode size are accordingly defined. 
4. The constructor arguments need to be retrieved from code appended at the end of a contract bytecode. This is done in the init part of creation/deployment code. Actually, when deploying a contract whose constructor contains parameters, the arguments are appended to the end of the code as raw hex data. OMG do they have padding?
5. Study and analyze any contract’s bytecode : For debugging, first decompile the bytecode into opcodes and then follow the execution of those opcodes and track associated changes to the stack, memory, storage etc, using Remix. If we call a contract’s function in remix and debug it, it is similar to reversing the calldata of the execution plus the consequences of the calldata
6. before a function’s body is executed, the function’s parameters are loaded into the stack (whenever possible)(by the function wrapper), so that the upcoming code can consume them. All functions written in Solidity update the free memory pointer after every use of memory during execution. Every single function in Solidity, when compiled to EVM bytecode, will initialize this pointer. This keeps a record of the offset, to prevent overwriting any used memory location
7. The structure that fetches parameters embedded at the end of the bytecode, as well as the structures that copy the runtime code and return it, can be considered boilerplate code and generic EVM opcode structures. JUMP and return opcodes are imp to identify standalone structures in evm bytecode and then split the entire code into chunks
8. how Solidity compiler’s generated EVM bytecode routes incoming transactions : Using calldata to derive selector and matching it against available external/public functions via the dispatcher (some sort of switch statement that simply routes execution to the correct part of the code)
9. the calldata is an encoded chunk of hexadecimal numbers that contains information about what function of the contract we want to call, and it’s arguments or data. Simply put, it consists of a “function id”, which is generated by hashing the function’s signature (truncated to the first leading four bytes) followed by the abi-encoded arguments data
10. When calling a function evm does the following things :
- Initialize the free memory pointer
- short calldata check ie. (if less than 4 bytes calldata, ( if yes, go to fallback if exists) otherwise revert(0,0)
- matching function selector of calldata with actual selectors of all external/public functions one by one ⇒ if it matches any case, take to that jumpdest ie. the ABI wrapper of the matched function. this wrapper will be in charge of un-packaging the transaction’s data for the function’s body to consume, as well as preparing the output data for returning to the caller
- otherwise reach at fallback(if it exists)or revert

1. DUP is mostly used because the next operation will consume values from the stack and we want to keep a value around on the stack for some further operations (for ex. the function selector needs to be extracted from calldata through a long list of opcodes so we want to preserve the result if further operations need it)
2. calldataload only loads 32 bytes starting from offset. We use offset zero, and load first 32 bytes on the stack to find the selector . The calldata total may actually be > 32 bytes but it doesn’t matter because we just want to extract the first 4 bytes to determine the selector(when capturing the selector and masking it with 0x00000000ffffffff)
3. The function dispatcher(in runtime code) sits at the gate of every contract out there (at least all those compiled from Solidity) and redirects execution to the appropriate location in the code.(function wrappers) It’s the way in which Solidity gives a contract’s bytecode the ability to emulate multiple entry points, and hence, an interface
4. Function wrapper code exists after the switch case statements for function selector is over. It jumps into and out of the function body from here. (ie. we have jump and jumpdest opcodes together in close lines). This is how the code reaches the body and then leaves it to come back to the wrapper. after it comes back, it copies any return arguments from the stack to memory and then RETURN value from memory
5. Every function has its own wrapper, and all wrappers are clubbed together in the code before all function bodies that are clubbed together. The function dispatcher doesn’t direct to the function body but to the wrapper. The wrapper has non-payable checks and argument checking etc. and then redirects to actual body
6. If we have enabled the optimizer, if there is a reusable chunk of code in some function’s wrapper, it could be called into from some other function body if it needs that ( for ex. code that grabs a `uint256` value from the stack and returns a `uint256` via memory). The Solidity compiler could be noticing that part of the code generated for these two wrappers is the same, and deciding to reuse the code to save on gas
7. There is an additional chunk - calldata unwrapper in the function wrapper if there are function parameters. In this the calldata is loaded starting from after 4 bytes and masked with 0xff(AND operation) of the datatype size(ex. 0xffff… 40 fs ie. 20 bytes for an address) this is called type checking
8. A function’s wrapper is an intermediary that unpacks the calldata for a function’s body to use, routes execution to it, and then repacks whatever comes back for the user. This wrapper structure is there for all functions that are part of the public(external) interface of a contract in Solidity. IDK and how are internal /private functions integrated into this ? Through jump statements from function body of actual function call.
9. Before the time a function body is executed, the function’s arguments should be sitting comfortably in the stack (or in memory if the data is dynamic), anxious to be used/ consumed.
10. Inside the function body, the incoming parameters are cast to their correct type (ex. address into 20 bytes from 32 byte stack words) using 0xfff… 40 fs mask(AND operation). In any of these masking operations, the trailing bits of the mask are 0s so that AND operation yields 0 and only the required 20 initial bytes(as an example for an address) remain true to their value.
11. To calculate mapping storage slot, sha3 opcode is used to hash in memory. So the mapping’s storage slot is placed in memory starting at 0x20 and the mapping key starting at 0x00. SHA3 takes two parameters - starting position and length of bytes to hash
12. In the runtime bytecode, metadata hash (hashing of the metadata file) is appended at the end after function bodies( constructor arguments are appended at the end of creation bytecode, so the order is init code ⇒ runtime code ⇒ metadata hash ⇒ constructor arguments if any). This is basically non-executable code. The compiler is hashing the contract’s metadata (which includes information about the contract such as its source code, compiler settings, optimizer, devdoc, userdoc, ABI, evmversion etc.) and injecting this hash into the contract’s own bytecode!. The encoding of this metadata hash is interpreted as opcodes which is wrong, instead its a byte sequence. This is the byte pattern :

`0xa1 0x65 'b' 'z' 'z' 'r' '0' 0x58 0x20 <32 bytes swarm hash> 0x00 0x29`

So in order to retrieve the data, the end of the deployed bytecode can be checked to match that pattern and use the Swarm hash to retrieve the file.

1. This hash can be used in [Swarm](https://swarm-guide.readthedocs.io/en/latest/introduction.html) as a lookup URL to find the contract’s metadata. Swarm is basically a decentralized storage system, similar to [IPFS](https://ipfs.io/). The idea is that we can query swarm and find the config info like the version of compiler used and the source code. the metadata hash can be used by wallet applications to fetch the contract’s metadata, extract it’s source, recompile and verify that the produced bytecode matches the contract’s bytecode, then fetch the contract’s JSON ABI and look at the [NATSPEC](https://github.com/ethereum/wiki/wiki/Ethereum-Natural-Specification-Format) documentation of the function being called.
2. This is CBOR encoding where the decentralized storage system’s name and version is also stored to help retrieval (for ex. swarm version zero bzzr0 or ipfsr0 )
3. We can check the pattern using a utf8 to hex converter to find the string characters of pattern. and setup a local swarm node to lookup the hash
4. We get an INVALID opcode by the bytecode interpreters when a hex byte isn’t identified as a valid opcode. ignore the bytecode when I see a `LOG1`, followed by a `PUSH6`and a few `INVALIDs`, understanding that what I am seeing is the metadata hash injection that the Solidity compiler makes.
5. Remember memory is a byte array meaning we can start our reads (and our writes) from any memory location. We are not constrained to multiples of 32. Memory is linear and can be addressed at the byte level.
6. Remember elements in memory arrays in Solidity always occupy multiples of 32 bytes (this is even true for `bytes1[]`, but not for `bytes`and `string`). So when an array is defined, The compiler will have determined how much space is required through the array size and the default array element size. (32 * arrayElements bytes). This memory is allocated. The free memory pointer has to be updated at any memory operation. 
7. Now the array needs to be initialized, for example with zero value. Here, the calldatacopy takes full size of the array ie. 32 * arrayElements. Next if a specific index of this array has to be assigned, its index is pushed to stack and verify that it is less than length of array. If it isn’t the execution jumps to a different part of the bytecode which handles this error state. Now opcodes determine where in memory(the start location of an array index) the value needs to be written for it to correspond to the correct array index.
8. In storage, a fixed size array of size n takes up n storage slots, every element is stored in its own slot. (if its element type takes up 32 bytes or maybe > 16 bytes individually). How is dynamic array or fixed array with type < 32 bytes stored in storage ?????
9. When sload is used for a slot that packs multiple variables within itself, the whole content will be loaded. Storing multiple variables in a single slot without overwriting existing data and retrieving a variable’s specific bytes from a 32-byte slot. ⇒ first value is stored on the lower bits end ie. right end, then second value is stored to the left of it, agnostic to internal alignment of bits of a value 
10. Remember that hexadecimal numbers are ultimately seen as binary numbers by the machine. This is important as a number of bitwise operations are used in slot packing : ADD, OR & NOT (opcodes with same name)
11. AND of two x bit numbers - arrange both horizontally and compare vertically - 1 if both are 1 else 0. OR - 0 if both bits are 0 else 1. NOT(takes one value, perform on each bit of a number) performs logical negation on each bit. Bits that are 0 become 1, and those that are 1 become 0. Perform the operation for all couples of bits and the final result is obtained. 
12. The SSTORE and SLOAD on packed slots is very complex and unexplanable.  : Allocate position of the variable to be stored using EXP opcode(1 byte offset raised to the starting byte of variable from the order of filling ie. right to left) and then multiply by the value to be stored, this gives slot content with only the new variable value inserted at the correct position (What about the other variables that were already stored ?) ⇒ multiply the exp result by 0xffffffff (because we are storing a 4-byte variable, count fs according to that) ⇒ this acts as a bit mask and we get 0xfff… on the variable’s full location but other bits are zeros(in the exp result) ⇒ Then performing a NOT on this gives 0xfff…00000000 …. ffff for all bits except the 4 bytes our variable is concerned with ⇒ then we separately load the slot and use AND operation with bit mask 0xffff …. 00000000 … fff(depends on bit mask obtained in previous step) ⇒ This gives value of the storage slot excluding the 4 bytes where we are going to store our variable.(if it was already written it would get overwritten because of these operations) 
13. The idea is to get the variable value at its position in a blank slot, plus the other existing contents of the actual slot we are concerned with, and then combine these two values using OR operation. This will store variable at the specified location while keeping all other existing contents of the slot unchanged. During this process, we first clear any written info at specified location, prepare the value to be stored separately and then combine it with rest of the existing slot. 
14. For reading of a value : take exp result after raising to starting position of bytes of variable (from right to left) : Suppose we have a 8 byte variable at the lower bits and before it our variable v resides, so we multiply exp by 0x08 which is actually ending position of v so to say but the starting position when we look from right to left ⇒ load the slot and divide its contents by the exp result (this is basically right shift and the last 8 bytes(for ex.) now vanish and new 16 zeros get introduced at the left end of storage slot content ⇒ now our desired value sits at the end of this result because last 8 bytes have been removed. Now we use a bit mask 0x0000 ….000ffffffffffffffff (fs according to 8 bytes for ex. depending on size of variable v to be read and not the starting position of v we earlier used) Now AND operation on the bitmask and the result of DIV in previous step removes all other preceding contents before our concerned variable and only our variable v’s value remains at the end. 
15. In the state root, *In actuality, the key is the hash of the Ethereum address and the value is the RLP encoded Ethereum account. In storageRoot of ethereum account too, Again in actuality, there is RLP encoding of the values & hashing of the keys that goes on as part of this process.(these hashes may represent merkle tree branch paths)*
16. *When the EVM executes a smart contract, a [context](https://www.evm.codes/about) is created for it. The context consists of : code, storage, memory, stack, calldata, return data. This context is preserved to the original EOA caller when a contract delegatecalls another contract. If we dont pass value while calling, the delegatecalled contract can still read (but can’t use eth if we have not explicitly sent value with this delegatecall) msg.value values that we sent with the original call. Perservation of context : msg variables, caller info, contract state variables etc. WTF delegatecall can’t pass eth ?*
17. Conceptually a “Delegate Call” effectively allows you to copy and paste a function from another contract into your contract. It will be run as if it were executed by your contract
18. Under the hood, solidity compiler maps declared state variables to storage slots and then EVM doesn’t see variables in bytecode as variables but slots. This is why delegatecall pattern requires same order of variable declaration in both contracts because it will manipulate the slot the called contract wants to according to its reference(ie. its own declaration order of variables) regardless of semantics or naming of variable. This will lead to accidental replacement of values. (For example if called contract’s code modifies uint value; variable which is declared at position 2 inside it, then it will modify whatever variable is defined at position 2 in the calling contract, regardless of whether it is uint value;)
19. For DELEGATECALL opcode we have the following input variables;
- `gas`: amount of gas to send to the sub [context](https://www.evm.codes/about) to execute. The gas that is not used by the sub context is returned to this one.
- `address`: the account which [context](https://www.evm.codes/about) to execute.
- `argsOffset`: byte offset in the [memory](https://www.evm.codes/about) in bytes, the [calldata](https://www.evm.codes/about) of the sub [context](https://www.evm.codes/about).
- `argsSize`: byte size to copy (size of the [calldata](https://www.evm.codes/about)).
- `retOffset`: byte offset in the [memory](https://www.evm.codes/about) in bytes, where to store the [return data](https://www.evm.codes/about) of the sub [context](https://www.evm.codes/about).
- `retSize`: byte size to copy (size of the [return data](https://www.evm.codes/about)).

CALL opcode has exactly the same input variables with one additional value.

- `value`: [value](https://www.evm.codes/#34) in [wei](https://www.investopedia.com/terms/w/wei.asp) to send to the account. (CALL only)

Delegate call doesn’t require a value input as it is inherited from its parent call. WTF but the value is not transfered

Both call and delegatecall have one output variable “success” which is 0 if the sub context reverted otherwise it returns 1.

> Delegatecall will return success ‘True’ if it is called on an address that is not a contract and so has no code. This can cause bugs if code expects delegatecall functions to return `False` when they can’t execute.
> 
The delegatecall picks up calldata (prepeared using abi.encodewithsignature etc. and to be used for the message call) from the memory and passes to the destination contract. The retSize parameter in it is zero when the called function isn’t returning anything. If it returns something retSize ≠ 0 and the returned data would be copied at a defined offset in memory

Transaction Receipts and Event logs :

1. EVM nodes are not required to keep logs forever and can remove old logs to save space. Contracts cannot access log storage so they are not required for nodes to execute a contract. Contract storage on the other hand is required for execution so cannot be removed.
2. The “Transaction Trie” is the data set that generates the transactionsRoot and records the transaction request vectors. Transaction request vectors are the pieces of information required to execute a transaction.

### Transaction Execution :

The fields included in a transaction can be seen below.

- Type = The transaction type (LegacyTxType, AccessListTxType, DynamicFeeTxType).
- ChainId = The EIP155 chain ID of the transaction.
- Data = The input data of the transaction
- AccessList = The access list of the transaction
- Gas = The gas limit of the transaction
- GasPrice = The gas price of the transaction
- GasTipCap = The gasTipCap per gas of the transaction
- GasFeeCap = The fee cap per gas of the transaction
- Value = The ether amount of the transaction
- Nonce = The sender account nonce of the transaction
- To = The recipient address of the transaction. For contract-creation transactions, To returns nil
- RawSignatureValues = The V, R, S signature values of the transaction

Only the sent txn data is stored in txn trie, actual execution info is stored in recipt trie (gas used, event logs, success of txn)

1. Fields of transaction receipt for a particular txn :
- Type = The transaction type (LegacyTxType, AccessListTxType, DynamicFeeTxType).
- PostState (root) = The StateRoot post-execution of the transaction. You may note it’s 0x in the query above this is likely due to [EIP-98](https://github.com/ethereum/EIPs/issues/98).
- CumulativeGasUsed = Sum of gasUsed by this transaction and all preceding transactions in the same block.
- Bloom (logsBloom) = Bloom filter for event logs
- Logs = Array of log objects
- TxHash = The transaction hash that the receipt is associated with
- ContractAddress = Address of the deployed contract if the transaction was a contract creation. 0x000…0 if the transaction isn’t a contract creation.
- GasUsed = Gas used by this transaction
- BlockHash = Hash of the block this transaction occurred in
- BlockNumber = Block number for the block this transaction occurred in
- TransactionIndex = Transactions index within the block. The index determines which transaction is executed first. The transaction that is at the top of the block has an index 0.
1. In event logs(array of log objects above ⇒ each object has many fields like contract address, topics, data, txhash, blockhash, blocknumber, logIndex, txn index, removed?), Topics are indexed values. By default, topic1 is hash of event signature. The non-indexed parameters of the event are just stored in the data field. We can have max of 4 topics ie. max of 3 indexed parameters. Each topic is 32 bytes in size. If the type of an indexed parameter is larger than 32 bytes (i.e. string and bytes), the actual data isn’t stored, but rather the keccak256 digest of the data is stored. The non-indexed parameters are added to the data field with proper padding acc to its type. and sequential to the parameters order.
2. There is one case when the first topic in event logs isn’t the hashed event signature. This is when we declare an anonymous Event. This opens up the possibility of having 4 indexed parameters rather than 3 but we lose the ability to index on the Event name. One other advantage of anonymous events is that they can be cheaper to deploy since they don’t force you to use 1 additional topic. 
3. The address field is the address of the contract that emitted the event. One important note on this field is that it will also be indexed despite it not being included in the topics section. This is to facilitate filtering of data
4. the LOG opcodes of which there are 5. They go from LOG0 for when no topics are included to LOG4 when 4 topics are included (including the topic1 default hash of event signature). It takes in
- offset = memory offset, which represents the start location of the data field input
- length = length of the data to read in from memory
- topic1 = value for topic1
- topic2 = value for topic2
- topic3 = value for topic3

Offset and length define where in memory the data is located for the data section.

1. The secret to how indexed items enable faster lookup is Bloom filters.

> A Bloom filter is a data structure designed to tell you, rapidly and memory-efficiently, whether an element is present in a set.
> 
> 
> The price paid for this efficiency is that a Bloom filter is a **probabilistic data structure**: it tells us that the element either *definitely is not* in the set or *may be* in the set.
> 
> The base data structure of a Bloom filter is a **Bit Vector**.
> 
1. Machine state vs World state
![Machine state](https://github.com/chinmay-farkya/EVM-notes/assets/93861625/901ac1c1-ed50-424d-8585-072b3178b2a6)

2. ![Machine state 2](https://github.com/chinmay-farkya/EVM-notes/assets/93861625/a5a0d4fe-ba83-4708-a4b5-6e4834bae071)

3. ![Machine state 3](https://github.com/chinmay-farkya/EVM-notes/assets/93861625/037a0091-2037-481f-8404-3176c4ce44b7)


for BYTE instruction, the order is opposite ie. left to right so byte(0) references Most significant bit while for push instructions its always starts from right to left

1. Wallets fetch data using network requests (in this case, via HTTP) to hit JSON-RPC endpoints exposed by nodes. For example, reading an account’s nonce. Then the wallet builds a txn filling in all txn fields using info retrieved from nodes. 
2. in some way or another most wallets are implementing ABI-encoding to interact with contracts. The wallet as an asbtraction takes care of everything. The ERC20 balances you see in the wallet are results from read requests made by the wallet to nodes to query the token balance of your address. 
3. a command-line utility called [cast](https://book.getfoundry.sh/reference/cast/cast-calldata.html), which is able to ABI-encode the call with the specific arguments. Then this sequence of bytes is included in the data field
4. Different wallets may take different approaches to deciding how much to pay for gas. Strategies to determine the right fees may involve querying gas-related information from nodes (such as the minimum base fee accepted by the network). Popular wallets also use more sophisticated off-chain services to fetch gas price estimations and suggest sensible values to their users.
5. The `eth_feeHistory` endpoint is exposed by some nodes to allow querying transaction fee data. If you're curious, read [here](https://docs.infura.io/infura/networks/ethereum/json-rpc-methods/eth_feehistory) or play with it [here](https://composer.alchemyapi.io/?share=eJwdyEEKgCAQBdC7.LULCyTwBK1atomIoSaKUkMnIqK7J_0e78G40OphtYJnuULcfjuWJUwNOYZF9jAz12uSEG8oHBTJtbSfnGA7FDrfTsJJMrrSKKNVZXr07wcDJR59), or see the spec [here](https://github.com/ethereum/execution-apis/blob/main/src/eth/fee_market.json#L14-L94). To determine the gas limit, there's a handy mechanism that wallets may use to simulate a transaction before it is really submitted. It allows them to closely estimate how much gas a transaction would consume, and therefore set a reasonable gas limit. Gas estimations can be done using a node's endpoint called `eth_estimateGas`
6. There are two other gas fields : maxFeePerGas(gasFeeCap) and maxPriorityFeePerGas(gasTipCap) based on EIP-1559 : https://www.blocknative.com/blog/eip-1559-fees. Remember that the `maxPriorityFeePerGas`
 and `maxFeePerGas` will ultimately depend on the network conditions at the moment of sending the transaction
7. the `accessList`
 field. Advanced use cases or edge scenarios may require the transaction to specify in advance the account addresses and contracts' storage slots to be accessed, thus making it somewhat cheaper. Right now its is set to empty list because it doesn’t save much gas, but may be relevant in future. https://eips.ethereum.org/EIPS/eip-2930
8. The importance of signing a txn : the txn requests made by you via wallet are routed to the nodes through rpc endpoints but how do the nodes know that you have sent the txn and not someone else ? Once a wallet has collected enough information to build the transaction, and you hit SEND, it will digitally sign your transaction using your account's private key (that your wallet has access to)
9. what's actually being signed is the `keccak256` hash of the concatenation between the transaction's type and the [RLP encoded](https://ethereum.org/en/developers/docs/data-structures-and-encoding/rlp/) content of the transaction.

`keccak256(0x02 || rlp([chainId, nonce, maxPriorityFeePerGas, maxFeePerGas, gasLimit, to, amount, data, accessList]))`

1. The process of signing the txn produces a signature : the v r and s values. These travel along the txn. The next step is *serializing* the signed transaction. That means encoding the pretty object above into a raw sequence of bytes, such that it can be sent to the Ethereum network and consumed by the receiving node. v, r, s are added to the above rlp list. everything other than keccak256 represents the encoding. 
2. Once built, signed and serialized, the transaction must be sent to an Ethereum node. There's a handy JSON-RPC endpoint that nodes may expose where they can receive such requests. It's called `eth_sendRawTransaction`. The result included in the response of this JSON HTTP request contains the transaction's hash: `bf77c4a9590389b0189494aeb2b2d68dc5926a5e20430fb5bc3c610b59db3fb5`. This 32-bytes-long sequence of hex characters is the unique identifier for the submitted transaction.
3. what happens when an Ethereum node receives the serialized signed transaction ? Dive into geth code using deliriusz.eth ‘s articles
4. The geth code first deserializes the txn to access txn fields easily. and then starts validating the txn ⇒ ensuring txn fee is not above hard cap of 1 ether ⇒ ensuring cross-chain replay protection using chainID ⇒ sending the txn to the txn mempool (this pool represents the set of transactions that the node is aware of at a specific moment, not included in the blockchain yet.) 
5. Before actually adding the txn to mempool, it checks if it has not added the txn already, then verifying ECDSA signature.
6. ⇒ tx gas limit not above block gas limit ⇒ txn size is not above max size ⇒ nonce being the expected one ⇒ sender having enough funds to cover the cost (value + gas fees). ⇒ logging stuff ⇒ returning txn hash. These validation rules can be custom txn prioritization rules based on the mempool management approach of the node operator. 
7. Metamask etc. are connected to the traditional nodes so the txn is bound to appear on public mempools. There's a handy endpoint some nodes expose, called `eth_newPendingTransactionFilter`. Perhaps a good-old friend of [frontrunning](https://www.paradigm.xyz/2020/08/ethereum-is-a-dark-forest) [bots](https://arxiv.org/pdf/1904.05234.pdf)
8. There are more specialized nodes that feature private mempools. These prevent public exposure of txns before they are included in a block : https://github.com/flashbots/mev-geth. such mechanisms usually consist of establishing private channels between transaction senders and block builders. Like the [Flashbots Protect service](https://docs.flashbots.net/flashbots-protect/rpc/quick-start)
9. The transaction you send through a node reaches other nodes that are block producers who verify it. This propogation and exchange of txns is handled by various p2p protocols. All nodes are listening and broadcasting txns along with their peers(max 50 by default for geth). To favor efficiency, only a random subset of connected nodes ([the square root](https://github.com/ethereum/go-ethereum/blob/v1.10.18/eth/handler.go#L621-L622) 🤓) are [sent the full transaction](https://github.com/ethereum/go-ethereum/blob/v1.10.18/eth/protocols/eth/peer.go#L183-L198). The rest is [only sent the transaction hash](https://github.com/ethereum/go-ethereum/blob/v1.10.18/eth/protocols/eth/peer.go#L213-L223). These could request back the full transaction if needed.
10. A transaction cannot linger in a node's mempool forever. If it's not first dropped for other reasons (e.g., the pool is full and the transaction is underpriced, or it gets replaced with a new one with higher nonce/price), it may be automatically [removed](https://github.com/ethereum/go-ethereum/blob/v1.10.18/core/tx_pool.go#L390-L396) after a certain period of time (by default, [3 hours](https://github.com/ethereum/go-ethereum/blob/v1.10.18/core/tx_pool.go#L184)). Valid transactions in the mempool that are considered ready to be picked up and processed by a block builder are [tracked](https://github.com/ethereum/go-ethereum/blob/v1.10.18/core/tx_pool.go#L1338-L1345) in a list of [pending transactions](https://github.com/ethereum/go-ethereum/blob/v1.10.18/core/tx_pool.go#L254). This data structure [can be queried](https://github.com/ethereum/go-ethereum/blob/v1.10.18/core/tx_pool.go#L536) by block builders to obtain processable transactions that are allowed to make it into the chain..
11. The txn then gets picked up by mining node and is prepared for execution. Execution is done in a spawned up EVM environment, along with txn and msg context and the previous state.
12. there can only be a maximum of 1024 nested calls in a single transaction (call depth limit)
13. In Ethereum Mainnet, all the precompiled contracts (addresses 0x01 to 0x09 ) have atleast 1 wei in balance. 
14. The data of the transaction itself, plus the block's hash and number, can always be retrieved at the node's `eth_getTransactionByHash` endpoint. The transaction's receipt can be requested at the `eth_getTransactionReceipt` endpoint

