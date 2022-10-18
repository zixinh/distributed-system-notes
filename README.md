# distributed-system-notes
sources:
- Designing Data-Intensive Applications from Martin Kleppmann
- [Distributed Systems By Martin Kelppmann](https://www.youtube.com/playlist?list=PLeKd45zvjcDFUEv_ohr_HdUFe97RItdiB)


#### Two General Problems
- Key is that each general does not know if their last messenger got through (Unreliable link). And once their counterpart sends reply, the counterpart starts to wonder if reply gets through, and so on -- there is always a party who is not sure if their message gets through.

#### LSM-Tree vs B-Tree

#### SSTable

#### Clock Synchronization
- problem: each node's clock might be drifted (because clock is made of quartz) and out of sync with other nodes
- solution: each node periodically sends requests to NTP server (Network Time Protocol) to sync with most accurate clock
<img width="1275" alt="Screen Shot 2022-10-17 at 8 23 04 PM" src="https://user-images.githubusercontent.com/28737133/196307487-4a06b029-9ae8-4f73-a2df-e89b752b929e.png">
