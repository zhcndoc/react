---
title: "React Server Components 中的拒绝服务与源代码泄露"
author: The React Team
date: 2025/12/11
description: 安全研究人员在尝试利用上周关键漏洞的补丁时，发现并披露了 React Server Components 中另外两个漏洞。高危漏洞拒绝服务（CVE-2025-55184）和中危漏洞源代码泄露（CVE-2025-55183）


---

2025 年 12 月 11 日，作者：[The React Team](/community/team)

_更新于 2026 年 1 月 26 日。_

---

<Intro>

安全研究人员在尝试利用上周关键漏洞的补丁时，发现并披露了 React Server Components 中另外两个漏洞。

**这些新漏洞不允许远程代码执行。** React2Shell 的补丁在缓解远程代码执行利用方面仍然有效。

</Intro>

---

新漏洞披露如下：

- **拒绝服务 - 高危**: [CVE-2025-55184](https://www.cve.org/CVERecord?id=CVE-2025-55184), [CVE-2025-67779](https://www.cve.org/CVERecord?id=CVE-2025-67779), 和 [CVE-2026-23864](https://www.cve.org/CVERecord?id=CVE-2026-23864)（CVSS 7.5）
- **源代码泄露 - 中危**: [CVE-2025-55183](https://www.cve.org/CVERecord?id=CVE-2025-55183)（CVSS 5.3）

由于新披露漏洞的严重性，我们建议立即升级。

<Note>

#### 先前发布的补丁存在漏洞。 {/*the-patches-published-earlier-are-vulnerable*/}

如果你已经为之前的漏洞进行了更新，则还需要再次更新。

如果你更新到了 19.0.3、19.1.4 和 19.2.3， [这些是不完整的](#additional-fix-published)，你还需要再次更新。

升级步骤请参见 [上一条帖子中的说明](/blog/2025/12/03/critical-security-vulnerability-in-react-server-components#update-instructions)。

-----

_更新于 2026 年 1 月 26 日。_

</Note>

在修复措施全部发布完成后，将提供这些漏洞的更多细节。

## 需要立即采取的行动 {/*immediate-action-required*/}

这些漏洞存在于与 [CVE-2025-55182](/blog/2025/12/03/critical-security-vulnerability-in-react-server-components) 相同的包和版本中。

包括以下包的 19.0.0、19.0.1、19.0.2、19.0.3、19.1.0、19.1.1、19.1.2、19.1.3、19.2.0、19.2.1、19.2.2 和 19.2.3：

* [react-server-dom-webpack](https://www.npmjs.com/package/react-server-dom-webpack)
* [react-server-dom-parcel](https://www.npmjs.com/package/react-server-dom-parcel)
* [react-server-dom-turbopack](https://www.npmjs.com/package/react-server-dom-turbopack?activeTab=readme)

修复已回补到 19.0.4、19.1.5 和 19.2.4 版本。如果你正在使用上述任意包，请立即升级到任一已修复版本。

和之前一样，如果你的应用的 React 代码不使用服务器，那么你的应用不受这些漏洞影响。如果你的应用不使用支持 React Server Components 的框架、打包器或打包器插件，那么你的应用不受这些漏洞影响。

<Note>

#### 关键 CVE 导致后续漏洞被发现是很常见的。 {/*its-common-for-critical-cves-to-uncover-followup-vulnerabilities*/}

当披露一个关键漏洞时，研究人员会仔细检查相邻的代码路径，寻找变种利用技术，以测试初始缓解措施是否可以被绕过。

这种模式不仅出现在 JavaScript 领域，在整个行业都很常见。例如，在 [Log4Shell](https://nvd.nist.gov/vuln/detail/cve-2021-44228) 之后，随着社区继续研究原始修复，披露了额外的 CVE（[1](https://nvd.nist.gov/vuln/detail/cve-2021-45046)、[2](https://nvd.nist.gov/vuln/detail/cve-2021-45105)）。

额外披露可能令人沮丧，但通常表明响应周期是健康的。

</Note>

### 受影响的框架和打包器 {/*affected-frameworks-and-bundlers*/}

一些 React 框架和打包器依赖、具有 peer 依赖，或包含了存在漏洞的 React 包。以下 React 框架和打包器受到影响：[next](https://www.npmjs.com/package/next)、[react-router](https://www.npmjs.com/package/react-router)、[waku](https://www.npmjs.com/package/waku)、[@parcel/rsc](https://www.npmjs.com/package/@parcel/rsc)、[@vite/rsc-plugin](https://www.npmjs.com/package/@vitejs/plugin-rsc) 和 [rwsdk](https://www.npmjs.com/package/rwsdk)。

升级步骤请参见 [上一条帖子中的说明](/blog/2025/12/03/critical-security-vulnerability-in-react-server-components#update-instructions)。

### 托管服务提供商的缓解措施 {/*hosting-provider-mitigations*/}

和之前一样，我们已与多家托管服务提供商合作，应用临时缓解措施。

你不应依赖这些措施来保护你的应用，仍需立即更新。

### React Native {/*react-native*/}

对于不使用 monorepo 或 `react-dom` 的 React Native 用户，你的 `react` 版本应在 `package.json` 中固定，并且不需要其他额外步骤。

如果你在 monorepo 中使用 React Native，并且安装了受影响的包，则你应当仅更新受影响的包：

- `react-server-dom-webpack`
- `react-server-dom-parcel`
- `react-server-dom-turbopack`

这对于缓解安全通告是必要的，但你不需要更新 `react` 和 `react-dom`，因此不会在 React Native 中引起版本不匹配错误。

更多信息请参见 [这个 issue](https://github.com/facebook/react-native/issues/54772#issuecomment-3617929832)。

---

## 高危：多个拒绝服务 {/*high-severity-multiple-denial-of-service*/}

**CVEs:** [CVE-2026-23864](https://www.cve.org/CVERecord?id=CVE-2026-23864)
**基础分数:** 7.5（高）
**日期**: 2026 年 1 月 26 日

安全研究人员发现 React Server Components 中仍然存在额外的 DoS 漏洞。

这些漏洞可通过向 Server Function 端点发送特制的 HTTP 请求来触发，并可能导致服务器崩溃、内存不足异常或 CPU 过度使用；具体取决于所利用的存在漏洞的代码路径、应用配置和应用代码。

1 月 26 日发布的补丁缓解了这些 DoS 漏洞。

<Note>

#### 发布了额外修复 {/*additional-fix-published*/}

最初针对 [CVE-2025-55184](https://www.cve.org/CVERecord?id=CVE-2025-55184) 中 DoS 的修复是不完整的。

这使得先前版本仍然存在漏洞。19.0.4、19.1.5、19.2.4 版本是安全的。

-----

_更新于 2026 年 1 月 26 日。_

</Note>

---

## 高危：拒绝服务 {/*high-severity-denial-of-service*/}

**CVEs:** [CVE-2025-55184](https://www.cve.org/CVERecord?id=CVE-2025-55184) 和 [CVE-2025-67779](https://www.cve.org/CVERecord?id=CVE-2025-67779)
**基础分数:** 7.5（高）

安全研究人员发现，可以构造并发送恶意 HTTP 请求到任何 Server Functions 端点，当 React 对其反序列化时，可能导致一个无限循环，从而挂起服务器进程并消耗 CPU。即使你的应用没有实现任何 React Server Function 端点，如果你的应用支持 React Server Components，也仍然可能存在漏洞。

这会形成一种漏洞利用向量，攻击者可能借此阻止用户访问产品，并可能对服务器环境造成性能影响。

今天发布的补丁通过防止无限循环来缓解此问题。

## 中危：源代码泄露 {/*low-severity-source-code-exposure*/}

**CVE:** [CVE-2025-55183](https://www.cve.org/CVERecord?id=CVE-2025-55183)
**基础分数**: 5.3（中）

一名安全研究人员发现，向存在漏洞的 Server Function 发送恶意 HTTP 请求，可能会不安全地返回任何 Server Function 的源代码。利用这一问题需要存在一个 Server Function，它显式或隐式地暴露了一个字符串化参数：

```javascript
'use server';

export async function serverFunction(name) {
  const conn = db.createConnection('SECRET KEY');
  const user = await conn.createUser(name); // 隐式字符串化，在 db 中泄露

  return {
   id: user.id,
   message: `Hello, ${name}!` // 显式字符串化，在回复中泄露
  }}
```

攻击者可能能够泄露以下内容：

```txt
0:{"a":"$@1","f":"","b":"Wy43RxUKdxmr5iuBzJ1pN"}
1:{"id":"tva1sfodwq","message":"Hello, async function(a){console.log(\"serverFunction\");let b=i.createConnection(\"SECRET KEY\");return{id:(await b.createUser(a)).id,message:`Hello, ${a}!`}}!"}
```

今天发布的补丁阻止了对 Server Function 源代码的字符串化。

<Note>

#### 只有源代码中的秘密可能会被泄露。 {/*only-secrets-in-source-code-may-be-exposed*/}

以硬编码方式写入源代码中的秘密可能会被泄露，但运行时秘密（例如 `process.env.SECRET`）不受影响。

泄露代码的范围仅限于 Server Function 内部的代码，其中可能包括其他函数，具体取决于你的打包器进行了多少内联。

务必针对生产构建进行验证。

</Note>

---

## 时间线 {/*timeline*/}
* **12 月 3 日**：泄露由 [Andrew MacPherson](https://github.com/AndrewMohawk) 向 Vercel 和 [Meta Bug Bounty](https://bugbounty.meta.com/) 报告。
* **12 月 4 日**：初始 DoS 由 [RyotaK](https://ryotak.net) 向 [Meta Bug Bounty](https://bugbounty.meta.com/) 报告。
* **12 月 6 日**：React 团队确认这两个问题，并开始调查。
* **12 月 7 日**：创建初始修复方案，React 团队开始验证并规划新的补丁。
* **12 月 8 日**：通知受影响的托管服务提供商和开源项目。
* **12 月 10 日**：托管服务提供商已实施缓解措施，补丁已验证。
* **12 月 11 日**：Shinsaku Nomura 向 [Meta Bug Bounty](https://bugbounty.meta.com/) 报告了额外的 DoS。
* **12 月 11 日**：补丁发布，并作为 [CVE-2025-55183](https://www.cve.org/CVERecord?id=CVE-2025-55183) 和 [CVE-2025-55184](https://www.cve.org/CVERecord?id=CVE-2025-55184) 对外披露。
* **12 月 11 日**：内部发现缺失的 DoS 情况，已修补并作为 [CVE-2025-67779](https://www.cve.org/CVERecord?id=CVE-2025-67779) 对外披露。
* **1 月 26 日**：发现、修补并对外披露了额外的 DoS 情况，编号为 [CVE-2026-23864](https://www.cve.org/CVERecord?id=CVE-2026-23864)。
---

## 归因 {/*attribution*/}

感谢 [Andrew MacPherson (AndrewMohawk)](https://github.com/AndrewMohawk) 报告了源代码暴露问题，感谢 [RyotaK](https://ryotak.net)（来自 GMO Flatt Security Inc）和 Bitforest Co., Ltd. 的 Shinsaku Nomura 报告了拒绝服务漏洞。感谢 [Mufeed VH](https://x.com/mufeedvh)（来自 [Winfunc Research](https://winfunc.com)）、[Joachim Viide](https://jviide.iki.fi)、[RyotaK](https://ryotak.net)（来自 [GMO Flatt Security Inc](https://flatt.tech/en/)）以及腾讯安全 云鼎实验室的 Xiangwei Zhang 报告了其他 DoS 漏洞。
