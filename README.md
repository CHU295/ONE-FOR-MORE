# 一种新的前端架构思维方式

## 前言

刚来这家公司时，因为业务场景的需求，让做一个新前端项目，可以快速部署业务项目，尽可能简单快速的完成，于是就有了这个项目；

精简了无数功能模块，还在编写中

## 项目规划

为了尽可能的**人尽其才,物尽其用**,把工作分成了下面几块

### 业务侧配置

这块基本由前后端辅助产品完成，通过 excel 配表+python 导表工具=json 文件的方式实现。

这块的主要功能是生成项目配置，包括前端页面的信息表，后端数据库的键值，常量表，公共表等等；

展开为

- 前端页面的所有内容都是通过读取信息表生成
- 后端的数据库键值结构通过读取数据表生成

其中前端是根据不同的前端模板(后面具体讲)加载不同的 json，然后生成不同标签节点。

举个例子，前端 page 模板 LIST 其中的table，表头直接读取json生成，前端写页面的时候不需要配置了，直接用产品配置的生成，后续产品想要修改页面的文字自行修改即可

这里整个过程都由py脚本实现，通过全自动的Jenkins执行，一键实现excel到json的转换，文件的copy移动替换。

体现在日常就是，产品改表，运行导表脚本，拉取分支代码，就可以获得新的json，全程无感

### 前端配置

首先看一下前端项目结构

#### 目录结构

```
│ index.js              入口文件
├─assets                资源文件
│  ├─css 样式文件
│  │      index.less    样式主入口
│  │      reset.less    重置样式，例如覆盖默认html样式
│  │      tool.less     工具样式，例如marginTop-10 
│  ├─img
│  └─js 
│          axios.js     axios配置项
│          common.js    通用js，常为模块组件直接复用的逻辑
│          utils.js     工具js，独立于项目之外的逻辑，例如日期转换
├─project               项目
│  │  index.js          项目主入口js，根据node参数调用不同目录的文件
│  ├─default            项目default
│  │  │  index.js       项目配置
│  │  │  router.js      项目自带路由
│  │  └─data            存放excel转换的json
│  └─test               项目test
│      │  index.js
│      │  router.js
│      └─data
└─source                代码资源
    ├─api               接口类
    │      auth.js      
    │      Interface.js 
    ├─component         UI组件
    │  ├─select
    │  └─table
    ├─layout            网页布局
    │  ├─Front
    │  │      index.js
    │  └─H5
    ├─module            业务模块
    │  ├─createRoute
    │  │      index.js
    │  └─mainInfo
    └─page              页面
        ├─A
        │      index.js
        └─B
                index.js
```
与普通相关差异不大，主要是多了一个project目录，也就是项目配置目录，在使用的时候根据node参数加载不同的子目录，以获得不同的配置项，生成不同的项目。

为了保证打包体积，这里把路由拆分成项目独有

## 项目核心点
1. 多业务项目隔离，通过执行不同的命令参数，在node运行的时候注册不同的全局变量，配置模块根据全局变量读取不同的项目配置，获取不同的路由配置等等，实现不同业务项目的代码隔离，保证编译打包速度体积的稳定性；
2. PC移动端兼容，例如PC引入的antd在移动端需要使用antd mobile，在项目运行的时候通过配置判断运行环境，然后通过nodejs脚本，循环需要修改的文件，替换引入依赖。
3. 网页编辑器，在尝试使用了monaco-editor及codemirror后，发现与实际业务需求不太吻合，于是便使用codemirror然后接管了相关api，自己完成了输入监听，词法判断，语句替换；
4. 无限递归的模板筛选，
    1. 通过配置筛选变量，不同的变量支持大于/小于/不等于/包括/为空/不为空/等于/其中之一/范围/等等十几种逻辑关系，并对应输入框/下拉框/级联/树/日期/范围选择/等等不同ui组件；
    2. 另外支持一种模板筛选，通过配置不同的模板实现不同的功能，比如在 一个月 内 入院次数 大于 3 次的 白血病 患者；
    3. 加上维护一个特定的树形结构，达到无限递归的逻辑判断。
5. 使用react-dnd 完成表单的拖拽填充，配置规范的数据结构，通过封装的drag与drop组件，在拖拽的时候动态校验所有表格是否符合填充规则，并且反馈到表格校验上面；

## 扩展
### antd集成及动态主题
demo地址

最近一直忙上面那个无限递归的组件，扩展了很多东西进去，没来得及整理日常:smile:

## 后序
其实这个实现方式只是个折中方案，我们真正想做的是网页编辑器，在网页端直接修改，然后存储到数据库，替代excel这一步，但是由于时间精力人力等各方面的限制，新的方案推进一直比较缓慢。

## 可能存在的问题
耦合性太强，同一个组件十几个项目使用，单元测试工作量巨大
后期技术栈破坏性升级维护成本太大
````

# 学习中，后续再更新
原实际项目已经运行了两年，并且部署到各大医院完美运行，其中包括内网环境等等，经受的主各种考验
