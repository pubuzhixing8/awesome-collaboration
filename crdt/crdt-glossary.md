# CRDT 词汇表
原文链接：  [https://crdt.tech/glossary#crdt-glossary](https://crdt.tech/glossary#crdt-glossary)  

**Add-wins set (AWSet):**
A set datatype in which additions take precedence over removals. For example, if one replica removes and re-adds an element, while another replica concurrently removes the element, then the merged outcome is that the element is in the set. Contrast with remove-wins set.
一种集合数据类型，其中添加优先于删除。 例如，如果一个副本删除并重新添加一个元素，而另一个副本同时删除该元素，则合并的结果是该元素在集合中。 与 remove-wins 集相对。
**Commutative Replicated Data Type (CmRDT):**
An old name for “operation-based CRDT”. The term “operation-based” is now preferred.
“基于操作的 CRDT”的旧称。 现在首选术语“基于操作”。
**Concurrent:**
Two events are concurrent if they happened without knowledge of each other (i.e. the node on which event A occurred had not heard of event B at the time of A, and vice versa).
如果两个事件在彼此不知情的情况下发生（即发生事件 A 的节点在 A 的时间没有听说过事件 B，反之亦然），则这两个事件是并发的。
**Convergence:**
The key correctness property of CRDTs. Convergence requires the following: for any two replicas, if they have processed the same set of updates, then those replicas must be in the same state (regardless of the order in which they processed those updates).
CRDT 的关键正确性属性。 收敛需要以下条件：对于任何两个副本，如果它们处理了相同的更新集，那么这些副本必须处于相同的状态（无论它们处理这些更新的顺序如何）。
**Convergent Replicated Data Type (CvRDT):**
An old name for “state-based CRDT”. The term “state-based” is now preferred.
“基于状态的 CRDT”的旧名称。 术语“基于状态”现在是首选。
**Delta-CRDT:**
A variant of a state-based CRDT in which replicas don’t always send their entire state, but rather send state differences (deltas). See Delta State Replicated Data Types by Paulo Sérgio Almeida, Ali Shoker, and Carlos Baquero.
基于状态的 CRDT 的一种变体，其中副本并不总是发送它们的整个状态，而是发送状态差异（增量）。 请参阅 Paulo Sérgio Almeida、Ali Shoker 和 Carlos Baquero 撰写的 Delta State Replicated Data Types。
**Effector:**
In operation-based CRDTs, an effector is a function that updates the state of a replica by applying an operation.
在基于操作的 CRDT 中，效应器是通过应用操作来更新副本状态的函数。
**Generator:**
In operation-based CRDTs, a generator is a function that processes a data modification action taken by a user, and produces an operation that can be sent over the network and applied by an effector.
在基于操作的 CRDT 中，生成器是一个函数，它处理用户采取的数据修改动作，并产生可以通过网络发送并由效应器应用的操作。
**Last write wins (LWW):**
A timestamp is attached to each update, and updates with higher timestamps always overwrite values written by updates with lower timestamps. However, if a replica has already applied an update with a higher timestamp, and it subsequently receives an update with a lower timestamp, the latter update is ignored. This approach is one way of ensuring convergence.
每个更新都附加一个时间戳，具有较高时间戳的更新总是覆盖具有较低时间戳的更新写入的值。 但是，如果副本已经应用了具有较高时间戳的更新，并且随后接收到具有较低时间戳的更新，则忽略后一个更新。 这种方法是确保收敛的一种方式。
**Observed-remove set (ORSet):**
Another name for add-wins set. “Add-wins” is now preferred.
add-wins 集的另一个名称。 “Add-wins”现在是首选。
**Operation-based CRDT:**
A CRDT in which each modification of the replica data is encoded as an operation, and operations are sent over the network. The system must ensure that operations are not lost (by resending if necessary) and that each operation is applied once (by ignoring duplicates). Operation-based CRDTs ensure that any two concurrent operations commute; that is, replicas can apply those operations in either order, and the outcome is the same.
CRDT，其中数据的每次修改都被编码为一个操作，并且操作通过网络发送。 系统必须确保操作不会丢失（必要时通过重新发送）并且每个操作都应用一次（通过忽略重复项）。 基于操作的 CRDT 确保任意两个并发操作可交换； 也就是说，副本可以按任一顺序应用这些操作，结果是相同的。
**Optimistic replication:**
An approach to replication in which a replica can process updates locally, without waiting for communication with other replicas. This means that replicas can become temporarily inconsistent, but CRDTs ensure that replicas nevertheless converge towards a consistent state (see Strong Eventual Consistency). See Optimistic Replication by Yasushi Saito and Marc Shapiro.
一种复制方法，其中副本可以在本地处理更新，而无需等待与其他副本的通信。 这意味着副本可能会暂时不一致，但 CRDT 确保副本仍然会收敛到一致状态（请参阅强最终一致性）。 请参阅 Yasushi Saito 和 Marc Shapiro 的乐观复制。
**Remove-wins set (RWSet):**
A set datatype in which removals take precedence over additions. For example, if one replica removes and re-adds an element, while another replica concurrently removes the element, then the merged outcome is that the element is not in the set. Contrast with add-wins set.
一种集合数据类型，其中删除优先于添加。 例如，如果一个副本删除并重新添加一个元素，而另一个副本同时删除该元素，则合并的结果是该元素不在集合中。 与加赢集相反。
**Replication:**
maintaining a copy of some data on multiple computing devices (maybe servers, maybe end-user devices). Those copies are called replicas.
在多个计算设备（可能是服务器，也可能是最终用户设备）上维护一些数据的副本。 这些复制版本称为副本。
**State-based CRDT:**
A CRDT in which replicas synchronise by sending each other their entire state over the network; when one replica receives such a state from another replica, it uses a merge function to combine the two states. This merge function is defined in such a way that it is commutative (i.e. merge(a,b) = merge(b,a)), associative (i.e. merge(a,merge(b,c)) = merge(merge(a,b),c)), and idempotent (i.e. merge(a,a)=a).
一个 CRDT，其中副本通过在网络上相互发送它们的整个状态来同步； 当一个副本从另一个副本接收到这样的状态时，它会使用合并函数来组合这两个状态。 这个合并函数是这样定义的，它是可交换的（即merge(a,b) = merge(b,a)），关联的（即merge(a,merge(b,c)) = merge(merge(a) ,b),c)) 和幂等（即合并（a，a）=a）。
**Strong Eventual Consistency (SEC):**
A formal consistency model for CRDTs. It requires convergence (see above) and eventual delivery (if one replica has processed an update, every other replica that has not failed will also eventually process it). See Conflict-Free Replicated Data Types by Marc Shapiro, Nuno Preguiça, Carlos Baquero, and Marek Zawirski.
CRDT 的正式一致性模型。 它需要收敛（见上文）和最终交付（如果一个副本已经处理了更新，所有其他没有失败的副本也将最终处理它）。 请参阅 Marc Shapiro、Nuno Preguiça、Carlos Baquero 和 Marek Zawirski 撰写的无冲突复制数据类型。
**Tombstone:**
A special object used in some CRDTs to indicate that a value is absent (e.g. because it has been deleted). Tombstones may increase the memory consumption of a CRDT, as they continue to exist even if the corresponding data has been deleted at the application level. There are algorithms for garbage-collecting tombstones and freeing their memory.
某些 CRDT 中使用的特殊对象，用于指示值不存在（例如，因为它已被删除）。 Tombstones 可能会增加 CRDT 的内存消耗，因为即使在应用程序级别删除了相应的数据，它们也会继续存在。 有一些算法用于垃圾收集墓碑并释放它们的内存。

**词汇：**
Eventual adj 最终的