<!--
 * @Author: Li Zhiliang
 * @Date: 2020-12-02 16:24:12
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2020-12-02 16:32:58
 * @FilePath: /FE-Interview.git/project/process.md
-->

# 说下工作流程（开发流程、代码规范、打包流程等）

## 一、拿到原型图，先自我解析需求，画出思维导图，流程图

在未拿到 UI 给定的 PSD 时，可以先理清我们的需求
    
- 依赖的外部资源

- 后端提供的接口

- UI 出图的大概布局

- 后期频繁改动的地方

需要实现的效果

- 下拉刷新

- 动画效果

- 吸顶效果

- 懒加载、预加载、防抖、节流

## 二、产品召集项目相关人员，开需求讨论会，产品讲解原型

1. 理解产品的需求，提出质疑：这是什么功能，怎么做，为啥这么做

2. 评估实现难度和实现成本，是否有潜在技术问题/风险

3. 对比自己整理的需求图，如果有和自己想的不符合的，提出疑问

4. 理解 PM 提出此次需求的目的，明白哪些内容是重点，哪些次要，可以适当取舍

5. 如果产品要求提供时间，简单项目可以预估，复杂项目不可马上给出时间，需要仔细评估，评估时间包含开发、自测、测试人员测试、修复 bug、上线准备

## 三、会后进一步整理需求

1. 细化细节，整理有疑问的地方，与产品、设计等其他人进行确认

2. 评估项目完成时间--影响因素：需要的人力、 中间插入的需求、 开发、 自测、 测试人员测试、 修复 bug、 上线准备、 其他风险（如技术选型错误等）

3. **初步制定排期表**

## 四、需求二次确认（开发中遇到不确定的，依旧需要找相关人员进行需求确认，杜绝做无用功）

1. IM 工具沟通确认

2. 邮件确认

3. 小型需求/项目相关讨论会

4. **确定最终排期表**

## 五、开发

1. 技术选型

2. 搭建开发环境：工具链

3. 搭建项目架构

4. 业务模块划分
    
    - 优先级排序
    
    - 新项目介入，需要当前项目和介入项目的相关负责人 Pk 优先级，随后调整项目排期
    
    - 开发过程中发现工作量与预期有严重出入，需要尽早向其他项目人员反馈，方便其修改时间安排

5. 定制开发规范

    - 开发规范
        
        - commit 提交格式:[改动文件类型]：[改动说明]
        
        - 单分支开发或者多分支开发
            
            - 小项目、并行开发少，则只在 master 主分支开发
            
            - 中大项目，需求复杂，并行功能多，则需要分为 master、developer、开发者分支；需要开发者自创一个分支开发，合并到 developer，确认无问题后，发布到 master，最后上线

    - 代码规范
        
        1. jsconfig.json
        
        2. .postcssrc.js
        
        3. .babelrc
        
        4. .prettierrc（vscode 插件 prettier-code fomatter）— 注意与 eslint 要保持一致
        
        5. .editorconfig
        
        6. .eslintrc.js（强制开启验证模式）
    
    - 源码管理
    
    - 版本管理
    
    - 安全管理

## 六、自测

1. 手动测试

2. 单元测试

3. 集成测试

## 七、提测---测试人员测试

1. 开发人员修复 bug

2. 期间不可接手耗时大的需求

3. 有不确定优先级高低的需求，需要各个需求方互相 pk 优先级，再确定做与不做，不能因此拖延项目完成点

4. 测试修复 bug 时间可能比开发时间还长，因此开发者预估开发时间不能乐观

## 八、上线

1. 上线准备 a. 域名申请 b. 备案申请 c. 服务器申请 d. 部署

2. 测试线上环境

    - 有 bug 回到修复 bug 环节

3. 日志监控
    
    1. 调用栈
    
    2. sourcemap
    
    3. 本地日志
    
    4. 用户环境、IP
    
    5. 低成本接入
    
    6. 统计功能
    
    7. 报警功能

## 九、维护

技术创新（对现有的技术领域以及具体项目实现方法进行优化）

1. 提高效率:例如 jenkins 构建部署

2. 减少成本

3. 提升稳定性

4. 安全性