---
title: react-test-renderer 弃用警告
---

## ReactTestRenderer.create() 警告 {/*reacttestrenderercreate-warning*/}

react-test-renderer 已弃用。每当调用 ReactTestRenderer.create() 或 ReactShallowRender.render() 时，都会触发警告。react-test-renderer 包仍可在 NPM 上使用，但不会再维护，并且可能会因新的 React 特性或 React 内部实现的变更而失效。

React 团队建议将你的测试迁移到 [@testing-library/react](https://testing-library.com/docs/react-testing-library/intro/) 或 [@testing-library/react-native](https://callstack.github.io/react-native-testing-library/docs/start/intro)，以获得现代且受良好支持的测试体验。


## new ShallowRenderer() 警告 {/*new-shallowrenderer-warning*/}

react-test-renderer 包不再从 `react-test-renderer/shallow` 导出浅渲染器。这实际上只是将之前提取出来的独立包 `react-shallow-renderer` 重新打包了一次。因此，你可以通过直接安装它，以相同的方式继续使用浅渲染器。参见 [Github](https://github.com/enzymejs/react-shallow-renderer) / [NPM](https://www.npmjs.com/package/react-shallow-renderer)。
