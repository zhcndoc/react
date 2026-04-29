---
title: 版本控制策略
---

<Intro>

所有 React 的稳定构建都经过高度测试，并遵循语义化版本控制（semver）。React 也提供不稳定的发布通道，以鼓励对实验性功能尽早反馈。本页面说明你可以对 React 发布内容有什么预期。

</Intro>

本版本控制策略描述了我们对 `react` 和 `react-dom` 等包版本号的处理方式。过去发布版本的列表请参见 [Versions](/versions) 页面。

## 稳定发布 {/*stable-releases*/}

React 的稳定发布（也称为 “Latest” 发布通道）遵循 [语义化版本控制（semver）](https://semver.org/) 原则。

这意味着，对于版本号 **x.y.z**：

* 当发布 **关键错误修复** 时，我们通过更改 **z** 数字来进行 **补丁发布**（例如：15.6.2 到 15.6.3）。
* 当发布 **新功能** 或 **非关键修复** 时，我们通过更改 **y** 数字来进行 **次版本发布**（例如：15.6.2 到 15.7.0）。
* 当发布 **破坏性变更** 时，我们通过更改 **x** 数字来进行 **主版本发布**（例如：15.6.2 到 16.0.0）。

主版本发布也可以包含新功能，而任何发布都可以包含错误修复。

次版本发布是最常见的发布类型。

我们知道用户会继续在生产环境中使用旧版本的 React。如果我们得知 React 中存在安全漏洞，我们会为受影响的所有主版本发布回溯修复补丁。

### 破坏性变更 {/*breaking-changes*/}

破坏性变更对所有人来说都很不方便，因此我们尽量减少主版本发布的次数——例如，React 15 于 2016 年 4 月发布，React 16 于 2017 年 9 月发布，而 React 17 于 2020 年 10 月发布。

相反，我们会在次版本中发布新功能。这意味着，尽管名字看起来不起眼，次版本发布通常比主版本发布更有趣，也更有吸引力。

### 对稳定性的承诺 {/*commitment-to-stability*/}

随着 React 随时间演进，我们会尽量减少利用新功能所需的工作量。在可能的情况下，我们会保持较旧的 API 仍然可用，即使这意味着要把它放到一个单独的包中。例如，[mixins 多年来一直不被推荐](https://legacy.reactjs.org/blog/2016/07/13/mixins-considered-harmful.html)，但它们至今仍可通过 [create-react-class](https://legacy.reactjs.org/docs/react-without-es6.html#mixins) 获得支持，而且许多代码库仍然在稳定的旧代码中继续使用它们。

超过一百万开发者在使用 React，共同维护着数百万个组件。仅 Facebook 的代码库就有超过 50,000 个 React 组件。这意味着我们需要尽可能让升级到新版本 React 变得简单；如果我们在没有迁移路径的情况下做出大改动，人们就会被困在旧版本上。我们会在 Facebook 内部测试这些升级路径——如果我们少于 10 人的团队都能独自更新 50,000+ 个组件，我们希望其他任何使用 React 的人也都能顺利升级。在许多情况下，我们会编写 [自动化脚本](https://github.com/reactjs/react-codemod) 来升级组件语法，然后将其包含在开源发布中，供所有人使用。

### 通过警告逐步升级 {/*gradual-upgrades-via-warnings*/}

React 的开发构建包含许多有帮助的警告。只要有可能，我们都会在为未来的破坏性变更做准备时添加警告。这样，如果你的应用在最新发布版本中没有警告，它就会与下一个主版本兼容。这使你能够一次升级一个组件。

开发环境中的警告不会影响应用的运行时行为。这样，你可以放心，开发构建和生产构建之间的应用表现会是相同的——唯一的区别是生产构建不会记录这些警告，而且它更加高效。（如果你发现并非如此，请提交 issue。）

### 什么算作破坏性变更？ {/*what-counts-as-a-breaking-change*/}

通常情况下，我们*不会*因为以下变更而提升主版本号：

* **开发警告。** 由于这些不会影响生产环境行为，我们可能会在主版本之间添加新警告或修改现有警告。事实上，正因为如此，我们才能可靠地对即将到来的破坏性变更发出警告。
* **以 `unstable_` 开头的 API。** 这些是作为实验性功能提供的，其 API 我们尚未有足够把握。通过以 `unstable_` 前缀发布，我们可以更快地迭代，并更早地得到稳定的 API。
* **React 的 Alpha 和 Canary 版本。** 我们提供 React 的 alpha 版本作为尽早测试新功能的一种方式，但我们需要足够的灵活性，才能根据 alpha 阶段学到的内容进行更改。如果你使用这些版本，请注意在稳定版发布前，API 可能会发生变化。
* **未文档化的 API 和内部数据结构。** 如果你访问诸如 `__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED` 或 `__reactInternalInstance$uk43rzhitjg` 之类的内部属性名，不提供任何担保。后果自负。

这一策略的设计是务实的：当然，我们不希望给你带来麻烦。如果我们把以上所有变更都提升为主版本，我们最终会发布更多主版本，并给社区带来更多版本管理上的痛苦。这也意味着我们无法像希望的那样快速推进 React 的改进。

话虽如此，如果我们预计这个列表中的某项变更会给社区带来广泛问题，我们仍会尽最大努力提供渐进式迁移路径。

### 如果次版本发布不包含新功能，为什么它不是补丁发布？ {/*if-a-minor-release-includes-no-new-features-why-isnt-it-a-patch*/}

次版本发布可能不包含新功能。[这是 semver 允许的](https://semver.org/#spec-item-7)，其中指出 **“[次版本] MAY be incremented if substantial new functionality or improvements are introduced within the private code. It MAY include patch level changes.”**

不过，这确实引出了一个问题：为什么这些发布不直接作为补丁发布来编号？

答案是，对 React（或其他软件）的任何更改都带有一定风险，可能会以意想不到的方式破坏功能。想象这样一个场景：某个修复一个 bug 的补丁发布，意外引入了另一个 bug。这不仅会打扰开发者，还会损害他们对未来补丁发布的信心。尤其是在原始修复针对的是一个在实践中很少遇到的 bug 时，这一点更令人遗憾。

我们在保持 React 发布无 bug 方面有相当不错的记录，但补丁发布对可靠性的要求更高，因为大多数开发者都认为它们可以在没有负面后果的情况下直接采用。

基于这些原因，我们只将补丁发布保留给最关键的 bug 和安全漏洞。

如果某个发布包含非必要更改——例如内部重构、实现细节变更、性能改进，或小型 bug 修复——即使没有新功能，我们也会提升次版本号。

## 所有发布通道 {/*all-release-channels*/}

React 依赖一个蓬勃发展的开源社区来提交 bug 报告、发起 pull request，以及 [提交 RFC](https://github.com/reactjs/rfcs)。为了鼓励反馈，我们有时会分享包含尚未发布功能的 React 特殊构建。

<Note>

本节对从事框架、库或开发者工具的开发者最相关。主要使用 React 来构建面向用户应用的开发者，通常不需要担心我们的预发布通道。

</Note>

React 的每个发布通道都面向不同的使用场景：

- [**Latest**](#latest-channel) 用于稳定、符合 semver 的 React 发布。你从 npm 安装 React 时得到的就是它。这是你今天已经在使用的通道。**直接消费 React 的面向用户应用会使用这个通道。**
- [**Canary**](#canary-channel) 跟踪 React 源代码仓库的主分支。可以把它们看作是下一个 semver 发布的候选版本。**[框架或其他精选环境可能会选择使用固定版本的 React 来采用这个通道。](/blog/2023/05/03/react-canaries) 你也可以使用 Canaries 在 React 与第三方项目之间进行集成测试。**
- [**Experimental**](#experimental-channel) 包含实验性 API 和功能，这些在稳定版发布中不可用。它们同样跟踪主分支，但启用了额外的功能开关。可用它来在这些功能发布前进行试用。

所有发布都会发布到 npm，但只有 Latest 使用语义化版本控制。预发布版本（Canary 和 Experimental 通道中的版本）会根据其内容的哈希和提交日期生成版本号，例如 Canary 的 `18.3.0-canary-388686f29-20230503` 和 Experimental 的 `0.0.0-experimental-388686f29-20230503`。

**Latest 和 Canary 通道都官方支持面向用户的应用，但预期不同**：

* Latest 发布遵循传统的 semver 模型。
* Canary 发布 [必须固定版本](/blog/2023/05/03/react-canaries)，并且可能包含破坏性变更。它们适用于精选环境（如框架），这些环境希望按自己的发布节奏逐步发布新的 React 功能和 bug 修复。

Experimental 发布仅用于测试，我们不保证不同发布之间行为不会变化。它们不遵循我们为 Latest 发布所使用的 semver 协议。

通过将预发布版本发布到与稳定版相同的注册表中，我们能够利用许多支持 npm 工作流的工具，例如 [unpkg](https://unpkg.com) 和 [CodeSandbox](https://codesandbox.io)。

### Latest 通道 {/*latest-channel*/}

Latest 是用于稳定 React 发布的通道。它对应于 npm 上的 `latest` 标签。对于所有面向真实用户发布的 React 应用，它都是推荐通道。

**如果你不确定应该使用哪个通道，那就是 Latest。** 如果你直接使用 React，这就是你已经在使用的通道。你可以期待 Latest 的更新极其稳定。版本遵循语义化版本控制方案，如前文所述。[（前文）](#stable-releases)

### Canary 通道 {/*canary-channel*/}

Canary 通道是一个预发布通道，跟踪 React 仓库的主分支。我们使用 Canary 通道中的预发布版本作为 Latest 通道的候选发布。你可以把 Canary 看作是更新更频繁的 Latest 超集。

最近一次 Canary 发布与最近一次 Latest 发布之间的变更幅度，大致相当于两个 semver 次版本发布之间的差异。然而，**Canary 通道不符合语义化版本控制。** 你应该预期 Canary 通道中相邻发布之间偶尔会出现破坏性变更。

**不要直接在面向用户的应用中使用预发布版本，除非你遵循 [Canary 工作流](/blog/2023/05/03/react-canaries)。**

Canary 中的发布会以 npm 上的 `canary` 标签发布。版本号由构建内容的哈希和提交日期生成，例如 `18.3.0-canary-388686f29-20230503`。

#### 将 canary 通道用于集成测试 {/*using-the-canary-channel-for-integration-testing*/}

Canary 通道也支持 React 与其他项目之间的集成测试。

React 的所有变更在发布给公众之前都会经过广泛的内部测试。然而，整个 React 生态中存在大量不同的环境和配置，我们不可能对每一种都进行测试。

如果你是第三方 React 框架、库、开发者工具或类似基础设施类项目的作者，你可以通过定期针对最近的变更运行你的测试套件，帮助我们为你的用户以及整个 React 社区保持 React 的稳定性。如果你有兴趣，请按以下步骤操作：

- 使用你偏好的持续集成平台设置一个 cron 任务。CircleCI 和 Travis CI 都支持 cron 任务。[CircleCI](https://circleci.com/docs/2.0/triggers/#scheduled-builds) 和 [Travis CI](https://docs.travis-ci.com/user/cron-jobs/) 都支持 cron 任务。
- 在 cron 任务中，使用 npm 上的 `canary` 标签，将你的 React 包更新到 Canary 通道中最新的 React 发布。使用 npm cli：

  ```console
  npm update react@canary react-dom@canary
  ```

  或者使用 yarn：

  ```console
  yarn upgrade react@canary react-dom@canary
  ```
- 用更新后的包运行你的测试套件。
- 如果一切通过，太好了！你可以预期你的项目将兼容下一个 React 次版本发布。
- 如果出现意外问题，请通过 [提交 issue](https://github.com/facebook/react/issues) 告诉我们。

使用这种工作流的项目有 Next.js。你可以参考他们的 [CircleCI 配置](https://github.com/zeit/next.js/blob/c0a1c0f93966fe33edd93fb53e5fafb0dcd80a9e/.circleci/config.yml) 作为示例。

### Experimental 通道 {/*experimental-channel*/}

与 Canary 类似，Experimental 通道也是一个跟踪 React 仓库主分支的预发布通道。不同于 Canary，Experimental 发布包含额外的功能和 API，这些还未准备好向更广泛范围发布。

通常，对 Canary 的更新会伴随对应的 Experimental 更新。它们基于相同的源代码修订，但使用了不同的一组功能开关构建。

Experimental 发布可能与 Canary 和 Latest 发布有很大不同。**不要在面向用户的应用中使用 Experimental 发布。** 你应该预期 Experimental 通道中的发布之间会频繁出现破坏性变更。

Experimental 中的发布会以 npm 上的 `experimental` 标签发布。版本号由构建内容的哈希和提交日期生成，例如 `0.0.0-experimental-68053d940-20210623`。

#### 什么会进入实验性发布？ {/*what-goes-into-an-experimental-release*/}

实验性功能是那些尚未准备好向更广泛公众发布的功能，并且在最终定型之前可能会有很大变化。有些实验可能永远不会定型——我们设置实验的原因就是为了测试提议变更的可行性。

例如，如果在我们宣布 Hooks 时 Experimental 通道已经存在，那么我们会在 Hooks 可用于 Latest 之前的数周，就将其发布到 Experimental 通道。

你可能会觉得针对 Experimental 运行集成测试很有价值。是否这样做取决于你。不过请注意，Experimental 的稳定性甚至比 Canary 更低。**我们不保证 Experimental 发布之间的任何稳定性。**

#### 我如何了解更多关于实验性功能的信息？ {/*how-can-i-learn-more-about-experimental-features*/}

实验性功能可能会有文档，也可能没有。通常，实验在接近于在 Canary 或 Latest 中正式发布之前，不会编写文档。

如果某个功能没有文档，它可能会伴随一个 [RFC](https://github.com/reactjs/rfcs)。

当我们准备宣布新的实验时，我们会发布到 [React 博客](/blog)，但这并不意味着我们会公开宣传每一个实验。

你始终可以参考我们公开的 GitHub 仓库的 [历史记录](https://github.com/facebook/react/commits/main)，以获取完整的变更列表。
