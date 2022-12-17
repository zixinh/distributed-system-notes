# distributed-system-notes
sources:
- Designing Data-Intensive Applications from Martin Kleppmann
- [Distributed Systems By Martin Kelppmann](https://www.youtube.com/playlist?list=PLeKd45zvjcDFUEv_ohr_HdUFe97RItdiB)

## index
- time
  - [clock synchronization](#clock-synchronization)
  - [vector clock](#vector-clock)
- replication
  - [state update with retry](#state-update-with-retry)
- database
  - [LSM-Tree vs B-Tree](#lsm-tree-vs-b-tree)
  - [SSTable](#sstable)
- misc
  - [two general problem](#two-general-problem)

#### two general problem
- Key is that each general does not know if their last messenger got through (Unreliable link). And once their counterpart sends reply, the counterpart starts to wonder if its reply gets through, and so on -- there is always at least one party who is not sure if their message gets through.
- Even with multiple rounds of confirmation, still uncertain. For example, after several rounds of confirmations, A sends confirmation to B, and starts to worry not deliver, if B receive it, and B sends back confirmation, and B worries its delivery, and if A receives it and send back a confirmation, A starts to worry again, and so on.... `because this unreliable link, confirmation is always required to confirm if the message is delivered, however, the new confirmation itself is unreliable as well that needs another confirmation.`

#### LSM-Tree vs B-Tree

#### SSTable
- sorted string table: for each log-structured storage segment, the sequence of key-value pairs is sorted by key

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

#### state update with retry
- problem: because network sometimes may fail, how to prevent duplicated state updates
- solution: using idempotent function
<p align="center"> 
<img width="616" alt="Screen Shot 2022-12-17 at 12 06 24 PM" src="https://user-images.githubusercontent.com/28737133/208253182-47b2f7b0-0b85-438f-8fe9-ee6c1cf5d0fd.png">
</p>
