

IPython3.x中包括了notebook server,qt console等等，是最完整的一个release。在IPython4.x中，与语言无关的部分，如notebook格式，消息协议，qtconsole，notebook web应用等，都移到了一个新的项目中去，并命名为Jupyter。IPython则专注于交互式的Python，其中一部分为jupyter提供了一个Python内核。

Jupyter有很多子项目，如notebook，console，qtconsole等。

Jupyter Notebook包含两个组件：

- 一个web应用：一个基于浏览器的工具，用于交互式创作文档
- notebook文档：.ipynb包含了web应用上的所有可见内容，包括输入输出，解释性文本，数学运算，图片等。

web应用的主要特征：

- 在浏览器上编辑代码，有自动的语法高亮，缩进，补全等
- 使用富媒体（如html）来展示运算结果

notebook文档：