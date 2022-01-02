原文地址：  [https://josephg.com/blog/crdts-go-brrr/](https://josephg.com/blog/crdts-go-brrr/)   作者：josephg

几年前，我真的被一篇学术论文所困扰。

法国的一些研究人员进行了比较，展示了实现实时协作编辑的多种方式（如 Google Docs）。 他们实现了很多算法——CRDTs 和 OT 算法等等。 他们对它们进行了基准测试，以查看它们的表现。 （酷！！）一些算法运行得相当好。 但其它的算法需要 3 秒以上的时间来处理编辑会话中的简单粘贴操作。 哎呀！

那是什么算法？ 好吧，这很尴尬，但是..这是我的。 我的意思是，它不是我发明的 - 但它是我用于 ShareJS 的算法。 我们用于 Google Wave 的算法。 我知道一个事实（坚持）该算法并没有花费 3 秒来处理大型粘贴事件。那么测试中究竟发生了什么？

我仔细看了他们的论文。 在他们的实现中，当用户粘贴一大块文本（如 1000 个字符）时，他们的代码将插入拆分为 1000 个单独的操作，而不是使用 1000 个字符创建 1 个操作。 并且这些操作中的每一个都需要单独处理。 好吧 - 当然，如果你这样做会很慢！ 这不是操作转换算法的问题。 这只是它们  *特定实现*  的问题。

令人气愤的是，有几个人给我发了这篇论文的链接，并（有针对性地）问我对此有何看法。 写成一篇已发表的科学论文，这些速度比较似乎是关于宇宙的一个事实。 这可能不是真的 - 一些代码的实现细节，可能由一个过度紧张的研究生编写，这可能是他们一堆代码中的一部分。

“不！不是每个人的Review技术都正确！请相信我！”。 但是我没有发表论文来证明我的主张。 我可以很好工作的代码，但感觉没有一个聪明的计算机科学人员关心这个。 我又是谁呢？ 这不重要。

---


即使谈论这些东西，我们也有语言问题。 我们将每个系统描述为一个“算法”。 Jupiter 是一种算法。 RGA 是一种算法。 但实际上有两个非常不同的方面：

1. 并发编辑的黑盒  *行为*  。 当两个客户端同时编辑相同的文本区域时，会发生什么？ 它们是否合并，如果合并，按什么顺序合并？ 都有些什么样的规矩？
1. 系统的白盒  *实现*  。 我们使用什么编程语言？ 什么数据结构？ 代码优化得如何？


如果某些学者的代码运行缓慢，这实际上教会了我们什么？ 也许这就像测试。 代码测试的全部通过，但永远不能  *证明*  没有错误。 同样，缓慢的实现虽然很慢，但永远不能证明系统的每个实现都会很慢。 如果您等待的时间足够长，就会有人发现更多错误。 而且，也许有人可以设计一个更快的实现。

多年前，我将旧文本 OT 代码翻译成 C、Javascript、Go、Rust 和 Swift。 每个实现都具有相同的行为和相同的算法。 但性能甚至不接近。 在 javascript 中，我的转换函数每秒运行大约 100 000 次。 不错！ 但是 C 中的相同函数每秒执行 20M 次迭代。 那快了 200 倍。 不可思议！

学者们是在测试这段代码的慢版本还是快版本？ 也许，在没有注意到的情况下，他们有一些算法的快速版本和其他算法的慢版本。 这些信息从论文上是无法得到的！

## 使 CRDT 变快

如您所知，我最近对 CRDT 产生了兴趣。 对于初学者来说，CRDT（无冲突复制数据类型）是一种花哨的编程工具，可以让多个用户同时编辑相同的数据。 它们让您可以毫无延迟地在本地工作。 （您甚至不必在线）。 当您与其他用户和设备同步时，一切都会神奇地同步并最终保持一致。 CRDT 最好的部分是它们可以完成所有这些工作，甚至不需要云中的中央计算机来监视和控制一切。

我想要没有谷歌的谷歌文档。 我希望我的应用程序能够在我的所有设备之间无缝共享数据，而无需期望所依赖的某些初创公司的服务器在下一个十年仍然存在。 我认为它们是协作编辑的未来。 也许所有软件的未来 - 但我还没有准备好谈论这个。

但是你在学术论文中读到的大多数 CRDT 都非常慢。 十年前，我决定停止阅读学术论文并将其驳回。 我认为 CRDT 有一些固有的问题。 每个字符的 GUID？对位置的东西感觉很疯狂！ 但是——承认这一点很尴尬——我想我和那些研究人员犯了同样的错误。 我正在阅读描述不同系统行为的论文。 我认为这意味着我们知道如何以最佳方式实施这些系统。 哇，我错了。

错到什么程度？ 好。 运行此  [编辑跟踪](https://github.com/automerge/automerge-perf/)  ，  [Automerge](https://github.com/automerge/automerge/)  （一种流行的 CRDT，由一位  [受欢迎的研究人员](https://github.com/automerge/automerge/)  编写）需要近 5 分钟才能运行。 我有一个  [新的实现](https://github.com/josephg/diamond-types)  ，可以在 56 毫秒内处理相同的编辑跟踪。 那是 0.056 秒，快了 5000 多倍。 这是我从优化工作中获得的最大速度提升 - 我对此感到非常高兴。

让我们谈谈为什么 automerge 目前很慢，我将带您完成使其超快的所有步骤。

首先，我们需要从：

### automerge 是什么?

Automerge 是一个帮助您进行协作编辑的库。 它由马丁·克莱普曼 (Martin Kleppmann) 撰写，他的著作和  [出色的演讲](https://martin.kleppmann.com/2020/07/06/crdt-hard-parts-hydra.html)  让他颇有名气。 Automerge 基于一种称为 RGA 的算法，如果您对此感兴趣，可以在学术论文中阅读该算法。

  [https://www.youtube.com/watch?v=x7drE24geUw&t=1237s](https://www.youtube.com/watch?v=x7drE24geUw&t=1237s)  

Automerge（以及 Yjs 和其他 CRDT）将共享文档视为字符列表。 文档中的每个字符都有一个唯一的 ID，每当您插入到文档中时，您都会为要插入的内容命名。

想象下我（seph）在一个空文章中输入“abc”。Automerge 创建 3 个 items：

- 插入 ’  *​a*  ‘ (seph, 0) 在ROOT之后
    - 插入 '  *​b*  ' (seph, 1) 在 (seph, 0) 之后
        - 插入 '​  *​i*  ' (seph, 2) 在 (seph, 1) 之后


![image.png](https://atlas-rc.pingcode.com/files/public/61cae115e429e861587f6008/origin-url)

让我们看看Mike（人名）在 ’  *​a*  ‘ 和 ’  *​b*  ‘ 之间插入一个 'X'，我们得到 ’aXbc‘，我们得到如下：

- 插入 ’  *​a*  ‘ (seph, 0) 在ROOT之后
    - 插入 '  *​X*  ' (mike, 0) 在 (seph, 0) 之后
    - 插入 '  *​b*  ' (seph, 1) 在 (seph, 0) 之后
        - 插入 '​  *​i*  ' (seph, 2) 在 (seph, 1) 之后


![image.png](https://atlas-rc.pingcode.com/files/public/61cae223e429e861587f600b/origin-url)

请注意“X”和“b”都共享同一个父级。 当用户在文档中的同一位置同时键入时，就会发生这种情况。 但是我们如何确定哪个字符先出现呢？ 我们可以只使用他们的代理 ID 或其他东西进行排序。 但是啊，如果我们这样做，文档最终可能会变成 abcX，即使 Mike 在 b 之前插入了 X。 那真的会很混乱。

Automerge (RGA) 通过巧妙的技巧解决了这个问题。 它为每个Item添加一个额外的整数，称为  *序列号*  。 每当你插入一些东西时，你将Item的序列号设置为比你见过的最大序列号大 1：

- 插入 ’  *​a*  ‘ (seph, 0) 在ROOT之后   *seq: 0*
    - 插入 '  *​X*  ' (mike, 0) 在 (seph, 0) 之后   *seq: 3*
    - 插入 '  *​b*  ' (seph, 1) 在 (seph, 0) 之后   *seq: 1*
        - 插入 '​  *​i*  ' (seph, 2) 在 (seph, 1) 之后   *seq: 2*


这是一个算法版本 “哇，我看到一个序列号，竟然这么大！”。 “是吗？我的更大！”

规则是子项首先根据它们的序列号排序（更大的序列号在前）。 如果序列号匹配，则更改必须是并发的。 在这种情况下，我们可以根据代理 ID 对它们进行任意排序。 （我们这样做是为了让所有对等点最终得到相同的结果文档。）

Yjs - 我们稍后会看到更多 - 实现了一个名为 YATA 的 CRDT。 YATA 与 RGA 相同，只是它通过稍微不同的 hack 解决了这个问题。 但这里的区别并不重要。

Automerge (RGA) 的行为由该算法定义：

- 构建树，将每个Item连接到其父项
- 当一个项目有多个子项时，先按序列号再按 ID 对它们进行排序。
- 结果列表（或文本文档）通过使用深度优先遍历把树打平制作。


那么你应该如何  *实现*   automerge 呢？ automerge 库以一种显而易见的方式完成它，即将所有数据存储为一棵树。 （至少我是这么认为的 - 在输入“abc”之后，这是 automerge 的  [内部状态](https://gist.github.com/josephg/0522c4aec5021cc1dddb60e778828dbe)  。呃，呃，我不知道这里发生了什么。所有这些 Uint8Arrays 都在做什么？无论如何，automerge 库有效，通过这些Items构建一个Tree。

对于一个简单的基准测试，我将使用 Martin 自己制作的  [编辑跟踪](https://github.com/automerge/automerge-perf/)  来测试自动合并。 这是马丁输入学术论文的逐字记录。 此跟踪中没有任何并发编辑，但用户几乎从未真正将光标放在完全相同的位置然后输入，所以我不太担心这一点。 我也只计算在  *本地*  应用此跟踪所需的时间，虽然这并不理想，但还可以（代表是正确的）。 如果您喜欢这类事情，Kevin Jahns（Yjs 的作者）在这里有一个更广泛的  [基准测试](https://github.com/dmonad/crdt-benchmarks)  。 这里的所有基准测试都是在我的 chonky ryzen 5800x 工作站上完成的，在合适的时候使用 Nodejs v16.1 和 rust 1.52。 （剧透！）

编辑轨迹有 260 000 次编辑，最终文档大小约为 100 000 个字符。

正如我上面所说，automerge 需要不到 5 分钟的时间来处理此跟踪。 这只是每秒 900 次编辑，这可能没问题。 但是当它完成时，automerge 正在使用 880 MB 的 RAM。 哇！ 这是每次按键 10kb 的内存。 在高峰期，automerge 使用了 2.6 GB 的 RAM！

为了了解有多少开销，我将把它与  [基准](https://gist.github.com/josephg/13efc1444660c07870fcbd0b3e917638#file-js_baseline-js)  进行比较，在基准中我们只是将所有编辑直接拼接到一个 javascript 字符串中。 这丢弃了我们进行协作编辑所需的所有信息，但它让我们了解 javascript 的运行速度。 事实证明，在 V8 上运行的 javascript 速度很快：

|Test|Time taken|RAM usage|
|---|---|---|
|**automerge (v1.0.0-preview2)**|291s|880 MB|
|*Plain string edits in JS*|0.61s|0.1 MB|


这是一个图表，显示了在整个测试过程中处理每个操作所花费的时间，平均每组 1000 个操作。 我认为这些峰值是 V8 的垃圾收集器试图释放内存。

![image.png](https://atlas-rc.pingcode.com/files/public/61caef34e429e861587f601e/origin-url)

在接近尾声的最慢峰值中，一次编辑需要 1.8 秒的处理时间。 糟糕。 在实际的应用程序中，整个应用程序（或浏览器选项卡）有时会在您打字的过程中冻结几秒钟。    

当我们将所有内容平均并缩放 Y 轴时，图表更易于阅读。 我们可以看到平均性能随着时间的推移逐渐（大致线性地）变差。

![image.png](https://atlas-rc.pingcode.com/files/public/61cc1ac3e429e861587f606e/origin-url)

### 为什么 automerge 这么慢?

Automerge 运行缓慢的原因有很多：

1. 随着文档的增长，Automerge 的基于核心树的数据结构变得又大又慢。
1. Automerge 大量使用   [Immutablejs](https://immutable-js.github.io/)  。 Immutablejs 是一个库，它为 javascript 对象提供类似 clojure 的 copy-on-write 语义。 这是一组很酷的功能，但 V8 优化器和 GC 努力优化使用 immutablejs 的代码。 因此，它会增加内存使用量并降低性能。
1. Automerge 将每个插入的字符视为一个单独的Item。 还记得我之前讲过的论文，复制+粘贴操作很慢吗？ Automerge 也是如此！


Automerge 从来没有考虑过性能。 他们的团队正在研究算法的替代   [Rust 实现](https://github.com/automerge/automerge-rs/)  来运行 wasm，但在撰写本文时它尚未落地。 我想让 master 分支正常工作，但是在它准备好之前他们有一些问题需要解决。 切换到 automerge-rs 后端也不会使此测试中的平均性能更快。 （尽管它确实将内存使用量减半并使性能更平滑。）

---


> 你不能让电脑更快。 你只能让它做更少的工作。

我们如何让计算机在这里做更少的工作？ 通过检查代码和改进许多小东西，可以获得很多性能上的胜利。 但是 automerge 团队有正确的方法，最好从整体优化开始，在转向优化单个方法之前修复核心算法和数据结构。优化的函数很可能在优化核心算法和数据结构时被删除掉，那么这样的小的优化就没有意义了。

到目前为止，Automerge 最大的问题是其复杂的基于树的数据结构。 我们可以用更快的方式替换它。

## 改进数据结构

幸运的是，有一种更好的方法来实现 CRDTs，这是   [Yjs](https://github.com/yjs/yjs)   中首创的。 Yjs 是由 Kevin Jahns 制作的另一个（竞争）开源 CRDT 实现。 它很快，文档完善，生态完善。如果我今天要构建支持协作编辑的软件，我会使用 Yjs。

Yjs 不需要整篇博文来讨论如何使其更快，因为它已经非常快了，我们很快就会看到。 它通过使用一个聪明的、明显的数据结构“技巧”实现，我认为该领域的其他人没有想到它。 而不是像 automerge 那样将 CRDT 实现为树：

```
state = {
  { item: 'a', id: ['seph', 0], seq: 0, children: [
    { item: 'X', id, seq, children: []},
    { item: 'b', id, seq, children: [
      { item: 'c', id, seq, children: []}
    ]}
  ]}
}
```

Yjs 只是将所有Item放在一个单一的列表中：

```
state = [
  { item: 'a', id: ['seph', 0], seq: 0, parent: null },
  { item: 'X', id, seq, parent: ['seph', 0] },
  { item: 'b', id, seq, parent: ['seph', 0] },
  { item: 'c', id, seq, parent: [..] }
]
```

这看起来很简单，但是如何在列表中插入一个新项目呢？ 使用 automerge 很容易：

1. 找到 Parent Item
1. 将New Item 插入到 Parent Item 的 Children 中的正确位置


但是实现上面的目标会很复杂：

1. 找到 Parent Item
1. 从 Parent Item 之后开始，遍历列表直到我们找到应该插入新项的位置（？）
1. 插入那里，拼接成数组


本质上，这种方法只是一种花哨的插入排序。 我们正在使用列表实现列表 CRDT。 天才！

这听起来很复杂 - 你是如何确定 New Item 的位置的？但它的实现复杂程度和它的数据复杂度一样。 很难理解，但是一旦理解了，大约20行代码就可以实现整个插入功能：

（但如果这看起来令人困惑，请不要惊慌 - 我们可能可以让地球上今天理解此代码的每个人都进入一个小会议室。）

```
const automergeInsert = (doc, newItem) => {
  const parentIdx = findItem(doc, newItem.parent) // (1)

  // Scan to find the insert location
  let i
  for (i = parentIdx + 1; i < doc.content.length; i++) {
    let o = doc.content[i]
    if (newItem.seq > o.seq) break // Optimization.
    let oparentIdx = findItem(doc, o.parent)

    // Should we insert here? (Warning: Black magic part)
    if (oparentIdx < parentIdx
      || (oparentIdx === parentIdx
        && newItem.seq === o.seq
        && newItem.id[0] < o.id[0])
    ) break
  }
  // We've found the position. Insert at position *i*.
  doc.content.splice(i, 0, newItem) // (2)

  // .. And do various bookkeeping.
}
```

我在我的实验性   [*reference-crdts*](https://github.com/josephg/reference-crdts/blob/main/crdts.ts)   代码库中使用这种方法实现了 Yjs 的 CRDT (YATA) 和 Automerge。   [这是插入功能，还有一些注释](https://github.com/josephg/reference-crdts/blob/fed747255df9d457e11f36575de555b39f07e909/crdts.ts#L401-L459)  。 这个函数的 Yjs 版本在同一个文件中，如果你想看看。 尽管是来自不同的论文，但插入的逻辑几乎相同。 尽管代码不尽相同，但这种方法在语义上与实际的 automerge、Yjs 和 sync9 代码库相同。 （  [Fuzzer verified (TM)](https://github.com/josephg/reference-crdts/blob/main/reference_test.ts)  ）。

如果你有兴趣更深入地了解这一点，我在几周前的一次  [braid](https://braid.org/)  会议上  [讨论了这种方法](https://invisiblecollege.s3-us-west-1.amazonaws.com/braid-meeting-10.mp4#t=300)  。

重要的一点是这种方法有一下更多好处：

1. 我们可以使用平面数组来存储所有内容，而不是不平衡的树。 这使得计算机处理的一切都变得更小、更快。
1. 代码真的很简单。 更快更简单移动  [Pareto efficiency frontier](https://en.wikipedia.org/wiki/Pareto_efficiency)  。 这样做的想法是罕见的，而且是真正的黄金。
1. 您可以像这样实现很多 CRDT。 Yjs、Automerge、Sync9 和其他工作。 您可以在同一个代码库中实现许多列表 CRDT。 在我的 reference-crdts 代码库中，我同时实现了 RGA（automerge）和 YATA（Yjs）。 他们共享他们的大部分代码（除了这个函数之外的所有代码）并且他们在这个测试中的表现是相同的。


理论上，当在文档中的同一位置存在并发插入时，该算法会减慢速度。 但这在实践中真的很少见 - 您几乎总是在父项之后插入。

使用这种方法，我对 automerge 算法的实现比真正的 automerge 快了大约 10 倍。 它的内存效率提高了 30 倍：

|Test|Time taken|RAM usage|
|---|---|---|
|automerge (v1.0.0-preview2)|291s|880 MB|
|**reference-crdts (automerge / Yjs)**|31s|28 MB|
|*Plain string edits in JS*|0.61s|0.1 MB|


我希望我能将  *所有*  这些差异归因于这个甜美而简单的数据结构。 但是这里的很多区别可能只是 immutablejs 对 automerge 进行了处理。

它比 automerge 快得多：

![image.png](https://atlas-rc.pingcode.com/files/public/61cc2dd0e429e861587f608b/origin-url)

## 1000次扫描导致死机

我们现在正在使用干净且快速的核心数据抽象，但实现仍然  *不快*  。 我们需要修复此代码库中的两大性能瓶颈：

1. 找到要插入的位置，以及
1. 实际把它插入到数组中


（这些行为在上面的代码块中标记为   *(1)*   和   *(2)*  ）。

为了理解为什么这个代码是必要的，假设我们有一个文档，它是一个项目列表。

```
state = [
  { item: 'a', isDeleted: false, id: ['seph', 0], seq, parent: null },
  { item: 'X', isDeleted: false, id, seq, parent: ['seph', 0] },
  { item: 'b', isDeleted: true,  id, seq, parent: ['seph', 0] },
  { item: 'c', isDeleted: false, id, seq, parent: ['seph', 1] },
  ...
]
```

其中一些项目可能已被删除。 我添加了一个 isDeleted 标志来标记哪些。 （不幸的是，我们不能将它们从数组中删除，因为其他插入可能依赖于它们。见鬼! 但这是其它时候需要讨论的问题。）

想象一下，文档中有 150 000 个数组项，代表 100 000 个尚未删除的字符。 如果用户在文档中间（文档位置 50 000）键入一个 'a'，那么它对应于我们数组中的什么索引？ 为了找出答案，我们需要扫描文档（跳过已删除的项目）以找出正确的数组位置。

因此，如果用户在位置 50 000 插入，我们可能必须线性扫描超过 75 000 个项目或其他东西才能找到插入位置。 哎呀！

然后当我们实际插入时，代码会这样做，这是双重的：

```
doc.content.splice(destIdx, 0, newItem)
```

如果数组当前有 150 000 个项目，javascript 将需要将 newItem 之后的每个 Item 向后移动一次。 这部分发生在本机代码中，但是当我们移动这么多 Items 时它可能仍然很慢。 （旁白：V8 在这方面的速度实际上令人怀疑，所以也许 v8 没有在内部使用数组来实现数组？谁知道！）

但一般来说，将一个 item 插入到一个包含   *n*   个 items 的文档中大约需要   *n*   个步骤。 等等，不 - 比这更糟糕，因为删除的项目仍然存在。 插入到曾经有   *n*   个 items 的文档需要   *n*   个步骤。 这种算法相当快，但每次输入都会变慢。 插入   *n*   个字符将花费   *O(n^2)*  。

如果我们放大上图，您可以看到这一点。 这里发生了很多事情，因为 Martin 的编辑位置在文档周围跳来跳去。 但是向右上方有很强的线性趋势，这就是我们所期望的在插入时花费的时间是   *O(n)*   的：

![image.png](https://atlas-rc.pingcode.com/files/public/61cdc2d2e4058eb01f92e7e1/origin-url)

为什么特别是这种形状？ 为什么性能在接近尾声时变得更好？ 如果我们简单地绘制在整个编辑轨迹中每个编辑发生的  *位置*  ，使用相同的水平和垂直刻度，结果是一条非常熟悉的曲线：

![image.png](https://atlas-rc.pingcode.com/files/public/61cdc40fe4058eb01f92e7e2/origin-url)

看起来应用更改所花费的时间主要取决于扫描文档数组所花费的时间。

## 改变数据结构

我们能解决这个问题吗？ 我们可以！ “我们”是指 Kevin 在 Yjs 中解决了这些问题。 他是怎么做到的？

所以请记住，有两个问题需要解决：

1. 我们如何找到特定的插入位置？
1. 我们如何高效地在该位置插入内容？


Kevin 通过思考人类实际上如何编辑文本文档来解决第一个问题。 通常在我们打字的时候，我们实际上并不会在一个文档周围跳来跳去。 Yjs 不会在每次编辑时扫描文档，而是缓存用户进行编辑的最后一个（索引、位置）对。 下一个编辑可能与上一个编辑非常接近，因此 Kevin 只需从上一个编辑位置向前或向后扫描。 这对我来说听起来有点狡猾 - 我的意思是，这是一个很大的假设！ 如果编辑随机发生怎么办？！ 但人们实际上并不会随意编辑文档，因此它在实践中效果很好。

（如果两个用户同时编辑文档的不同部分怎么办？Yjs 实际上存储了一整套缓存位置，因此无论他们在文档中的哪个位置进行更改，每个用户附近几乎总是有一个缓存的光标位置。 )

Yjs一旦找到目标插入位置，就需要高效插入，而不是复制所有现有的项目。 Yjs 通过使用双向链表而不是数组来解决这个问题。 只要我们有一个插入位置，链表就允许在恒定时间内插入。

Yjs 还做了一件事来提高性能。 人类通常会输入一系列字符。 因此，当我们在文档中输入“hello”时，不是存储：

```
state = [
  { item: 'h', isDeleted: false, id: ['seph', 0], seq, parent: null },
  { item: 'e', isDeleted: false, id: ['seph', 1], seq, parent: ['seph', 0] },
  { item: 'l', isDeleted: false, id: ['seph', 2], seq, parent: ['seph', 1] },
  { item: 'l', isDeleted: false, id: ['seph', 3], seq, parent: ['seph', 2] },
  { item: 'o', isDeleted: false, id: ['seph', 4], seq, parent: ['seph', 3] },
]
```

Yjs 仅仅存储：

```
state = [
  { item: 'hello', isDeleted: false, id: ['seph', 0], seq, parent: null },
]
```

最后那些讨厌的粘贴事件也会很快！

这是相同的信息，只是存储得更紧凑。 不幸的是，我们无法使用此技巧将整个文档折叠为单个项目或类似内容。 该算法只能在 ID 和父项按顺序排列时折叠插入 - 但每当用户键入一串字符而不移动光标时就会发生这种情况。 这种情况经常发生。

在这个数据集中，使用跨度换算将数组条目的数量减少了 14 倍。 （180k 条目下降到 12k）。

现在有多快？ 这让我大吃一惊——在这个测试中，Yjs 比我的 reference-crdts 实现快 30 倍。 而且它只使用大约 10% 的 RAM。 它比 automerge 快 300 倍！

|Test|Time taken|RAM usage|
|---|---|---|
|automerge (v1.0.0-preview2)|291s|880 MB|
|reference-crdts (automerge / Yjs)|31s|28 MB|
|**Yjs (v13.5.5)**|0.97s|3.3 MB|
|*Plain string edits in JS*|0.61s|0.1 MB|


老实说，我对这次测试中使用的 ram Yjs 的使用量感到震惊并有点怀疑。 我确信 V8 中有一些魔法使这成为可能。 这是非常令人印象深刻的。

Kevin 说他编写并重写了 Yjs 的部分内容 12 次，以使这段代码运行得如此之快。 如果有程序员版的跑速社区，他们会喜欢 Kevin。 我甚至不能把 Yjs 放在与其他算法相同的规模上，因为它太快了：

![image.png](https://atlas-rc.pingcode.com/files/public/61cdc7d8e4058eb01f92e7e4/origin-url)

如果我们隔离 Yjs，您可以看到它的性能基本持平。 与其他算法不同，它不会随着文档的增长而变慢：

![image.png](https://atlas-rc.pingcode.com/files/public/61cdc801e4058eb01f92e7e5/origin-url)

但我不知道接近尾声的那些尖峰是什么。 从绝对值来看，它们很小，但仍然很奇怪！ 也许当用户在文档周围移动光标时会发生这种情况？ 或者当用户删除块时？ 我不知道。

这很好，但真正的问题是：我们能走得更快吗？ 老实说，我怀疑我能否让纯 javascript 比 Kevin 在这里管理的更快地运行此测试。 但也许……只是也许我们可以……

## 比Javascript更快

当我告诉 Kevin 我认为我可以做出比 Yjs 快得多的 CRDT 实现时，他不相信我。 他说 Yjs 已经优化得非常好，可能不可能再快得多。 “如果你只是把它移植到 Rust，也许会快一点。但不会快很多！现在 V8 真的很快！！”

但我知道一些 Kevin 不知道的事情：我知道内存碎片和缓存。 Rust 不仅仅是  *更快*  。 它也是一种低级语言，它为我们提供了控制分配和内存布局所需的工具。

> Kevin 现在也知道这一点，他正在努力应用到  [Yrs](https://github.com/yjs/y-crdt)  上，看看他是否能夺回表现桂冠。

想象一下我们在 javascript 中的文档项之一：

```
var item = {
  content: 'hello',
  isDeleted: false,
  id: ['seph', 10],
  seq: 5,
  parent: ['mike', 2]
}
```

这个对象在内存中实际上是这样的：

![image.png](https://atlas-rc.pingcode.com/files/public/61cdc9b3e4058eb01f92e7e6/origin-url)

坏消息：  *你的电脑讨厌这个*  。

这很糟糕，因为所有数据都是碎片化的。 都是用指针隔开的。

> 是的，我知道，V8 在可能的情况下尽最大努力防止此类事情发生。 但它不是魔法。

像这样排列数据，计算机必须为每一项一项地分配内存。 这很慢。 然后垃圾收集器需要额外的数据来跟踪所有这些对象，这也很慢。 稍后我们需要读取该数据。 要读取它，您的计算机通常需要从主内存中获取它，这 - 您猜对了 - 也很慢。

主存读取有多慢？   [At human scale](https://gist.github.com/hellerbarde/2843375)  ，每次 L1 缓存读取需要 0.5 秒。 从主内存读取需要接近 2 分钟！ 这是单次心跳与刷牙所需时间之间的差异。

像 javascript 一样安排内存就像编写购物清单一样。 但不是“奶酪、牛奶、面包”，你的清单实际上是一个寻宝游戏：“沙发底下”、“冰箱顶上”等等。 沙发底下有一张小纸条，上面写着你需要牙膏。 毋庸置疑，这使得去杂货店购物需要做很多工作。

为了更快，我们需要将所有数据压缩在一起，以便计算机可以在每次读取主内存时获取更多信息。 （我们想要一次阅读我的购物清单来告诉我们我们需要知道的一切）。 正是因为这个原因，  **链表在现实世界中很少使用——内存碎片会破坏性能**  。 我也想摆脱链接列表，因为用户有时会在文档周围跳来跳去，这在 Yjs 中具有线性性能成本。 这在文本编辑中可能没什么大不了的，但我希望这段代码在其他用例中也能很快。 我不希望程序需要那些缓慢的扫描。

我们无法在 javascript 中解决这个问题。 javascript 中奇特的数据结构的问题是你最终需要大量的奇异对象（比如固定大小的数组）。 所有这些额外的对象都会使碎片变得更糟，因此由于您的所有工作，您的程序最终通常会运行得更慢。 这与 immutablejs 有相同的限制，也是为什么它的性能在发布后的十年中没有太大提高。 V8 优化器非常聪明，但它不是魔术，聪明的技巧只能让我们到此为止。

但我们不仅限于 javascript。 即使在制作网页时，我们现在也有 WebAssembly。 我们可以将其编码为  *任何内容*  。

为了看看我们到底能走多快，我一直在悄悄地用 Rust 构建一个名为   [Diamond](https://github.com/josephg/diamond-types)   类型的 CRDT 实现。 Diamond 与 Yjs 几乎相同，但它在内部使用  [range tree](https://en.wikipedia.org/wiki/Range_tree)  而不是链表来存储所有项目。

在引擎下，我的 range tree 只是一个稍微修改过的 b-tree。 但通常当人们谈论 b-trees 时，他们指的是   [BTreeMap](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html)  。 那不是我在这里做的。 b-tree 的每个内部节点不存储键，而是存储该 item 的 children 中的字符总数（递归）。 因此我们可以通过字符位置查找文档中的任何 item，或者在   *log(n)*   时间内在文档中的任何位置插入或删除。

此示例显示了存储当前具有 1000 个字符的文档的树：

![image.png](https://atlas-rc.pingcode.com/files/public/61cdcd39e4058eb01f92e7e7/origin-url)

> *这是一个range tree，对吧？ 关于*  [*wikipedia article on range trees*](https://en.wikipedia.org/wiki/Range_tree)  *对我在这里所做的事情的描述非常薄弱。*

这解决了我们之前的线性扫描问题：

1. 当我们想在位置 200 处找到项目时，我们可以遍历树并向下遍历。 在上面的示例中，位置为 350 的项目必须在此处的中间叶节点中。 树非常整洁 - 我们可以在树中仅将 Martin 的编辑跟踪存储在 3 个级别中，这意味着在此基准测试中，我们可以在大约 3 次读取中从主内存中找到任何项目。 实际上，这些读取中的大部分已经在您的 CPU 缓存中。
1. 更新树也很快。 我们更新一个叶子，然后更新它的父级和它的父级的字符数，一直到根。 同样，经过 3 个左右的步骤后，我们就完成了。 比在 javascript 数组中打乱所有内容要好得多。


在这个测试中，我们从不合并来自远程对等方的编辑，但无论如何我也做得很快。 合并远程编辑时，我们还需要通过 ID 查找项目（  *例如 ['seph', 100]*  ）。 Diamond 几乎没有索引可以通过 ID 搜索 b-tree。 不过，该代码路径并未在此处进行基准测试。 它很快，但现在你必须相信我的话。

我没有使用 Yjs 缓存最后一个编辑位置的技巧——至少现在还没有。 它可能会有所帮助。 我只是还没试过。

Rust 使我们可以完全控制内存布局，因此我们可以将所有内容都紧密地打包。 与图中不同，我的 b-tree 中的每个叶节点都存储了一个包含 32 个条目的块，这些条目打包在内存中的固定大小数组中。 用这样的结构插入会导致一些 memcpy-ing，但是一点 memcpy 就可以了。 Memcpy 总是比我想象的要快 - CPU 每个时钟周期可以复制几个字节。 它不是对主内存查找的史诗般的追捕。

为什么是 32 个条目？ 我用一堆不同的块大小运行了这个基准测试，32 个运行良好。 我不知道为什么结果是最好的。

说到快，到底有多快？

如果我们将  [此代码编译为 webassembly](https://github.com/josephg/diamond-js)   并像在其他测试中一样从 javascript 驱动它，我们现在可以在 193 毫秒内处理整个编辑跟踪。 这比 Yjs 快 5 倍。 尽管做了所有支持协作编辑的工作，但编辑原生 javascript 字符串的速度比我们的基线测试快了 3 倍！

Javascript 和 WASM 现在是一个瓶颈。 如果我们跳过 javascript 并  [直接在 rust](https://github.com/josephg/diamond-types/blob/42a8bc8fb4d44671147ccaf341eee18d77b2d532/benches/yjs.rs)   中运行基准测试，我们可以在短短 56 毫秒内处理此编辑跟踪中的所有 260k 编辑。 这比我们开始使用 automerge 时快 5000 倍以上。 它每秒可以处理 460 万次操作。

|Test|Time taken|RAM usage|
|---|---|---|
|automerge (v1.0.0-preview2)|291s|880 MB|
|reference-crdts (automerge / Yjs)|31s|28 MB|
|Yjs (v13.5.5)|0.97s|3.3 MB|
|*Plain string edits in JS*|0.61s|0.1 MB|
|**Diamond (wasm via nodejs)**|0.19s|???|
|**Diamond (native)**|0.056s|1.1 MB|


性能如黄油般顺滑。 b-tree 不关心编辑发生的位置。 该系统在整个文档中速度一致。 Rust 不需要垃圾收集器来跟踪内存分配，因此没有神秘的 GC 峰值。 由于内存非常紧凑，因此处理整个数据集（全部 260 000 个）只会导致对 malloc 的 1394 次调用。

![image.png](https://atlas-rc.pingcode.com/files/public/61cdd013e4058eb01f92e7e8/origin-url)

噢真可惜。 它太快了，你几乎看不到它旁边的 yjs (fleexxxx)。 让我们放大一点，看看那条平线：

![image.png](https://atlas-rc.pingcode.com/files/public/61cdd03ae4058eb01f92e7e9/origin-url)

嗯，几乎是一条直线。

请记住，此图表显示的是慢速版本。 该图表由 javascript 生成，通过 WASM 调用 Rust。 如果我在本地运行这个基准测试，它又快了大约 4 倍。

为什么 WASM 比本地执行慢 4 倍？ 对 WASM VM 的 javascript 调用真的那么慢吗？ LLVM 是否更好地优化了原生 x86 代码？ 还是 WASM 的内存边界检查会减慢它的速度？ 我很好奇！

## 数组结构还是结构数组？

这个实现还有另一个小的、重要的变化——我不确定我是否喜欢它。

在 Rust 中，我实际上是在做这样的事情：

```
doc = {
  textContent: RopeyRope { 'hello' },

  clients: ['seph', 'mike'],

  items: BTree {[
    // Note: No string content!
    { len:  5, id: [0, 0], seq, parent: ROOT },
    { len: -5, id: [1, 0], seq, parent: [0, 0] }, // negative len means the content was deleted
    ...
  ]},
}
```

请注意，文档的文本内容不再存在于项目列表中。 现在它在一个单独的数据结构中。 我为此使用了一个名为   [Ropey](https://crates.io/crates/ropey)   的 Rust 库。 Ropey 实现了另一个 b 树来有效地管理文档的文本内容。

这不是普遍的胜利。 不幸的是，我们来到了令人不安的工程权衡之地：

1. Ropey 可以进行文本特定的字节打包。 所以使用ropey，我们使用更少的RAM。
1. 插入时，我们需要更新 2 个数据结构而不是 1 个。这使得一切都慢了两倍多，并且使 wasm 包的大小增加了一倍（60kb -> 120kb）。
1. 对于许多用例，无论如何我们最终都会将文档内容存储在其他地方。 例如，如果您将此 CRDT 与 VS Code 挂钩，则编辑器将始终保留该文档的副本。 因此，根本不需要将文档存储在我的 CRDT 结构中。 这种实现方法可以很容易地关闭那部分代码。


所以我仍然不确定我是否喜欢这种方法。

但无论如何，我的 CRDT 实现在这一点上是如此之快，以至于算法的大部分时间都花在了更新 ropey 中的文档内容上。 Ropey 本身需要 29 毫秒来处理此编辑跟踪。 如果我只是......关闭ropey会怎样？ 这只小狗到底能跑多快？

|Test|Time taken|RAM usage|Data structure|
|---|---|---|---|
|automerge (v1.0.0-preview2)|291s|880 MB|Naive tree|
|reference-crdts (automerge / Yjs)|31s|28 MB|Array|
|Yjs (v13.5.5)|0.97s|3.3 MB|Linked list|
|*Plain string edits in JS*|0.61s|0.1 MB|*(none)*|
|Diamond (wasm via nodejs)|0.20s|???|B-Tree|
|Diamond (native)|0.056s|1.1 MB|B-Tree|
|*Ropey (rust) baseline*|0.029s|0.2 MB|*(none)*|
|**Diamond (native, no doc content)**|0.023s|0.96 MB|B-Tree|


Boom。 这虽然有点没用，但现在比 automerge 快 14000 倍。 我们在 23 毫秒内处理了 26 万次操作。 那是每秒 1100 万次操作。 我可以通过按键使我的家庭互联网连接饱和，而且我还有 CPU 可用。

---


我们可以计算每个算法处理编辑的平均速度：

![image.png](https://atlas-rc.pingcode.com/files/public/61cdd28de4058eb01f92e7ea/origin-url)

但这些数字具有误导性。 请记住，automerge 和 ref-crdts 并不稳定。 它们一开始很快，然后随着文档的增长而变慢。 尽管 automerge 平均每秒可以处理大约 900 次编辑（这速度快到用户不会注意到），但在此基准测试期间最慢的编辑使 V8 停滞了整整 1.8 秒。

如果我使用对数刻度，我们可以将所有内容放在一个漂亮的图表中。 这看起来非常整洁：

![image.png](https://atlas-rc.pingcode.com/files/public/61cdd2f8e4058eb01f92e7eb/origin-url)

呵呵 - 看看底部的两行。 yjs 和 diamond 的抖动相互映衬。 yjs 变慢的时期，diamond 变快。 我想知道那里发生了什么！

但对你的直觉来说，对数可读是垃圾食品。 在线性尺度上，数据如下所示：

![image.png](https://atlas-rc.pingcode.com/files/public/61cdd368e4058eb01f92e7ec/origin-url)

我的朋友们，这就是让计算机少做很多工作的方法。



## 结论

多年前我读过的那篇愚蠢的学术论文说一些 CRDTs 和 OT 算法很慢。 每个人都相信这篇论文，因为它是已发表的科学。 但是论文是错误的。 正如我所展示的，我们  *可以*  使 CRDsT 变快。 如果我们对我们的实施策略发挥创意，我们  *可以*  让他们快的疯狂。 通过正确的方法，我们可以使 CRDT 变得如此之快，以至于我们可以与原生字符串的性能竞争。 那篇论文中的表现数字不仅仅是错误的。 他们是“一位亿万富翁，猜测一根香蕉值 1000 美元”，这有点错误。

但你知道吗？ 我现在有点欣赏那篇论文。 他们的错误是可以的。 这是人类。 我曾经在学术上感到不足 - 也许我永远不会那么聪明！ 但这整件事让我意识到一件显而易见的事情：科学家不是神，从天而降，带着真理的礼物。 不，他们是美丽的、有缺陷的人，就像我们其他人一样。 擅长我们所痴迷的一切，但在其他任何地方都处于中等水平。 我可以很好地优化代码，但我仍然把西葫芦和黄瓜搞混了。 而且，无论我从朋友那里得到什么戏弄，那都可以。

十年前，Google Wave 确实需要一个高质量的列表 CRDT。 当 CRDT 的论文开始出现时，我感到非常兴奋。   [LOGOOT](https://hal.inria.fr/inria-00432368/document)   和   [WOOT](https://hal.inria.fr/inria-00445975/document)   看起来很重要！ 但是当我意识到算法太慢且效率低下而无法实际使用时，那种兴奋就消失了。 我犯了一个大错误——我认为如果学者们不能让他们快速完成，那么没有人能做到。

但有时最好的作品来自具有不同技能的人之间的合作。 我不擅长学术论文，我很擅长让代码运行得很快。 然而，在我自己的领域，我甚至没有尝试提供帮助。 研究人员正在尽自己的一份力量使 P2P 协作编辑工作。 我只是对他们嗤之以鼻，继续致力于运营转型。 如果我提供帮助，也许十年前我们会有快速、可行的 CRDT 用于文本编辑。 哎呀！ 结果证明协作编辑需要我们所有人的协作。 真讽刺！ 谁能猜到？！

嗯，这花了十年时间，来自一群聪明人的一些辛勤工作和一些伟大的想法。 Martin 为 Automerge 发明的二进制编码系统非常出色。 通过使用递增（agent id、sequence）元组来避免 UUID 的系统是天才。 我不知道是谁提出的，但我喜欢它。 当然，我在此描述的 Kevin 的列表表示 + 插入方法使一切变得更快、更简单。 我敢打赌，在过去十年中，一定有 100 名聪明人在没有任何人提到的情况下从这个想法中走出来。 我怀疑我也不会想到它。 我的贡献是使用运行长度编码的 b-trees 和巧妙的索引。 并且展示 Kevin 的快速列表表示可以适用于任何 CRDT 算法。 我认为之前没有人注意到这一点。

现在，经过十年的等待，我们终于找到了如何制作快速、轻量级的列表 CRDT 实现。 实用的去中心化实时协同编辑？ 我们下一次来找你。

## 附录 A: 我想为我的应用程序使用 CRDT。 我该怎么办？

如果您现在正在构建基于文档的协作应用程序，则应该使用 Yjs。 Yjs 具有稳定的性能、低内存使用和强大的支持。 如果您需要帮助在您的应用程序中实现 Yjs，Kevin Jahns 有时会接受金钱以换取帮助将 Yjs 集成到各种应用程序中。 他用它来资助全职从事 Yjs（和相关）的工作。 Yjs 已经运行得很快，很快它就会变得更快。

automerge 团队也很棒。 我和他们就这些问题进行了一些很好的对话。 他们将性能作为 2021 年的第一期，并计划使用许多这些技巧来加快自动合并。 当您阅读本文时，它可能已经快得多了。

Diamond 真的很快，但在我与 Yjs 和 Automerge 具有同等功能之前还有很多工作要做。 一个好的 CRDT 库除了操作速度之外还有很多。 CRDT 库还需要支持二进制编码、网络协议、非列表数据结构、presence（光标位置）、编辑器绑定等。 在撰写本文时，Diamon 几乎没有做这些。

如果你想要数据库语义而不是文档语义，据我所知，还没有人在 CRDT 之上做得很好。 您可以使用使用 OT 的   [ShareDB](https://github.com/share/sharedb/)  。 我多年前编写了 ShareDB，它使用良好、维护良好并经过实战测试。

展望未来，我对   [Redwood](https://github.com/redwood/redwood)   感到兴奋——它支持 P2P 编辑并计划全面支持 CRDT。

## 附录 B：谎言、该死的谎言和基准

这是真的吗？ 是的。 但是性能很复杂，我不会在这里讨论全貌。

首先，如果你想使用我自己运行的任何基准测试，你可以。 但一切都有些混乱。

JS 纯字符串编辑基线、Yjs、automerge 和 reference-crdts 测试的基准代码都在这个   [github gist](https://gist.github.com/josephg/13efc1444660c07870fcbd0b3e917638)   中。 一团糟; 但凌乱的代码总比缺少代码好。

您还需要   [josephg/crdt-benchmarks](https://github.com/josephg/crdt-benchmarks)   中的 automerge-paper.json.gz 来运行大多数这些测试。 在这个版本中，reference-crdts benchmark依赖于   [josephg/reference-crdts 中的 crdts.ts](https://github.com/josephg/reference-crdts/tree/fed747255df9d457e11f36575de555b39f07e909)  。

Diamond's benchmarks 来自   [josephg/diamond-types, at this version](https://github.com/josephg/diamond-types/tree/42a8bc8fb4d44671147ccaf341eee18d77b2d532)  . Benchmark 靠运行   `RUSTFLAGS='-C target-cpu=native' cargo criterion yjs`  . 可以通过编辑   [src/list/doc.rs](https://github.com/josephg/diamond-types/blob/42a8bc8fb4d44671147ccaf341eee18d77b2d532/src/list/doc.rs#L15)   顶部的常量来启用或禁用内联绳索结构更新。. 您可以通过运行   `cargo run --release --features memusage --example stats`  查看内存统计信息.

Diamond 使用此   [this wrapper](https://github.com/josephg/diamond-js/tree/6e8a95670b651c0aaa7701a1a763778d3a486b0c)   编译为 wasm，硬编码以指向来自 git 的 diamond-types 的本地副本。 wasm 包使用 wasm-opt 进行了优化。

这些图表是在   [ObservableHQ](https://observablehq.com/@josephg/crdt-algorithm-performance-benchmarks)   上制作的。

### Automerge 和 Yjs 做同样的事情吗？

在这篇文章中，我一直在比较 RGA（automerge）和 YATA（Yjs + 我的 Rust 实现）的实现性能。

这样做的前提是假设 YATA 和 RGA 的并发合并行为基本相同，并且您可以在不更改实现或实现性能的情况下在 CRDT 行为之间进行交换。 这是一个我认为以前没有人看过的新颖想法。

我对这个声明充满信心，因为我在我的  [参考 CRDT 实现](https://github.com/josephg/reference-crdts)  中展示了它，当使用 Yjs 或 automerge 的行为时，它具有相同的性能（和几乎相同的代码路径）。 冲突严重的编辑跟踪可能存在一些性能差异 - 但在实践中这种情况极为罕见。

我也相信您可以修改 Yjs 以实现 RGA 的行为，而无需更改 Yjs 的性能。 您只需要：

1. 更改 Yjs 的 integrate 方法（或进行替代），该方法对并发编辑使用略有不同的逻辑
1. 在每个项目中存储 seq 而不是 originRight
1. 将 maxSeq 存储在文档中，并使其保持最新和
1. 更改 Yjs 的二进制编码格式。


我和 Kevin 讨论过这个问题，他认为在他的库中添加 RGA 支持没有任何意义。 这不是任何人真正要求的。 在预置项目时，RGA 可能会有奇怪的  [交错](https://www.cl.cam.ac.uk/~arb33/papers/KleppmannEtAl-InterleavingAnomalies-PaPoC2019.pdf)  。

对于 Diamond ，我让我的代码接受一个类型参数，用于在 Yjs 和 automerge 的行为之间切换。 我不确定我是否愿意。 Kevin 可能是对的 - 我不认为这是人们想要的。

---


嗯，Yjs 有一种方式比 automerge 具有明显的优势：  *当*  文档中的每个项目都被删除时，Yjs 不会记录。 仅记录每个项目是否已被删除。 这有一些奇怪的含义：

1. 在每次删除发生时进行存储对内存使用和磁盘存储大小有很大的影响。 添加此数据后，diamond 的内存使用量从 1.12mb 增加到 2.34mb，并使系统速度降低约 5%。
1. Yjs 没有存储足够的信息来实现按键编辑重播或其他类似的东西。 （也许这就是人们想要的？记录每个错误的击键是否很奇怪？）
1. Yjs 需要将有关哪些Item已被删除的信息编码到版本字段中。 在 diamond 中，版本是几十个字节。 在 yjs 中，版本是 ~4kb。 随着文档的增长，它们会随着时间的推移而增长。 Kevin 向我保证，这些信息在实践中基本上总是很小的。 他可能是对的，但这仍然让我感到奇怪的紧张。


目前，Diamond 的主分支包括时间删除。 但是这篇博文中的所有基准测试都使用   [yjs 风格的 diamond 类型分支](https://github.com/josephg/diamond-types/tree/yjs-style)  ，它与 Yjs 的工作方式相匹配。 这可以与 yjs 进行更公平的比较，但 Diamond 1.0 的性能配置可能略有不同。 （这里有很多关于 diamond 尚未打磨的的双关语，但我现在还不够敏锐。）

### 这些基准衡量错误的东西

这篇文章只测量了重放本地编辑跟踪所需的时间。 我正在测量由此产生的 RAM 使用量。 可以说，接受来自用户的传入更改只需要  *足够*  快地发生。 手指根本不会打字很快。 一旦 CRDT 可以在大约 1 毫秒内处理任何本地用户编辑，速度更快可能无关紧要。 （并且自动合并通常已经很好地执行了，除非一些不幸的 GC 暂停。）

实际上重要的的指标是：

1. 文件在磁盘或网络上占用多少字节
1. 文档保存和加载需要多长时间
1. 更新静态存储的文档（in database）需要多长时间（更多内容见下文）


我在这里使用的编辑跟踪也只有一个用户进行编辑。 当用户进行并发编辑时，阴影中可能潜伏着病态的性能案例。

我这样做是因为我还没有在我的 reference-crdts 实现或钻石中实现二进制格式。 如果我这样做了，我可能会复制 Yjs & automerge 的二进制格式，因为它们非常紧凑。 所以我希望所有这些实现之间得到的二进制大小是相似的，除了删除操作。 加载和保存的性能可能大致反映我上面显示的基准。 或许。 或者也许我错了。 我以前错了。 找出答案会很有趣。

---


我认为目前还没有人认真对待另一项绩效衡量标准。 也就是说，我们如何更新静态文档（在数据库中）。 大多数应用程序都不是协作文本编辑器。 通常，应用程序实际上是在与充满微小对象的数据库进行交互。 这些对象中的每一个都很少被写入。

如果您想使用 Yjs 或 automerge 今天更新数据库中的单个对象，您需要：

1. 将整个文档加载到 RAM 中
1. 做出改变
1. 再次将整个文档保存回磁盘


这将非常缓慢。 对此有更好的方法 - 但据我所知，根本没有人在做这件事。 我们可以使用你的帮助！

> Kevin 说你可以调整 Yjs 的提供者以合理的方式实现这一点。 我很想看到它在行动。

---


还有另一种快速制作 CRDT 的方法，我在这里根本没有提到，那就是   *​pruning（修剪）*  。 默认情况下，像这样的列表 CRDT 只会随着时间的推移而增长（因为我们必须为所有已删除的 items 保留 tombstones ）。 CRDT 的很多性能和内存成本来自加载、存储和搜索不断增长的数据集。 有一些方法可以通过找到完全摆脱某些数据的方法来解决这个问题。 比如Yjs的GC算法，或者   [Antimatter](https://braid.org/antimatter)  。 也就是说，git 存储库只会随着时间的推移而增长，而且似乎没有人会介意太多。 也许只要底层系统足够快就无所谓了？

### 这个旅程的每一步都改变了太多的变数

优化过程中的每一步都涉及对多个变量的更改，我不会孤立这些更改。 例如，从 automerge 转移到我的 reference-crdts 实现发生了变化：

1. 核心数据结构（tree 到 list）
1. 删除了 immutablejs
1. 删除了 automerge 的前端/后端协议。 显然，无论出于何种原因，在整个自动合并过程中弹出的所有 Uint8Array 都消失了。
1. javascript 风格完全不同。 （FP javascript -> imperative）


我们从这一切中获得了 10 倍的性能。 但我只是在猜测 10 倍的加速应该如何在所有这些变化中分配。

从 reference-crdts 到 Yjs，从 Yjs 到 Diamond 的跳跃同样是单一的。 Diamond 和 Yjs 之间的速度差异有多少与内存布局无关，而与 LLVM 的优化器有关？

automerge-rs 并不比 automerge 快这一事实让我相信 Diamond 的性能不仅仅归功于 Rust。 但老实说我不确认。

所以，是的。 这是对我的方法的合理性持批判的态度。 如果这个问题困扰着您，我希望有人能够分解我在此处展示的实现之间的每个性能差异，并梳理出更详细的细分。 我会把它读出来的。 我喜欢基准化分析这些故事。 这很正常，对吧？

## 附录 C: 我还是不明白——为什么 automerge 的 javascript 这么慢？

因为它并没有试图变得快。 查看   [automerge](https://github.com/automerge/automerge/blob/d2e7ca2e141de0a72f540ddd738907bcde234183/backend/op_set.js#L649-L659)   中的这段代码：

```
function lamportCompare(op1, op2) {
  return opIdCompare(op1.get('opId'), op2.get('opId'))
}

function insertionsAfter(opSet, objectId, parentId, childId) {
  let childKey = null
  if (childId) childKey = Map({opId: childId})

  return opSet
    .getIn(['byObject', objectId, '_following', parentId], List())
    .filter(op => op.get('insert') && (!childKey || lamportCompare(op, childKey) < 0))
    .sort(lamportCompare)
    .reverse() // descending order
    .map(op => op.get('opId'))
}
```

这在每次插入时调用，以确定应如何对项目的子项进行排序。 我不知道它多热（译者：可能是比喻说CUP 产生的温度），但有  *很多关于这个的事情*  很慢：

1. 我可以在这个函数中发现 7 处内存分配。 （应该提升 2 闭包）。 （你能全部找到吗？）
1. 在调用此方法之前，项目已经排序为 reverse-lamportCompare。 对反排序列表进行排序是对任何内容进行排序的最慢方法。 这个代码应该只反转 lamportCompare 中的参数（或否定返回值），而不是 sorting ，然后 reverse()'ing 。
1. 目标是将新项目插入到已排序的列表中。 使用 for 循环可以更快地做到这一点。
1. 这段代码将 childId 包装到一个 immutablejs Map 中，以便参数匹配 lamportCompare - 然后再次解开它。 停下——我要死了！


但在实践中，这段代码将被 WASM 调用替换为 automerge-rs。 也许在您阅读本文时它已经被   [automerge-rs](https://github.com/automerge/automerge-rs)   取代了！ 所以没关系。 尽量不要去想它。 绝对不要提交任何 PR 来解决所有悬而未决的问题。   *抽搐*  。

## 致谢

这篇文章是   [Braid 项目](https://braid.org/)  的一部分，由  [Invisible College](https://invisible.college/)  资助。 如果这是您想为之做出贡献的工作，请与我们联系。 我们正在招聘。

感谢在此帖子上线之前提供反馈的所有人。

特别感谢 Martin Kleppmann 和 Kevin Jahns 在 Automerge 和 Yjs 方面所做的工作。 Diamond 站在巨人的肩膀上。