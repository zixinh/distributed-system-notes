# distributed-system-notes
#### sources
- Designing Data-Intensive Applications from Martin Kleppmann
- [Distributed Systems By Martin Kelppmann](https://www.youtube.com/playlist?list=PLeKd45zvjcDFUEv_ohr_HdUFe97RItdiB)
- all over internet

#### guideline
- read quickly - understand the concept in overall system, no need to understand details 
- only write notes if I believe it's going to be useful in real world settings

## index
- [time](#time)
  - [clock synchronization](#clock-synchronization)
  - [vector clock](#vector-clock)
- [replication](#replication)
  - [state update with retry](#state-update-with-retry)
  - [tombstone and replica state sync](#tombstone-and-replica-syncing)
  - [read and write quorum](#read-and-write-quorum)
- [consensus](#consensus)
  - [total-order broadcast](#total-order-broadcast)
  - [raft](#raft)
  - [atomic commit](#atomic-commit)
- [database](#database)
  - [log-structured database](#log-structured-database)
  - [SSTable & LSM-Tree](#sstable-and-lsm-tree)
  - [B-Tree](#b-tree)
  - [LSM-Tree vs B-Tree](#lsm-tree-vs-b-tree)
  - [clustered index vs unclustered index](#clustered-index-vs-unclustered-index)
  - [multi-column index](#multi-column-index)
  - [full-text search and fuzzy index](#full-text-search-and-fuzzy-index)
- [misc](#misc)
  - [two general problem](#two-general-problem)


## time
#### clock synchronization
- problem: each node's clock might be drifted (because clock is made of quartz) and out of sync with other nodes
- solution: each node periodically sends requests to NTP server (Network Time Protocol) to sync with most accurate clock
  - be careful of using `getCurrentTimestamp` to get time duration, as NFP might happen at anytime and skew the clock, which leads to negative time duration. Use `nanotime` instead.
<p align="center"> 
<img width="300" alt="Screen Shot 2022-10-17 at 8 23 04 PM" src="https://user-images.githubusercontent.com/28737133/196307487-4a06b029-9ae8-4f73-a2df-e89b752b929e.png">
</p>

#### vector clock
- for a system of N nodes, each node has a vector <t1, t2,...tn>, where ti refers to logic timestamp of node i. 
- benefit: 
  - each vector corresponds to unique event at unique node because each event is incremented by 1 on node i's timestamp
  - represents happen-before relationships if V(a) < V(b) because if tk at V(a) < tk at V(b), then it means there is a new event to increment node B's counter from node A (does not mean directly, it just means there is a new event happen from A to B's causal chain)
  - represents concurrent relationships if V(a) || V(b) which is V(a) !<= V(b) and V(a) !>= V(b), because timestamps are incomparable, some events happened and node A and B is not directly related, so its corresponding counter is not updated
<p align="center"> 
<img width="300" alt="Screen Shot 2022-10-26 at 12 06 32 AM" src="https://user-images.githubusercontent.com/28737133/197932236-d618a23e-6ef2-4b66-856f-452bd15ca748.png">
</p>

## replication
#### state update with retry
- problem: because network sometimes may fail, how to prevent duplicated state updates
- solution: using idempotent function
<p align="center"> 
<img width="300" alt="Screen Shot 2022-12-17 at 12 06 24 PM" src="https://user-images.githubusercontent.com/28737133/208253182-47b2f7b0-0b85-438f-8fe9-ee6c1cf5d0fd.png">
</p>

#### tombstone and replica syncing
- problem: in case of certain network requests fail and one replica is outdated, how to sync across replicas?
- solution: for each requests, we assigned a timestamp and set desired value with that timestamp in a pair (tombstone), periodically replicas will have a syncing call to compare each value with latest timestamp, if one replica has a value with newer timestamp, then the other replica with older timestamp will sync with the other replica.
<p align="center">
<img width="300" alt="Screen Shot 2022-12-17 at 6 38 00 PM" src="https://user-images.githubusercontent.com/28737133/208269893-187edd55-290c-4d73-b53b-10618c8f00b7.png">
</p>

#### read and write quorum
- assuming a cluster has `n` replicas where each replica can be read and write, **to ensure that user always reads the most up to update value of a key `k`**, we need to update key `k` on `w` write replicas and user needs to make read requests to `r` replicas on the value of `k` where `w + r > n`. 
- `w + r > n` ensures that there is always at least one replica that has the most updated value (see venn diagram in picture). occasionally read requests may give outdated value of key `k`, but as each update has a logical timestamp on it, the value with latest logical timestamp is the most up to date value of key `k`, and user will compare all read requests' timestamp and determine the latest one.
<p align="center">
<img width="300" alt="Screen Shot 2022-12-17 at 8 36 02 PM" src="https://user-images.githubusercontent.com/28737133/208273290-d0779f6c-13e4-4b22-97ee-b5475c3290b3.png">
</p>


## consensus
#### total-order broadcast

#### raft

#### atomic commit

## database

#### log-structured-database
- a series of log, segment by segment, and host a hash table between every key and its byte offset across all segments
- read: check if key in hash table, if yes, get the data byte offset, and get the value
- write: append the new key-value pair to last segment, if key is in hash table, update byte offset
- compaction/merging: periodically remove outdated key-value pairs across segments

#### SSTable and LSM-Tree
- SSTable ~ Sorted String Table / LSM-Tree ~ Log-Structured Merge Tree
- comparing to log-structured database, for each segment, key-values pairs are sorted by key, and host a hash table across all segments with sparse keys, not for every key
- write: database will host an in-memory tree structure or memtable, and it will append the new key/value to corrsponding position in the tree, if the tree reaches certain size, database will convert this tree to a SSTable segment, update sparse hash table and create a new empty tree
- read: first check in-memory tree, if not there, check hash table, and because keys are sorted, just need to find keys that are closed to target key, and search that area. e.g. hash table has key dad and dog, if target key is deer, then deer's value must be between dad's offset and dog's offset.
- compaction/merge: basically comparing segments from start to end, because each segment is sorted, merge is much easier

#### B-Tree
- block tree: a tree of blocks where each block is a limited key-value pairs
- only leaves are storing actual values and non-leaves layer represents a more abstract key ranges and each key refers to a block that contains granular key ranges. 
- example: top of the tree has the most general key range, e.g. 100 200 300 and each key's value points to a block saves more granular key range, e.g. 200 'value refers to a block where it has 200 230 260, and key 230's value refers to a block that saves key range e.g. 238 246 254 and so on until leave node. 
- [image source](https://cs.appstate.edu/~crr/cs3460/examples/btrees.html)
<p align="center">
<img width="416" alt="Screen Shot 2023-01-06 at 12 22 33 AM" src="https://user-images.githubusercontent.com/28737133/210935801-3d44cbdf-ce81-479c-b57b-073e9beeba63.png">
</p>

#### LSM-Tree vs B-Tree
- both are index to boost read & write, but has some tradeoffs
- generally B-tree faster for reads, LSM-tree faster for writes
- practically need to evaluate case by case 

#### clustered index vs unclustered index
- mainly two ways to store values in index 
  - clustered index: actual row is stored with key within index
  - unclustered index: a reference to row is stored with key within index, and actual rows are stored in a file known as *heap file*
- a hybrid index is called covering index where some info + reference of row is stored with key within index

#### multi-column index
- one common approach is to transform multi-dimensional data to a one-dimensional data, and feed them into normal index like B-Tree
  - for example, concatenated index: append multiple columns to a single key and index them using that key
  - other transformations is ok too e.g. convert GPS coordinates to a space-filling curve
- more specialized multi-column index for different use case is available e.g. R-tree for spatial indexes

#### full-text search and fuzzy index
- one common method is to convert keys to a finite state machine over the characters used in the keys e.g. Elastic Search
- other methods available too using document classification and machine learning

## misc

#### two general problem
- Key is that each general does not know if their last messenger got through (Unreliable link). And once their counterpart sends reply, the counterpart starts to wonder if its reply gets through, and so on -- there is always at least one party who is not sure if their message gets through.
- Even with multiple rounds of confirmation, still uncertain. For example, after several rounds of confirmations, A sends confirmation to B, and starts to worry not deliver, if B receive it, and B sends back confirmation, and B worries its delivery, and if A receives it and send back a confirmation, A starts to worry again, and so on.... `because this unreliable link, confirmation is always required to confirm if the message is delivered, however, the new confirmation itself is unreliable as well that needs another confirmation.`
