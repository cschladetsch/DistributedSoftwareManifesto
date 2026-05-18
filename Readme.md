# Propagation Manifold

## An Information-Physics Model for Distributed Systems

Reference implementation: [KAI](https://github.com/cschladetsch/KAI). This a distributed object model for C++ with reflection, persistence, cross-process communication, and an incremental tri-color garbage collector.

---

## I. The Problem With Global State

Distributed computing has long operated on a flawed assumption: that a network of machines can and should share a singular, synchronized reality.

This assumption has costs. Paxos and Raft exist to force consensus. NTP exists to approximate a universal clock. Distributed locks exist to freeze system state long enough to observe it safely. These are not solutions to hard problems; they are compensation mechanisms for a bad premise.

The physics of our universe do not support global simultaneity. Einstein established that simultaneity is observer-relative. Any architecture that requires the opposite is fighting the laws of the medium it runs on.

Propagation Manifold is an architectural model that accepts this constraint rather than fighting it. It replaces synchronization as a goal with *perspective consistency*. Each node maintains a coherent local view, and the system's behaviour emerges from the interaction of those views.

![Demo](Demo1.jpg)

See the [live interactive model](https://cschladetsch.github.io/DistributedSoftwareManifesto) for the influence weight formula.

---

## II. Propagation-Bounded State: Propagation Physics

Standard network architectures treat state changes as instantaneous from the software's perspective. An object is at position A; then it is at position B. This step-function model is the root cause of jitter. The discontinuity propagates through the system as instability.

Propagation Manifold caps information propagation at a configurable propagation velocity. This is the Propagation Constant.

This has a useful consequence: when an event occurs at Node A, it does not immediately update Node B. Instead, it generates a *wavefront* that travels through the network fabric at the defined velocity. Because the wave has a measurable speed, distant nodes observe the *approach* of an influence before it arrives. This provides lead time. The most valuable resource in distributed state management.

State transitions are handled as curves rather than step functions. The result is phase stability: the system absorbs change rather than snapping between states.

![Diagram](diagram.png)

---

## III. Saliency-Driven Sampling: The Witness Layer

Traditional distributed systems stream data continuously. This is expensive and largely unnecessary. Most of the data being streamed describes regions of the simulation that nobody is currently observing.

Propagation Manifold samples rather than streams. Autonomous agents, called Witnesses, reside at every coordinate in the network fabric. Every N milliseconds, a Witness samples its local environment, collapses the surrounding influence waves into a stable snapshot, and records it.

Two properties follow from this:

**Natural hysteresis.** If a variable oscillates between states a thousand times between samples, the rest of the system never observes the oscillation. High-frequency noise is filtered by the act of observation itself.

**Saliency-proportional compute.** The sampling rate is governed by local activity. Witnesses in regions with active observers run at 60Hz. Witnesses in unobserved regions drop to a quiescent low-frequency state; once per minute or less. Computation is allocated proportionally to what is being observed. Unobserved regions cost almost nothing.

This is a form of adaptive scaling: the system self-organises its resource allocation based on attention rather than a fixed global tick rate.

---

## IV. Influence-Weighted Consensus: The Reputation Model

Without a central server, how is order maintained?

The answer is the Summation of Influence. Each node carries an Influence Weight. When Node A receives data from Node B, it does not treat it as ground truth. It calculates an Influence Delta using the inverse square law and temporal decay:

$$
W_{\text{influence}} = \frac{W_{\text{source}}}{1 + \text{dist}^2} \times e^{-\lambda t}
$$

Where $W_{\text{source}}$ is the sender's historical reliability score, $\text{dist}$ is the propagation-bounded distance, and $e^{-\lambda t}$ is message entropy over time.

A misbehaving node; whether due to a bug or deliberate fault injection, does not cause a crash. The surrounding nodes detect deviation from local averages. The rogue node's reputation weight decays statistically until its influence approaches zero. No explicit banishment is required; the system stops weighting its output.

This mirrors flocking behaviour in nature: local rules, applied consistently, produce coherent global order without central coordination.

See the [live interactive model](https://cschladetsch.github.io/DistributedSoftwareManifesto) for the influence weight formula.

---

## V. Live Continuation Migration

Moving a live execution — a Continuation — from one machine to another without a stop-the-world event is one of the harder problems in distributed systems.

In the Propagation Manifold, a Continuation in transit is treated as a traveling state wavefront moving at the configured propagation velocity through the network. As it passes through intermediate nodes, the Witnesses along the path record its passage and update their local directional vectors, creating a propagation trail.

By the time the Continuation arrives at its destination, the neighboring machines have already pre-patched their references. They saw the state approaching and prepared. The handoff is not a hard cutover: it is the final step in a gradual handover that began the moment migration was initiated.

---

## VI. Distributed Garbage Collection: Tri-Color Reclamation

Memory reclamation in the Propagation Manifold is a natural consequence of the same propagation physics governing everything else.

Objects exist in one of three states:

- **White:** Not sampled by any Witness for an extended period. Candidates for reclamation.
- **Grey:** Currently within the observation radius of an active agent.
- **Black:** High influence weight; part of the current active reality ground.

Because the garbage collector's marking front travels at the same propagation velocity as mutations, it cannot be outrun by the mutation front. A flying reference cannot be collected before it is observed. The reclamation model is self-consistent with the propagation model, with no special cases required.

---

## VII. Summary

Propagation Manifold is a distributed systems architecture built on four premises:

1. Global simultaneity is physically impossible and architecturally counterproductive. Design for perspective consistency instead.
2. Bounded propagation velocity provides lead time and eliminates step-function state transitions.
3. Saliency-driven sampling allocates compute proportionally to observed activity, not to a fixed global rate.
4. Reputation-weighted influence produces emergent consensus without central coordination.

The reference implementation is KAI. Core behaviours have been stable for over 16 years. The approach is applicable to game servers, simulation systems, and any distributed architecture where latency, scale, and fault tolerance are primary constraints.
