# A State-transition Framework to Answer Complex Questions over Knowledge Base

## Abstract 

文章提出了一种基于状态转移的方法来实现将一个复杂的的自然问句转化为查询的sparql语句。四种op和一种基于学习的转化方法。

## 1. Introduction

- Target

  - Flexiable strategy to answer complex questions
  - without handcrafted templates
  - without restricting the query graph's structure

- challenges

  - Multi or implicit relations—很多方法只能抽取一种关系，以及对于没有关键词标记的隐含关系难以识别

  - Multi or no entities

  - variablle and co-reference

  - Composition

- 生成SQG

  - 先识别node（variable 和 entity）
  - 然后使用ops with conditions 来进行state transistion
    - 状态转移使用的是基于SVM的排序算法来进行下一状态的选择。

    

## 2. Semantic Query Graph Generation

### 2.1 Node Recognition

- 节点类型：entity node， type node，literal node， variable node

- 识别方法

  - 使用序列标注的方法来识别各种类型节点，识别entity/type node效果不好
    - 实体node会出现比较复杂和比较长的情况不利于识别
  - 本文采用了两种方法结合

    - 针对entity/type nodes，

    - 针对variable/literal nodes，使用blstm-crf方法。

### 2.2 SQG’s Structure Construction

贪心算法：考虑到每一步骤去查找搜索空间的时候，都是指数级别的搜索空间，因此我们无法通过枚举的方式来进行处理，文章采用了一种贪心的算法，一个好的中间状态倾向于有更大可能被访问，同时也会有更多的后续状态。

- connect
  - 链接两个节点，当两节点之间的关系在知识库中存在这个关系
- merge
  - 主要是为了解决一些共指的问题
- expand
  - 解决query中存在隐含信息
- fold
  - 解决冗余的关系，实现关系合并，从而匹配知识库中的关系

#### 减少搜索空间

为了减少搜索的空间，需要加一些限制条件

- 对connect的限制

  - 当两节点的根据依存句法分析的最短路径之间没有其他节点，此时可以尝试connect两个节点

- 对merge的限制

  - 只有当两个节点中有疑问词，因为疑问词可以出现共指的情况

- 对expand的限制

  - 操作的节点必须是一个variable节点，因为只有variable node需要被解析

- 对fold的限制

  - 操作节点的至少存在一个边，在知识库找不到对应的关系

### 2.3 Finding entity/relation candidates and matches of SQG

- 实体抽取

- 关系抽取

  - 给定两个节点，我们需要抽取两个节点之间的关系，文章使用MCCNN的方法利用句法和句子的信息。

  - 第一个信道的信息是两节点之间依存句法树的信息。

  - 第二个信道的信息，整个句子除去所有节点外的信息，这里包含了所有的relation信息。

  - 针对两节点是相邻节点的情况，我们会在第三个信息将两节点以及其类型信息作为输入

  - 这里可以抽取多种的关系。

## 3. Learning the reward function

SQG的一个中间状态采用不同的op会转换到不同的后续状态，文章通过一个激励函数的输出最高的那个后续状态作为下一个状态。文章中采用的是一个线性分类模型。

### features

- Nodes

  - 将最高的confidence作为当前节点的confidence
  - 所有实体的平均confidence
  - 当前节点中constant nodes和variable nodes 的个数

- Edges

  - 将最高的confidence作为当前边的confidence
  - 问句中所有的边的个数  

- Matches

  - 当所有的后续状态的confidence都比较低的时候，会terminate

    


​    