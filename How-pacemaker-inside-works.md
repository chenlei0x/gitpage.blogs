---
title: How pacemaker inside works
date: 2018-04-23 11:56:25
tags: [pacemaker,ha,crmsh,crm]
categories: HA
---

This article tries to illustrate how pacemaker inside works, what components pacemaker comprises, and how the components works together.

# what is pacemaker
>Pacemaker is a cluster resource manager, that is, a logic responsible for a life-cycle of deployed software — indirectly perhaps even whole systems or their interconnections — under its control within a set of computers (a.k.a. nodes) and driven by prescribed rules. 
>
>It achieves maximum availability for your cluster services (a.k.a. resources) by detecting and recovering from node- and resource-level failures by making use of the messaging and membership capabilities provided by your preferred cluster infrastructure (either Corosync2 or Heartbeat3 ), and possibly by utilizing other parts of the overall cluster stack.

From my perspective, pacemaker is a set of software which help maintain each nodes which makes up a cluster.
It also guarantee each resource(process in fact) prescribed works well without user's interaction.


# Internal Components
* Cluster Information Base(CIB)
* Cluster Resource Management deamon (CRMd)
* Local Resouce Management deamon(LRMd)
* Policy Engine (PEngine or PE, only *one* PE exist within the whole cluster)
* Fencing daemon (STONITHd)

<!--more--> 

![----> means notify](http://clusterlabs.org/pacemaker/doc/en-US/Pacemaker/1.1/html/Clusters_from_Scratch/images/pcmk-internals.png)
X----->Y  means X notifies or orders Y


```bash
# pacemaker and its subprocesses

        ├─pacemakerd─┬─attrd
        │            ├─cib
        │            ├─crmd
        │            ├─lrmd
        │            ├─pengine
        │            └─stonithd

```
## CIB
> The CIB uses XML to represent both the cluster’s configuration and current state of all resources in the cluster.
The contents of the CIB are automatically kept in sync across the entire cluster and are used by the PEngine to compute the ideal state of the cluster and how it should be achieved.

CIB records, maintains and syncs the state description of the cluster.
CIB provides user-space tool [`cibadmin`](https://clusterlabs.org/pacemaker/man/cibadmin.8.html) to help users interact with CIB deamon. Any change happening on any node will be synced to the cluster.
## PE
The Policy Engine is the component that takes the cluster’s current state, decides on the optimal next state and produces an ordered list of actions to achieve that state.
Although the `pengine` process is active on all cluster nodes, it is only doing work on one of them. The “active” instance is chosen through the `crmd`’s DC election process and may move around as nodes leave/join the cluster.

## crmd



>PE generate the list of instructions  and then feed them to the Designated Controller (DC). Pacemaker centralizes all cluster decision making by electing one of the CRMd instances to act as a master. Should the elected CRMd process (or the node it is on) fail, a new one is quickly established.
>
>The DC carries out the PEngine’s instructions in the required order by passing them to either the Local Resource Management daemon (LRMd) or CRMd peers on other nodes via the cluster messaging infrastructure (which in turn passes them on to their LRMd process).

In summary:
* Exists on each node,  issues DC election, and finally one of crmd will serve as DC(designated controller)
* Only DC waits for instructions from PE, and retransfer them to other CRMs.
* All crmd will tells lrmd what to do next.

## lrmd
* Do whatever crmd wants
* Report the results to DC

## stonithd
>In some cases, it may be necessary to power off nodes in order to protect shared data or complete resource recovery. For this, Pacemaker comes with STONITHd.
>In Pacemaker, STONITH devices are modeled as resources (and configured in the CIB) to enable them to be easily monitored for failure, however STONITHd takes care of understanding the STONITH topology such that its clients simply request a node be fenced, and it does the rest.

In summary
Stonithd receives fence request and execute it!

# How decisions are made
A decision means the action the cluster will take to fit for the change. Decisions are made by PE.
When a state changes, like a node joining the cluster or after a service fails, the follow steps happen;

1.  The node elected as the DC will review it's CIB.
2.  It will compare the new state against the state it wishes to be in given the changes.
3.  Next it decides what action or actions are needed to get to that state.
4.  With this plan in place, the DC determines which steps it can perform and which steps other nodes need to perform.
    1.  Anything it can perform is passed down to the lrmd via it's local crmd.
    2.  Anything that needs to be acted on by other nodes is sent over the cluster messaging and picked up by each node's crmd. These messages are then passed down to each nodes lrmd.
    3.  The lrmd initiates the actual actions requested.
    4.  All nodes report back to the DC the results of their action.
    5.  The DC will determine, based on the results of the actions taken on the node(s), if any further action is required.

All actions that are taken, be they local to the DC or on a remote node, are executed in the same order across all nodes. This ordering is ensured thanks to a function of the cluster communication layer called "virtual synchrony", which is provided by corosync's use of the totem protocol.

# crmsh & pcs
Both serves as a powerful cluster management tool, but they are different in design models.
Do not get confused with crmd and crmsh. `crmd` is a pacemaker component, while `crmsh` is used to interact with pacemaker to manage a cluster.
crmsh communicates with pacemaker through `cibadmin` command.



# reference
-  pacemaker-from-scratch.pdf
-  https://www.alteeve.com/w/CIB
- http://blog.clusterlabs.org/blog/2013/debugging-pengine



> Written with [StackEdit](https://stackedit.io/).
