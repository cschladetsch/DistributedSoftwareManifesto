Distributed systems spend enormous energy fighting a problem they created themselves: the assumption of shared global state.

Paxos, Raft, NTP, distributed locks -- these aren't solutions to hard problems. They're compensation mechanisms for a flawed starting premise.

I've been working on an architecture that abandons that premise entirely. A few of the core ideas:

**Propagation-capped state.** Instead of treating state changes as instantaneous, information travels at a bounded velocity through the network. Nodes see influence approaching before it arrives. That lead time turns jagged state transitions into smooth curves and eliminates a whole class of jitter.

**Saliency-driven sampling.** No streaming. Autonomous sampling agents update at rates proportional to observed activity -- high frequency where players are present, near-zero where nothing is happening. Computation follows attention, not a fixed tick rate.

**Reputation-weighted consensus.** No central authority. Nodes weight incoming data by the historical reliability of the sender, decayed over time and distance. Misbehaving nodes lose influence gradually. The system doesn't crash -- it just stops listening.

**Live continuation migration.** Running execution state moves between machines along the propagation path, pre-warming destination nodes before arrival. No stop-the-world. No hard cutover.

**Propagation-speed GC.** The garbage collector runs at the same velocity as mutations, so the marking front can never be outrun. Memory reclamation becomes a natural consequence of the same physics governing everything else.

The reference implementation is KAI -- a distributed object model in C++ with reflection, persistence, cross-process communication, and an incremental tri-color GC.

Full writeup linked below if you want the details.
https://github.com/cschladetsch/DistributedSoftwareManifesto

#distributedsystems #cpp #gameservers #systemsdesign #architecture

