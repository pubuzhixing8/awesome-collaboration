原文地址：  [https://www.aha.io/blog/text-editor](https://www.aha.io/blog/text-editor)   

这是一个痛苦的认识。 您刚刚在错误跟踪器的文本编辑器中添加了一个漂亮的多页描述，并附有照片和一个简短的截屏视频。 然后你的同事在他们去吃午饭的时候把窗户开着，帮助修正了一个错字……并覆盖了你刚刚做的一切。 噗——所有这些工作都消失了。

关于 Rails，我最喜欢的事情之一是它解决了大多数应用程序 99% 的问题——你知道，获取一些信息，用它做一些事情，然后把这些信息放回人们面前。 但随着您的应用越来越受欢迎，同一记录中输入的信息也越来越多，而且通常是多人同时输入。 这可能是灾难性的。

如果每个人都可以同时工作在同一条记录上，该有多好？ 如果记录可以处理所有这些更新，那么您不必像《蝇王》中的一群滞留的青春期少女那样协商谁来进行更新？

随着越来越多的组织将所有工作流程和数据转移到网上，人们期望每个人都可以实时协同工作。 科技公司正在围绕协作编辑或在现有产品中添加协作来构建整个业务。

这是我们在 Aha 一直在考虑的事情！ 一段时间。 我们知道我们的客户希望在我们的应用程序中创建注释或编写功能描述时能够无缝协作。 因此，我们一直在研究如何才能提供最佳的协作文本编辑器体验。

如果您不希望您的客户为了完成工作而不得  **不通过海螺壳（话语权、靠喊）**  ，那么您也需要它。

**协同编辑是什么样的？**

作为开发人员，您可能会想到 Git。 您对文档进行更改，其他人对其文档进行更改，您合并，然后你们中的一个人修复冲突。

这个过程的一部分很棒。 您可以直接进行更改，而无需等待其他任何人。 这就是所谓的乐观地做出改变，从某种意义上说，你可以做一些事情而不必先告诉其他人并假设你的改变会实现。

这个过程的一部分不是很好。 当你我同时编辑同一个文档时，我们不想每隔几分钟就被打断处理冲突。 像这样工作的文本编辑器将无法使用。 但冲突不会经常发生。 它们只会在我们尝试同时编辑同一个地方时发生，这并不常见。

但是，当冲突确实发生时，如果我们没有被打扰怎么办？ 系统可以对要做什么做出最好的猜测，如果它错了，我们中的一个人可以修复它。 从理论上讲，这似乎是一个永远行不通的可怕想法。 在实践中，它主要是有效的。

那么这在实践中是如何运作的呢？ 系统不需要是正确的，它只需要保持一致，并且需要努力保持你的意图。

下图显示了这一点。 如果我输入“hello”这个词，系统应该尽最大努力确保“hello”最终出现在文档的某个地方。 那是意图。

![image.png](https://atlas-rc.pingcode.com/files/public/60e97796f6d53dd3455c4ff7/origin-url)

如果其他人同时在同一地点键入“bye”？ 我们的两个文档最终应该完全相同，无论是“hellobye”还是“byehello”——文档需要保持一致。

冲突怎么办？ 嗯，人是天生的冲突解决机器。 如果你正走在走廊上，有人要走进你（找你），你会停下来。 可能你们两个都会移到同一边。 然后也许你们会移到另一边，然后你会笑。 但最终你们中的一个会移动，另一个会静止不动，一切都会好起来的。

因此，您希望您的编辑器快速响应使用它的人。 如果您正在输入，您不想等待网络请求才能看到您输入的内容。 并且您希望编辑您的文档的其他人看到您的更改。 您希望尽快完成所有这些工作。 你怎么能做到这一点？

您可能会想到的第一件事是发送差异，如下图所示。 “  This person 1 changed line 5 from this to this  ” 但是很难在差异中看出意图。 所有的差异只是告诉你发生了什么变化，而不是为什么。

![image.png](https://atlas-rc.pingcode.com/files/public/60eaa148f6d53d2bf85c4ffb/origin-url)

更好的方法是考虑一个人可以采取的行动：“我在位置 5 插入了字符‘a’。” “我删除了位置 8 之后的字符‘b’。”

![image.png](https://atlas-rc.pingcode.com/files/public/60eaa197f6d53d86925c4ffc/origin-url)

您几乎可以随心所欲地进行这些行为（或操作）。 从“插入一些文本”到“使这部分文本加粗”的所有内容。 您可以将这些应用到文档，当您这样做时，文档会发生变化。 因此，如下所示，应用此操作会将“Hello”更改为“Hello, world”。

![image.png](https://atlas-rc.pingcode.com/files/public/60eaa39ef6d53d28fa5c4ffd/origin-url)

如果我们有操作，我们可以发送操作，我们可以通过应用操作更改文档，那么我们几乎就有了协作。

如果客户端 A 向客户端 B 发送了一个“在 5 处插入‘world’”操作怎么办？ 客户端 B 可以应用该操作，您将拥有相同的文档！ Binggo - 工作已完成，完美无缺。 实际上它不是。

要记住 — 你可能改动文档在相同的时间。那么，让我们看看两个 客户端的场景。像下图展示的那样，他们每一个都是文本 “at”的文档。

![image.png](https://atlas-rc.pingcode.com/files/public/60eaa504f6d53d2b075c4ffe/origin-url)

现在，左边的客户端输入 “c” 在位置 0，产生了单词 “cat”。在相同的时间，另外一个客户端输入 “r” at position 1，产生了单词 “art”。

![image.png](https://atlas-rc.pingcode.com/files/public/60eaa561f6d53d51535c4fff/origin-url)

现在，右边的客户端的收到了来自其他客户端的 “insert c at 0” 操作以“cart”结束。目前，还好。但是左边的客户端收到了其他客户端的操作“insert r at 1”，以“crat”结束。

![image.png](https://atlas-rc.pingcode.com/files/public/60eaa720f6d53de6175c5000/origin-url)

我不知道 “crat” 是什么（你呢？）

比“crat”更糟，我们已经违反了最重要的规则。两个文档需要互相兼容另外一个 — 他们需要以相同的状态结束。因为现在，如果左边的客户端删除了字符 “a” 在位置2之后，它也在另外一个客户端删除了 “r” 而不知道它正在做的是什么。它是错误的并且以人不能修复的方式被破坏。

那么，一些别的事情需要发生。它就是   operational transformation   操作转换。

**Transforming operations**

那么让我们再看看问题。我们有两个同时发生的操作，这意味着它们都来自同一个文档状态——我们之前谈到的“at”示例。

![image.png](https://atlas-rc.pingcode.com/files/public/60eaa9c9f6d53d33945c5001/origin-url)

每当我们在同一个文档上发生两个操作时，这意味着我们可能需要更改其中之一。这是因为您只能按顺序一个接一个地进行更改——就像我们刚刚看到的“cart”和“crat”一样，顺序很重要。在这个客户端上，这意味着插入“r”必须改变，以便它可以在插入“c”之后发生。那么这会是什么样子呢？

好吧，在你插入“c”之后，原来的位置 1（下面用黄色突出显示）现在是位置 2。一切都移动了一个。如果“r”介于“a”和“t”之间（就像它在另一个客户端上所做的那样），它应该进入位置 2——对吗？所以，当你得到“在 1 处插入 r”时，你在应用它之前将它转换为“在 2 处插入 r”。

![image.png](https://atlas-rc.pingcode.com/files/public/60eaaa6af6d53d75905c5002/origin-url)

另一边呢？它得到“在 0 处插入 c”。但是位置 0 没有移动，所以“在 0 处插入 c”可以保持原样。

![image.png](https://atlas-rc.pingcode.com/files/public/60eaaa9ff6d53d90565c5003/origin-url)

你想要做的是说，“如果操作 A 和操作 B 同时发生，我怎么能改变操作 B 以适应操作 A 所做的？”

That can sometimes be abstract and hard to think about. So I draw boxes instead. (Yep, I have lots of pieces of paper filled with boxes.) But check this out below. In the upper-left corner, I write a document state — “at.”

这有时可能是抽象的，难以思考。所以我画框代替。 （Yep, I have lots of pieces of paper filled with boxes.）但请在下面查看。在左上角，我写了一个文档状态——“at”。

![image.png](https://atlas-rc.pingcode.com/files/public/60eaaad5f6d53d1fda5c5004/origin-url)

I draw an arrow going right and I write one of the operations (“insert c at 0”) on it. Then in the upper-right, I write what things look like after that happens (“cat”). Then I draw a line going down.

我画了一个向右的箭头，然后在上面写了一个操作（“在 0 处插入 c”）。然后在右上角，我写下事情发生后的样子（“cat”）。然后我画一条向下的线。

![image.png](https://atlas-rc.pingcode.com/files/public/60eaab41f6d53d136e5c5005/origin-url)

接下来，我画了一个向下的箭头。这个得到另一个操作（“在 1 处插入 r”）。和以前一样：我写下事情发生后的样子（“art”）。我画了一个向右的箭头。我们最终得到了您在下面看到的内容。

![image.png](https://atlas-rc.pingcode.com/files/public/60eaab7cf6d53d678a5c5006/origin-url)

现在我们要做出决定。在右下角，文档应该是什么样的？这就是考虑用户的期望会有所帮助的地方，但有时您必须做出决定。

不过，在这里，答案并不含糊——应该是“购物车”。将“cat”变成“cart”需要什么？ “在 2 处插入 r。”将“art”变成“cart”需要什么？ “在 0 处插入 c。”所以我们将填写空白箭头。这两个箭头是我们的两个转换操作。

![image.png](https://atlas-rc.pingcode.com/files/public/60eaabb1f6d53db2165c5007/origin-url)

一旦你知道了这一点，就可以很容易地测试驱动转换操作的代码。顶部和左侧是您的输入，右侧和底部是您的预期输出。

不过，在我们开始编写这些之前，还有最后一个问题需要解决。你如何打破关系？如果两个客户端都试图在完全相同的位置插入文本，谁的文本最先结束？

记住——你不一定是对的，你只需要保持一致。所以，选择一个一致的决胜局。如果您正在与服务器通信，您可以决定服务器总是获胜。或者你可以给每个客户一个随机 ID，最大的一个获胜。只要保持一致。

**写转换函数**

我们有一些要转换的操作和一些预期的返回值。这个转换函数实际上是什么样子的？

我们将从这个开始：

```
def transform(top, left, win_tiebreakers = false)
  bottom = transform_operation(top, left, win_tiebreakers),
  right =  transform_operation(left, top, !win_tiebreakers)
  [bottom, right]
end
```

它将顶部与左侧转换为底部箭头，然后将左与顶部转换为向右箭头，然后返回它们，这完成了我们的正方形。 但这只是提出问题。 transform_operation 函数是什么样的？

让我们关注以   `right =`   开头的那一行。你如何转换左边的操作，让它表现得好像它发生在上面的操作把所有东西都移过来并变成右箭头——“在 1 处插入 r”之后？

```
# ours:   { type: :insert, text: “r”, position: 1 }
# theirs: { type: :insert, text: “c”, position: 0 }
def transform_operation(ours, theirs, win_tiebreakers)
  # TODO: handle other kinds of operations

  transformed_op = ours.dup

  if ours[:position] → theirs[:position] || 
    (ours[:position] == theirs[:position] && !win_tiebreakers )
    transformed_op[:position] = 
      transformed_op[:position] + theirs[:text].length
  end

  transformed_op
end
```

如果我们现在只是考虑插入文本，那么编写 transform_operation 并不太难。我们写了一个待办事项以备后用。

接下来，我们返回一个新操作，因为我们不想通过更改传递给我们的操作来搞砸任何事情。在   `if`   行中，我们回答以下问题：什么会导致我们的位置发生变化？

如果另一个客户在我们的位置之前插入文本，我们将需要移过去。如果他们在与我们相同的位置插入文本，而我们失去了决胜局，我们也将不得不移过去。如果发生这些情况中的任何一种，我们需要将我们的位置移动他们插入的文本的长度。如果他们正在输入一个字符，我们就会移动一个字符。就像我们之前看到的一样——因为有人在我们的“r”之前输入了“c”，我们需要移动，否则我们会得到“crat”。

这和转换函数一样简单，但大多数都遵循相同的模式：

1. 检查其他操作是否会以某种方式影响我们。
1. 如果影响，则返回一个考虑到该影响的新操作。
1. 否则，返回原始操作。


转换可能会变得复杂。 但它们在数学意义上非常实用，这使得它们易于测试。 这些函数必须满足一些数学特性。    

1. 如果您有两个状态相同的文档...
1. 然后应用第一个操作，然后应用转换后的第二个操作……
1. 您最终会得到相同的文档，就好像您应用了第二个操作，然后是转换后的第一个操作。


那是一中表达。但要形象化它的真正含义，请看下面这个方块，从左上角开始。从那里开始，如果您先选择顶部箭头，然后选择向右箭头，您最终会得到与您选择向左箭头和底部箭头相同的文档。

![image.png](https://atlas-rc.pingcode.com/files/public/60eba783f6d53d7b015c5013/origin-url)

如果你正确地改变事物，无论你走哪条路，你最终都会到达同一个目的地。数学使测试转换函数变得更加容易。您可以生成一大堆随机操作，将它们相互转换，将它们应用到文档，并且——只要文档最终相等——你就知道这些转换函数是有效的。

因此，即使转换函数可能会变得复杂，但要确保它们起作用并不难。

现在，这些方图只有在有两个客户端同时发送操作时才真正起作用——对吗？您实际上只能让两个箭头从左上角射出并到达左下角。如果你有三个客户，你会得到三维图，如果你有四个，你会得到四维图，依此类推。并且通过这些图表的每条路径都必须以相同的状态结束。

但是如果你有一个单一的事实来源，一个中央服务器，这一切都会变得容易得多。您有几个二维图而不是三维图——每个客户端-服务器连接一个。客户之间不直接交谈。他们通过服务器交谈。 （作为 Rails 开发人员，我们大多数人都相当习惯于依赖后端服务器。）所以从现在开始，让我们假设我们有一个服务器并且我们的操作通过它。

**什么时候需要转换操作？**

我们刚刚谈到了转换函数。这些函数转换操作，因此您最终得到的操作序列都将在同一个文档中结束。但是您还需要另一条信息。

我们仍然需要知道要转换哪些操作。为此，我们有一个控制算法。为了弄清楚这一点，算法需要知道两个文档是否相同，这样我们才能判断两个操作是否同时发生。

由于我们正在与服务器交谈，因此这很容易 - 服务器是您的真相来源。它可以为每个文档提供一个唯一的版本号，并使用该编号来判断两个文档是否相同。

一旦我们有了文档版本，我们就可以跟踪每个操作发生在哪个文档版本中。因此我们将文档的版本号添加到我们创建的每个操作中，如下所示：

```
→ operation
{
  type: :insert,
  text: “r”,
  position: 1,
  version: 2
}
```

假设我们的“at”示例是版本 2。我们有两个客户端在同一个文档上运行操作（“insert C”和“insert r”），因此它们也得到版本 2。

通过这种方式，您可以判断这些操作是同时发生的，并且您知道需要对它们进行转换。应用每个操作后，我们最终会得到一个新的文档版本。

但是，如果您在同步之前更进一步，会发生什么？如果两个客户端在相互交谈之前分别运行两个操作，而不是一个操作呢？

**转换多操作**

就像我们之前的例子一样，如果你画一个正方形，这样你就可以更容易地形象化，这样你就可以看到发生了什么。

![image.png](https://atlas-rc.pingcode.com/files/public/60ebac96f6d53d3bfc5c5014/origin-url)

您在顶部有一些箭头，在左侧有一些箭头，并且您想用右侧的箭头和底部的箭头来完成正方形。您可以使用您已经编写的相同转换函数。

但这里有一个技巧。因为这不是一个正方形。正如你在下面看到的，它实际上是四个。

![image.png](https://atlas-rc.pingcode.com/files/public/60ebad2ef6d53db9545c5015/origin-url)

这非常重要，因为一个正方形的右侧成为下一个正方形的新左侧。

![image.png](https://atlas-rc.pingcode.com/files/public/60ebad6bf6d53d80e45c5016/origin-url)

This is a little brain-bending. So, do not worry if it does not really sink in at first. It took years for people to find and fix this problem.

这有点伤脑筋。 因此，如果它一开始没有真正理解它，请不要担心。 人们花了数年时间才发现并解决这个问题。

请记住，您必须填写这些方格中的每一个。 正方形的每一边只能进行一次操作。 对于每个方块，从顶部然后右侧移动必须产生与从左侧然后底部移动相同的值。

所以你的转换算法有点像这样：

1. 取两个操作列表：顶部列表和左侧列表。
1. 创建两个空列表：右侧列表和底部列表。
1. 将第一个 top 操作和 first left 操作转换为底部和右侧的值。
1. 将底部值推到底部列表中。




- 保留右边的值（我称之为“tranformed left”），因为接下来您将使用它。


![image.png](https://atlas-rc.pingcode.com/files/public/60ebb087f6d53d5aec5c501b/origin-url)

- 然后，变换下一个 top 操作和“transformed left”操作。 （你一直抓着的那个。）
- 将返回的底部操作推到底部列表中。
- 如果你有更多的顶部元素，你只需不断地将右边的元素变成新的左边元素，然后继续前进。




- 到达一行末尾后，将最后一个正确的值推送到您的右侧列表中。


![image.png](https://atlas-rc.pingcode.com/files/public/60ebb126f6d53d26715c501e/origin-url)

现在您的右侧列表中有一个元素和一整行底部操作。将底部列表转换为新的顶部列表，并使用左侧列表的第二个元素重复整个过程。最终，您会一直走到底部并完成两个列表。

![image.png](https://atlas-rc.pingcode.com/files/public/60ebb18af6d53d0abf5c501f/origin-url)

在代码中看到这一点可能会有所帮助：

```
def transform(left, top)
  left = Array(left)
  top = Array(top)
  
  return [left, top] if left.empty? || top.empty?

  if left.length == 1 && top.length == 1
    right = transform_operation(left.first, top.first, true)
    bottom = transform_operation(top.first, left.first, false)
    return [Array(right), Array(bottom)]
  end

  right = []
  bottom = []

  left.each do |left_op|
    bottom = []
    
    top.each do |top_op|
      right_op, bottom_op = transform(left_op, top_op)
      left_op = right_op
      bottom.concat(bottom_op)
    end
    
    right.concat(left_op)
    top = bottom
  end

  [right, bottom]
end
```

首先要做的是确保我们只处理数组以使以后的代码更简单。这样，对于我们的控制算法的其余部分，我们只考虑转换操作列表。

接下来，我们处理一些简单的情况。如果我们的左侧列表或顶部列表为空，则意味着我们不必做任何事情。从用户的角度来看，这可能是当你是唯一一个做出改变的人，或者你离开办公桌而其他人在做改变的时候。没有什么可以改造的。

如果您只是将一个操作转换为另一个操作，这与您之前看到的简单正方形完全相同。您将左操作转换为顶部操作以获得正确的操作，然后反向执行以获得底部操作。

现在是棘手的部分 - 当我们连续进行多个操作时。第 13 和 14 行创建了一些空数组，以便在我们得到它们时挂在我们的转换操作上。对于第一行，我们遍历顶部的每个操作。

我们通过递归调用这个转换函数来转换它们——通常，这会遇到这两种简单情况之一，所以不值得考虑太多。它返回新的转换操作，我们将保留这些操作。

我们恢复了正确的操作。 请记住，我们下次使用该操作作为新的左操作。 所以让我们将右操作设置为左操作。 接下来，我们将返回的底部操作添加到底部列表中。

一旦我们完成了一整行，最后一个操作就是我们最终得到的操作，因此我们将其添加到我们的右侧列表中。这使用了 left_op，但此时，left_op 和 right_op 是相等的。

然后，在下一次循环中，我们的底部操作列表成为新的顶部列表，我们继续进行。这就像我们之前看到的第二次迭代一样。

并且，当这完成后，我们将右侧和底部列表返回给用户。小菜一碟吧？

现在，一个好处 - 您可能不必自己编写。那是因为控制算法是通用的。您可以为各种不同的应用程序使用相同的功能，而不必更改它。你的控制算法根本不关心你的操作实际做什么。

您的运营实际上应该做什么？无论您的应用需要什么！

只要您可以编写不违反转换属性的转换函数，您就可以整天发明新的操作。这很棒，因为您可以越来越接近代表一个人实际在做什么。

但就像所有事情一样，有一个权衡。您拥有的操作越丰富，您往往拥有的操作就越多。并且您拥有的操作越多，您需要编写的转换函数就越多，而且越难使它们正确。

当我解决这个问题时，我有 13 种不同的操作，最终我编写了一百多个转换函数。 但是有更具体的操作意味着当两个人编辑文档的同一部分时，我可以保持一些非常强烈的用户意图。

**如何让协作更轻松？**

您可以做一些事情来使编写协作编辑器变得容易，而其他一些事情则几乎不可能。

1. 考虑运营，而不是状态变化：如果您计划转变运营，您必须从运营的角度来考虑——一个人可以采取的行动。如果您只存储完整的文档状态，那么您的前路可能会很艰难


有办法解决这个问题。您有时可以查看文档的差异并从中推断操作。但是这样你会失去很多用户意图。想想“在位置 1 插入 t”而不是“文档从 a 更改为 at”。

1. 保持线性：如果您可以将文档视为一组事物（字符、丰富的对象等），那就容易多了。 转换数组索引只是加法和减法。 有时您也可以线性地表示树。 只需在数组中包含表示“进入子树”或“退出子树”的项目。 在这种情况下，它仍然很容易转换，但使用起来有点困难。


字符数组比分层数据更容易转换，但如果必须转换树，这并不是最糟糕的事情。 除了使用索引，您还可以使用索引数组。 例如，这个节点可以通过路径 [1, 1] “child 1 of child 1”到达。

![image.png](https://atlas-rc.pingcode.com/files/public/60ebb890f6d53d51bb5c5021/origin-url)

1. 使您的数据尽可能可转换：字符串可以很容易地转换。 你可以找出一些与数字有关的东西，比如将它们相加。但是，如果您对自定义对象进行了相互冲突的编辑，那么您的决定就会困难得多。


**这如何配合？**

我们有文档状态。 （为了简单起见，我们称它们为字符数组。）文档也有一个版本。每个客户端以及服务器在某个时间点都有一份文档副本。

您可以立即对自己的文档应用操作，因此无需等待查看。然后您将其发送到服务器，该服务器将其发送给其他客户端。

![image.png](https://atlas-rc.pingcode.com/files/public/60ebbb3df6d53d98b25c5023/origin-url)

有时，服务器会说，“那很好，我还没有看到任何新的操作，我的版本和你的一样。”它将确认您的版本，您更新您的文档版本，并且每个人都保持同步。

![image.png](https://atlas-rc.pingcode.com/files/public/60ebbb66f6d53d5a285c5024/origin-url)

其他时候，服务器会说，“我无法进行该操作，因为我看到了不同的文档。但这里是你的版本和我的版本之间的所有操作。”

当这种情况发生时，您将这些操作转变为您的操作，因为从您的角度来看，您的操作已经发生了。 记住？ 用正确的方式执行它。 然后，您将转换后的服务器中的操作应用到您的文档。

之后，您针对所有服务器的操作转换您的操作，因为从服务器的角度来看，您的操作发生在他们的操作之后。服务器还没有看到你的。然后您将转换后的版本发送回服务器——希望这一次，服务器会接受它。该过程如下所示。

![image.png](https://atlas-rc.pingcode.com/files/public/60ebbc64f6d53d8f2c5c5027/origin-url)

现在，一切都是一致的。如果同时发生多个操作，我们必须做更多的工作，但思路还是一样的。

**你还需要什么？**

当您转换操作时，您可以构建一个可以同时处理多人编辑的文本编辑器。 但这还不足以真正提供出色的体验。 我会告诉你两个原因。

首先，如果您有几个人在编辑同一个文档，那么在用户看来，字母和单词似乎是凭空出现的。 用户不知道在什么地方会发生变化，或者在它发生之前将要发生什么。 如果其他编辑文档的人的光标是可见的，这样用户就会知道会发生什么，那就太好了。

其次，假设用户在打字时犯了一个错误并点击了撤消。文本编辑器可以撤消两种不同的更改： 它应该撤消您所做的最后一次更改吗？或者任何人所做的最后一次更改？

让我们将您只撤消自己的更改的场景称为“本地撤消”。第二，您可以在其中撤消其他人的更改“全局撤消”。

如果您尝试过具有这些不同撤消风格的文本编辑器，您很快就会明白本地撤消感觉很正常。如果您键入一个字符，撤消应该删除该字符，无论其他人后来键入了什么。要拥有出色的协作编辑器，我们需要添加光标报告和本地撤消。

**光标同步**

让我们从一个哲学问题开始。什么是光标，真的吗？如果您的文档是一个列表，那么光标实际上只是该数组中的一个位置。

在下面的文档中，你有“你好”，我的光标在“e”之前。你可以说光标在位置 1。如果它在“o”之后，你会说它在位置 5。

![image.png](https://atlas-rc.pingcode.com/files/public/60ec29e6f6d53d7cf45c506a/origin-url)

其他人的光标呢？它们也可以是数字，但您可能还想知道谁的光标是谁的。您可以附加某种标识符。我们将只使用一个数字并将其称为客户 ID。因此，有两个数字：位置和客户 ID。

现在我们的文档有点复杂，但还不错。我们有我们的东西列表、一个版本、我们自己的光标偏移量和一个远程光标列表。这足以让您可以随心所欲地渲染文本编辑器和那些光标。

![image.png](https://atlas-rc.pingcode.com/files/public/60ec2a81f6d53d5c3d5c506b/origin-url)

添加字符或删除字符会发生什么？让我们回到我们的第一个例子，“cart”。假设我们正在看我们的屏幕，客户端 2 将光标留在位置 2，在“a”和“r”之间。

![image.png](https://atlas-rc.pingcode.com/files/public/60ec2b04f6d53d30245c506c/origin-url)

然后我们运行操作，“在位置 1 插入 h”。现在我们有了“图表”。它应该在哪里为客户端 2 绘制光标？将它保持在原来的位置是最有意义的——在“a”和“r”之间，对吧？

所以我们可以假设“将光标放在这个位置”是一个操作，我们将它转​​换为我们的“在位置 1 插入 h”操作，如下所示。我们在光标前插入了一个字符，因此我们将它移到一个位置上。

![image.png](https://atlas-rc.pingcode.com/files/public/60ec2b9ef6d53d09b15c506d/origin-url)

这是我们的规则：无论何时执行操作，您都需要针对该操作转换您所知道的所有光标，以将它们保持在正确的位置。 这些转换往往非常简单——大多数与 insert_text 转换完全相同，因为您只是在两者中移动一个位置。

一旦您从另一个客户端收到一个光标，您就需要知道另一条信息。该光标来自哪个版本的文档？

如果光标位于您的客户尚未看到的文档版本上，则您的客户无法绘制它——因为您没有该文档。光标可能指向位置 15，但您的文档只有 10 个字符。因此，您可以稍后保留光标（如果您愿意）或忽略它并希望其他客户端稍后再次发送它。

如果光标位置来自旧版本，它也可能不适用于文档的当前版本。发生这种情况时，您可以在该版本之间的所有操作中转换光标。例如，如果您的文档是版本 2 并且您看到版本 1 光标，您可以将其转换为将您的文档从版本 1 转换为版本 2 的操作。或者，如果您希望看到更新的游标，也可以忽略它很快就好了。

如果光标在同一版本上，您可能认为这很清楚。它是……但前提是服务器已确认您的所有操作。但是，如果您在版本 15，并且另一个客户端的光标在版本 15 上，但是您运行了尚未发送到服务器的插入“h”操作？嗯，看起来像这样。

![image.png](https://atlas-rc.pingcode.com/files/public/60ec2e35f6d53d8e4e5c506e/origin-url)

您必须针对该操作转换其他客户端的光标。该客户端在位置 3 向您发送了一个光标，您必须将其移动到位置 4。

发送光标怎么样？您可以随时发送光标，只要您没有进行服务器尚未确认的任何更改。否则，您的光标可能对客户没有意义，他们将不知道该怎么做。服务器确认您的操作后，您可以再次开始发送光标。

**协同 Undo**

Just like with cursors, to figure out how to handle local undo, we have to understand how undo usually works. Remember, we are thinking in operations — “insert ‘a’ at position 3.”

How would you undo that? You would run the operation, “remove ‘a’ at position 3.” How would you redo? You would run the operation, “insert ‘a’ at position 3.”

These two operations are inverses of each other — they cancel each other out. If you run an operation and then run its inverse, it is as though the original operation never happened. Which is exactly what you want with undo.

Undo also works like a stack. The last thing you did is the first thing you undo.

So, if our text editor was not collaborative, here is how you would apply an operation with undo:

- You perform an operation, like “insert h before position 1.”
- You invert that operation, so it becomes “remove h at position 1.”
- Then, you push that inverted operation onto the undo stack.


What about when you hit undo?

- You pop the operation (“remove h at position 1”) off the stack.
- You apply it as if you were performing it to begin with.
- Then, if you want to support redo, you invert it again and push the inverse onto the redo stack.


Simple enough, right? Let’s see how that breaks when other people are collaborating with you. You run “insert s, 4” — that pushes “remove s, 4” onto the undo stack. And you send the insert to the server.

![image.png](https://atlas-rc.pingcode.com/files/public/60ec3160f6d53dec385c506f/origin-url)

A little bit later, the server sends you the operation, “insert h at 1” — this is not happening simultaneously, so you do not have to transform it. Now our state is “charts.”

![image.png](https://atlas-rc.pingcode.com/files/public/60ec317ff6d53d75d05c5070/origin-url)

Now look at our undo stack. What would happen if you hit undo? You would run “remove s at 4” — but there is no “s” at position 4, right?

![image.png](https://atlas-rc.pingcode.com/files/public/60ec31eaf6d53dcb4b5c5071/origin-url)

Clearly, we are missing a step. When you get the operation from the server, you need to transform all the operations in your undo stack against that operation. So, the undo stack is “remove s, 4.” We receive “insert h, 1″ and have to transform the undo stack so it looks like “remove s, 5.”

Now, when we undo, we run “remove s at 5” it deletes the “s” at position 5 and everything is great.

When you receive an operation, you have to transform the undo stack against that operation. Luckily, we already have a function (that big transform one from earlier) that is really good at transforming lists of operations against other lists of operations.

We can just use this:

```
def transform_stacks(remote_op)
  self.undos, _ = transform(self.undos, remote_op)
  self.redos, _ = transform(self.redos, remote_op)
end
```

Here is how collaborative local undo would work then:

- When you perform an operation, invert it and push it on the stack.
- When you receive an operation, transform the stack against it.
- When you undo, pop the top item off the stack and run it, sending it to the collaboration server.


This mostly works, but it is not perfect. In fact, it can violate some rules that you should have with undo. For example, if every client undoes a set of operations and then redoes them, the document should be in the same state as it was originally. Sometimes, with this method, it is not.

But this is a pragmatic balance between complexity and good-enough behavior. And I am not the only one who thinks so — almost all collaborative text editors that I have used, including Google Docs, can fail undo in the exact same ways.

Putting it all together

The following is enough to make collaboration work with any kind of app. You start with a document, which can be as simple of an array of things, a version, a cursor, a list of remote cursors, and an undo stack. You have operations that act on that state, such as insert character and remove character. These operations know which version of the document they came from.

You have a set of transformation functions, which take two operations that happened at the same time and transform them so they can be run one after the other.

You have a control algorithm, which can take two lists of operations and transform each side against each other to come up with documents that end up in the same place. You have functions to transform cursors and functions to send and receive cursors, transforming them on the way in.

And you have an undo stack and a redo stack, which hold inverted operations that get transformed whenever a remote operation comes in.

When you perform an operation, you:

- Apply it to your document.
- Transform all the cursors against it.
- Send it to the server.
- Send your current selection once everything calms down.


When you receive an operation, you:

- Transform your pending operations against it to complete the transformation square.
- Apply the transformed operation to your document, and send your pending transformed operations to the server.
- Transform all the cursors you know about against the operation you received and transformed.
- Transform your undo stack against it as well.


When you change your cursor position and you have no pending operations:

- Send your current cursor position.


When you get a cursor from someone else:

- If the cursor is for an older version of the document, either ignore it, or transform it up to your current version.
- If it is for the current version of the document, transform it against any pending operations.
- If it is for a future version of the document, either ignore it or hold onto it until you see that version of the document.


I have a   [demo](http://justinweiss-editor.herokuapp.com/)   that puts all this together, which you can play with.

Where to go next

There are a lot of ways to build collaborative applications, but this is a good one to start with. It works for all different kinds of apps, it is not too hard to build, and it is extremely flexible. It is a model you will see a lot of companies use.

But it is not perfect because:

- This model needs a server to work.
- There are some edge cases, especially around undo, that would add a lot of complexity if you want to fix them.
- Depending on what you are building, there are other collaboration methods that might be easier or more correct.


If you want to build peer-to-peer collaboration that does not rely on a central server, take a look into conflict-free replicated data types (CRDTs). Same thing if you are just dealing with plain text — CRDTs tend to be great at that. CRDTs are newer collaboration methods that fit some specific kinds of text editors really well and they are getting even better.

If you are using operational transformation and you do not want to write the server or control algorithm yourself, take a look at   [ShareDB](https://github.com/share/sharedb)  . If you want to check out CRDTs,   [Y.js](http://y-js.org/)  ,   [Gun](https://gun.eco/)  , and   [Automerge](https://github.com/automerge/automerge)   are all really cool projects.


现在，我喜欢我们可以在 Aha 完成我们的工作！ 远程。 团队中的每个人都在家庭办公室工作——整个公司是完全分布式的。 我喜欢远程工作变得越来越流行。

这也让一些事情变得更加困难。 在一个项目上一起工作可能很困难。 最糟糕的是，当事情变得困难时，这些项目有时根本不会产生。 我喜欢能够让一个团队聚在一起完成比我们自己更大的事情。 但我不想担心做一个小的改变会破坏你的大事。

协同编辑是一种神奇的体验。 突然之间，这东西不再是你的，而是我们的。

我有信心做出改变，因为我的贡献不会与你的冲突。 我希望这种魔法无处不在，即使我不一直使用它。 因为在同一件事上工作的两个人应该让它变得更好，而不是更糟。

Ever since I joined the Aha! team, I have worked on some truly interesting projects. And I have only worked on a few of the many, many, many interesting projects we have going on at Aha!

So you like solving cool problems for great customers and you want to work for a fast-growing, remote, and profitable software company?   [We are hiring](https://www.aha.io/company/careers/current-openings)   and I would love to collaborate with you.