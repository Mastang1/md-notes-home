
---
# 1. 基础流程
 ==1. 根据eclipse的流程，需要通过plugin development module进行开发，但是内网没有；所以实际的流程是基于原有的module plugin进行开发；==
 ==2. 理解plugin.xml中各个extension point 对应的files关联方式（xpath指定short path）==
 ==3. 理解 xml、xdm的schema 和 data的关系==
 ==4. 理解EB tresos规定的xdm中node的类型、node的属性在GUI中改的作用==
 ==5. 宏代码模板的基本原理理解==
 ==6. 核心是熟悉xpath常用的API、在EB中的调试方式等==
 ==7. 生成的plugin在发布时候遇到bug，原因是删除了meta-info路径下的manifest.mf文件导致，另外该文件中的Bungle-symbolicname作为plugin的名字，否则别的PC中的plugin不会识别==

# 2. todo