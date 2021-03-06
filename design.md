<!-- TOC -->

- [1 需求分析](#1-需求分析)
    - [1.1 需求概述](#11-需求概述)
        - [1.1.1 目标概述](#111-目标概述)
        - [1.1.2 需求特性](#112-需求特性)
        - [1.1.3 需求场景](#113-需求场景)
    - [1.2 功能需求](#12-功能需求)
        - [1.2.1 文档管理](#121-文档管理)
        - [1.2.2  图片管理](#122--图片管理)
        - [1.2.3  常用工具](#123--常用工具)
    - [1.3 性能需求](#13-性能需求)
        - [1.3.1 索引机制](#131-索引机制)
        - [1.3.2 缓存机制](#132-缓存机制)
        - [1.3.3 自动缓存优化](#133-自动缓存优化)
- [2 系统架构](#2-系统架构)
    - [2.1 分层设计](#21-分层设计)
    - [2.2 系统特性](#22-系统特性)
    - [2.3 对外接口](#23-对外接口)
- [3 总体设计](#3-总体设计)
    - [3.1 交互设计概要](#31-交互设计概要)
        - [3.1.1 目录树视图](#311-目录树视图)
        - [3.1.2 片段编辑视图](#312-片段编辑视图)
        - [3.1.3 文档编辑视图](#313-文档编辑视图)
        - [3.1.4 查看-编辑的统一](#314-查看-编辑的统一)
        - [3.1.5 文档查看视图](#315-文档查看视图)
        - [3.1.6 回收站](#316-回收站)
    - [3.2 存储方案](#32-存储方案)
        - [3.2.1 片段式存储](#321-片段式存储)
        - [3.2.2 内容安全性设计](#322-内容安全性设计)
            - [安全机制](#安全机制)
            - [片段历史记录](#片段历史记录)
            - [文档历史记录](#文档历史记录)
        - [3.2.3 版本管理](#323-版本管理)
        - [3.2.4 配置设计思路](#324-配置设计思路)
- [4 详细设计](#4-详细设计)
    - [4.1 片段操作](#41-片段操作)
        - [4.1.1 字段设计](#411-字段设计)
        - [4.1.2 创建](#412-创建)
            - [[创建片段]算法](#创建片段算法)
            - [[保存片段]算法](#保存片段算法)
            - [[独立备份]算法](#独立备份算法)
        - [4.1.3 查询及显示](#413-查询及显示)
        - [4.1.4 修改](#414-修改)
            - [[撤消更改]算法](#撤消更改算法)
            - [[系统托管]算法](#系统托管算法)
            - [[取消托管]算法](#取消托管算法)
        - [4.1.5 删除](#415-删除)
            - [[回收片段]算法](#回收片段算法)
            - [[批量回收片段]算法](#批量回收片段算法)
            - [[片段链回收]算法](#片段链回收算法)
            - [[还原片段]算法](#还原片段算法)
            - [[删除片段]算法](#删除片段算法)
    - [4.2 文档操作](#42-文档操作)
        - [4.2.1 字段设计](#421-字段设计)
        - [4.2.2 创建](#422-创建)
            - [[创建文档]算法](#创建文档算法)
        - [4.2.3 查询及显示](#423-查询及显示)
            - [[文档加载]算法](#文档加载算法)
        - [4.2.4 修改](#424-修改)
            - [[文档修改]算法](#文档修改算法)
            - [[片段引用替换]算法](#片段引用替换算法)
        - [4.2.5 删除](#425-删除)
            - [[移除引用]算法](#移除引用算法)
            - [[回收文档]算法](#回收文档算法)
            - [[删除文档]算法](#删除文档算法)
    - [4.3 文档打开过程](#43-文档打开过程)
    - [4.3.1 文档查询](#431-文档查询)
    - [4.3.2 文档组装](#432-文档组装)
    - [4.3.3 文档显示](#433-文档显示)
    - [4.5 系统优化算法](#45-系统优化算法)
        - [4.5.1 统一定时任务](#451-统一定时任务)
    - [4.6 应用本地缓存过程](#46-应用本地缓存过程)
- [5 附录](#5-附录)
    - [5.2 插件建议](#52-插件建议)
        - [5.2.1 对页面段落内的链接，调用外部浏览器打开。](#521-对页面段落内的链接调用外部浏览器打开)
        - [5.2.2 倡议：基于自定义事件的回调约定](#522-倡议基于自定义事件的回调约定)
        - [1.树形菜单的图标](#1树形菜单的图标)
        - [2. 代码引用](#2-代码引用)

<!-- /TOC -->


# 1 需求分析


## 1.1 需求概述

### 1.1.1 目标概述

**目标系统**

系统主题： 基于Web的资源管理系统

`资源`的范畴： web文档、Web应用、图片、附件等。

基本上，目标系统可分解为`信息收集与管理`和`Web应用管理`两大功能，但在界面上是融合一体的。

此处`信息`的范畴： 一切可使用Web表达的信息，例如API文档、网络文章、个人笔记等。


**系统要求**

* 基于Web，功能开放
* 具有高度扩展性，利用HTML与CSS结合，使信息以最佳的方式展现
* 具有编辑、收集资源和自我编辑的功能
* 支持数据导出

本系统是基于Webkit特定条件的个人软件，因此兼容性并不重要。

### 1.1.2 需求特性

**通用文档存储**

文档主要指HTML形式的文档，同时也包括TXT/XML/Markdown，以及外部资源文档（PDF/Word/PPT等）。
文本型的文档可存入数据库，二进制文件则可收集其路径信息方便查找和归类管理。
根据存储类型调用不同的处理脚本实现定制化界面和功能。
***文档内容与表现完全分离***，存储文档时只需要存入内容即可，大大减少文档存储空间。


**可编程、可定制**

提供自定义配置存储和开放的样式、脚本权限，并提供操作这些配置的通用、便捷API。
强大的模板化定制功能，可基于模板实现内容与表现分离、数据为主的高效存储。

**文档即应用**

系统要支持文档和应用的通用管理，就要无区别的对待这两者。
因此，系统本身不关心它们的功能或界面问题，只关心存储问题，如资源的存储、依赖分析、整理、维护等。 
比如：

* 图片  
* 脚本或引入外部文件  
* 附件（包括压缩文件，非必要的功能：可在线提取内部文件、预览）  
* 链接 （链接资源识别与管理、调用外部程序打开等）  

以上都是文档和应用可能包含的依赖，因此，`文档`被视为一种特殊的应用，以查看为主，但也具有一些交互操作，如折叠、示例运行等。
API手册、博文、笔记等都是常见的具体类型。

注意文档和应用仍然有一些区别：

> `应用`与`文档`的区别主要在于前者具有界面样式和功能脚本的独立性。
不同`应用`之间没有共享的模板，或者模板就是应用界面主体。

**便捷的管理**

• 全面的属性索引，以及灵活的实体关联（例如文档与文档、文档与附件、文档属性之间）  
• 强大的界面，搜索、分类树、标签、附件集中管理、回收站等。  
• 多对多的信息分类机制：同一内容实体，提供多个分类维度。
`分类`总是参考实体的某种特征而进行，由于存在多种分类依据，信息分类无法良好地解决以下问题：

1. 怎样判断新的信息实体应该归属到哪个分类？它是否可以同时被归属到其它分类？
2. 在不知道实体信息的情况下，如何根据模糊印象快速定位到所要搜寻的子域？
3. 如何提高查找某个无关键信息实体的命中率？

> 很多情况下，我们不记得目标信息实体在录入系统时被给予的特征描述，只能根据此时所能想到的特征描述去查找，如果二者不对应，可能就一时无法找到目标信息。
多分类索引机制为信息实体提供了不同维度的特征录入，因此可大大降低定位查找难度。

• 多方式搜索与推荐功能：当查看文档属性，或者搜索结果较少时，选择性地提供**类似主题文档**推荐。
为此，系统提供自动发现功能，根据关键字的相似度计算算法自动罗列相关性最大的几篇文档及其位置信息等。
用户可以手动为这些文档创建关联。
（例如用户填写文档关键字后系统自动搜索数据库并列出候选文档，只要点击文档对应的“关联”按钮即可**自动将关联链接信息添加到文档头部或尾部(可点击)**。

• 自动标签管理：一个文档可以有多个标签，某些常见的标签可以存储到数据库中。
标签众多，系统应该提供自动标签记忆和积累功能。
按标签频度和出现的类别个性化的提供标签自动添加的候选列表。
通过标签体系建立文档之间的模糊关联。

todo **`标签`与`分类`的区分：标签只指示关键字，或者联想的关键字，领域类的关键字应该是一种具体的分类。**
示例： 

++ 编程语言类别，属于分类，如C++.
++ 具体的问题领域，属于标签，如MVC、递归等。
++ 同样的关键字可能是分类也可能是标签，如`IE`，如果当前文档是介绍IE的某个特性，或提到IE较多，则可以归纳为标签。
   如果是介绍IE浏览器下特有的API，则可归纳为分类，而标签为`api`。

总之，标签具有“关键字”的含义，是比较具体的特征概括，而分类是更上层抽象，是种类归属。


### 1.1.3 需求场景

**诗歌、歌词收集分类工具**

1. XML存储
2. XSLT转换

***样式定制***：简单颜色、背景（颜色或图片）、排版。
手动添加诗歌，存储为XML文档，按作者分类。



## 1.2 功能需求

### 1.2.1 文档管理

文档管理包括对文档的收集、整理、查看与编辑四大部分。是个很大的范畴，在此包括有：

1. 文档的即时编辑与预览（窗口较小，可通过快捷键查看预览）
2. 常见文章的保存，支持HTML / Markdown / XML格式
3. 管理还包含分类搜索、文档属性的查看与修改、跨文档链接的处理
4. 文档的定时备份（到其它分区，甚至加密上传到云）
5. 数据整体导出功能：系统可以按照目录树的结构将全部文本文档整体导出，通过将各文档分类的编号替换成对应的分类名，并以分类名创建文件夹即可。


__文档创建__

首先需要创建文档模板，可为每个分类自定义样式模板（主题）和结构模板。
第一次定义后系统自动生成相应的模板文件并为以后的所有该分类的文档应用这套模板。
用户创建模板时可以实时预览（例如提示用户自建测试样例，分栏编辑和预览）。

__文档编辑__

文档编辑可以在主窗口中设计，也可以采用在新窗口打开，编辑和实时预览双屏模式。

除了日常技术文档资料，还包括很多种类型的文档，例如
*IT词典、英语词汇与例句、医药健康知识、代码仓库、诗歌收藏 …*
不同类型的文档可以有不同的界面风格与操作接口。

### 1.2.2  图片管理

不同类型的图片具有不同的浏览风格需求，以自定义插件方式提供浏览界面，此外还有以下要求：

1. 图片保存信息全面，涉及尺寸大小、格式、颜色风格、图片内容类型、关联标签（如具体内容、用途）、图片质量（评级）等信息，方便快速搜索。
2. 图片浏览支持全窗口模式。

图片需要独立的视图管理，以及定制化的多条件搜索。

### 1.2.3  常用工具

常用工具包括：

+ 设计辅助类，如颜色拾取、base64加解密、各种编码转换、简单图片处理（Canvas支持）；  
+ 通讯类，如实时聊天、局域网UDP通信基础，提供多人协作API和双人对战游戏基础API。  
+ 多媒体类，RAP在线音乐平台。  
+ 网络工具，如即时页面抓取并精简处理，即时保存至本地（很方便），还有RSS订阅器等。  
+ 项目管理类。支持上传文件到服务器指定目录，然后在多个客户端共享、实时浏览并显示更新状态列表，有助于团队开发。  

•文档辅助功能：提供一些常见的辅助操作，例如编码转换、语法加亮（这是个难度较大的问题）、
简繁体转换等功能，会很有用处。



[每日日记]

[图文教程]（新浪、百度经验）

代码中英文字体混排（.zh-fit-font样式类）


页面DEMO链接，hover出现“引入浏览”提示链接功能，点击则可以将代码直接插入到当前位置的下方。

堆叠式页面缓存设计（类似win7桌面窗口切换程序的堆叠），通过简单的交互达到切换页面，避免状态丢失或者重新查询等问题。
（例如鼠标移到边缘触发“半重叠”移位展开，与iPhone上的safari选项卡效果一致）。
这种设计同时规避了界面上的多选项卡式的复杂交互设计的弊端。（水平方向上可能已经有固定的选项卡用于应用切换，而不是
文档的切换），文档的堆叠在垂直方向上展开，这样对纵、横向的层次空间利用达到了均衡。

树形菜单提供：
	1. 子节点范围搜索。
	2. Tab切换树分类依据，例如按照Tags分类树，按照时间分类树（不是树，而是一个时间线组件视图）。
	3. 支持多个标签的组合过虑。

搜索视图提供：
	1.搜索视图（like Google ）
	2.详情列表（like 百度网盘）
	3.缩略图视图



## 1.3 性能需求

### 1.3.1 索引机制

按分类依据转换视图的响应要迅速，所以要能够提供手动创建索引的支持，并对索引表进行管理。例如：按语言种类建立的索引（或视图），按图片尺寸建立的索引等，可以大大提升系统的响应速度和减少内存占用。

### 1.3.2 缓存机制

缓存是一种特殊的存储对象，保存了所有能加快系统启动初始化速度的数据。
存储数据库比XML存储更快，并且实现上无需再增加XML处理的复杂函数，保持了良好的一致性。
这个目录树集中了各个数据表的目录，犹如Unix文件目录结构，数据表（类似于磁盘分区）被挂在目录树的第一级子节点上。

**目录树缓存：**

系统启动时从缓存表中加载数据并恢复上一次退出时目录树的状态，*包含所有界面上可见的节点数据*。

缓存机制的实现有几种可选的方案：
1. 整体数据直接存储于一个JSON文本中（外部文件或数据库），这可能遇到分隔问题。
2. 抽取每条记录的必要信息以以及它们之间的关系，并汇聚到一张表中。
如果可能，尽量将索引机制的某些部分合并到缓存机制中来，以减少索引表的数量。当索引过多时，SQL的事务会变得繁重。

**利用缓存变异步为同步：**

一旦数据被获取并缓存在内存中，下次读取时直接返回即可。旦由于很多操作需要读取数据库，这是个异步的操作过程，
因此会出现以下问题：首次查询需要以异步回调的方式处理业务逻辑，再次查询时，可以同步返回数据，这样编码时
需要实现异步和同步两个版本的函数，导致调用过程不一致，并且代码冗余。通常API的设计是可以提供同步和异步两个
版本的，旦业务操作不同，两个版本会夹杂很多重复的业务处理逻辑。

解决以上问题的方法，就是利用缓存，一次性加载所有需要首次异步操作并缓存的数据。具体过程如下：

在程序安装后首次使用时，首次启动会异步查询数据库获取初始化数据并缓存于内存，此时界面同步显示初始化提示界面。
这个过程会有一些慢，通常1-3秒。加载完成后，系统将缓存数据整理再存储到缓存表中，这样，下次启动直接一次性异步
获取所有必要初始化数据，此时界面同样有初始化过程，但等待时间很短，异步回调后，系统才正式执行初始化，此时的
数据处理都是同步的。通常这个初始化是一个惟一的入口，如`App.run()`。

### 1.3.3 自动缓存优化

系统根据文档的片段引用记录表，可以找出被引用最多的那些片段，它们很可能是一些常用的库文件，例如
jQuery、underscore.js，将这些文件自动缓存在本地，并且读取一次后缓存在内存中，实现最大优化。
（在一个任务计划[schedule]中执行定期更新操作）

# 2 系统架构

## 2.1 分层设计


事件模型、框架层 => 底层SQL存储API => 基础服务设计与抽象、接口设计 => 开放服务API => 界面设计与API开发

## 2.2 系统特性

数据复杂。
必须提供的丰富的视图。

鉴于以上特性，如果不对系统中的实现逻辑和数据加以分离，将导致极大的耦合性和代码冗余。
因此必须对系统的数据处理需求加以细分、完整化。
这样在视图中以简单的函数调用即可实现某个视图所需的功能。

## 2.3 对外接口

对外接口主要是系统的插件支持问题。由系统主要为HTML文档和图片，需要很多脚本插件以完成数据的格式化显示、图片的插件式管理和提升用户体验等，这通常要用到JQuery、语法高亮等插件，这些插件使用的频率相当高，而且需要方便引用，所以不适合存储在数据库中。
为此，系统提供一些全局变量，如 $AppDir 代表系统所在目录（绝对路径），这样只需要设定插件的相对路径即可，在系统文件夹中设定一个插件目录用于存放插件，在要用到插件的文档中使用相对路径进行引用，即使系统整体搬移也不会有问题。


•如果可能，采用JQuery模拟添加自动依附的工具栏，并可设置系统的布局等属性。

数据处理复杂，除整体设计框架和数据结构的设计之外，系统数据的删除与更新将成为焦点问题。
必需合理地进行数据的抽象分析，理清各数据之间的联系与依赖关系等。
这也是使用SQLite的理由之一，SQLite强大的事务处理支持可以使很多关联性的工作由SQL来完成。

打开文档用矩形长廊左右滚动切换（像Ubuntu安装界面一样），活动文档数量达到最大限制时采用替换方式。


# 3 总体设计

## 3.1 交互设计概要

### 3.1.1 目录树视图

节点延迟加载，缓存按需同步更新。
对树的节点数据存储可用HashMap进行缓存。

### 3.1.2 片段编辑视图

片段编辑提供“[保存片段](#frag-save-or-update)”、“[独立备份](#frag-save-alone)”、“[回收片段](#frag-recycle)”操作。
如果修改片段内容后不想保存，直接关闭页面或重新导航即可。

注：只能编辑独立片段。系统应该根据预判结果动态显示[独立备份]操作按钮。预判规则为，当前片段已有id并且是叶子节点，且没有被文档引用。

### 3.1.3 文档编辑视图

为了简洁和便捷的操作体验，新建和修改操作将共享同一个视图，但在操作功能集上有些差别。

文档编辑视图由片段编辑视图和一些额外的操作元素组合而成，提供编辑视图和文档源码预览视图的平滑无缝切换。

文档编辑是以片段为单位一块一块编辑的，每个编辑区域对应一个片段。


编辑界面生成过程：

[模板分析程序]()对模板内容进行占位符分析，然后由系统对照占位结构生成对应的片段编辑区域，用户手动粘贴即可。
系统根据相应的编译器是否提供反编译接口来决定是否保留自动分析入口，自动分析及处理逻辑由编译器脚本提供。
自动处理逻辑将输入内容分割并输出为与文档内容存储结构类似的对象，粘贴内容后，系统生成预览图，但不会自动保存。

针对任意存储类型的片段，程序应该具有简单的自动预测语言类型的功能逻辑，预置预测的结果并提供纠正机会。

界面细则：

+ 新建文档时，默认根据模板占位符添加可编辑区域进行片段内容填充，每个非空的编辑区块被存储为一个片段；
+ 编辑区具有针对片段的操作功能入口，停靠于激活编辑区一侧；
+ 编辑区域高度随内容在一定范围内动态伸缩；
+ 根据对模板占位符的可重复性属性分析结果，相应的编辑区一侧会提供手动增减编辑区的操作按钮，并且它们之间可以拖动以调整顺序；

- TODO 编辑区内容类型设计 可以选择下拉菜单手动更改为其它类型，一旦类型变更，将重置引用（清除片段链历史记录？）

编辑区包含以下操作：

[撤消更改](#frag-reload) - 丢弃更改，重新载入。
[取消托管](#frag-untie) - 当编辑片段tied=0时，点击后切换按钮为绿色。
[系统托管](#frag-tieup) - 当编辑片段tied=1时，点击后切换按钮为灰色。
[独立备份](#frag-save-alone) - 一次性按钮，在同一次编辑会话中，点击一次后禁用（如果操作失败需再次启用）。
[移除引用](#doc-remove-ref) - 界面上显示为[-]按钮，仅当操作已存在片段时，执行该算法，否则仅删除界面元素。


文档实体包含以下操作：

[保存文档](#doc-save) - 保存文档成功后，应该界面指示其它引用了该片段的文档和片段。 - TODO 待设计
[回收文档](#doc-recycle) - 新建文档时，不出现该操作入口。
[取消]() - 取消后返回之前的界面，对编辑区做的更改将不生效。


点击”[-]“与“清空编辑区”操作的细微差别：
界面清空编辑区不会产生任何数据更改，直到文档保存的时候，执行[移除引用](#doc-remove-ref)算法。
而删除编辑区操作则会立即对已存在的片段执行[移除引用](#doc-remove-ref)算法。

### 3.1.4 查看-编辑的统一

提供“以源码查看”方法，并支持选项卡拆分查看和整体查看视图的切换功能，整体查看时，块之间仍提供
内联的“展开/折叠”显示功能，类似 brackets.io 的设计。

文档的源码查看视图可以方便地过渡到编辑视图，此时视图过渡到以片段为独立单位的编辑区域。
- TODO 待完善设计

### 3.1.5 文档查看视图

文档视图包括以下基本内容：
1） 标题区
2） 内容区
3） 属性区：以Tab选项卡分类显示。属性必须包括：
其它附属分类、关键字（即标签）、文档创建日期、最近修改日期、**文档（虚拟）路径**、附注、关联文档、相关推荐（动态计算的结果）。
		说明：文档路径格式： 分类名/一级子分类名/二级子分类名/…/文档名
关联文档是用户手动添加的一种持久存储的硬性关联，而推荐文档是系统根据关键字向用户推送的文档链接，是动态计算的结果。
系统会主动推送相关文档，这有助于相关领域的信息汇集，减少用户查找操作。
创建关联是经用户确定的关联关系，而推送是系统根据算法确认的。

4） 工具栏
提供查找、高亮等快捷操作。(利用脚本操作，使用DOM更新内容而不更改数据～)

底部有关于当前打开文档的附加信息。为提高空间利用率，采用tab选项卡分组显示。如文档的属性、链接、关键词、所属分类等。

### 3.1.6 回收站

回收站显示被回收的文档、片段、附件列表。列表项包括一些概要信息，如删除时间、剩余存活时间等。
回收站中的内容不可再编辑。

## 3.2 存储方案

### 3.2.1 片段式存储

**片段如何组装成文档？**

所有代码按块类型（或者语言类型，如CSS、JavaScript、HTML）分开存储（同一张表中的不同数据项），
这样可以将样式、HTML结构、脚本算法等进行独立处理（搜索和查看），很多常用的算法脚本和样式也可以重用。
此外，还可以对它们分开进行独立编辑、自由拼装和选择性调用。

片段被定义为内容的最小实体单位。片段 = 类型信息 + 内容。
这样，所有的“片段”被无区别地对待，存储于同一张表中，从物理角度上看，“该表提供了所有文档的内容基础”，
而实际的文档实体则存储在另一张表中，仅包含组成文档的片段id信息，以及文档的模板id信息即可。


**模板类型存储示例：**

```html
	<!doctype html>
	<html lang="en">
	<head>
		<meta charset="UTF-8">
		<title>{$title}</title>
	{$style}
	</head>
	<body>
	{$content}

	{$script}
	</body>
	</html>
```

如果content占位标记的内容没有限制，可以包含混杂内容，如同时包含HTML和JS，甚至SVG等等。
因此在内容创建时可根据需要进行灵活拆分。

**模板对重复模式的支持**

模板的占位片段包含了可重复属性、必须非空属性等，根据这些属性可以设计出不同的重复模式，如下。

1. 局部模式：简单的占位片段连续重复

一种设计方式是，将重复的引用直接以数组存储为对应的key值，例如

*[code sample 1]*
```javascript
content: {title: "AAA", content: [1,2], script: [3,4,5]}
```

以上存储格式可转换为另一种格式（更加灵活）：

*[code sample 2]*
```javascript
content: [
	{title: "AAA", content: 1},
	{content: 2},
	{script: 3},
	{script: 4},
	{script: 5}
]
```

2. 整体模式：当文档包含多个内容实体时，以模板为整体进行数据重复

*[code sample 3]*

```html
  <!-- 散文 -->
  <article class="essay">
    <header>
       <span class="essay-title">{$title}</span>
       <p class="essay-info">{$desc}</p>
    </header>
  	<section class="essay-body">${body}</section>
  </article>
```

```javascript
// 文档的content存储
[
	{"title": "小马过河", "desc": "xxxxxxxxxxxxx", "body": 34},
	{"title": "大象穿过沙漠", "desc": "yyyyyyyyyyyyy", "body": 35},
	{"title": "海洋世界", "desc": "zzzzzzzzzzzzz", "body": 36}
]
```

最后编译结果：

*[code sample 4]*
```html
  <!-- 散文 -->
  <article class="essay">
    <header>
       <span class="essay-title">小马过河</span>
       <p class="essay-info">xxxxxxxxxxxxx</p>
    </header>
  	<section class="essay-body">
  		...
  	</section>
  </article>
  
  <!-- 散文 -->
  <article class="essay">
    <header>
       <span class="essay-title">大象穿过沙漠</span>
       <p class="essay-info">yyyyyyyyyyyyy</p>
    </header>
  	<section class="essay-body">
  		...
  	</section>
  </article>
  
  <!-- 散文 -->
  <article class="essay">
    <header>
       <span class="essay-title">海洋世界</span>
       <p class="essay-info">zzzzzzzzzzzzz</p>
    </header>
  	<section class="essay-body">
  		...
  	</section>
  </article>
```

3. 混合模式：部分类型的交替重复

如果*[code sample 1]*中content位被设计成必须非空，则示例内容不能被存储，合法的内容示例如下：

```javascript
content: [
	{title: "AAA", content: 1},
	{content: 2},
	{content: 3, script: 6},
	{content: 4, script: 7},
	{content: 5, script: 8}
]
```

支持重复编译过程的编译器称之为**重复式编译器**。

前两种模式可看作混合模式的特例，实际上只需要关心每个片段自身的规则限制：可重复性和非空限制。
存储时，如果某个key值为空并且无非空限制，则直接删除这个key。

界面上提供两种内容增减控制按钮：

	1. 如果整体是可重复的，可一次性添加/移除一整块模板。
	2. 如果某个占位是可重复的，可添加/移除独立占位区。

编译器的这种重复编译模式实现了片段内容的细粒度控制：

	1. 减少了冗余的结构信息；
	2. 非核心数据、简短内容可以不存储为片段，直接填充在文档content字段对象的某个key中，成为文档“私有内容”，简化清理、提高片段信息质量；

如何直接将内容填充在文档的content字段？

	这种情况下，内容不被抽象为片段存储，因此在模板设计上需要考虑这一点。可以定制模板编辑视图：
	动态添加两种类型的输入区（textarea），一种为“片段”，另一种为”直接内容“。
	编辑区以focus: placeholder=>value，blur: value=>placeholder的交互方式编辑其对应的key值描述信息。

模板生成程序：

	1. 编辑模板内容，完成后，点击[下一步]；
	2. 程序生成编辑区布局，用户给每块编辑区定义其属性规则；
	--3. 选择布局模型，并调整各编辑区大小（或者可以自定义css内容）；
	3. 存储模板内容和属性信息，结束；

模板一般不能适用于所有场景，对于复杂的实例类型，允许自定义模板，或者不使用模板。（如angularjs应用）

+ 模板不支持循环语法，虽然这能明显压缩大量重复模式的内容，但解析稍微复杂，更重要的是存储一致性难以保证。
+ 模板不支持嵌套，因为这同样会增加编译复杂度。

以上两种方式是过度的设计，会导致文档的片段组成结构过于复杂。

片段的定义边界是：

	只解决内容的拆分和共享问题，通过模板定义实现拆分，通过片段存储实现内容共享，编译程序实现共享内容的重组。
	增加对循环或嵌套的支持无疑是相当于为模板引入了类似预处理语言的特性。

因此，如何处理重复模式的压缩应该交由脚本程序自行解决。（例如Mustache模板引擎可以完成这样的事情）
纯粹的数据可以直接存储为 XML片段 或 JSON数据脚本，甚至是HTML注释，调用简单的脚本解析过程即可，
并且，包含重复模式处理逻辑的脚本片段可以达到同样的数据压缩效果，并且更加灵活。

### 3.2.2 内容安全性设计

#### 安全机制

文档与片段是多对多的关系，片段被文档共享引用，因此需要一种安全的机制保证文档内容的独立性。
当共享片段被修改时，系统以创建副本的方式保证文档完整性。
这种机制使得整个系统的内容共享与版本记录之间达到了一种平衡：

	1. 对于大容量的（例如库和框架源码）、内容相对稳定的（例如古文诗词）内容，表现出共享特征；
	2. 重要的、反复修订的内容（例如学习笔记），表现出版本记录特征 —— 用户根据自己的喜好来灵活处理以减少内容冗余和加强版本管理。

多数情况下内容的共享是侧重点 —— 系统的默认行为是在文档、片段编辑不影响其它内容的情况下尽量避免创建新的副本。

#### 片段历史记录

片段的历史记录是树状的，通过片段的prev信息串接。
*历史记录的对象是片段而不是文档*，这一点要认清。因为片段和片段链都是完全共享的，一条链上有多个文档，内容修改只能构成片段本身的历史痕迹。
将这种”版本“概念附加在文档上是没有意义的，至少不能完整对应到文档的某次修改。

**在设计上也要避开”文档历史“的概念，只从片段的角度来展现版本数据**

#### 文档历史记录

文档的内容编辑是通过直接操作片段来间接完成的，因此，文档的历史版本是由各组成片段的版本链共同、间接体现的。
并且，文档的历史版本并不完全对应到片段的整个历史版本，透过文档只能看到从目标片段的初始引用版本到现在的
引用版本之间的历史痕迹，**这条历史记录只是整个片段历史记录树的某个分支的子链**。

从设计的角度考虑，暂不提供文档的历史记录查看支持。但在原理上，可以通过时间戳来鉴别某一个文档历史版本对应的N个片段版本 —— 
这N个片段是在同一个时间点上被修改的，*系统在文档修改逻辑上保证了这一点*。

如何标记文档历史记录的起止？

系统将记录文档各组成片段的首次引用版本（并不是片段自身的最初版本）和当前引用版本，并持续跟踪（新增或删除）。

扩展设计：
	片段的副本存储机制不适用于完整的历史记录，因为不是每次修改都会保存为副本，相邻父子片段节点之间存在丢失修改痕迹的情况。
	但如果每次都存副本，频繁的修改将造成大量的内容冗余，降低了管理效率。
	
	后期如需要完整的内容撤销策略，可以借鉴git的版本对比算法和存储管理机制，这种机制只记录相邻两个父子节点之间的每次修改。

### 3.2.3 版本管理


版本管理是一个功能子集，提供依赖查询与分析、清理助手等功能。

### 3.2.4 配置设计思路

**细节**

类型划分，包括“文档”类和“应用”类，区别通常如下：
	文档类需要使用统一的模板并自动套用，而且是仅需要一个内容填充位。
	而应用类则需要自行指定各部分的内容，并且应用的操作不局限于查看。
	二者有一定的相似性，严格来说，“文档”类属于一种特殊的应用。可能包含更特殊的文档，比如API手册文档。

	重要的一点：数据可以提供本地缓存，很多场景下，缓存可能是为了满足应用的需求，例如：
		* 应用需要一种完整的HTTP Server模型来运作。
		* 某种应用缓存可能有利于提高效率，避免数据库查询。
		* 部分测试数据可能需要缓存以支持应用方便的修改，或者生成临时数据。
	即便应用需要缓存，它们仍然将被保存在数据库中，缓存只是临时的副本，因为需要考虑以下因素：
	    * 应用繁多时，如何共享同样的库和框架？
	    * 如果应用都缓存在文件系统，如何与文档搜索等数据库管理流程统一？
	应用保存在数据库有利于保护数据、节省数据空间（因为并非所有的应用都适合缓存在文件系统）
	最终目的还是为了方便管理。
文档链接URL，可能需要两种，一种是本地存储的引用URL，另一种是网络来源URL.
“在新标签页中打开”功能，页面被固定到新标签页中时，如何支持刷新操作？
	方案一：所有数据库查询的资源被内存缓存，监听刷新操作，直接渲染。
	方案二：监听刷新操作，重新查询。（查询程序为全局函数）

** 应用缓存的数量有一定限制，例如最多缓存10个本地应用，缓存策略可以根据使用时间进行筛选**
例如窗口激活时对当前应用开始计算使用时间，窗口失焦后，停止计时（这是一个通用计时组件，不应该专为缓存而设计），再次激活窗口继续累加计时。
最后将总停留计时与已缓存应用的计时相比较，淘汰掉计时最小的，并缓存新的应用。（**缓存即导出过程**）


# 4 详细设计

## 4.1 片段操作

### 4.1.1 字段设计

片段存储包含一些重要的字段，以下列出：

* @name 片段名称。由文档创建的片段，其名称为程序自动命名，可以合理利用名称存储额外信息。
* @host 片段宿主。不为NULL时，它的值具有传递性；为NULL时，因文档编辑自动创建的副本其host为文档id，手动编辑视图备份的副本其host仍为NULL。
* @tied 片段独立性标志。其值具有传递性，可以通过”[取消托管]()“/”[系统托管]()“操作修改，*打破连续传递性*。独立的片段被视为重要内容，不随文档一起删除。
* @alive 片段可用性标志。值为0时，片段不可用，即被锁定。详见[回收文档](#doc-recycle)。

host字段标示片段的生成途径，即“how it was born”。它的值只与创建方式有关，与其它字段没有关联。
tied字段标示片段是否具有独立的生命周期，即”how it will die“。如果值为1，则生命周期为[系统托管]()，以文档引用计数的方式被回收。

注意事项：

- [回收文档](#doc-recycle)时，其引用片段的host值并不立即置NULL，仅仅在真正执行删除的时候修改。
- [还原文档]()时，文档自身及其组成片段alive标志位复位置1即可。


### 4.1.2 创建

创建内容节点并存储，分为”创建新片段“和”创建副本”两种行为。

#### [创建片段]算法

<a name="frag-create">【定义】</a>

新建片段并存入数据库。从无到有，如果指定了独立标志位，则按该标志位存储。
如果是从已有片段创建分支，则tied标志位继承；
否则，如果是直接创建，tied为0，由文档创建则tied为1。

【参数】

- @fragObj 待保存的片段数据对象
- @docid 文档id，将赋给片段host，默认为NULL

【步骤】

1. 设F=fragObj，取F.id，如果是undefined，则设F.tied=[docid!=NULL]，且F.prev=NULL；
2. 否则设F.prev=F.id，并删除F.id；
3. 设F.host=[docid]，存储F；
4. 结束；

#### [保存片段]算法

<a name="frag-save-or-update">【定义】</a>

在编辑某个片段并保存时，根据情况新建片段、修改片段内容或创建副本。

【参数】

- @fragObj 待保存的片段数据对象
- @docId 可选，当前编辑的文档id，默认为NULL

【步骤】

1. 取fid=fragObj.id；
2. 如果 !isNaN(fid) && isLeaf(fid) && !hasRef(fid)，则片段直接更新保存，转至4结束；
3. 传递参数fragObj, docId，调用[创建片段](#frag-create)算法；
4. 结束；

【规则】

仅在直接编辑片段时会用到该算法。

- TODO 文档、片段均有修改历史记录，每次修改应该有修改注解，说明修改了什么内容，像github提交一样。
- TODO [依赖查询算法](): 为避免冗余，界面指示片段的依赖实体（文档和子孙片段），并提供更新操作入口（耗计算，可手动配置是否自动查询）

#### [独立备份]算法

<a name="frag-save-alone">【定义】</a>

将重要的片段存为新的副本并标记为未托管状态，使其获取单独修改的权限。

【参数】

- @fragObj 当前片段对象
- @docid 文档id，将赋给片段host，默认为NULL

【步骤】

1. 设fragObj.tied=0（由于是引用传递，可能要先copy）；
2. 传递参数fragObj, docid，执行[创建片段](#frag-create)算法；
3. 结束；

【规则】

独立备份的片段不随文档一起删除，需要手动删除。
独立备份的片段将获取单独编辑的权限，系统托管的片段不具有此权限。


### 4.1.3 查询及显示


### 4.1.4 修改

#### [撤消更改]算法

<a name="frag-reload">【定义】</a>

1. 重新载入片段内容。
2. 结束。

#### [系统托管]算法

<a name="frag-tieup">【定义】</a>

1. 将目标片段的tied字段更新为1；
2. 保存，结束；

【规则】

- TODO host为NULL的独立片段，其没有依附的文档对象，不能对其执行[系统托管]()更改

#### [取消托管]算法

<a name="frag-untie">【定义】</a>

该操作将特定版本的系统托管片段直接更改为非托管状态，而不创建副本。
该操作不会影响到关联的其它片段版本。

【步骤】

1. 将目标片段的tied字段更新为0；
2. 保存，结束；


### 4.1.5 删除

#### [回收片段]算法

<a name="frag-recycle">【定义】</a>

如果被检测片段不再有文档引用它，则回收这个片段。

【参数】

- @fid 片段id

【步骤】

1. 如果 !hasRef(fid)，则[[Set]]< fid, alive = 0 >；
2. 结束；

【规则】

[1]片段回收没有株连性，即不会影响到它的父节点。
[2]片段回收不更改它的adate属性。


#### [批量回收片段]算法

<a name="frag-batch-recycle">【定义】</a>

批量回收给定的片段集合，使用批量判定算法。

【参数】

- @fidArr 片段id集合

【步骤】

-- TODO 独立片段也有回收的概念（需要手动回收），这里仅是自动回收算法？

1. 对集合fidArr中的每一项fid:
	如果
		[[Get]]< fid >.tied && !hasRef(fid)
	则
		[[Set]]< fid, alive = 0 >
2. 结束；

【规则】

批量回收算法本身会跳过非系统托管的片段。

【相关SQL语句】

	update Fragment set alive=0 where tied=1 and not exist (
		select 1 from RefChain where ifid in (${fidArr}) or cfid in (${fidArr})
	);

#### [片段链回收]算法

<a name="frag-chain-recycle">【定义】</a>

当文档或者片段被回收时，回收相关片段链上满足回收条件的系统托管节点。

【参数】

- @fid 片段id
- @docId 切断对该片段引用的文档id（不一定是宿主），默认值为null表示直接回收某个片段

【步骤】

1. 取 F = [[Get]](fid)；
2. 如果F.tied，传递fid，执行[回收片段](#frag-recycle)算法；
3. 如果 F.host != docId || F.prev == null， 转至5结束；
4. 取 fid = F.prev，转至1；
5. 结束；

[注解]条件3中
	F.prev == null表明已是头节点，这没什么难理解的，关键是
	F.host != docId 表明，从父节点开始已不再是当前文档的寄生片段，不受当前文档影响。
	具体情况：
		1. 若F.host=null,F可能为独立片段，或者它的原宿主文档早已被删除（不存在了）；
		2. 若F.host有值，F还被其它文档引用着，F及其父链不应该被影响；

两种删除法设计：

	1. 删除文档时，对于片段的处理，只删除从初始引用到当前引用之间的部分，这不会导致片段链的拆分；
	2. 删除文档时，参照host进行片段链的删除，这会导致片段链的拆分；

[另一种算法：先收集可回收片段的id集，集中一次性更新]

【规则】

[1] 被回收的片段不可再被索引，以避免被新创建的文档重新引用这类的情况；
[2] 文档所引用的片段，要么是被这个文档创建（即宿主为当前文档），要么该片段是文档的初始引用片段；（**重要结论**）

- TODO 
配置：[x]删除文档时，保留被独立片段引用的共享片段。（包装函数：根据配置项启用或禁用isLeaf()判断即可。）
需要考虑的是，当配置发生变更时，如何整理数据？

#### [还原片段]算法

<a name="frag-recover">【定义】</a>

还原被回收的片段。

【参数】

- @fid 片段id

【步骤】

1. 如果 !hasRef(fid)，则[[Set]]< fid, alive = 1 >；
2. 结束；

【规则】

[1]被托管的片段只能以还原文档的形式间接还原，独立片段可直接还原操作。


#### [删除片段]算法

<a name="frag-delete">【定义】</a>

从片段所在链上删除自身，并重新衔接断开的链。

【参数】

- @fid 片段id

【步骤】

1. 设fragObj = [[Get]](fid)；
2. [可选]验证 fragObj.tied = 1 && !hasRef(fid)，如果为false则结束；
3. 设F = [[GetNext]](fid), [[Set]]< F, prev = fragObj.prev >；
4. [[Remove]](fid)；
5. 结束；

【规则】

删除操作只能由系统自动执行，只能从回收站删除，不能直接删除。
只能删除不被引用的片段，可配置系统是否在删除前自动验证操作安全性。


## 4.2 文档操作

### 4.2.1 字段设计

### 4.2.2 创建

#### [创建文档]算法

<a name="doc-create">【定义】</a>

创建一个新的文档，可能有部分片段引用现有的，另一部分则由文档创建（包括从现有片段修改而产生的副本），需要加以区分。

【参数】

- @docObj 文档数据对象
- TODO 待定（自上而下设计：先从视图保存文档时需要的参数进行设计，再考虑API层）

【步骤】

- TODO may change
文档编辑好后，按以下步骤进行存储：

1. 存储文档实体信息，content预置为空，获取文档id，记为docId；
2. 遍历编辑区域，每个编辑区域被存储为一个片段，设集合S, O为空，步骤如下；
	1. 如果片段是被引用的，且未被修改（dirty标志位为0），则将片段id加入集合S，转至5（结束）；
	2. 如果片段是被引用的，旦已被修改（dirty标志位为1），则更改片段属性 [[id]]=>[prev]，[[tied]]=>[tied]；
	3. 如果片段是新创建的，则更改片段属性 null=>[prev], [[tied|1]]=>tied；
	4. 更改片段属性 docId=>[host]，并将当前片段对象加入集合O；
	5. 处理下一个片段；
3. 存储文档新增的片段集合O，在每个插入记录的回调中记录id信息到集合N，并记入文档content数据对象，记为N；
4. 更新文档content字段；
5. 将文档的片段组成S+N记录到初始引用表；
6. 存储新增的标签；
7. 存储文档-标签关联信息，触发器更新标签计数；
8. 存储文档-分类关联信息；

### 4.2.3 查询及显示


#### [文档加载]算法

- TODO 
【定义】

【步骤】


### 4.2.4 修改

#### [文档修改]算法

- TODO 
【定义】

【步骤】

文档修改点击保存时，如果被修改的片段没有被其它文档引用（the [getRefDocIds()]() algorithm），
且是叶子节点（the [isLeaf()]() algorithm），则片段直接保存。
否则[创建片段](#frag-create)的新副本，并更新文档内容的引用，以及文档修改时间戳。

高级配置选项：
    1. 启用历史记录（在一次编辑交互过程中，每个被修改片段的首次保存将自动备份，再次保存则直接更新）
    	1.1 允许手动备份片段（一个操作按钮，仅在片段内容自保存之后再次被修改时可用）

the [getRefDocIds(fid, [excludeDocId])]() algorithm:

() => select did from RefChain where (ifid=[fid] or cfid=[fid]) and did!=[excludeDocId];

the [getRefFragIds(docId)]() algorithm:

select ifid, cfid from RefChain where did=[docId]; => return [ifid].union([cfid]);

the [isLeaf(fid)]() algorithm:	-- TODO 被回收的非叶子节点可能被误判断为叶子节点，是否应该取消回收判断的限制？

select id from Fragment where prev=[fid] and alive=1 limit 1; => return !results.length;

the [hasRef(fid)]() algorithm:

select id from RefChain join Doc on RefChain.did=Doc.id where (ifid=[fid] or cfid=[fid]) and Doc.alive=1 limit 1; => return !!results.length;

the [hasInitialRef(fid)]() algorithm:

select id from RefChain join Doc on RefChain.did=Doc.id where ifid=[fid] and Doc.alive=1 limit 1; => return !!results.length;

the [hasCurrentRef(fid)]() algorithm:

select id from RefChain join Doc on RefChain.did=Doc.id where cfid=[fid] and Doc.alive=1 limit 1; => return !!results.length;


#### [片段引用替换]算法

<a name="doc-replace-ref">【定义】</a>

将文档的某个片段引用重新指向其它任意一个片段，将旧的历史引用记录销毁并重新建立新的引用记录。

【参数】

- @docId 被修改的文档id
- @nfid 新的引用片段id
- @ofid 当前引用片段id

【步骤】

[开启存储过程]
1. 更新RefChain中[cfid]=ofid && [did]=docId的行，更新[ifid]=nfid, [cfid]=nfid；
-2. 传递docId, ofid，执行[移除引用](#doc-remove-ref)算法；
2. 传递ofid, docId，执行[片段链回收](#frag-chain-recycle)算法；
3. 更新文档content中的片段引用id为片段nfid；
4. 结束；

【规则】

本操作将重设初始引用，丢失历史。
因移除引用可能导致片段被回收，因此无法在同一次编辑会话中撤消恢复之前的引用。

*一个片段不能在同一文档中重复引用*: 文档当前引用片段id不能重复（可引用同一片段的不同历史版本），初始引用无此限制。


### 4.2.5 删除

#### [移除引用]算法

<a name="doc-remove-ref">【定义】</a>

从文档上移除对某个片段的引用，并尝试[片段链回收](#frag-chain-recycle)。
*引用移除操作不可撤销，但仍可通过日志的手段手动恢复引用*（如果片段被回收，仅在回收周期内可恢复）。

【参数】

- @docId 当前操作文档id
- @fid 将被移除引用的片段id

【步骤】

1. 从引用记录中删除满足条件cfid=[fid] and did=[docId]的记录（记录日志）；
2. 传递fid, docId，执行[片段链回收](#frag-chain-recycle)算法；
--3. 设F=[[Get]]< fid >，如果F.host==docId，则[[Set]]< fid, host = NULL >，取fid=F.prev并迭代此过程（记录日志）；
	--TODO 如果只有部分能回收，回收的片段其host可以不清除（这样可以完整恢复，但仍要针对文档内容的恢复设计一种方案，例如输出更改前后的上下文对比视图），但未被回收的片段其host仍被保留会有什么问题？
	（想像如果回收的片段到期被删除了，则这些保留了host但未被其host文档引用的片段一定有被其它文档直接引用，或者是非托管的片段，
	这些残留的host值并不会产生任何负面影响，至少可能有些历史痕迹参考用途）
3. 从文档内容中删除对片段的引用（日志记录key变化的前后值）；- TODO 恢复日志支持
4. 结束；

【规则】

[1] 该操作不一定导致有片段被回收，但被回收的片段在允许的时间内可以还原，并可以通过日志手段恢复引用。
	- TODO 回收站显示被回收的托管片段，设计良好的日志格式以支持日志恢复。（当托管片段的host文档未处于回收状态时，则其在回收站显示，否则，不显示）
	- TODO 将日志设计为独立的数据库文件，一般只有增删操作，没有改操作，比较稳定，并且可以方便地进行可视化管理。
	- TODO 日志系统尽量简单灵活，能支持移植确保特殊情况下可用性。
[2] 仅在编辑文档时允许该操作。


#### [回收文档]算法

<a name="doc-recycle">【定义】</a>

将文档和引用链上可删除的片段置入回收站，延时删除。

【参数】

- @docId 待回收的文档id

【步骤】

算法一：

1. 更新文档alive标记为0；
2. 执行getFragIds()算法，提取文档的当前内容引用片段，记为集合A；
3. 对A中的每个fid，传递fid, docId，执行[片段链回收](#frag-chain-recycle)算法；
4. 结束；

算法二（优化版）：

1. 更新文档alive标记为0；
2. 执行getFragIds()算法，提取文档的当前内容引用片段，记为集合A；
3. 设S=< >，迭代各条链上节点，步骤如下：
	3.1. 映射转换：B = A(fid)=>[[Get]](fid)；
	3.2. 将A并入S；
	3.3. 重设A=< >，对B中每个记录F，如果F.host == docId && F.prev != null，则将F.prev加入A；
	3.4. 如果A非空，则转至3.1，否则跳出；
4. 对S中的记录执行[批量回收片段](#frag-batch-recycle)算法；
7. 结束；


【规则】

回收站不直接显示托管的片段，但有方法在回收站中查询文档的组成片段，选择执行还原并取消托管操作。
- TODO: 多个含回调的异步操作能否被包含到一个事务中执行？如果可以，那么这些异步操作有各自独立的回调吗？

【相关SQL语句】

	select * from Fragment where id in fidArr and host=docId;
	select prev from Fragment where id in fidArr and prev is not null; - 不需要

#### [删除文档]算法

删除文档是一个与创建文档同样复杂的过程，分为两个阶段完成。
第一阶段为“回收”：仅仅给内容实体标记一种状态，不再被检索或编辑，在系统规定的时间内仍可恢复到正常状态。
第二阶段为“删除”：在周期检测中，删除无剩余生命值的文档及片段，即真正的物理数据删除操作。

<a name="doc-delete">【定义】</a>

超过回收站存活期后，彻底删除文档。

【参数】

- @docId 文档id

【步骤】

1. 删除分类关联数据、文档标签数据；- TODO SQL语句；
--2. 获取文档引用片段中alive=0 || tied && !hasRef(fid)的子集，记为集合A，删除引用记录数据；
2. 获取文档引用片段中alive=0的子集，记为集合A，删除引用记录数据；
3. 对A中的每一个fid，调用[删除片段](#frag-delete)算法；
4. 更新所有host为docId的片段，设置host=null；
5. 删除文档自身记录，触发事件。
6. 结束；

【相关事件】

doc::deleted, doc::before-delete ?

【相关SQL语句】
-- TODO
	[select id from Fragment where id in < S > and not exist (select id from RawFrag where id=Fragment.id)]
	T <= select id from Fragment where id in < S > and id not in (select fid from RawFrag where fid in < S > union select id from Fragment where prev in < S > and id not in < S >)
	update Fragment set prev=null where prev in < T >;
	delete from Fragment where id in < T >;

## 4.3 文档打开过程

## 4.3.1 文档查询

## 4.3.2 文档组装

## 4.3.3 文档显示

## 4.5 系统优化算法

### 4.5.1 统一定时任务

软删除的数据记录了两个时间点：

1. 生命值（600s）
2. 软删除操作时间（当前时间，以下简称 删除时间）

每次应用启动，启用一个周期任务循环（taskLoop），默认周期1s（可配置）。
并且，此任务循环并不直接处理任务，它包含若干个频率分支。
使用倍率判断衍生出不同的周期任务，如周期数i%60==0代表60s周期的minLoop。
包含如下几种周期： 10s, 30s, 1min, 10min(600s), 30min(1800s)
每个任务在加入队列时记录了当前loop的计数，例如:

```JavaScript

		var __task_callbacks__ = [];

		taskLoop = {
			// @options.loop 执行次数
			// 因为不能随意指定周期，各周期方法不共用，单独声明。
			add600sLoop: function(callback, options) {
				if(__task_callbacks__.length > MAX_TASK_100) {
					//warn... return;
				}
				__task_callbacks__.push({delay: 600 * 1000, digest: callback});	// 一次性任务，超过指定延时后执行一次，并删除任务
				__task_callbacks__.push({delay: 600 * 1000, loop: 3, digest: callback});
			},
			remove: function() {	// 移除某个任务
			}
		};
```

任务循环如下：

```JavaScript

	// const APP_START_TIME = Date.now();

	function loop(){
		runtime = Date.now() - APP_START_TIME;
		_timer++;	// 不准确，应取Date.now()和起始时刻之差。
		var list = __task_callbacks__.fiter(function(task){
			if((_timer >= task.endLoop) {
				task.digest();
				return true;
			}
			return false;
		});
	}
	setInterval(loop, 1000);	// 不要使用匿名函数？
```

这种方式极大地减少了任务处理的判断次数。
对软删除列表中的数据启用一个延时删除任务，并将此任务加入taskLoop的1min周期队列，
即每隔1min检查一次删除任务队列中是否存在已过期的任务。
过期任务删除成功后，转为真正删除状态，此时同步删除数据库中的数据（无法再找回）。
应用退出时，删除任务队列中可能还存在未到期的任务，退出时登记程序退出时间即可。
下次启动时，重新启动新的延时任务，并更新软删除数据的生命值，新的生命值计算方法如下：
新生命值 = 原生命值 - (应用上次退出时间 - (删除时间 > 上次启动时间 ? 删除时间 : 上次启动时间) ）

## 4.6 应用本地缓存过程



# 5 附录


## 5.2 插件建议

### 5.2.1 对页面段落内的链接，调用外部浏览器打开。
最好可以配置外部软件路径，增加通用性规则以方便灵活配置。

### 5.2.2 倡议：基于自定义事件的回调约定

回调事件参数信息应该尽可能详细、有用。例如：

doc.trigger(事件:e[附带文档属性])



### 1.树形菜单的图标
用列表实现，样式控制图标（list-style-image）。对li添加class属性可应用不同的图片。
可以减少&lt;img /&gt;标签。

### 2. 代码引用

1. 引用标记（无副本）
示例：

<!-- code_uid1 -->
<!-- code_uid2 -->

侧栏工具栏菜单：[搜索][代码][图片][链接]



无脚本客户端保持可用性方案

主页面->无脚本页面->检测脚本启用->location.replace("有脚本页面")
loction.href创建了一条新的历史记录，导致用户点击后退返回无脚本页面，而location.replace()
方法不会（覆盖旧页面的历史记录）创建，这样点击将退到主页面。（—— PPK谈javascript, P35）


www.quirksmode.org (兼容性列表)


P111 例46

select Sname from student where [not exist
	(select * from Course where Cno not in
		(select Cno from SC where SC.Sno=student.Sno)
	)
]

select Sname from S,SC shere S.Sno=SC.Sno group by Sno
	having count(*)=(select count(*) from C);


js不支持尾递归，使用while消除递归：

```js
sum[0] = sum[1] = 1;

while(i++ < n) {
	sum[i] = sum[i-1] + sum[i-2];
}
return sum[i];

或:

n1=n2=1, n3;
while(i++ < n) {
	n3 = n1+n2;
	n1 = n2;
	n2 = n3;
}
return n3;


## Todo
- [-] add editor bindings for common style opération (bold, underline, add link, images...)
  - [ ] lists, bloquotes, titles
  - [ ] images, links
- [ ] add recent list file management to the file menu
- [-] better css for markdown rendering
  - [-] add some styles
- [ ] export generated output to pdf
- [ ] integrate with github api to read and save directly from github
