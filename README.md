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
<img width="1920" alt="Screen Shot 2022-10-17 at 8 22 59 PM" src="https://user-images.githubusercontent.com/28737133/196307272-f736f63e-b286-4ff3-98bb-af0ec3770c22.png">
