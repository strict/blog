# 类分析模型应用的前端逻辑表达

## 背景

神策分析前端最初基于 jQuery 技术栈开发，目前依然支撑着关键业务，包括概览和各种分析模型。jQuery 是一个 JavaScript 库，创建于 2006 年，当时互联网主要还是 PC 端浏览型页面，jQuery 封装了对 DOM 的操作以及事件处理、动画、Ajax 等等，可以方便各种网页交互效果的实现，但并不适合开发复杂的 Web 应用。在神策分析过去的开发中，积累了大量基于这种技术栈的过程式代码，如果继续依赖 jQuery 技术栈，新功能的开发和代码维护成本都非常高。

2018 年开始引入 React 技术栈，为团队带来了全新的开发模式，基于 React 可以将应用的 UI 界面拆解为组件树，以声明式的 JSX 语法表达 UI 界面，屏蔽了 DOM 操作的复杂度。再结合 DvaJS 的数据流管理方案，对大型应用的开发进行分层，数据层用于管理各种业务逻辑，视图层以组件化的模式进行拆解，分层 + 组件化，为前端应用的开发模式带来了巨大的提升。

React 技术栈为应用开发提供了基础支撑，在此之上依然需要编写大量的业务代码，也会产生很多类似代码，例如各个分析模型页面，背后的业务模型是相同的，但具体的查询条件、UI 界面不一样，需要写各种不同的代码逻辑来表达，随着功能的迭代，业务代码是一个持续退化的过程，后续维护成本会持续增加。

针对不同业务场景的前端应用开发，需要有更多抽象模型的定义，实现更高效的开发模式。

## 目标

为分析模型类应用，提供一种统一的逻辑表达方式，提升逻辑表达效率。

## 总体思路

**业务模型**

![](https://cdn.nlark.com/yuque/__puml/462f374a06bcd19062e6d5746e1ed120.svg#lake_card_v2=eyJjb2RlIjoi5YmN56uv5bqU55SoLT7mnI3liqHnq6_mjqXlj6M6IHJlcXVlc3Qg5p-l6K-i5p2h5Lu2XG7mnI3liqHnq6_mjqXlj6MtLT7liY3nq6_lupTnlKg6IHJlc3BvbnNlIOiuoeeul-e7k-aenCIsInR5cGUiOiJwdW1sIiwiaWQiOiJic1FINiIsInVybCI6Imh0dHBzOi8vY2RuLm5sYXJrLmNvbS95dXF1ZS9fX3B1bWwvNDYyZjM3NGEwNmJjZDE5MDYyZTZkNTc0NmUxZWQxMjAuc3ZnIiwiY2FyZCI6ImRpYWdyYW0ifQ==)

各个分析模型应用本质上就是一条查询请求，具体到 UI 界面，查询条件会拆解为各种输入控件供用户进行配置，如：事件下拉菜单、属性下拉菜单、时间选择控件等等，输入控件的值最终组合成完整的查询条件发送给服务端，返回的计算结果渲染为各种图和表格。

在基于 React 技术栈的开发模式上，这个场景大体的实现是：各种`表单输入组件`用于呈现 UI 效果，用户设置表单值后触发组件的 `onChange` 回调函数，回调执行框架提供的 `dispatch` 方法并传入一个 `action` ，把消息传递到 `model` 层，model 层有对应的方法来处理 action，组装成完整的查询参数，发送请求给服务端接口，返回数据后更新应用的 `state`。

这一系列代码逻辑，表达的其实就是 **UI 与业务模型之间的约定**，既然是约定，就能以一种特定的数据结构来表达。

将业务逻辑的表达由编写各种代码转变为特定数据结构的描述，可以带来的好处非常明显：

- 编写代码的效率与质量取决于工程师的编程能力，通过数据结构来表达，只用理解数据结构的描述规则。
- 业务代码随着功能的迭代会逐渐退化，数据结构不会。
- 通过数据结构可以看到分析模型应用的完整业务表达，新人能更快的熟悉业务逻辑。

## 详细设计

实现新的模式需要三块内容支撑：

- 数据结构的设计，覆盖分析模型应用所有业务逻辑的表达。
- 运行引擎的实现，输入数据结构即可实现应用的所有业务逻辑。
- 应用开发思路的设定，基于数据结构 + 运行引擎提供一种标准化的应用拆解思路。

### 数据结构

```javascript
/**
 * @file 事件分析业务逻辑描述
 */

// 指标设置组件
import Measures from 'components/Measures';
// 全局筛选组件
import Filter from 'components/Filter';
// 分组配置组件
import ByFields from 'components/ByFields';
// 聚合单位组件
import Unit from 'components/unit';
// ...
// 引入指标配置相关的各种组件

export default {
  // Key 为查询条件字段
  by_fields: {
    // 字段对应的组件
    component: ByFields,
    // 组件属性，可以直接设置值或通过函数生成值，函数接收应用的 state 对象
    // 运行引擎内置了一些属性值，包括：
    //   - onChange 组件值改变后的回调函数
    //   - validateStatus 字段值的校验状态，error 表示值不合法
    props: state => {
      return {};
    },
    // 字段默认值，可以直接设置值或通过函数生成值，函数接收应用的 state 对象
    defaultValue: state => {
      return {};
    },
    // 用于配置一个组件输出多个条件字段值 
    oneMany: value => {
      // 返回的 k-v 替代配置中定义的 key
      return {};
    },
    // 关联的字段，其他字段值改变后会影响当前字段值
    follow: {
      // 关联的字段值或字段值列表 []
      field: '',
      // 关联字段值改变后的处理函数
      process: (state, followData, currentValue) => {
        // 可以根据其他字段的改变决定是否返回新的值或处理副作用
        return {};
      }
    },
    // 字段值改变后的处理函数
    afterChange: (prevValue, currentValue) => {
      // 处理副作用
    },
    // 字段值改变后是否触发查询
    isTrigger: true
  }
};
```

数据结构可以表达以下逻辑：

| # | 逻辑 | 表达方式 |
| :--- | :--- | :--- |
| 1 | 查询条件各个字段与组件的映射关系 | 配置的 `key` 表示查询条件字段，`component` 表示对应的组件 |
| 2 | 组件依赖的数据 | 通过 `props` 设置 |
| 3 | 组件交互过程中依赖的异步数据 | 通过 `props` 设置，典型的场景如：属性值输入框输入时需要获取 `values`，可以在 `props` 上设置一个工具函数来获取数据 |
| 4 | 组件输入不合法的反馈 | 分主动校验和被动显示，主动校验由组件内部逻辑实现，如输入框失去焦点时校验，被动显示由运行引擎通过 `props` 传入的 `validateStatus` 字段设置 |
| 5 | 组件的输出需要赋值给多个字段 | 通过 `oneMany` 设置，典型的场景如：时间控件组件可以输出 `from_date`、`to_date`、`compare_from_date`、`compare_to_date` 等多个字段 |
| 6 | 字段的默认值 | 通过 `defaultValue` 设置 |
| 7 | 字段的联动关系 | 通过 `follow`设置， 典型的场景如：新增指标后需要查询事件列表属性值的交集，已经选中的分组值如果未包含在内，则需要重置为 `总体` |
| 8 | 字段值改变后要处理的副作用 | 通过 `afterChange` 设置，如值变化命中某些规则时要弹提示框 |
| 9 | 无 UI 的字段 | 可以不设置 `component`，典型的字段如：`request_id` |
| 10 | 字段值改变后是否触发查询 | 通过 `isTrigger` 设置 |


### 运行引擎

运行引擎的核心是实现各个业务组件与业务模型的自动绑定，组件行为能自动触发后续的业务流程。

具体实现分为两块

**高阶组件**

业务组件由运行引擎提供的高阶组件包装后输出，高阶组件用于连接业务组件与运行引擎的`控制中心`。

**控制中心**

所有业务组件的 `onChange` 事件回调函数都会将值传递给控制中心，控制中心根据业务组件传入的值以及业务逻辑的整体配置生成完整查询条件，触发查询请求。

**核心流程**
![分析模型前端业务流程.png](https://cdn.nlark.com/yuque/0/2020/png/89120/1582974232612-d0602744-23c9-432a-8922-1023a02fa689.png#align=left&display=inline&height=338&name=%E5%88%86%E6%9E%90%E6%A8%A1%E5%9E%8B%E5%89%8D%E7%AB%AF%E4%B8%9A%E5%8A%A1%E6%B5%81%E7%A8%8B.png&originHeight=338&originWidth=1078&size=49366&status=done&style=none&width=1078)

### 应用开发思路

各类分析模型应用的开发由三部分组成：业务逻辑配置、`model` 层、视图层。

**1. 业务逻辑配置**

开发一个分析模型先从业务逻辑的配置开始，了解分析模型的设计，约定服务端 API，按照数据结构的约定，配置查询条件的每一个字段，描述字段对应的 UI 组件、默认值、关联字段等逻辑。

分析模型的维护也是这个视角，以条件字段为粒度，看要迭代的功能、要修的 bug 与哪个字段关联，再分析相关的配置要怎么调整。

**2. model 层**

基于 [`DvaJS`](https://dvajs.com/api/#model) 的 model，运行引擎封装了业务流程的实现，model 层所需的逻辑非常少。

```javascript
export default {
  namespace: 'segmentation',

  state: {
    reportData: null
  },

  subscriptions: {
    setup({ dispatch, history }) {
      history.listen(({ pathname, hash = '' }) => {
        // 响应路由变化
      });
    }
  },

  effects: {
    /**
     * 路由变化处理
     * @param {string} search 查询参数
     */
    *handleRoute({ search }, { put, select, take }) {
      // 进入事件分析时触发查询
    },

    /**
     * 触发查询
     * @param {Object} queryParams 查询参数
     */
    *query({ queryParams }, { call, put, select }) {
      // 发起查询请求
    }
  },

  reducers: {
    /**
     * 查询成功
     * @param {Object} reportData API 返回的报表数据
     */
    querySuccess(state, { reportData }) {
      return {
        ...state,
        reportData
      };
    }
  }
};
```

**3. 视图层**

按照查询条件的约定，以条件字段为粒度拆解出对应的 UI 组件，组件实现只用关注自身的输入和输出，输入为组件依赖的数据、提供的配置项，输出为对应的查询条件字段值。

> 皮成，2020.02.20