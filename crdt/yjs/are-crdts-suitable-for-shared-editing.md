# CRDT 是否适合共享编辑？

> 【 原文链接 】：   [https://blog.kevinjahns.de/are-crdts-suitable-for-shared-editing/](https://blog.kevinjahns.de/are-crdts-suitable-for-shared-editing/)  


[CRDT](https://crdt.tech/)通常被誉为构建协作应用程序的 "holy grail"，因为它不需要中央机构来解决同步冲突。它为扩展后端基础设施开辟了新的可能性，也非常适合作为完全不需要服务器的分布式应用程序的数据模型。

但是，一些文本编辑器开发人员反映不要使用它们，因为它们会产生过大的开销。
就在最近，Marijn Haverbeke 写了一篇文章，反对使用 CRDT 作为 CodeMirror 6的数据模型:

> [..] 这样做的成本是巨大的，最后，我认为收敛位置的要求太模糊了，不足以证明这种额外的复杂性和内存使用的合理性。  [(来源)](https://marijnhaverbeke.nl/blog/collaborative-editing-cm.html)  

[Xi Editor](https://github.com/xi-editor/xi-editor)   使用 CRDT 作为其数据模型，以允许不同的进程（语法高亮器、类型检查器等）在不阻塞进程的情况下同时访问编辑器状态。他们回归到同步模型是因为…

> [..] CRDT并没有承担起它(足够的)责任   [(来源)](https://github.com/xi-editor/xi-editor/issues/1187#issuecomment-491473599)  


显然，每个人都认识到 CRDT 有很大的潜力，但是得出的结论是，使用它们的内存开销对于真实的应用程序来说肯定太昂贵了。

他们提出了一个合理的观点。大多数 CRDT 为在文档中创建的每个字符分配了一个唯一的ID。为了确保文档始终能够收敛，CRDT模型即使在删除字符时也会保留此元数据。

这对于像 JavaScript 这样的动态语言来说似乎特别昂贵。其他语言允许你在内存中使用   [structs](https://en.wikipedia.org/wiki/Struct_(C_programming_language))   (例如 C 或 Rust )有效地表示所有这些字符和 ID 。在 JavaScript 中，所有东西都表示为一个对象 —— 基本上是一个需要跟踪其所有键和值的键值映射。CRDT 为文档中的每个字符分配多个属性，以确保解决冲突。仅仅将文档表示为 CRDT 的内存开销可能是巨大的。

每个用户交互都会创建更多的元数据，CRDT 需要保留这些元数据以确保冲突解决。CRDT 创建数百万个对象来存储所有这些元数据的情况并不少见。JavaScript 引擎管理堆上的这些对象，检查它们是否被引用，如果可能的话，对它们进行垃圾收集。另一个主要问题是，随着对象创建数量的增加，创建对象的成本呈指数级增长。如下图所示:

> 创建一个对象的平均时间随着堆上对象的数量的增加而增加 [(数据源)](https://jsperf.com/cost-of-objects)  
> 
> ![image.png](https://atlas-rc.pingcode.com/files/public/60efa878f6d53ddae85c5186/origin-url)


因此，问题就来了，CRDT 是否真的适合在 web 上进行协同编辑，或者它是否会带来太高的成本从而无法在实践中实现。

或许你不认识我，我是一个 CRDT 实现库   [Yjs](https://github.com/yjs/yjs)   的作者，Yjs 是专门为在 web 上构建协同编辑应用程序而设计的。

在本文中，我将向你介绍 CRDT 的简单优化，并研究使用 Yjs 进行协同编辑的具体性能权衡。我希望使你相信，即使对于具有较长编辑历史的大型文档，开销实际上也是很小的。



## Yjs

Yjs 是一个使用 CRDT 作为数据模型来构建协作应用程序的框架。它拥有不断增长的扩展生态系统，可以使用不同的编辑器 (   [ProseMirror](https://docs.yjs.dev/ecosystem/editor-bindings/prosemirror)  ,   [Remirror](https://docs.yjs.dev/ecosystem/editor-bindings/remirror)  ,   [Quill](https://docs.yjs.dev/ecosystem/editor-bindings/quill)  ,   [CodeMirror](https://docs.yjs.dev/ecosystem/editor-bindings/codemirror)  , ..)，不同的网络技术(   [WebSocket](https://docs.yjs.dev/ecosystem/connection-provider/y-websocket)  ,   [WebRTC](https://docs.yjs.dev/ecosystem/connection-provider/y-webrtc)  ,   [Hyper](https://docs.yjs.dev/ecosystem/connection-provider/y-hyper)  , ..)，以及不同的持久化层 (   [IndexedDB](https://docs.yjs.dev/ecosystem/database-provider/y-indexeddb)  ,   [LevelDB](https://docs.yjs.dev/ecosystem/database-provider/y-leveldb)  ,   [Redis](https://docs.yjs.dev/ecosystem/database-provider/y-redis)  , ..)来进行协同编辑。大多数协同编辑解决方案与特定的编辑器和特定的后端绑定。使用 Yjs，你可以通过自定义的通信通道，通过点对点 WebRTC 网络，或通过可伸缩的服务器基础设施，使任何受支持的编辑器协作和交换文档更新。我对这个项目的愿景是，你可以简单地使用对你的项目有意义的技术组合你的协作应用程序。



这不仅仅是一个很酷很典型的业余项目。Yjs 是一种久经考验的技术，被多家公司用来实现协作。我只是在这里提到我的赞助商:

-   [Nimbus Note](https://nimbusweb.me/note.php)   使用Yjs水平扩展协同注释编辑。
-   [Room.sh](https://room.sh/)   是一个会议软件，允许通过WebRTC进行协作编辑和绘图。

我维护了一组可重复的基准测试，用于比较不同的 CRDT 实现。Yjs 是迄今为止速度最快、编码最高效的基于 web 的 CRDT 实现。在本文中，我经常将   [crdt-benchmarks](https://github.com/dmonad/crdt-benchmarks)    测试库中包含的特定基准测试称为“  [[B1.11]](https://github.com/dmonad/crdt-benchmarks/#b1-no-conflicts)  ”。



## 数据表示

你可能已经熟悉了 CRDT 工作原理的一般概念。如果没有，你想要深入这个未知领域，我推荐这个有趣的互动系列:

> [关于crdt的有趣互动系列](https://lars.hupel.info/topics/crdt/01-intro/)  

> [找到更多与CRDT相关的资源是一个很好的切入点](https://crdt.tech/resources)  
 

[Researchgate](https://www.researchgate.net/publication/310212186_Near_Real-Time_Peer-to-Peer_Shared_Editing_on_Extensible_Data_Types)   上提供了描述 Yjs 冲突解决算法 YATA 的概念论文。 这里讨论的概念非常通用，几乎可以扩展到任何 CRDT。


为了确定性能成本，我们将研究 Yjs 如何维护数据。请耐心听我描述如何使用 JavaScript 对象表示 CRDT 模型。这将是相关的。

与其他 CRDT 类似，YATA CRDT 为每个字符分配一个唯一 ID。然后将这些字符保存在一个双链表中。

唯一的 ID 是   [Lamport Timestamps](https://en.wikipedia.org/wiki/Lamport_timestamp)  .。它们由一个唯一的用户标识符和一个随着每个字符插入而增加的逻辑时钟组成。

当用户从左到右键入内容“ABC”时，它将执行以下操作： insert(0, "A") • insert(1, "B") • insert(2, "C")。 对文本内容建模的 YATA CRDT 的链表将如下所示：


> 插入内容“ABC”的CRDT模型（假设用户具有唯一的客户端标识符“1”）
> 
> ![image.png](https://atlas-rc.pingcode.com/files/public/60efcac4f6d53d28ba5c519b/origin-url)


请注意如何通过唯一的客户端 ID 和不断增加的时钟计数器的组合来唯一标识每个字符。

Yjs 将链表中的项表示为 Item 对象，该对象包含一些内容(在本例中为 String )、唯一ID、到相邻 Item 对象的链接，以及与 CRDT 算法相关的附加元数据。

所有的 CRDT 都会为每个字符分配某种唯一的 ID 和附加的元数据，这对于大型文档来说非常消耗内存。我们不能删除元数据，因为它是解决冲突的必要条件。Yjs 还唯一地标识每个字符和分配元数据，有效地表示了这些信息。较大的文档插入表示为单个 Item 对象，使用字符偏移量唯一地单独标识每个字符。下面的 Item 将字符“A”唯一标识为 {client:1,clock:0}，字符“B”为 {client:1,clock:1}，依此类推......

Yjs 对链表中item的内部表示：
```
Item {
    id: { client: 1, clock: 0 },
    content: 'ABC',
    ...
}
```


如果用户将大量内容复制/粘贴到文档中，则插入的内容由单个 Item 表示。此外，从左到右写入的单字符插入可以合并为单个 Item。重要的是，我们能够在不丢失任何元数据的情况下拆分和合并项。

CRDT 模型的这种复合表示及其拆分功能首先在“  [用于智能和大规模协作系统的字符串 CRDT 算法](https://kundoc.com/pdf-a-string-wise-crdt-algorithm-for-smart-and-large-scale-collaborative-editing-sys.html)  ”中进行了描述。 Yjs 为 YATA 调整了这种方法，还包括合并 Item 对象的功能。



## 操作成本 

考虑到这个简单的优化，让我们看看文档上的修改数量与需要保留的元数据数量之间的关系。我们将通过创建的 Item 对象的数量来度量元数据，然后检查单个 Item 的成本是多少。

每个用户与文本编辑器的交互都可以表示为  **插入**  或  **删除**  操作。

- `insert(index: number, content: string)`  任何大小的插入都会创建集成到文档中的单个 Item。在某些情况下，集成需要拆分现有的 Item。因此，每次插入最多可以创建两个 Item。
- `delete(index: number, length: number)`   删除一个 Item 只会将其标记为已删除。即   `item.deleted = true`  。因此，Item 的删除是自由的，并且会减少使用的内存量，因为 Item.content 可以被删除。Item 不需要保留执行冲突解决的内容。但是删除一系列内容可能需要拆分两个现有 Item。因此，删除的代价也最多为创建两个 Item。


通过使用 CRDT 的复合表示，元数据的数量只与用户产生的操作数量相关，而不是插入字符的数量。这在实践中有很大的不同。大多数较大的文档是通过复制粘贴现有内容或将段落移动到其他位置来创建的。任何类型的操作，甚至是复制-粘贴和撤销/重做，最多只能创建两个 Item 对象。

Yjs 还支持富文本和结构化文档。上面的语句(元数据的数量只与操作的数量相关)对于这类文档仍然成立。但是，测量更复杂操作的操作成本超出了本文的范围。实践中一个有趣的观察是，在结构化文档(例如，使用   [y-prosemirror](https://docs.yjs.dev/ecosystem/editor-bindings/prosemirror)   绑定到    [ProseMirror](https://prosemirror.net/)   编辑器)上的长时间编辑会话的文档大小实际上比线性文本文档小得多。这是由于其他优化在 Yjs 中起作用，可能会在另一篇文章中进行研究。


## 测量性能

在学术研究中，通过 CRDT 每秒可以处理的(并发)操作量来衡量性能已经成为一种常见的做法。对于具有高吞吐量的数据库应用程序，这可能是一个重要的基准测试。但是在协同编辑应用程序中，相对较少的用户每秒只产生几次操作。因此，单个操作的集成过程需要1纳秒还是100纳秒并不重要。此外，很少发生冲突，因为大多数 CRDT 是相对字符寻址的，只有两个用户同时在同一位置插入一个字符时才会发生冲突。

当我们只使用特定场景(例如，随机位置的插入数)来对性能进行基准测试时，我们最终可能得到一个只在这个特定场景中表现良好的CRDT。在实践中，其他性能特征也发挥了作用。我试图在   [crdt-benchmarks](https://github.com/dmonad/crdt-benchmarks)   资源库中捕获不同场景中的相关性能特征。它表明某些 CRDT 在某些场景中表现良好，但在其他场景中表现不佳。例如，RGA 实现在附加内容时表现良好，但在只附加内容时则表现很差。用于协同编辑的 CRDT 在所有场景中都应该表现良好。下面的列表描述了协同编辑应用程序的生命周期，并提供了对不同性能特征相关性的更多了解。

1. 文档是从网络或本地文件加载的。解析已编码的文档通常需要大量时间，尤其是在文档很大的情况下。  `parseTime`   表示解析已编码文档所需的时间。在我看来，  `parseTime`   是最重要的性能特征，因为它在应用程序开始时阻塞了进程。所有其他任务都可以在用户不工作时执行。
2. 文档与其他对等方一致。 如果文档在脱机时被修改，可能会发生冲突。   [[B2]](https://github.com/dmonad/crdt-benchmarks#b2-two-users-producing-conflicts)   基准测试仅由两个客户端（例如在客户端-服务器环境中）产生的同步冲突。   [[B3]](https://github.com/dmonad/crdt-benchmarks#b3-many-conflicts)   基准测试测量在多个客户端之间的同步冲突（在 p2p 环境中可能发生的同步冲突）所需的时间。在大多数CRDT实现中，需要在加载文档时再次解决同步冲突，所以在查看基准测试时要特别注意 parseTime。
3. 用户对本地文档应用变更。Yjs 使用一个链表来表示文档中的字符。除非编辑器直接处理 CRDT 模型，否则编辑器将使用索引位置应用  `insert`  和  `delete`  操作。CRDT 实现需要遍历其内部表示以找到位置，执行更改，然后生成发送给其他对等方的更新。  `time`   表示执行某个任务所需的时间(例如，添加100万个字符，同步N个并发更改，..)。  [[B1]](https://github.com/dmonad/crdt-benchmarks#b1-no-conflicts)   基准测试模拟单个用户对文档执行更改而实际上不会产生冲突。它表明，在简单地对文档应用更改时，某些 CRDT 有着很大的开销。
4. 协作者将文档更新发送到远程对等方。 我们假设在远程对等点上应用单个更新不会花费大量时间。   [[B2]](https://github.com/dmonad/crdt-benchmarks#b2-two-users-producing-conflicts)   和   [[B3]](https://github.com/dmonad/crdt-benchmarks#b3-many-conflicts)   基准测试涵盖了应用多个操作。大多数 CRDT 实现都支持某种形式的增量更新功能。每次更改都会产生一个小的更新，发送给其他对等点。  `avgUpdateSize`   表示文档更新的平均大小。 它只是确认一个 CRDT 产生小的增量更新。
5. 文档存储在数据库中或发送给远程对等方。  `encodeTime`  表示将文档转换为二进制表示所花费的时间。  `docSize`  表示编码文档的大小。

[crdt-benchmarks](https://github.com/dmonad/crdt-benchmarks)   自述文件显示了许多与协同编辑相关的不同场景的性能特征。 在本文中，我们只介绍了几个衡量最相关性能特征的基准：

- `memUsed`  应用所有更改后JavaScript引擎的堆大小。在 crdt-benchmark 库中，  `memUsed`  只是所用内存的近似值，因为我们不能可靠地运行垃圾收集器来删除以前基准测试的痕迹。我单独运行了本文的基准测试，并在性能检查器中直接测量堆大小，以获得更准确的结果。
- `docSize`  编码文档的大小。Yjs有一个非常高效的编码器，可以将 Item 对象写入二进制压缩格式。它通过网络(WebSocket, HTTP, WebRTC， ..)发送给其他客户端，所以我们要确保文档大小是合理的。
- `parseTime`  解析已编码文档所花费的时间。在我们从网络上收到编码文档后，我们希望尽快呈现它。因此，我们期望在合理的时间内解析它。


## 最糟糕的情况

在Yjs的最佳情况下，用户从左到右写内容。在这种情况下，所有操作都合并到一个Item中。

最糟糕的情况是用户从右向左编写内容。这个场景准确地反映了没有复合优化的Yjs的性能开销。请注意，这个场景不是自然发生的，因为即使是从右到左的书写系统(例如 Hebrew )也会按照逻辑顺序(从左到右)存储数据。

当用户写大文档时，通常会产生多少插入操作？如果我在单个文件中从头开始编写 Yjs，我将编写大约 20 万个字符。整个 CodeMirror 源代码由 568k 个字符组成。假设 Yjs 需要处理一百万个插入操作同时从右向左写。这相当于一个很大的文件。情况有多糟?

我们担心 Yjs 使用太多内存来表示所有这些 Item 对象。 毕竟，JavaScript 中的内存使用效率似乎很低，而且我们还不得不担心创建 JavaScript 对象的时间呈指数级增长。

为了对最坏情况进行基准测试，我们将通过在位置 0 产生一百万个单字符插入操作来创建一百万个 Item 对象：

```
import * as Y from 'yjs'

const ydoc = new Y.Doc()
const ytext = ydoc.getText('benchmark')

// Insert one million 'y' characters from right to left
// 从右到左插入一百万个“y”字符
for (let i = 0; i < 1000000; i++) {
    ytext.insert(0, 'y')
}

// transform ydoc to its binary representation
// 将 ydoc 转换为其二进制表示
const encodedDocument = Y.encodeStateAsUpdateV2(ydoc)
const docSize = encodedDocument.byteLength
console.log(`docSize: ${docSize} bytes`) // => 1,000,046 bytes

// Measure time to load the Yjs document containing 1M chars
// 测量加载包含 1M 个字符的 Yjs 文档的时间
const start = Date.now()
const ydoc2 = new Y.Doc()
Y.applyUpdateV2(ydoc2, encodedDocument)

const parseTime = Date.now() - start
console.log(`parseTime: ${parseTime} ms`) // => 368.39 ms
```
对解析包含一百万个字符插入的文档的时间进行基准测试，无需优化。 等效于  [ [B1.3]](https://github.com/dmonad/crdt-benchmarks/#b1-no-conflicts)  ，其中 N=1,000,000

**结果:**

- memUsed: 112 MB
- docSize: 1,000,046 bytes
- parseTime: 368.39 ms

事实证明，最坏的情况并不算太糟糕。 Yjs 不会消耗过多的内存。 考虑到应用于文档的更改量，我想说 112 MB 的总体内存消耗是可以忍受的。 在不到 400 毫秒的时间内解析该大小的文档似乎也不错。 请记住，这绝对是 Yjs 最坏的情况。

我展示了元数据的数量只与产生的变化量有关，与插入的内容量无关。 从一百万次插入的角度来看：键盘压力测试机在每分钟 120 次击键的情况下需要 139 小时才能产生一百万次插入。(  [https://youtu.be/pYXGtxIfprM](https://youtu.be/pYXGtxIfprM)  )


## 检查内存使用情况

JavaScript 对象的工作方式类似于键值映射。 这意味着每个对象都需要跟踪其所有键，并将它们映射到各自的值。  [ C-struct](https://en.wikipedia.org/wiki/Struct_(C_programming_language))   不需要在内存中保留键，只以有效编码的格式保存值。因此，在 JavaScript 中处理大量对象时，自然会有很多恐惧。但是 JavaScript 引擎中的对象表示实际上非常有效。 当你创建许多具有相同结构的对象（它们都具有相同的 key entries）时，JavaScript引擎表示它们的效率几乎与   [ C-struct](https://en.wikipedia.org/wiki/Struct_(C_programming_language))   一样。在 V8/Chrome 中，这种优化被称为   **hidden classes**  。在 SpiderMonkey/Firefox 中，同样的优化被称为   **shapes**  。这种类型的优化实际上比 web 更古老，并且是所有 JavaScript 运行时引擎的一部分。 所以对象表示不是我们需要担心的。


> [V8 如何优化 JavaScript 代码的简要概述](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e) 

 
让我们跳回到最坏的情况并检查每个 Item 到底消耗了多少内存。

> 创建100万个项目时内存检查器的截图
> 
> ![image.png](https://atlas-rc.pingcode.com/files/public/60efe8d0f6d53d1b4b5c51c6/origin-url)


Item 由一些内容(在本例中是 ContentString )和一个 ID 对象组成。而 ID 则由一个不断增加的数字时钟和一个在此场景中不会改变的数字客户端标识符组成。我们只有超过一百万（number）对象，每个 Item 的成本是 88 字节的内存使用量，不包括其内容。你可以通过将 Item、ID 和（number）的“ Shallow Size ”相加并除以 Item 数量来获得此数字【（56000000 + 20000000+12000000）/ 1000000 = 88   **】**  。除了插入字符串的大小外，每个ContentString 对象还占用 16 个字节。
另一个 5.2 Mb 用于仅使用数组索引这些项目。 通常，与创建的项目数量相比，所需的索引信息数量可以忽略不计。

基于 web 的 CRDT 实现的性能与它创建的对象数量直接相关。分析最坏情况下的运行时性能，我们可以观察到 40% 的时间花在执行 V8 内存清理上 (Major & Minor GC)。

> 创建 100 万个项目所花费的时间
>
> ![image.png](https://atlas-rc.pingcode.com/files/public/60efeca6f6d53d798b5c51cb/origin-url)


这是动态编程语言的一大缺点。但是，在下一节中，我们将看到我们的优化在实践中减少了对象创建的数量，因此也显著提高了性能。


## 真实场景

Yjs针对人类输入行为进行了优化。一个很明显的观察是，文本通常是从左到右插入的。尽管我们经常需要纠正拼写错误，但我们倾向于删除整个单词，然后重新开始。Yjs利用了这种行为，并通过在单个 Item 中表示连续的插入来优化批量插入。将一个巨大的文本块复制粘贴到文档中也只会创建一个 Item。此外，删除是自由的，可以减少使用的内存量。正如我们在现实场景中看到的那样，这些优化在实践中产生了巨大的差异。

Martin Kleppmann 分享了他在撰写关于 “  [无冲突复制JSON数据类型](https://arxiv.org/abs/1608.03960)  ” 的长达17页的会议论文时创建的文本操作的  [编辑轨迹](https://github.com/automerge/automerge-perf)  。编辑跟踪包括182,315个单字符插入和77,463个单字符删除。最终的文档包含104852个字符(包括空格)。

在 Yjs 文档上应用编辑跟踪的基准测试结果    [[B4]](https://github.com/dmonad/crdt-benchmarks/#b4-real-world-editing-dataset)   证实了我的预测，即人们通常会产生批量插入：

- memUsed: 19,7 MB
- Item objects created: 10,971
    - 5,799 contain content（  包含的内容）
    - 5,172 are marked as deleted and don't contain any content（标记为已删除且不包含任何内容）
- docSize: 159,927 bytes
- parseTime: 20 ms

一个简单的实现会将 260k 单字符插入/删除的每一个都表示为一个单独的 JavaScript 对象。在Yjs中，完整的文档结构只包含 11k 个 Item 对象。实际使用的内存大约是 2.1 MB。其余的用于 V8 的内部代码优化(下一个基准测试证实了这一点)。

编码后的文档大小约为 160 kB。即使对于慢速网络设备，仅53%的文档大小开销也不会影响网络性能。实际上，与其他解决方案相比，Yjs 可能更有利，因为它允许你在浏览器数据库中存储文档，这样你只需要从网络中提取差异。即使没有管理编辑历史的中央授权，它也可以工作。

在 20 毫秒内解析完整会议论文的编辑轨迹是没问题的。尽管在集中式协同编辑解决方案中解析的开销接近于零，但我认为去中心化的好处超过了在实践中不易察觉的开销。


## 解析大文档

实际场景表明，在处理科学论文等中等大小的文档时，Yjs没有任何显著的开销。但是用 Yjs 写真正的大文档呢？当然，解析如此大的文档所需的时间将呈指数级增长。在本文开始时，我给出了基准测试结果，结果显示创建对象的时间随着已创建对象的数量呈指数级增长。此外，索引项的数据结构应该至少导致时间的对数增长。

基准测试   [[B4 x 100] ](https://github.com/dmonad/crdt-benchmarks/#b4-x-100-real-world-editing-dataset-100-times)  显示，解析文档的时间只随着操作的数量线性增加。我应用了超过 260k 次插入和删除操作的相同的编辑轨迹100次，从而产生了一个巨大的文档。解析此文档的时间仅线性增加（20 ms * 100）。

- memUsed: 220 MB
- docSize: 15,989,245 bytes
- parseTime: 1952 ms
- Item objects: 1,097,100

最终文档包含 10,485,200 个字符（1800 万次插入操作和 800 万次删除操作）。再者，换个角度来看 ：小说《权力的游戏:冰与火之歌》仅包含大约 160 万个字符（开玩笑，没有别的意思）。

以每分钟30个字符的速度计算，一个人需要连续写1.65年才能完成2600万次操作。这甚至都不考虑光标移动会在文档中产生碎片时间。Yjs仅使用220 MB内存就可以轻松处理2600万项更改。

这个基准测试表明，Yjs可以处理非常大的文档，解析文档的时间只是线性增长，而不是我们所怀疑的指数增长。即使在这种情况下，Yjs的性能开销也几乎不明显。从网络中拉取 10 MB 大小的大文档并在浏览器中显示的时间都比使用 Yjs 解析文档花费的时间要长得多。

创建对象时间的指数级增长并没有真正影响Yjs，因为它一开始创建的对象很少。测试证实，你需要应用实际数据集 1000 次，以创建 1000 万个 Item，这些 Item 在解析文档时会额外花费一秒钟的垃圾收集开销。到目前为止，解析文档的时间只会线性增加。

至于索引 Item 对象的数据结构，执行查询的时间确实随着已插入的项目数量呈对数增加。然而，开销是如此微不足道，以至于你无法度量它。我还怀疑 JavaScript 引擎会逐渐优化执行的代码并消除对数开销。在实践中，解析文档的时间只会随着用户交互的数量线性增加。


## 结论：CRDT 是否适合协同编辑？

当然！人类基本上不可能写出Yjs无法处理的文档。我详细的展示了 Yjs 在实际场景中表现非常出色，甚至在没有任何优化可以应用的情况下也表现良好。

Yjs 对性能的权衡是非常有利的。为了换取每次操作的少量内存，Yjs 允许你在任何网络堆栈（甚至是对等）与其他对等方同步文档。当然，它还具有与协同编辑相关的其他特性：

- 选择性撤销 / 重做管理
- 快照并能够恢复到旧文档状态
- 计算版本之间的差异，并由创建更改的用户呈现差异(git blame)
- 同步浏览器数据库的更改以允许离线编辑
- 一种协作感知模型，用于表示光标位置等位置
- 意识功能（目前谁在线，他们在做什么，..）

在我看来，跟我们分析的这些小开销相比，在分布式环境中使用协作感知模型所带来的好处是值得的。

如果你想了解更多关于 Yjs 的信息，请访问   [GitHub](https://github.com/yjs/yjs)   并在   [twitter](https://twitter.com/kevin_jahns)   上关注我，了解最新的发展。


维护 Yjs，关心 GitHub 问题，管理不断增长的社区占用了我大量的空闲时间。如果你想让我做更多的公共工作，比如写博客文章，那么请在 GitHub 上赞助 [我](https://github.com/sponsors/dmonad)  。


在下一篇博文中，我将讨论 CRDT 增加应用程序复杂性的概念。 Yjs 有一个协作感知模型，该模型非常有助于在协作（富）文本文档上实现注释、共享光标、位置标记、状态或建议等功能。 实现像 Yjs 这样的 CRDT 并正确地进行优化并非易事。 我将讨论用于测试分布式系统的方法，这些方法使我对文档总是收敛有很高的信心。


## Notes:

-   [Recent publications](https://arxiv.org/abs/1905.01302)   将 Yjs 描述为一种 CRDT，它执行某种垃圾收集方案来收集 tombstone 以减少文档的大小。由于本文中列出的优化，文档大小只会减小。Yjs不会执行会导致收敛问题的垃圾收集方案。公平地说，我在2016年发表的第一篇文章描述了一个垃圾收集方案，包括它的局限性。上面提到的垃圾收集方案确实能按预期工作，但在默认情况下从未启用，因为正如上面描述的那样，它只在特定的情况下工作，需要更多的工作。垃圾收集方法在2017年被移除，取而代之的是复合表示，以提高性能。这篇文章表明，即使 CRDT 在实践中工作也不需要 tombstone 垃圾收集。

- 经常有人说 CRDT 比   [OT](https://en.wikipedia.org/wiki/Operational_transformation)   快。我之前解释过，学术研究主要以 ops/second 来衡量性能，这并不能真正反映协同编辑应用程序的要求。在实践中，有些 CRDT 不适合共享编辑，因为它们在某些场景中表现不佳。在 parseTime 方面，集中式方法比 CRDT 更具优势。例如，在   [ShareDB](https://github.com/share/sharedb)   中，文档发送到客户端时几乎没有额外的元数据，因此可以更快地解析文档。去中心化方法会发送额外的元数据，这会产生解析开销。但是，本文表明这种开销在 Yjs 中并不重要。


## Edits:

- 2020/12/30 - 更新了关于  [从右到左书写系统](https://en.wikipedia.org/wiki/Right-to-left)  的论点。 以前，我担心 Yjs 在从右到左的书写系统（例如希伯来语和阿拉伯语）中不能很好地工作，因为复合优化仅适用于从左到右书写的文本。 但是一位用户通知我，即使使用的书写系统是从右到左，编辑器也始终按逻辑顺序（从左到右）存储文档内容。

