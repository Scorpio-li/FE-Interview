<!--
 * @Author: Li Zhiliang
 * @Date: 2020-11-25 15:42:00
 * @LastEditors: Li Zhiliang
 * @LastEditTime: 2020-11-25 15:42:02
 * @FilePath: /FE-Interview.git/project/webpack.md
-->
# Webpack 

## 1. webpack 原理

1. 初始化参数：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；

2. 开始编译：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译；

3. 确定入口：根据配置中的 entry 找出所有的入口文件；

4. 编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；

5. 完成模块编译：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系；

6. 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；

7. 输出完成：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。

## 2. babel 原理

babel的转译过程分为三个阶段：parsing、transforming、generating，以ES6代码转译为ES5代码为例，babel转译的具体过程如下：

1. ES6代码输入

2. babylon 进行解析得到 AST

3. plugin 用 babel-traverse 对 AST 树进行遍历转译,得到新的AST树

4. 用 babel-generator 通过 AST 树生成 ES5 代码


## 1. webpack中的loader和plugin有什么区别

- loader它是一个转换器，只专注于转换文件这一个领域，完成压缩、打包、语言编译，**它仅仅是为了打包**。并且运行在打包之前。

- 而plugin是一个扩展器，它丰富了webpack本身，为其进行一些其它功能的扩展。**它不局限于打包，资源的加载，还有其它的功能。**所以它是在整个编译周期都起作用。
