
我们团队最近在调研基于Slate的协同编辑方案，在 Slate 的一个陈旧的 Issue 中发生了一段激烈的讨论（核心是关于 OT 与 CRDT），觉得非常有意思就稍微整理了下。

Issue 原始链接：  [https://github.com/ianstormtaylor/slate/issues/259](https://github.com/ianstormtaylor/slate/issues/259)  

german-jablo commented 2021-6-24（讨论发起者）

TinyMCE   [explained](https://www.tiny.cloud/blog/real-time-collaborative-editing-slate-js/)   everything they had to do to achieve an OT with E2E encryption. Could someone please tell me if the solutions posted here meet that goal? Thank you very much for the help!

TinyMCE 解释了他们通过 E2E 加密实现 OT 所必须做的一切。 有人可以告诉我这里发布的解决方案是否符合该目标？ 非常感谢你的帮助！

TheSpyder commented（TheSpyder 是 Slate 的维护者，TinyMCE开源富文本编辑的核心开发者，在调研基于Slate的协同编辑时选择了OT，并发表了两篇相关的说明文章）

Not really. It's a lot easier to do E2E with CRDT, but I don't think any of those frameworks offer it today. We made it hard for ourselves by deciding to go the OT route - and then add E2E which I don't believe any other OT-based editors offer - but we're really happy with the result.

并不真地。 使用 CRDT 进行 E2E 会容易得多，但我认为今天这些框架中没有任何一个提供它。 我们决定走 OT 路线，这让我们自己很难过——然后添加 E2E，我不相信任何其他基于 OT 的编辑会提供这种服务——但我们对结果非常满意。

dmonad commented （dmonad 是 Yjs 的创作者，Yjs 是一个 CRDT 的实现框架，提供了实时协同编辑的核心实现，非常强大）

Both Skiff and Serenity Notes built E2E note-taking apps with the Yjs CRDT. The latter is now open-source and might provide a good starting point.

Skiff 和   [Serenity Notes](https://github.com/SerenityNotes/serenity-notes-backend)   都使用 Yjs CRDT 构建了 E2E 笔记应用程序。 后者现在是开源的，可能会提供一个很好的起点。

german-jablo commented

Thanks to both of you. I'm going to review those two tools you mention. I am more interested in OT than in CRDT because it preserves the intention better. Although as far as I know TinyMCE is the only OT with E2EE, and I am still not in a position to pay its price.

感谢你们俩。 我将回顾你提到的这两个工具。 我对 OT 比对 CRDT 更感兴趣，因为它更好地保留了意图。 虽然据我所知 TinyMCE 是唯一一个带 E2EE 的 OT，我仍然无法支付它的价格。

dmonad commented

> it preserves the intention better

I can say with some authority that this is not the case. OT has other advantages, but intention-preservation is not something that OT does particularly well.

我可以权威地说，事实并非如此。 OT 还有其他优点，但意图保存并不是 OT 做得特别好的。

german-jablo commented

Well, until now I had heard otherwise. I will do some tests to verify it.

Another problem I have is that I want my product to have a version history.

Also, I am not an expert but I understand that CRDT consumes more memory (since it does not "erase" characters). 
However it confuses me that some say it is faster / efficient than OT and others not.

Were those the advantages of OT you were referring to?

好吧，直到现在我还听说过其他情况。 我会做一些测试来验证它。

我的另一个问题是我希望我的产品有一个版本历史。

另外，我不是专家，但我知道 CRDT 消耗更多内存（因为它不会“擦除”字符）。 

然而，让我感到困惑的是，有人说它比 OT 更快/更高效，而其他人则不然。

那些是你所指的 OT 的优势吗？

LionsAd commented（这个人通俗的解释了 OT 和 CRDT 的区别）

@german-jablo You can decide for yourself:

OT is like a navigation system:

go 5 blocks left, go three blocks up, …

Once you’ve already moved 1 block left, the instructions read:

go 4 blocks left, go three blocks up, …

CRDT is an address, which never changes:

Go to client1-40 street

OT and CRDT are in the end equivalent ways and can actually transformed into each other (a paper has proven that).

And it makes intuitive sense:

If you have the address, then you can always get the directions of how to get to that address - regardless of how the state changes.

If you have directions, you can ensure that if the state (your position) changes that you still find the same address by transforming the operations how to get there.

I personally find it intuitively easier to “name” each character with a client unique ID.

It feels much easier to think of objects that change in relation to each other, than to transform operation stacks, but to each their own.

As for E2EE: We have an interview with Serenity Notes author and E2EE up at Tag1.com/yjs, but the secret nutshell is:

Just sync the whole document always.

Feels excessive, but if you think of the megabytes of data transferred on YouTube etc. it’s actually not a big deal.

Hope that helps!

OT 就像一个导航系统：

向左走 5 个街区，向上走三个街区，……

一旦你已经向左移动了 1 个街区，说明如下：

向左走 4 个街区，向上走三个街区，……

CRDT 是一个地址，永远不会改变：

去客户1-40街

OT 和 CRDT 最终是等效的方式，实际上可以相互转换（一篇论文已经证明了这一点）。

它具有直观的意义：

如果你有地址，那么你总能得到如何到达那个地址的方向——不管状态如何变化。

如果你有方向，你可以通过转换操作如何到达那里来确保如果状态（你的位置）发生变化，你仍然可以找到相同的地址。

我个人发现使用客户端唯一 ID 来“命名”每个角色在直觉上更容易。

考虑相对于彼此发生变化的对象，比转换操作堆栈更容易，但要对每个对象进行转换。

至于 E2EE：我们在 Tag1.com/yjs 上对 Serenity Notes 作者和 E2EE 进行了采访，但秘诀是：

只需始终同步整个文档。

感觉有点过分，但是如果您考虑在 YouTube 等上传输的兆字节数据，实际上这没什么大不了的。

希望有帮助！

mitar commented

There is also a middle ground:   [https://github.com/campadrenalin/ConcurrenTree](https://github.com/campadrenalin/ConcurrenTree)  

还有一个中间立场：  [https://github.com/campadrenalin/ConcurrenTree](https://github.com/campadrenalin/ConcurrenTree)  

TheSpyder commented

I'm not going to get into a mud-slinging match but I feel like someone needs to defend OT in this thread. I stand by what I have said in my blog posts - for simple cases (plain text, JSON) yes OT and CRDT are equivalent and CRDT is usually the better choice.

For a rich text editor, however, OT offers preservation of intent. That's a lot more than just mathematical equivalence. With Slate's 9 operations there is a matrix of 81 transforms to implement, but the payoff is that user intent is preserved. The key example I keep returning to is splitting a node. In Slate it's a split node operation, not a combination of inserts and deletes that a plain JSON OT or CRDT model would use to describe it. That offers very useful context when deciding how to transform conflicting operations.

> As for E2EE: We have an interview with Serenity Notes 
author and E2EE up at Tag1.com/yjs, but the secret nutshell is:  
Just sync the whole document always  
Feels excessive, but if you think of the megabytes of data transferred on YouTube etc. it’s actually not a big deal.

It is a big deal, and TinyMCE encryption is done at the operation level. Open up our demo and monitor the websocket connection - only tiny amounts of data are sent and received.

我不会参加一场泥泞的比赛，但我觉得有人需要在这个 Issue 中捍卫 OT。 我支持我在我的博客文章中所说的 - 对于简单的情况（纯文本，JSON）是的，OT 和 CRDT 是等效的，而 CRDT 通常是更好的选择。

然而，对于富文本编辑器，OT 提供了意图的保留。 这不仅仅是数学上的等价。 使用 Slate 的   [9 个操作](https://github.com/ianstormtaylor/slate/blob/main/packages/slate/src/interfaces/operation.ts#L119-L138)  ，需要实现一个包含 81 个转换的矩阵，但回报是保留了用户意图。 我保持返回的关键示例是拆分节点。 在 Slate 中，它是一个拆分节点操作，而不是普通 JSON OT 或 CRDT 模型用来描述它的插入和删除的组合。 在决定如何转换冲突操作时，这提供了非常有用的上下文。

> 至于 E2EE：我们在 Tag1.com/yjs 上对 Serenity Notes 作者和 E2EE 进行了采访，但秘诀是：  
只需始终同步整个文档。  
感觉有点过分，但是如果您考虑在 YouTube 等上传输的兆字节数据，实际上这没什么大不了的。

这是一个大问题，TinyMCE 加密是在操作级别完成的。 打开我们的演示并监控 websocket 连接 - 仅发送和接收少量数据。

german-jablo commented

I appreciate everyone's contribution. @TheSpyder What you are saying sounds like the holy grail of the RTC. From what little I understand, I think that the solution TinyMCE is working on is the best on the market in RTC, be it OT or CRDT. I think not many appreciate it because   [the research you did](https://www.tiny.cloud/blog/real-time-collaboration-end-to-end-encryption/)   is very technical.

It would be great if you could publish for more clumsy people like me the basics of the French paper you used and how you combined Jupiter / Soct5 to achieve the result.
By the way, do you have plans to integrate RTC in the core version?
我感谢每个人的贡献。 @TheSpyder 你所说的听起来像是 RTC 的圣杯。 据我所知，我认为 TinyMCE 正在开发的解决方案是 RTC 市场上最好的，无论是 OT 还是 CRDT。 我认为没有多少人会欣赏它，因为  [您所做的研究](https://www.tiny.cloud/blog/real-time-collaboration-end-to-end-encryption/)  非常技术性。

如果你能向像我这样笨手笨脚的人发布你使用的法国论文的基础知识以及你如何结合 Jupiter / Soct5 来实现结果，那就太好了。

顺便问一下，你们有计划在核心版本中集成RTC吗？

TheSpyder commented

> What you are saying sounds like the holy grail of the RTC.

I am not the first - there are others who have successfully built an OT-based collaborative editor with Slate. Some have posted in this thread. They are the ones who inspired us to do it.

我不是第一个 - 还有其他人成功地使用 Slate 构建了基于 OT 的协作编辑器。 有些人在这个帖子里发过帖子。 他们是激励我们去做这件事的人。

> I think not many appreciate it because the research you did is very technical.  
It would be great if you could publish for more clumsy people like me the basics of the French paper you used and how you combined Jupiter / Soct5 to achieve the result.

Ah. I am not Tim, who wrote that post; I am Andrew, author of the earlier posts. I will let him know there is interest in a deep-dive.
啊。 我不是写那篇文章的蒂姆； 我是安德鲁，  [早期帖子](https://www.tiny.cloud/blog/real-time-collaboration-ot-vs-crdt/)  的作者。 我会让他知道有人对它很感兴趣。

> By the way, do you have plans to integrate RTC in the core version?

That's still TBD. For now, our only announced plan is to include it with our premium offering; it will be cloud-only at launch with an on-prem version available later (we're hoping it will be in beta at launch).

那还是待定。 目前，我们唯一宣布的计划是将其包含在我们的高级产品中； 它将在发布时仅用于云，稍后将提供本地版本（我们希望它在发布时处于测试阶段）。

german-jablo commented

> I am not the first - there are others who have successfully built an OT-based collaborative editor with Slate. Some have posted in this thread. They are the ones who inspired us to do it.

Yes, but if I'm not mistaken there are none that are 

(1) E2EE compatible 

(2) that only store changes and

(3) allow a version history.

For everything else, thank you very much!

是的，但如果我没记错的话，没有一个是

(1) E2EE 兼容

(2) 只存储更改和

(3) 允许一个版本历史。

同时满足，非常感谢！

dmonad commented

@TheSpyder
> For a rich text editor, however, OT offers preservation of intent.

I know what you mean. But no conflict-resolution algorithm that exists can preserve the actual intent of the user because the algorithm doesn't understand what the user wants to do. All conflict-resolution approaches offer different tradeoffs when it comes to intent-preservation. Even CRDTs can offer a high degree of intent-preservation.

我明白你的意思。 但是现有的无冲突解决算法都不能保留用户的实际意图，因为该算法不了解用户想要做什么。 在意图保留方面，所有冲突解决方案都提供了不同的权衡。 甚至 CRDT 也可以提供高度的意图保留。

Your article made me kinda sad when I read it:   [https://www.tiny.cloud/blog/real-time-collaboration-ot-vs-crdt/](https://www.tiny.cloud/blog/real-time-collaboration-ot-vs-crdt/)   Citing from the article:

你的文章让我读起来有点难过：  [https://www.tiny.cloud/blog/real-time-collaboration-ot-vs-crdt/](https://www.tiny.cloud/blog/real-time-collaboration-ot-vs-crdt/)   
引用文章：
> If you can show me a rich text editor with support for CRDT, I can show you why I struggle to call it a "rich" text editor. Perhaps we should call them moderately wealthy text editors.  

如果您能向我展示一个支持 CRDT 的富文本编辑器，我就能向您展示为什么我很难称其为“富”文本编辑器。 也许我们应该称他们为中等富裕的文本编辑器。

Maybe you should have done a bit more research. Your split-node problem has been solved in Yjs since 2016. You completely misrepresent all the work that so many people put into different CRDT-based editor bindings. You just happened to use a very bad implementation of a CRDT to make the assumption that all CRDT implementations are bad.

也许你应该做更多的研究。 自 2016 年以来，您的拆分节点问题已在 Yjs 中得到解决。您完全歪曲了许多人投入到不同的基于 CRDT 的编辑器绑定中的所有工作。 你只是碰巧使用了一个非常糟糕的 CRDT 实现来假设所有 CRDT 实现都是糟糕的。

None of the 10 editors that Yjs supports has the behavior that you are describing:   [https://github.com/yjs/yjs-demos ](https://github.com/yjs/yjs-demos)  +   [https://www.tiptap.dev/](https://www.tiptap.dev/)   +   [https://remirror.io/](https://remirror.io/)  

Yjs 支持的 10 个编辑器中没有一个具有您所描述的行为：  [https://github.com/yjs/yjs-demos ](https://github.com/yjs/yjs-demos)  +   [https://www.tiptap.dev/](https://www.tiptap.dev/)   +   [https://remirror.io/](https://remirror.io/)  

Personally, I find the debate pointless to compare generic CRDTs vs OT. CRDTs can do everything that OT can do, and vice versa. Most "researchers" in this field have done half-assed research only to support their argument. Every CRDT paper that claims that it is "faster" than OT doesn't talk about the tradeoffs that they had to make. Every paper that claims that OT is clearly superior to CRDT is wrong because OT-operations can be represented on top of a CRDT. Yjs, for example, has full support for the   [OT RichText type](https://github.com/ottypes/rich-text)   (it transforms the OT operation to a CRDT operation to offer a stronger consistency model).

就个人而言，我发现比较通用 CRDT 与 OT 的争论毫无意义。 CRDT 可以做 OT 可以做的一切，反之亦然。 该领域的大多数“研究人员”都做了半途而废的研究，只是为了支持他们的论点。 每篇声称它比 OT“更快”的 CRDT 论文都没有谈论他们必须做出的权衡。 每篇声称 OT 明显优于 CRDT 的论文都是错误的，因为 OT 操作可以在 CRDT 之上表示。 例如，Yjs 完全支持   [OT RichText type](https://github.com/ottypes/rich-text)  （它将 OT 操作转换为 CRDT 操作以提供更强的一致性模型）。

So instead of mud-slinging, we should probably just compare implementations by their features and performance metrics. None of your numerous posts describes a single valid argument against the Yjs CRDT.

因此，我们应该只通过它们的特性和性能指标来比较实现，而不是泥泞。 您的众多帖子都没有描述反对 Yjs CRDT 的单一有效论点。

@german-jablo

If TinyMCE's OT approach does everything you are looking for, go ahead and use it. I just wanted to make clear that intent-preservation is not something that OT is inherently better at.

如果 TinyMCE 的 OT 方法可以满足您的所有要求，请继续使用它。 我只是想说明，意图保留并不是 OT 天生擅长的。

> Another problem I have is that I want my product to have a version history.
Also, I am not an expert but I understand that CRDT consumes more memory (since it does not "erase" characters). 
However, it confuses me that some say it is faster / efficient than OT and others not.
Were those the advantages of OT you were referring to?  

The slate-yjs binding currently doesn't support versions and tracking changes (only y-prosemirror does). CRDTs in general don't have to consume much more memory than OT (although some certainly do). It's part of the same fallacy that @TheSpyder ran into. They looked at a single bad implementation and judged that all implementations consume too much memory. Yjs has excellent performance metrics even for huge documents. You can write the Bible into a Yjs document while using less than 5MB of ram.

slate-yjs 绑定当前不支持版本和跟踪更改（只有 y-prosemirror 支持）。 通常，CRDT 不必比 OT 消耗更多的内存（尽管有些确实如此）。 这是 @TheSpyder 遇到的同样谬论的一部分。 他们查看了一个糟糕的实现，并判断所有实现都消耗了太多内存。 即使对于大型文档，Yjs 也具有出色的性能指标。 您可以使用不到 5MB 的内存将圣经写入 Yjs 文档。
---

Just to be clear. I have nothing against OT. Let's just stop with these pointless debates of citing papers from researchers that only want to popularize their approach. If you can't reproduce a "bad behavior" in a specific implementation, then you shouldn't make an argument that a certain thing is not possible. If OT works for you, that's good for you. Keep using it.

只是要清楚。 我不反对OT。 让我们停止这些关于引用研究人员的论文的毫无意义的辩论，他们只想普及他们的方法。 如果您无法在特定实现中重现“不良行为”，那么您不应该争论某件事是不可能的。 如果 OT 对你有用，那对你有好处。 继续使用它。

TheSpyder commented

I'm sorry you feel that way. But if we can't keep this on topic I'm going to lock the conversation. If you want to have a go at me please feel free to do it privately.

Every example you linked to - including ot-richtext - is either plain text, based on Quill, or based on Prosemirror. These are the editors I was describing that only support a subset of HTML (although prosemirror is better than it was when we made the call to use Slate+OT for TinyMCE in 2019). I have repeatedly said CRDT is perfect for that context and if those editors are sufficient more power to you.

This conversation is about Slate. Slate has a much more flexible document model, and while it isn't an officially supported yjs editor there are   [yjs bindings](https://bitphinix.github.io/slate-yjs-example/)   that do demonstrate this specific issue. And it's not like this is the only issue that comes up, it was just the easiest to show in a blog post.
[https://www.loom.com/share/1450a0c84f9b4cf58b4aedec6a0cc00a](https://www.loom.com/share/1450a0c84f9b4cf58b4aedec6a0cc00a)  

我觉得抱歉让你有种感觉。 但是，如果我们不能保持这个话题，我将锁定对话。 如果你想继续讨论，请随时私下进行。
您链接到的每个示例（包括 ot-richtext）都是基于 Quill 或 Prosemirror 的纯文本。 这些是我描述的仅支持 HTML 子集的编辑器（尽管 prosemirror 比我们在 2019 年呼吁将 Slate + OT 用于 TinyMCE 时更好）。 我一再说过 CRDT 非常适合这种情况，如果这些编辑器对你来说足够强大的话。

这个对话是关于 Slate 的。 Slate 有一个更灵活的文档模型，虽然它不是官方支持的 yjs 编辑器，但   [yjs binding](https://bitphinix.github.io/slate-yjs-example/)   确实演示了这个特定问题。 这并不是唯一出现的问题，它只是在博客文章中最容易展示的。

[https://www.loom.com/share/1450a0c84f9b4cf58b4aedec6a0cc00a](https://www.loom.com/share/1450a0c84f9b4cf58b4aedec6a0cc00a)  

dmonad commented

You are committing the same fallacy again. Just because you see one counter-example, you are judging that this is an inherent problem. I'm familiar with the Slate data model. Yjs has data types that enable you to linearize the content, similarly to how you do it. Slate-yjs just doesn't use this feature yet. This is not a hard problem to solve, as it can be seen on hand of numerous complex editor bindings that don't show the same behavior. OT doesn't have inherently better intention-preservation than CRDTs. This is just a wrong statement to make.

你又犯了同样的谬论。 仅仅因为你看到一个反例，你就判断这是一个固有的问题。 我熟悉 Slate 数据模型。 Yjs 具有使您能够线性化内容的数据类型，这与您的做法类似。 Slate-yjs 还没有使用这个功能。 这不是一个很难解决的问题，因为可以在众多复杂的编辑器绑定中看到，这些绑定不显示相同的行为。 OT 在本质上并不比 CRDT 具有更好的意图保留。 这只是一个错误的陈述。

TheSpyder commented

And you seem to have missed the conclusion of my article. If slate-yjs can be that good, please help this community by guiding it to get there. I'll help in any way I can. It would make my life a lot easier if TinyMCE could connect yjs to our Slate model instead of our custom OT solution :)

你似乎错过了我文章的结论。 如果 slate-yjs 可以那么好，请通过引导它到达那里来帮助这个社区。 我会尽我所能提供帮助。 如果 TinyMCE 可以将 yjs 连接到我们的 Slate 模型而不是我们的自定义 OT 解决方案，那将使我的生活更轻松:)

BitPhinix commented

@dmonad I'm really curious on how you would go about linearizing the content. Could you share some pointers?

我真的很好奇你将如何线性化内容。 你能分享一些点吗？

dmonad commented
Sure, I'm happy to help. In order to solve the split-node problem on text-nodes you can make use of the formatting attributes.

当然，我很乐意提供帮助。 为了解决文本节点上的拆分节点问题，您可以使用格式属性。

Assuming you have the text   **"Hello World"**  , where   **"Hello"**   is bold and   **"World"**   is bold and italic:

```
{
    "object": "text",
    "leaves": [
     {
        "object": "leaf",
        "text": "Hello",
        "marks": [italic]
      }, {
        "object": "leaf",
        "text": "World",
        "marks": [bold, italic]
      }
    ]
}
```

You can represent this text object using a   [Y.Text](https://docs.yjs.dev/api/shared-types/y.text)   and formatting attributes (this is basically equivalent to the idea of marks):

您可以使用 [Y.Text](https://docs.yjs.dev/api/shared-types/y.text) 和格式属性来表示此文本对象（这基本上等同于   `marks`   的想法）：

```
const ytext = new Y.Text()

ytext.insert(0, 'Hello World', { italic: true }) // insert "Hello World" as italic text
ytext.format(6, 5, { bold: true }) // assign bold formatting attributes to the word "World"
```
Or using the delta notation:
或者使用 delta 表示法：

```
ytext.applyDelta([
  { insert: 'Hello ', attributes: { italic, true } },
  { insert: 'World', attributes: { italic: true, bold: true } }
])
```

A formatting attribute can be any JSON-encodable key-value pair. You can granularly remove or update attributes without replacing the underlying text. These are meta-properties that you can assign to ranges of content in the Yjs document.

格式化属性可以是任何 JSON 可编码的键值对。 您可以在不替换底层文本的情况下精细地删除或更新属性。 这些是您可以分配给 Yjs 文档中的内容范围的元属性。

```
ytext.toDelta() // => [{ insert: 'Hello ', attributes: { italic, true } }, { insert: 'World', attributes: { italic: true, bold: true } }]
```

In order to solve the split-node scenario that Andrew described, you just need to map Y.Text with formatting attributes to a Slate text node. I recommend working with the Y.Text delta events that should map nicely to Slate's operations, and vice versa.

为了解决 Andrew 描述的拆分节点场景，您只需要将带有格式属性的 Y.Text 映射到 Slate 文本节点。 我建议使用应该很好地映射到 Slate 操作的 Y.Text delta 事件，反之亦然。

> I'll help in any way I can. It would make my life a lot easier if TinyMCE could connect yjs to our Slate model instead of our custom OT solution :)

@TheSpyder That would be great :) Let me know when you need help. For Yjs-specific discussions, I'm also available on the discussion board   [https://discuss.yjs.dev/](https://discuss.yjs.dev/)  
那太好了 :) 当您需要帮助时请告诉我。 对于特定于 Yjs 的讨论，我也可以在讨论板上找到   [https://discuss.yjs.dev/](https://discuss.yjs.dev/)  

BrentFarese commented
@dmonad and @TheSpyder or anyone else on this thread. We use Slate for   [Aline](https://www.aline.co/)   and are going to get to collaborative this year for sure. As with everyone, we have looked at OT vs. CRDT. YJS looks very interesting and we have considered using it.

We would definitely sponsor some open source work to improve official YJS-Slate bindings that would help the community (and allow us to use the bindings for Aline). We might be able to commit some resources ourselves in a couple of months too.

Is anyone interested in participating/co-sponsoring that type of work? I think it could be valuable to both Slate and YJS to extend the reach of both projects. @dmonad have you done anything like that in the past?

Thanks!

@dmonad 和 @TheSpyder 或此Issue上的任何其他人。 我们为   [Aline](https://www.aline.co/)   使用 Slate，今年肯定会进行协作。 与所有人一样，我们已经研究了 OT 与 CRDT。 YJS 看起来很有趣，我们已经考虑使用它。

我们肯定会赞助一些开源工作来改进官方 YJS-Slate 绑定，这将有助于社区（并允许我们使用 Aline 的绑定）。 我们也可以在几个月内自己投入一些资源。

有人有兴趣参与/共同赞助这种类型的工作吗？ 我认为扩大这两个项目的影响范围对 Slate 和 YJS 来说都是有价值的。 @dmonad 你过去做过类似的事情吗？

谢谢！


BrentFarese commented

Can we also list Slate bindings in YJS docs (maybe designate as WIP if they're not ready yet)?

我们是否还可以在 YJS 文档中列出 Slate 绑定（如果尚未准备好，可以指定为 WIP）？


dmonad commented

Hi @BrentFarese,

Thanks so much for offering this! @BitPhinix did an awesome job on the current slate-yjs binding. It would be great if you could sponsor him to work improve this implementation a bit.

非常感谢你提供这个！ @BitPhinix 在当前的 slate-yjs 绑定上做得很棒。 如果你能赞助他改进这个实现，那就太好了。

We finance Yjs additions with our   [open-collective](https://opencollective.com/y-collective)  . We can open a separate project for slate-yjs if @BitPhinix or someone else is interested in working on this. I don't have the time currently and can only offer my feedback.

我们通过我们的开放集体资助 Yjs 的增加。 如果@BitPhinix 或其他人对此感兴趣，我们可以为 slate-yjs 打开一个单独的项目。 我目前没有时间，只能提供我的反馈。

> Can we also list Slate bindings in YJS docs (maybe designate as WIP if they're not ready yet)?

Yeah, I thought I already added it. Will do it in a bit.

是的，我以为我已经添加了它。 一会就搞定。

BitPhinix commented

Thanks @dmonad! Opening a slate-yjs open collective project would be really helpful indeed.

I'd be happy to work on improving the current slate-yjs binding

谢谢@dmonad！ 打开一个 slate-yjs 开放集体项目确实会很有帮助。

我很乐意改进当前的 slate-yjs 绑定

BrentFarese commented

@dmonad and/or @BitPhinix let me know the link to the open collective when set up and we would be glad to contribute.

@dmonad 和/或 @BitPhinix 在设置时让我知道开放集体的链接，我们很乐意做出贡献。


hanspagel commented

Hi @BrentFarese! I’m running the y-collective together with Kevin, and we’re eager to bring new people from the Yjs eco system on board. 🙃

嗨@BrentFarese！ 我和 Kevin 一起经营 y-collective，我们渴望从 Yjs 生态系统中招募新人。 🙃

I’ve just created a slate-yjs project:   [https://opencollective.com/y-collective/projects/slate-yjs](https://opencollective.com/y-collective/projects/slate-yjs)   In other words, it’s open to collect donations. I’m in contact with @BitPhinix anyway, so I’ll discuss with him all further details, but that’s probably out of scope of this issue. If any of you wants to reach out in private, my inbox is open: humans@tiptap.dev. Happy to connect everyone with everyone and make the Yjs ecosystem a little bit better every day. ✌️

我刚刚创建了一个 slate-yjs 项目：https://opencollective.com/y-collective/projects/slate-yjs 换句话说，它是开放的，可以收集捐款。 无论如何，我与@BitPhinix 保持联系，因此我将与他讨论所有进一步的细节，但这可能超出了本问题的范围。 如果你们中有人想私下联系，我的收件箱是开放的：humans@tiptap.dev。 很高兴将每个人与每个人联系起来，让 Yjs 生态系统每天都变得更好。 ✌️


TheSpyder 的两篇文章链接：

[To OT or CRDT, that is the question](https://www.tiny.cloud/blog/real-time-collaboration-ot-vs-crdt/)  

[Collaboration needs a clean Slate](https://www.tiny.cloud/blog/real-time-collaborative-editing-slate-js/)  
