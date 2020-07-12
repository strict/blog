# 前端总体设计

> 记一款新产品的前端总体设计，供参考。

## 背景

> 略，此处用于描述业务背景。

## 总体思路

前端研发可以从工程诉求和系统实现两个方面来作总体设计，工程诉求关注研发流程上需要哪些支撑，系统实现关注概念的设定、功能的抽象、与开发模式的映射关系等等。

基础技术栈将选用 Vue，主要有两点考虑：

- 拓展团队技术范围，团队目前是 React 技术栈，引入一种新技术栈，在开发模式、典型功能场景的实现上能带给我们更多思考，同时 Vue 在集团也是主流技术栈之一，可以吸收其他团队的经验。
- 适应变化，当前使用的技术栈在未来可能会被淘汰，逐渐变成团队的技术债务，引入新技术栈的经验可以促使团队更积极的探索新技术，适应未来的变化。

## 工程诉求

### 开发概念

定义开发概念是应用开发的第一步，其中包含一系列约定，用于描述应用的拆解思路，一个大的应用怎样自上而下拆解为更小的单元，方便多人协作开发。

#### 目录规范

目录规范是资源的分类概念，定义整个应用由哪些资源组成，怎么组织。

```
.
├── src                          # 项目主目录
│   ├── main.ts                  # 入口文件
│   ├── router.ts                # 路由配置
│   ├── store                    # 状态管理
│   │   ├── index.ts             # 组装模块并导出 store
│   │   ├── root.ts              # 根级别的 mutation 和 action
│   │   └── modules              # 按业务分割的模块
│   │       ├── editor.ts
│   │       └── ...
│   ├── components               # 组件目录
│   │   ├── App.vue              # 根组件
│   │   ├── common               # 公共组件
│   │   ├── pages                # 页面组件
│   │   │   └── ...
│   │   ├── editor               # 按业务划分的组件
│   │   │   └── ...
│   │   └── reader
│   │       └── ...
│   ├── api                      # 对服务端请求的封装
│   │   └── ...
│   ├── util                     # 工具方法
│   │   └── ...
│   ├── assets                   # 静态资源
│   │   └── logo.svg
│   └── shims-vue.d.ts
├── public                       # 不经 webpack 处理的静态资源
│   ├── favicon.ico
│   └── index.html
├── test
│   └── unit                     # 单元测试
│       └── example.spec.ts
├── babel.config.js
├── tsconfig.json
├── jest.config.js
├── vue.config.js
├── package-lock.json
└── package.json
```

#### 组件化

将 UI 界面拆解为组件树，可以单独考虑每个组件的实现，基于 Vue [单文件组件](https://cn.vuejs.org/v2/guide/single-file-components.html) 的定义，一个 Vue 组件可以包含 `template`、`script`、`style` 三种语言块。

```html
<template>
  <div class="example">{{ msg }}</div>
</template>

<script>
export default {
  data () {
    // 数据对象
    return {
      msg: 'Hello world!'
    }
  },
  computed: {
    // 计算属性
  },
  watch: {
    // 侦听器
  },
  methods: {
    // 事件处理方法
  }
}
</script>

<style>
.example {
  color: red;
}
</style>
```

Vue 为组件逻辑提供了一种标准的组织结构，包括：

- `computed`，计算属性，处理复杂逻辑
- `watch`，侦听器，响应数据的变化
- `methods`，事件处理函数

基本涵盖了组件的大部分逻辑，将逻辑收敛这些地方，可以让模板更易于重用和维护。

#### 状态管理

开发大型应用时，通常需要有一种集中式的状态管理模式，把组件的共享状态抽取出来统一管理，通过一定的规则来实现状态的变更以及状态到视图的映射，维持视图和状态间的独立性，让代码变得更易于维护。

[基于 Vuex 的状态管理](https://vuex.vuejs.org/zh/)

![vuex](./assets/img/vuex.png)

当应用较大时，store 对象可能会变得臃肿，通过 Vuex 提供的 [`module`](https://vuex.vuejs.org/zh/guide/modules.html) 可以将 `store` 按业务作一些拆分。

#### 路由管理

单页面应用下，每个 URL 会对应一个特定的组件集合，路由也可以是一种状态，与视图形成映射关系，URL 变更时切换不同的组件。

[Vue Router](https://router.vuejs.org/zh/) 是 Vue.js 官方的路由管理器，除了提供基础的路由管理功能，还提供了进阶功能（如：导航守卫）方便典型场景下的需求实现。

基于这些概念的定义，就可以通过组件、状态、路由的划分对一个大型应用进行拆解，方便多人协作开发。

### 开发流程

从本地开发到部署上线需要有一系列工具支持，让写代码之外的事情更高效的完成。

#### 脚手架

脚手架用于帮助我们快速完成项目搭建的一些需求，包括：

- 创建项目，生成项目的基础目录结构、样板代码等等
- 提供默认的构建配置
- 提供本地开发环境

可以借助 [Vue CLI](https://cli.vuejs.org/zh/guide/) 来满足这些需求。

#### 自动化测试

代码质量直接影响软件的交付速度，如何保证代码质量是一个必须要考虑的问题，自动化测试是非常关键的手段。

根据[测试金字塔](https://martinfowler.com/testing/)的建议，我们应该编写许多单元测试，适当写一些更高层的 UI 测试，单元测试可以保证代码库里某个单元的执行符合预期。

![test-pyramid](./assets/img/test-pyramid.png)

**工具**

- [Jest](https://jestjs.io/)，JavaScript 测试框架。
- [Vue Test Utils](https://vue-test-utils.vuejs.org/zh/)，Vue 官方的单元测试实用工具库。

#### CI、CD

代码发布路径和集成方式。

- [分支模型和发布路径](https://www.yuque.com/picheng/blog/an65gi)

## 系统实现

有了工程层面的支撑，我们就可以在一套标准范式下进行多人协作开发，实现软件的持续交付。除此之外还需要对系统作合理的抽象，让系统功能与软件模型之间形成良好的映射关系，方便开发、方便维护、方便扩展。

### 产品特点

> 略，此处主要分析产品的核心资源是什么，系统由哪几块关键功能组成，各个功能的实现目标是什么。

| 系统功能 | 目标 |
| :-- | :-- |
| ... | ... |

### 技术诉求

基于产品特点和系统功能会产生一些技术诉求，主要包括：

- 页面组装规则：页面由一系列内容块组装而成。
- 内容块开发规范：内容块有展示和编辑两种状态，内容块的开发需要有一些约定，可以基于页面组装规则持续扩展。
- 可定制化：需要以灵活可配的方式支持企业的个性化需求。

### 页面组装规则

内容页面由页面容器和一系列内容块组装而成，页面容器是一个 `连接器`，在内容浏览状态将数据传给内容块，在内容编辑状态将编辑好的内容发送给服务端。

#### 数据结构

```javascript
const data = {
  // 源信息
  meta: {
    // 版本标识
    version: '',
  },
  // 内容列表，以内容块为粒度，数组顺序决定内容顺序
  items: [
    {
      // 内容块类型
      type: 'text',
      // 源信息，内部字段由具体组件约定
      meta: {
        style: {},
      },
      // 内容，内部字段由具体组件约定
      body: {
        heading: '',
        content: '',
      },
    }
  ]
};
```

#### ContentBlock

`ContentBlock` 组件是一个标准容器，用于接收内容数据，根据类型展示不同的内容组件，在编辑态时还会根据类型展示不同的编辑组件，同时提供通用的编辑逻辑，如：移动位置、删除、复制等等。

```html
<div id="content">
  <ContentBlock
    v-for="(item, index) in items"
    v-bind:data="item"
    v-bind:key="item.id"
  ></ContentBlock>
</div>
```

`ContentBlock` 只约定内容组件、编辑组件的输入输出，不关注具体的内部实现。

### 内容块开发规范

内容块的实现包含几类元素：

- 描述文件
- 内容组件
- 编辑组件

#### 描述文件

描述文件是一个标准结构，供 `ContentBlock` 组件使用。

```javascript
import Content from './Content.vue';
import EditControl from './EditControl.vue';

export default {
  // 内容块类型
  type: 'text',
  // 内容组件
  Content,
  // 编辑组件
  EditControl
}
```

#### 内容组件

内容组件实现课程内容的具体展示形式，同时在编辑态时也可以提供内容编辑能力。

```html
<Content
  v-bind:meta="meta"
  v-bind:body="body"
  v-bind:editable="editable"
  v-on:save="save"
></Content>
```

| 属性 | 类型 | 描述 |
| :-- | :-- | :-- |
| meta | object | 源信息 |
| body | object | 内容 |
| editable | boolean | 是否可编辑 |

| 事件 | 描述 |
| :-- | :-- |
| save | 处于编辑态时，编辑后的内容和源信息传递给事件处理函数 `{ meta: {}, body: {} }` |

#### 编辑组件

编辑组件为内容块提供完整的编辑能力，包括内容的编辑和风格的设置。

```html
<EditControl
  v-bind:meta="meta"
  v-bind:body="body"
  v-on:save="save"
></EditControl>
```

| 属性 | 类型 | 描述 |
| :-- | :-- | :-- |
| meta | object | 源信息 |
| body | object | 内容 |

| 事件 | 描述 |
| :-- | :-- |
| save | 编辑后的内容和源信息传递给事件处理函数 `{ meta: {}, body: {} }` |

**StyleSelect**

`StyleSelect` 是一个通用的风格设置组件，可供 `EditControl` 组件使用，内容组件支持多种风格时可以通过此组件来设置。

```html
<StyleSelect
  v-bind:options="options"
  v-on:change="change"
></StyleSelect>
```

```javascript
import textOnImage from './textOnImage.png';

const data = {
  options: [
    {
      // 名称
      label: 'Text on image',
      // 设置值
      value: 'textOnImage',
      // 预览图
      preview: textOnImage,
    },
  ],
};
```

**EditPanel**

`EditPanel` 是一个通用的内容编辑面板，可供 `EditControl` 组件使用。

```html
<EditPanel v-bind:isSettings="isSettings" v-bind:data="data" v-on:save="save">
  <template v-slot:content>
    <ContentEdit v-bind:data="data" v-on:save="save"></ContentEdit>
  </template>
</EditPanel>
```

| 属性 | 类型 | 描述 |
| :-- | :-- | :-- |
| data | object | 内容块数据 `{ meta: {}, body: {} }` |
| isSettings | boolean | 是否显示设置面板，用于内容块的通用设置，如：padding 值、背景色等等 |

| 事件 | 描述 |
| :-- | :-- |
| save | 编辑后的内容和源信息传递给事件处理函数 `{ meta: {}, body: {} }` |

| 插槽 | 描述 |
| :-- | :-- |
| content | 插入内容块的具体编辑组件 |

### 可定制化

可定制化要解决的问题是保持系统通用性的同时怎么满足企业的差异化需求。从开发的角度看，要避免针对特定客户的功能开发，开发的需求能复用给大量客户，以此摊薄开发成本。

#### 定制主题

主题配置在软件的可定制化上是一项标配功能，企业购买软件后通过更换主题可以让软件的配色与自身的品牌色调一致。

具体到前端应用的实现，主题配置基于 CSS 预处理器的变量来实现，编译时根据变量配置生成主题样式文件，在选择第三方 UI 库、开发业务组件时都需要考虑对主题配置的支持。

应用开发中通过一个变量文件来维护主题的配置，如：

```sass
/* Color
-------------------------- */
/// color|1|Brand Color|0
$--color-primary: #409EFF !default;
/// color|1|Background Color|4
$--color-white: #FFFFFF !default;
/// color|1|Background Color|4
$--color-black: #000000 !default;
$--color-primary-light-1: mix($--color-white, $--color-primary, 10%) !default; /* 53a8ff */
$--color-primary-light-2: mix($--color-white, $--color-primary, 20%) !default; /* 66b1ff */
$--color-primary-light-3: mix($--color-white, $--color-primary, 30%) !default; /* 79bbff */
$--color-primary-light-4: mix($--color-white, $--color-primary, 40%) !default; /* 8cc5ff */
$--color-primary-light-5: mix($--color-white, $--color-primary, 50%) !default; /* a0cfff */
$--color-primary-light-6: mix($--color-white, $--color-primary, 60%) !default; /* b3d8ff */
$--color-primary-light-7: mix($--color-white, $--color-primary, 70%) !default; /* c6e2ff */
$--color-primary-light-8: mix($--color-white, $--color-primary, 80%) !default; /* d9ecff */
$--color-primary-light-9: mix($--color-white, $--color-primary, 90%) !default; /* ecf5ff */
/// color|1|Functional Color|1
$--color-success: #67C23A !default;
/// color|1|Functional Color|1
$--color-warning: #E6A23C !default;
/// color|1|Functional Color|1
$--color-danger: #F56C6C !default;
/// color|1|Functional Color|1
$--color-info: #909399 !default;
```

基于变量的主题配置是编译时逻辑，实际应用上需要将主题配置作为一项线上服务提供给企业，避免在开发时维护不同企业的主题版本。

第一阶段的开发对可定制化暂时只考虑这么多，后续阶段再完善。

-----

> 皮成，2020.07.12