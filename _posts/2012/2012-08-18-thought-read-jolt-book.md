---
layout: post
comments: true
title: "书评-Jolt大奖精选丛书"
description: "在oschina写的书评"
categories: ["书评", "jolt"]
---

![Jolt大奖精选丛书][2]

**2012-08-18@oschina.net**
[书评链接][1]

**这次书评活动有6本书，写书评的确花了不少时间呀！**

###《编写有效用例》书评###
刚毕业的时候，在公司就看到这本书，当时看了一下，不过不怎么懂。
只是隐约知道有用例这么个概念，后来慢慢地接触到业务用例，测试用例等内容，才逐渐对用例有了一些理解。
再后来，因工作需要，经常使用UML来画用例图，小人的身影更是随处可见了。直到现在，虽然用例接触有不短的时间了，
不过用例这东西还真的没有非常系统的学习过。
       
没想到过了那么多年，又重新看到这本书，倍感亲切，让我有了系统学习一下的热情。
这本书和其他软件工程书籍不大一样，因为它专注于用例的编写。里边提供了很多写用例的原则，读完可以对用例概念有深入的认识。
另外，和很多书籍不一样的是，书中使用的例子都是作者实际项目中抽取出来的，而且例子数量非常的多，我非常喜欢真实饱满的例子
，因为这样才有现实指导意义。以前看过有些书，例子是简单的傻瓜化，遇到真实的场景显得苍白无力。
这本书，实战味道比较浓烈，学到一点用一点。即使是用例方面的新手，也很容易用得上。

另外，我注意到一点，书中强调了项目相关人员和利益模型，梳理系统用例与业务用例的边界，这些都是以前我不曾注意或不甚理解的地方。
我非常同意一书中的一个观点：我们需要的是有用的用例，而不必是一个最好的用例。
       
的确，实践总是我们的目的，学习的过程不要忘记练习一下。
对于我来说，有了这本书，以前的实践经验更容易得到修正，以后写用例一定会更加得心应手。

###《代码阅读》书评###
据我所知，《代码阅读》这本书是这个领域唯一的著作。
这本书我曾经读过两遍，我把书中的一些实践方式用于平时工作中，感觉收获很大。
       
首先，作者提供的例子都是现实存在的代码，并不是玩具程序。这样才让书中的例子更具说服力，作者提供的操作手法也更有借鉴意义。
例如我现在手头上的项目，不包括页面光java的代码就超过100w，还有数百万的c++代码，这些代码经历了很多人的摧残。
到现在，作为一个开发人员，每天都要面对着这么多代码，在这些代码上打转，努力去修复问题，添加新特性，
还需要每日和每周做code review，保证代码质量。阅读代码的能力显得格外重要，
因此我特别喜欢第6章和第9章关于大项目和架构方面的内容，让我有信心对付这些大块头项目。
        
另外，源代码开放活动已经成为软件业不可或缺的力量，而且现在成熟稳定的开源项目日益真多，每个人或多或少都会有所涉及。
一方面工作上的项目经常使用了这些开源代码，有时候出现问题需要定位，另一方面，通过开源代码，可以学习新的架构，
编程思想，优秀开发的技巧和经验，从而提高自己的能力。无论是什么样的情况，阅读开源代码是很有必要的。
而这本书，提供了阅读代码的最佳实践，能够帮助我们更快的理解这些代码。
        
当然，这本书不单单教导如何去阅读代码。我认为，书中关于编码规范，命名方式，文档方面的内容，从代码阅读的角度，
提出了自己的看法，所以当你进行编码规范规范制定等工作的时候，有一定的指导意义。如果你想立马学习代码阅读的技巧，
作为过来人，我觉得开头先研习一下书中第10章关于代码阅读工具的内容是个不错的开始，有工具的支持，边学习边操作，
理解书中的代码阅读技巧也会跟容易一些，乐趣也就多一些了。

###《持续集成：软件质量改进和风险降低之道》书评 ###
真正接触持续集成是2年前来到新公司的时候，那个时候公司请了ThoughtWork公司的咨询顾问来。
顾问给我们灌输了持续集成的概念，并使用CruiseControl帮助我们搭建了持续集成环境。
随着时间的推移，我们这里多个项目组都加入了持续集成的环境，并搬来了大电视让整个过程可视化，
并加了一些警示的声音，效果很是震撼呀。持续集成是个有趣的东西，
还制定了相应的军规：红灯停，绿灯行，黄灯停一停。(和电视上显示的项目状态的颜色有关)

正如书上说描述的，持续集成减少重复的过程，把编译，集成测试，部署等环节自动化，
的确节省了我们的时间，并避免了人工干预的风险。另外，我们在持续集成里边加入单元测试，
findbugs，checkstyle, cppunit等测试项，这样代码质量问题可以得到更快更好的反馈，经常因为有些问题代码被快速检测出来而感到庆幸。
       
样章只有持续测试这一章节的描述，项目的可靠性必须得通过持续不断的测试来保证。
正因为如此，我们在持续集成里边加入单元测试等功能之外，还使用自动化测试用例进行测试，
当然因为测试用例巨大，耗时较长，我们通常是选择在晚上进行的。
       
按照我们的理解，对项目有利的东西，只要能够自动化，都可以考虑加入持续集成，这将给项目带来更多的好处。

###《代码质量》书评###
工作了这么多年，也接触过各式各样的系统，看过千奇百怪的代码，知道代码质量的重要性对我们是多么的重要。
每每想起那些可怕的错综复杂的代码，还有那些线上千奇百怪的问题，总会深深感到不安，实在太让人揪心了。
所以在日常工作中，我非常强调代码质量，用各种培训去提升开发人员的技能水平，用代码评审等手段试图保证代码质量。
       
现今随着硬件成本的降低，人力成本的提升，我平时考察考察代码质量多从可读性，可维护性，性能的角度出发，
正因为如此，Diomidis Spinellis这本书的目录让我眼前一亮，它从多个维度考察代码质量，
讨论了如何满足非功能性需求，我从未用如此全面的角度衡量代码的质量，的确充实了我的知识库，
让我在平时工作中用更全面的眼光衡量代码的质量。
      
另外，和代码阅读一样，书中的丰富的例子来源于现实代码，让作者的观点更具说服力。
最近团队来了不少新员工，在代码质量控制方面花了不少功夫。有时候，他们并不理解代码质量的重要性，认为代码能运行就可以了。
让新员工理解这些非功能性需求的重要性，能够在平时中注意考虑非功能性需求的，这都是非常重要而且非常迫切的需求。
我想，借助于这本书理论与实践的结合， 继续这项工作更有信心了。

###《面向对象分析与设计》书评###
这是一本关于面向对象分析与设计的书籍，通常和对象打交道的书，几乎都是在写设计模式一样，
当我看到这本书的时候，第一反应就是这样。因为我使用面向对象语言进行开发设计已经有一段时间了，
除了设计模式方面的书籍，真的很少碰到面向对象方面让人耳目一新的著作。但在浏览了书本的目录和前言部分之后，
我对这本书有另外一个感觉: 这是学习掌握UML的一本绝佳的书籍。正是这一点吸引我继续阅读下去。
       
本身作者Grady Booch就是UML的创始人之一，书中也有大量关于UML图的内容，由他使用UML来讲述再好不过了。
另外，UML里边类图，状态图等，本质上也离不开面向对象理论的支持。理论与实践相结合，这是很多书籍做不到的地方，
也是我感觉使用面向对象进行分析设计经常遇到困难的痛处。所以，即使我使用面向对象进行设计开发，
也用过UML画过各式各样的图，但书中的内容仍然吸引着我。
   
这本书提供的样章相当的多，阅读花了不少时间。书中介绍了面向对象的各种概念，
并运用了大量的例子展示了面向对象设计的经验技巧，这些都是很难描述清楚的东西，
而且容易变得很枯燥的东西，庆幸的是，作者在这方面的文笔相当好，阅读起来非常顺畅。

###《灾难拯救：让软件项目重回轨道 》书评###
终于看完这本书的样章了！我不是一个项目管理人员，但我觉得了解一些项目方面的东西是有好处的。
我面对的也是一个不是很顺利的项目，有着比较纠结的代码基，人员众多而且人员流动也比较大，线上问题也时不时的出现。
虽然采用了一些敏捷开发的手段控制进度，但也不是特别的顺利。要命的是，这是一个需要长期维护发展的关键项目，不能随意推到重来。

我认为这本书对于我们这种类型项目来说，还是有相当的指导意义的 。
如前言说的，这是一本救治之书，它探讨了像我们这种随时面临失败的项目，如何去拯救，如何重回正轨。
书中提到面临失败项目，实施拯救需要的10个步骤。很显然，有些步骤我们已经在现实中采用了，
并且也取得了一些成果：例如确定最低目标，风险分析等内容。但能够完整描述这些步骤，
也真的是不容易做到，有了这本书的指导，我相信实施这些步骤是具有可操作性的。

整本书有一个顺序脉络在那里，就是步骤的描述，所以应该从头到尾的读会比较合适，因为每个步骤是有一些关联的，
如果跳来跳去，可能不能深入理解步骤的真正意义。最后一章是整本书的总结，把前面描述的10个步骤整合在一起，
并对时间安排，实施拯救计划进行指导。对于担心书中描述的理论是否可行的人来说，
这章无疑是打了强心针一样，只要按照书中的建议进行操作，应该是能够验证效果的，需要的只是：行动。 

 [1]: http://www.oschina.net/question/262659_63418?sort=default&p=3#answers
 [2]: /assets/images/jolt_book.png

{% assign series_list = "书评" %}
{% include series_list.html %}
