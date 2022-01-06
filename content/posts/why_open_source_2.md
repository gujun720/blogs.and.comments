---
title: "开源软件商业模式的探讨"
date: 2019-10-18T12:00:00+08:00
draft: false
tags: [开源,商业模式]
categories: [正文]
---

> 声明：我们的开源项目“ Milvus 向量搜索引擎”还处在社会主义初级阶段。以下内容是我们目前对开源工作的摸索，并非最佳实践。

## 开源许可证

既然我们决定了要开源，第一步便是要选择合适的开源许可证。虽然自由软件创始人 RMS 曾经倡导 Copyleft 概念，但 Copyleft 也是一种特殊的 Copyright 。

那么，什么是开源许可证？简单来说，一个许可证只要经过 OSI （ Open Source Initiative ）认证，就可以被称之为开源许可证。 OSI 有专门的流程来审核一个许可证是否符合开源定义（ [Open Source Definition](https://opensource.org/osd) ）。比如说， MongoDB 新设计的 SSPL （ Sever Side Public License ）在完成 OSI 认证之前， MongoDB 只能说自己的许可证是源码可用（ source available ），而不能说自己是开源（当然，这个限制属于行业惯例，没有强制性）。

目前主流的开源许可证，可以在 [OSI 网站](https://opensource.org/licenses)上查询到。网上也有很多文章去比较各个许可证之间的不同（[可参考阮一峰老师的博客](http://www.ruanyifeng.com/blog/2011/05/how_to_choose_free_software_licenses.html)），我就不一一赘述了。这里主要结合我们的自身情况来谈一下开源许可证的选择。开源许可证简单来说，可以分为三档：

- 严格，以 GPL 2.0 许可证为代表，典型软件是 MySQL
- 适中，以 Apache 2.0 许可证为代表，目前使用最广泛
- 宽松，以 BSD，MIT ， PostgreSQL 许可证为代表，典型软件是 PostgreSQL

熟悉数据库的朋友一定知道 MySQL 和 PostgreSQL 。 MySQL 是最流行的开源数据库，但 PostgreSQL 是衍生项目最多的开源数据库。现在的新项目很少使用 GPL 2.0 许可证，它的传染性应该是大家最有顾虑的地方。

对于推广基础技术来说，MIT/BSD 类的许可证是一个好选择。可能现在已经很少人使用 FreeBSD 。但它也还在不断的发展，因为采用非常宽松的 2-Clause-BSD 许可证， FreeBSD 被不少厂商用来开发自己的闭源系统。比如， Sony 的 Play Station 3 和 4 的系统都基于 FreeBSD ， 还有任天堂的 Swtich 游戏机也是。

Redis 也采用宽松的 3-Clause-BSD 许可证（相比 2-Clause 多了对商标的使用限制）。不过， Redis 整个工具链的许可证情况十分复杂。以至于当 Redis 切换部分组件的许可证时，引起了业界很大的误解。因此中途将许可证变严格是件有点敏感的事情。

![redis](/img/redis.png)

> 看起来颇为复杂的 Redis 许可矩阵。

如果上策太急，下策太缓。那么就选择中间的 Apache 2.0 。Apache 2.0 目前是 Apache 基金会与 CNCF 基金会推荐的默认开源许可证。

![apache](/img/apache2.png)

> Github 网站对 Apache 2.0 许可证的简易说明

Apache 2.0 像其他开源许可证一样不限制商业使用，专利授权也默认包含其中。不过 Apache 2.0 也明确规定了在此开源许可证下软件厂商的免责条款。这也就是开源软件公司提供订阅增值服务的法律基础。

不过即使是 Apache 2.0 这么成熟的开源许可证，大家还是有一个担心：公有云。

## 需要防范公有云厂商吗

开源软件与公有云的关系这两年有点紧张，一个比较流行的观点是公有云插管吸血开源软件，而对开源社区没有太多贡献。不少开源项目开始寻找在公有云面前保护自己的方法。毕竟公有云的出现，一定程度上打乱了原有的开源商业模式。最终用户通过购买云服务，从公有云服务商那里的到了保障，开源厂商被绕开了。

于是， [Common Clause](https://commonsclause.com/) 应运而生。 Common Clause 是一种附加条款，开源厂商依然需要选择一个基本的主许可证。最终的形式类似： Apache 2.0 + Common Clause 1.0 。

Common Clause 比较精炼，全文只有 3 句话，如下：

```text
The Software is provided to you by the Licensor under the License, as defined below, subject to the following condition.

Without limiting other conditions in the License, the grant of rights under the License will not include, and the License does not grant to you, the right to Sell the Software.

For purposes of the foregoing, "Sell" means practicing any or all of the rights granted to you under the License to provide to third parties, for a fee or other consideration (including without limitation fees for hosting or consulting/ support services related to the Software), a product or service whose value derives, entirely or substantially, from the functionality of the Software. Any license notice or attribution required by the License must also include this Commons Clause License Condition notice.
```

Common Clause 主要禁止他人在不增加开源软件价值的情况下，利用开源软件牟利。它的限制性主要体现在以下三点：

| 禁止他人 | 对软件生态构建的影响 |
| ---------- | ---------------------- |
| 利用原软件进行 license 销售 | 非常小。目前主流观点是开源软件不收取 license 费用，软件收费以按年支付的订阅模式进行。 |
| 提供软件支持服务 | 比较小。尤其是产品初期，大家都不了解，需要依靠官方支持。官方提供订阅模式，大家比较容易接受。产品的订阅收费模式能够得到保障。<br/><br />不过将来等产品成熟以后，需要考虑参照 Oracle 的 OCP 方式，提供一些针对专业人士的产品能力认证，以缓和这一点可能的影响。 |
| 提供软件托管服务 | 防范云厂商。限制云厂商不能通过托管云服务（ hosting ）的方式，利用原软件进行盈利。<br/><br />但不代表用户不能在云环境里使用该软件。Apache 2.0 + Common Clause 1.0 不会限制用户运行软件的硬件环境，只要用户购买订阅服务，产品方一样会提供支持服务。 |

假设第三方在开源软件的基础上构建了一整套面向用户的应用，这套新应用增加了原开源软件的价值，那么这套新应用不会受到任何限制。这一点保证了开源厂商与合作伙伴之间的合作关系不会受到影响。

不过 Common Clause 没有经过 OSI 认证，因此添加了 Common Clause 以后建议只说自己是源码可用（ source available ）。虽然会引起一定的争议，不过初创开源项目选择添加 Common Clause 看起来正受到越来越多人的理解。

然而，我们的开源项目并不打算加上 Common Clause 。有两个重要的原因。

### MongoDB 的启示

MongoDB 是开源项目成功的范例。 MongoDB 一开始就采用 AGPL 3.0 许可证。如果公有云要利用 MongoDB 提供服务，那么公有云厂商需要公布相关底层服务的源码。因此， AWS ， Azure， Google Cloud 等一众美国公有云都选择自行开发文档型数据库。而在美国以外， MongoDB 却很难用法律武器保护自己。 2018 年 10 月 MongoDB 修改新版本的许可证时，再次抱怨了公有云厂商对 MongoDB 利益的侵害，主要指的就是美国以外的公有云厂商。因此，志在全球的开源基础软件厂商其实很难仅靠一个许可证来对自己进行全面的保护。

另一方面，当 AWS 有了 DynamoDB ； Azure 有了 Cosmos DB ； Google Cloud 有了 Cloud Firestore 之后，文档数据库不再是 MongoDB 一家独大。在之后的移动互联网浪潮中，移动端的 MongoDB Mobile 没有达到期待中的影响力。毕竟 Realm 这样的移动端文档数据库可以直接和多个公有云文档数据库同步，极大的方便了移动开发者。2019 年 4 月， MongoDB 以 3900 万美元收购了 Realm 。

防范别人的同时也部分影响了自己的发展空间，是否值得？答案因人而异，开源项目需要结合自身情况作出一个选择。

### 四爷的新策略

据咨询公司 Gartner 的统计， Google Cloud 2018 年占据公有云 IaaS 市场 4.0% 的份额，排行全球第四。依然不及老大 AWS 市场占有率（ 47.8% ）的一个零头。 Google Cloud 想迎头赶上，他该怎么办？

在今年的 Google Cloud Next 大会上，新上任的 Google Cloud CEO 一举请来了 Redis Lab CEO 与 MongoDB CEO 帮忙站台。大会上 Google Cloud 推出了 Redis 的托管服务， MongoDB 上了 Google Cloud Marketplace 。后续 MongoDB 的 Atlas 云服务还和 Google Cloud 展开了一系列合作。

Redis 和 MongoDB 在开源界与互联网行业有较大的技术影响力。而且他们是开源界对公有云厂商开炮比较多的两家。近期他们又先后针对公有云厂商修改了自己的许可证。因此 Google Cloud Next 大会上传达的信息很有意思。 与成熟的开源厂商合作，看起来正是 Google Cloud 的新策略。这条路值得一试。毕竟，老四恐怕很难用老大的方法来战胜老大。

> “学我者生，似我者死。” —— 齐白石

Google Cloud 第一个想明白了。我相信会有越来越多的公有云厂商想明白这个问题，选择与成熟的开源厂商合作。所以对开源基础软件来说，当务之急是提升自身的成熟度，防范之心可以暂时放到一边。

## 商业设计

在上一篇文中，我们提到“ Apache 基金会拥有 1.9 亿行代码。根据 COCOMO II 模型估算，这些代码的开发成本超过 200 亿美元（ 2019 年报）。”如此算来，每一行代码的开发成本超过 200 美元。所以千万别觉得开源软件就该免费使用。

### 典型的开源商业模式

目前比较成熟的开源软件商业模式有以下几种：

- 订阅服务：开源许可证免除了厂商对软件质量与软件缺陷修复的责任。而这些都是企业级应用所必须的。因此，最自然的商业模式就是提供软件订阅服务，从而向用户提供生产级的服务支持响应和 hotfix 修复。
- 高级功能：比如 Redis 。核心部分的组件是开源的。但工具类软件，进阶功能（如多租户，无共享分布式架构等）都是收费的。
- 云服务：比如 Databricks 。 Spark 是开源的，但收费版本仅提供 Azure 和 AWS 上的云服务。
- 生态收益（仅限超大型开源厂商）：比如据华尔街分析师估算 Google 每年要支付近百亿美元给 Apple ，就为了 iPhone 上的默认搜索引擎入口。想想 Android 帮 Google 省了多少钱？

软件世界里有两个重大难题：一是大型软件系统的项目管理（人月神话），另一个是软件定价。关于项目管理，已经有了不少的研究与实践，大家多少有个参照物。而软件定价没有什么成熟的公式与模型。

但至少对于开源软件的定价，要避开下面两个坑：

- 定高价，打 1 折
- 不采用订阅模式

这些都是传统商业软件的模式。传统商业软件提供给客户的是资产，开源软件提供给用户的是**服务**。

如果大型用户要求对软件进行买断怎么办？大型用户倾向于一次性付费，并不是他们喜欢购买一堆软件资产。背后的原因在于大型用户内部的软硬件采购流程，需要采购人员与 IT 技术人员共同介入。而采购并不是技术人员的本职工作，以及事后的各种审计。因此技术人员更喜欢一次性买断，以省去未来的麻烦。请提醒他们，开源软件提供的是服务，服务是不能买断的，应该走更便捷的服务采购流程。

### 向 AWS 学习

基础软件的商业化是件很有挑战的事情。好在有很多成熟的企业可供我们参考。如果说 Oracle 是必须研究的传统商业软件公司，那么 AWS 毫无疑问就是必须好好学习的云服务公司。

刚才说软件定价没有什么成熟的公式与模型？其实 AWS 帮大家摸索了一个公有云上软件的定价方式。AWS Aurora 数据库据称是 AWS 上增长最快最赚钱的云服务。 Aurora 在技术上是非常创新的云原生数据库，带出了一众追随者。依据官方宣传：

> Amazon Aurora 的速度最高可以达到标准 MySQL 数据库的五倍、标准 PostgreSQL 数据库的三倍。它可以实现商用数据库的安全性、可用性和可靠性，而成本只有商用数据库的 1/10。（引用自 https://aws.amazon.com/cn/rds/aurora/ ）

那么这样一款技术如此先进的云上数据库是怎么定价的呢？以下对比 Aurora MySQL 所有可选的实例规格与 RDS MySQL 之间的定价：

| 云实例规格 | RDS MySQL | Aurora MySQL | Aurora 溢价 |
| -------------- | ---------: | ------------: | --------: |
| db.t3.small    | 0.034     | 0.041        | 20.59%   |
| db.t3.medium   | 0.068     | 0.082        | 20.59%   |
| db.t2.small    | 0.034     | 0.041        | 20.59%   |
| db.t2.medium   | 0.068     | 0.082        | 20.59%   |
| db.r5.large    | 0.24      | 0.29         | 20.83%   |
| db.r5.xlarge   | 0.48      | 0.58         | 20.83%   |
| db.r5.2xlarge  | 0.96      | 1.16         | 20.83%   |
| db.r5.4xlarge  | 1.92      | 2.32         | 20.83%   |
| db.r5.12xlarge | 5.76      | 6.96         | 20.83%   |
| db.r4.large    | 0.24      | 0.29         | 20.83%   |
| db.r4.xlarge   | 0.48      | 0.58         | 20.83%   |
| db.r4.2xlarge  | 0.96      | 1.16         | 20.83%   |
| db.r4.4xlarge  | 1.92      | 2.32         | 20.83%   |
| db.r4.8xlarge  | 3.84      | 4.64         | 20.83%   |
| db.r4.16xlarge | 7.68      | 9.28         | 20.83%   |

> 单位：美元/小时

当然 Aurora MySQL 和 RDS MySQL 的技术实现不太一样，同样实例规格需要的硬件也不能简单划上等号。不过考虑到 AWS 本身的体量，两者间硬件差异的成本应该是微乎其微的。可以大致认为 20% 的溢价来自 Aurora MySQL 软件。

基础软件上公有云 Marketplace 的时候怎么定价，总算有个参照物了。

## 后记

虽然写了两篇文章，但也只是涉及了开源的一小部分。开源模式可一点也不比传统商业软件的模式要简单。

其中比较关键的社区运营和开发者生态构建，我们也还在不断的摸索。等有一天我们形成自己的方法与风格，届时一定与大家分享。希望国内的基础软件同行都能一起进步。
