# TypeScript 实战工作坊

自信、有效的 TypeScript 编程实用指南

Ben Grynhaus, Jordan Hudgens, Rayon Hunte, Matt Morgan 和 Wekoslav Stefanovski

# TypeScript 实战工作坊

版权 © 2021 Packt 出版

所有权利保留。未经出版商事先书面许可，本课程的部分或全部不得以任何形式或通过任何手段进行复制、存储在检索系统中或以任何方式传输，但简要引用嵌入在评论文章或评论中除外。

在准备本课程的过程中，已尽一切努力确保所提供信息的准确性。然而，本课程中包含的信息销售时不附带任何明示或暗示的保证。作者、Packt 出版公司及其经销商和分销商不对由此课程直接或间接造成的或声称造成的任何损害承担责任。

Packt 出版公司已尽力通过适当使用大写字母来提供关于本课程中提到的所有公司和产品的商标信息。然而，Packt 出版公司不能保证此信息的准确性。

**作者：** Ben Grynhaus, Jordan Hudgens, Rayon Hunte, Matt Morgan 和 Wekoslav Stefanovski

**审稿人：** Yusuf Salman 和 Cihan Yakar

**管理编辑：** Mahesh Dhyani

**收购编辑：** Royluis Rodrigues 和 Sneha Shinde

**生产编辑：** Shantanu Zagade

**编辑委员会：** Megan Carlisle, Mahesh Dhyani, Heather Gopsill, Manasa Kumar, Alex Mazonowicz, Monesh Mirpuri, Bridget Neale, Abhishek Rane, Brendan Rodrigues, Ankita Thakur, Nitesh Thakur 和 Jonathan Wray

首次出版：2021 年 7 月

生产参考：1280721

ISBN：978-1-83882-849-3

由 Packt 出版有限公司出版

Livery Place, 35 Livery Street

Birmingham B3 2PB，英国

# 目录

## 前言

## 1. TypeScript 基础

### 简介

### TypeScript 的发展

### TypeScript 的设计目标

### 开始使用 TypeScript

### TypeScript 编译器

### 设置 TypeScript 项目

### 练习 1.01：使用 tsconfig.json 和开始使用 TypeScript

### 类型及其用途

### TypeScript 和函数

### 练习 1.02：在 TypeScript 中使用函数

### TypeScript 和对象

### 练习 1.03：使用对象

### 基本类型

### 练习 1.04：检查 typeof

### 字符串

### 数字

### 布尔值

### 数组

### 元组

### Schwartzian 转换

### 练习 1.05：使用数组和元组创建对象的高效排序

### 枚举

### Any 和 Unknown

### Null 和 Undefined

### Never

### 函数类型

### 创建自己的类型

### 练习 1.06：创建计算器函数

### 活动 1.01：创建用于处理字符串的库

### 总结

## 2. 声明文件

### 简介

### 声明文件

### 练习 2.01：从头创建声明文件

### 异常

### 第三方代码库

### DefinitelyTyped

### 分析外部声明文件

### 练习 2.02：使用外部库创建类型

### 使用 DefinitelyTyped 的发展工作流程

### 练习 2.03：创建棒球阵容卡应用程序

### 活动 2.01：构建热图声明文件

### 总结

## 3. 函数

### 简介

### TypeScript 中的函数

### 练习 3.01：在 TypeScript 中开始使用函数

### 函数关键字

### 函数参数

### 参数与参数

### 可选参数

### 默认参数

### 多个参数

### 剩余参数

### 解构返回类型

### 函数构造函数

### 练习 3.02：比较数字数组

### 函数表达式

### 箭头函数

### 类型推断

### 练习 3.03：编写箭头函数

### 理解 this

### 练习 3.04：在对象中使用 this

### 闭包和作用域

### 练习 3.05：使用闭包创建订单工厂

### 柯里化

### 练习 3.06：重构为柯里化函数

### 函数式编程

### 将函数组织到对象和类中

### 练习 3.07：将 JavaScript 重构为 TypeScript

### 导入、导出和 require

### 练习 3.08：导入和导出

### 活动 3.01：使用函数构建航班预订系统

### 使用 ts-jest 进行单元测试

### 活动 3.02：编写单元测试

### 错误处理

### 摘要

## 4. 类和对象

### 介绍

### 类和对象是什么？

### 练习 4.01：构建你的第一个类

### 使用构造函数扩展类行为

### this 关键字

### 练习 4.02：定义和访问类的属性

### 练习 4.03：将类型集成到类中

### TypeScript 接口

### 练习 4.04：构建接口

### 在方法中生成 HTML 代码

### 练习 4.05：生成和查看 HTML 代码

### 与多个类和对象一起工作

### 练习 4.06：组合类

### 活动 4.01：使用类、对象和接口创建用户模型

### 摘要

## 5. 接口和继承

### 介绍

### 接口

### 案例研究 – 编写你的第一个接口

### 练习 5.01：实现接口

### 练习 5.02：实现接口 – 创建原型博客应用

### 练习 5.03：为更新用户数据库的函数创建接口

### 活动 5.01：使用接口构建用户管理组件

### TypeScript 继承

### 练习 5.04：创建基类和两个扩展子类

### 练习 5.05：使用多层继承创建基类和扩展类

### 活动 5.02：使用继承创建一个原型车辆展厅 Web 应用

### 摘要

## 6. 高级类型

### 介绍

### 类型别名

### 练习 6.01：实现类型别名

### 类型字面量

### 练习 6.02：类型字面量

### 交集类型

### 练习 6.03：创建交集类型

### 联合类型

### 练习 6.04：使用 API 更新产品库存

### 索引类型

### 练习 6.05：显示错误消息

### 活动 6.01：交集类型

### 活动 6.02：联合类型

### 活动 6.03：索引类型

### 摘要

## 7. 装饰器

### 介绍

### 反射

### 设置编译器选项

### 装饰器的意义

### 横切关注点问题

### 解决方案

### 装饰器和装饰器工厂

### 装饰器语法

### 装饰器工厂

### 类装饰器

### 属性注入

### 练习 7.01：创建简单的类装饰器工厂

### 构造函数扩展

### 练习 7.02：使用构造函数扩展装饰器

### 构造函数包装

### 练习 7.03：为类创建日志装饰器

### 方法和访问器装饰器

### 实例函数上的装饰器

### 练习 7.04: 创建标记函数可枚举的装饰器

### 静态函数上的装饰器

### 方法包装装饰器

### 练习 7.05: 为方法创建日志装饰器

### 活动 7.01: 创建用于调用计数的装饰器

### 在装饰器中使用元数据

### Reflect 对象

### 练习 7.06: 通过装饰器向方法添加元数据

### 属性装饰器

### 练习 7.07: 创建和使用属性装饰器

### 参数装饰器

### 练习 7.08: 创建和使用参数装饰器

### 在单个目标上应用多个装饰器

### 活动 7.02: 使用装饰器应用横切关注点

### 总结

## 8. TypeScript 中的依赖注入

### 简介

### 依赖注入设计模式

### Angular 中的依赖注入

### 练习 8.01: 向 Angular 应用添加 HttpInterceptor

### Nest.js 中的依赖注入

### InversifyJS

### 练习 8.02: 使用 InversifyJS 实现 "Hello World"

### 活动 8.01: 基于依赖注入的计算器

### 总结

## 9. 泛型和条件类型

### 简介

### 泛型

### 泛型接口

### 泛型类型

### 泛型类

### 练习 9.01: 泛型集合类

### 泛型函数

### 泛型约束

### 练习 9.02: 泛型 memoize 函数

### 泛型默认值

### 条件类型

### 活动 9.01: 创建 DeepPartial<T> 类型

### 总结

## 10. 事件循环和异步行为

### 简介

### 多线程方法

### 异步执行方法

### 执行 JavaScript

### 练习 10.01：函数堆叠

### 浏览器和 JavaScript

### 浏览器中的事件

### 环境 API

### setTimeout

### 练习 10.02：探索 setTimeout

### AJAX（异步 JavaScript 和 XML）

### 活动 10.01：使用 XHR 和回调的影片浏览器

### Promise

### 练习 10.03：计数到五

### 什么是 Promise？

### 练习 10.04：使用 Promise 计数到五

### 活动 10.02：使用 fetch 和 Promise 的影片浏览器

### async/await

### 练习 10.05：使用 async 和 await 计数到五

### 活动 10.03：使用 fetch 和 async/await 的影片浏览器

### 总结

## 11. 高阶函数和回调

### 简介

### 高阶组件（HOC）简介 – 示例

### 高阶函数

### 练习 11.01：使用高阶函数编排数据过滤和操作

### 回调

### 事件循环

### Node.js 中的回调

### 回调地狱

### 避免回调地狱

### 在文件级别将回调处理程序拆分为函数声明

### 链式回调

### Promise

### async/await

### 活动 11.01：高阶管道函数

### 总结

## 12. TypeScript 中 Promise 指南

### 简介

### Promise 的演变和动机

### Promise 的结构

### Promise 回调

### then 和 catch

### 挂起状态

### 已满足状态

### 拒绝状态

### 链式调用

### 练习 12.01：链式调用 Promise

### finally

### Promise.all

### 练习 12.02：递归 Promise.all

### Promise.allSettled

### 练习 12.03：Promise.allSettled

### Promise.any

### Promise.race

### 使用类型增强 Promise

### 练习 12.04：异步渲染

### 库和本地 Promise - 第三方库、Q 和 Bluebird

### Promise 的 polyfill

### Promisify

### Node.js util.promisify

### 异步文件系统

### fs.readFile

### fs.readFileSync

### fs Promises API

### 练习 12.05：fs Promises API

### 与数据库一起工作

### 使用 REST 进行开发

### 练习 12.06：实现基于 sqlite 的 RESTful API

### 整合所有内容 - 构建 Promise 应用

### 活动 12.01：构建 Promise 应用

### 总结

## 13. TypeScript 中的 Async/Await

### 介绍

### 演变和动机

### TypeScript 中的 async/await

### 练习 13.01：转译目标

### 选择目标

### 语法

### async

### 练习 13.02：async 关键字

### 练习 13.03：使用 then 解析异步函数

### await

### 练习 13.04：await 关键字

### 练习 13.05：等待一个 Promise

### 语法糖

### 异常处理

### 练习 13.06：异常处理

### 顶层 await

### Promise 方法

### 练习 13.07：Express.js 中的 async/await

### 练习 13.08：NestJS

### 练习 13.09：TypeORM

### 活动 13.01：将链式承诺重构为使用 await

### 总结

## 14. TypeScript 和 React

### 介绍

### 类型化 React

### React 中的 TypeScript

### 你好，React

### 组件

### 有状态组件

### 无状态组件

### 纯组件

### 高阶组件

### JSX 和 TSX

### 练习 14.01：使用 Create React App 启动

### 路由

### 练习 14.02：React Router

### React 组件

### 类组件

### 函数组件（函数声明）

### 函数组件（箭头函数表达式）

### 无 JSX

### 函数组件中的状态

### React 中的状态管理

### 练习 14.03：React Context

### Firebase

### 练习 14.04：开始使用 Firebase

### 样式化 React 应用

### 主样式表

### 组件作用域样式

### CSS-in-JS

### 组件库

### 活动 14.01：博客

### 总结

## 附录
