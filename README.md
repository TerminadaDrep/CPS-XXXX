---
CPS: ?
Title: Increase delta between valid slots to acceptable network delay
Status: Open
Category: ?
Authors: Terminada
Proposed Solutions: []
Discussions: https://github.com/cardano-foundation/CIPs/pull/?
Created: 2024-10-22
License: CC-BY-4.0
---

# CPS-XXXX: Increase delta between valid slots to acceptable network delay

## Abstract
An underlying assumption in the design of Cardano's Ouroboros protocol is that the probability of a stake pool being permitted to update the ledger is proportional to the relative stake of that pool.  However, the current implementation does not properly realise this design goal.

Minor network delays endured by some participants cause them to face a greater number of fork battles.  The result is that more geographically decentralised participants do not obtain participation that is proportional to their relative stake.  This is both a fairness and security issue.

## Problem
The [Ouroboros Praos paper](<https://eprint.iacr.org/2017/573.pdf>) states that it is the "*probability of being permitted to issue a block*" that "is proportional to the relative stake a player has in the system".  However, it is clear from the security analysis section that it is the _probability of contributing to the ledger_ that is supposed to be proportional to the relative stake of the participant.  This is an important distinction because it matters little if a player has stake weighted permission to make blocks if the protocol later disproportionately drops that players blocks despite their honest performance.

With the current Ouroboros implementation where slots have 1 second duration, it is not uncommon to have consecutive slot leaders that are only 1 second apart.  Remote participants may be unable to receive the previous block within 1 second and so these block producers will make blocks that result in forks.  Ouroboros settles such forks by preferring the fork whose terminal block has the lowest VRF value.  This seems like a fair method because it is deterministic, neither player can alter the outcome, and each player has an equal chance of winning.  However, the problem is that the more geographically decentralised players will face a disproportionately greater number of "fork battles".  This in turn means that they will get more of their blocks dropped.  The net result is that the more geographically decentralised participants do not receive stake weighted _probability of contributing to the ledger_.  This is not only unfair, but also represents a deviation from the Ouroboros security analysis assumptions.

This might seem like a minor problem, but the effect is significant.  If the majority of the network reside in USA - Europe with close connectivity and less than 1 second propagation delays, then these participant pools will see 5% of their blocks suffering "fork battles" which will only occur when another pool is awarded the exact same slot (ie: a "slot battle").  They will lose half of these battles on average causing 2.5% of their blocks to be not adopted, or "orphaned".

However, for a pool that happens to reside on the other side of the world where network delays might commonly be just over 1 second, this pool will suffer "fork battles" not only with pools awarded the same slot, but also the slot before, and the slot after.  In other words, this geographically decentralised pool will suffer 3 times the number of slot battles, amounting to 15% of its blocks, and resulting in 7.5% of its blocks getting rejected.  The numbers are even worse for a pool suffering 2 second network delays because it will suffer 5 times the number of "fork battles" and see 12.5% of its blocks "orphaned".  This not only results in an unfair reduction in rewards, but also the same magnitude reduction in contribution to the ledger.

Considering that most stake pools are competing for 1% or less in fees, these are big numbers.  The obvious solution for the remote pool is to move its block producer to a server housed in USA or Europe.  This illustrates not only the centralisation problem created, but also the reduction in security that flows from running a block producer on someone elses computing hardware.

There is also another problem caused by the current deterministic resolution of "fork battles".  This problem stems from the fact that participants can know their block VRF value for any particular slot prior to production.  They can use this knowledge to decide whether to deliberately cause a fork with the last block if they know they will win the "fork battle".  For example, an attacker can employ custom software that chooses which block to build upon depending on whether their VRF value is lower than that of the last block.  The benefit they gain from taking such action is that they will cause other participants to lose blocks thereby earning a higher percentage of the total rewards and simultaneously gaining more control over the ledger.  Such a malicious pool, or coalition of pools, will look better to potential delegators because they will have a higher "luck percentage" on leader boards displayed by wallet apps, allowing them to accumulate more delegators, further increasing centralisation.

## Goals
Cardano should live up to the suggested [11 blockchain tenets](<https://iohk.io/en/blog/posts/2024/10/11/the-11-blockchain-tenets-towards-a-blockchain-bill-of-rights/>) proposed by Prof Aggelos Kiayias, in particular T8 and T11 which speak to treating participants fairly without asymmetries.

## Possible Solutions
1. Modify how stake pools calculate their slot leadership by including ```mod 2``` or ```mod 3``` in the formula so that only every second or third slot is a possible leader slot.  Then readjust the other parameters so that the Cardano network still produces a block every 20 seconds on average.

  A consequence of this change would be an increased number of true "slot battles", where two pools are awarded the exact same slot.  However such battles are fairly settled by preferring the lowest block VRF and cannot be gamed by a malicious actor.

2. Increase the slot duration to 2 or 3 seconds.  However this could have consequences for dapps that have assumed a slot will always be 1 second.

## Considerations
1. What is a reasonable expectation for block producers housed across the world to achieve in terms of network delays?  Is 2 seconds of network delay enough time before block producers get penalised, or would 3 seconds be better?  What countries, and their internet infrastructure, does Cardano consider as a minimum requirement in order to fairly contribute to its ledger?

2. Would there be any impact on the Ouroboros 5 second delta parameter?

3. Would it be appropriate to make the window in which the block VRF tie breaker rule is applied also the same 2 or 3 seconds?  So that only blocks with the exact same slot number are deterministicly settled using the block VRF, otherwise the node would prefer its current block.  Such a change would remove the ability of malicious actors to deliberately cause a fork with the previous block when they know their block VRF is lower.

4. What effect could any change have on the solution to [issue #2913: Consensus should favor expected slot height to ward off delay attack](<https://github.com/IntersectMBO/ouroboros-network/issues/2913>).

## Copyright
This CIP is licensed under [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/legalcode).

****
