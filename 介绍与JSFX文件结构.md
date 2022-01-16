# 介绍
这是一个针对为REAPER编写**JSFX**音频处理效果的参考指南。JSFX用动态编译的脚本语言EEL2编写，使你可以修改或者生成音频和MIDI，同时生成基于自定义向量的UI、展示分析图像。  
JSFX是一种很简单的文本文件，在载入REAPER时可以变成全功能插件。它们是分布在源代码中的，因此你可以修改已有的JSFX来满足你的需要，或者自己从头编写新的JSFX。（如果你修改已存的JSFX，我们推荐你将其保存为一个新文件，这样它就不会在REAPER更新时遗失。）  
这个指南会提供**关于JSFX文件结构的概述**、**编写语法**以及**一个含有所有函数和特殊变量的表单**。

# JSFX文件结构
JSFX文件是由一些**描述行**以及跟随其后的一个或多个代码块组成的。  
这些可以被规定的描述行有：
* desc:**Effect Description**
  这一行能且只能被规定一次，用于定义展示给用户的效果名称。理想情况下，这一行应该写在文件的第一行，这样它可以被快速识别为JSFX文件。    
* slider**1:5<0,10,1>slider description**
  你最多可以使用64次此描述行来规定变量，用户可以通过UI控制器（通常是一个推子和文本输入，不过也可以改变，详见下文）来改变这些变量。这些变量也可以在REAPER中自动化。  
  在上面这个例子中，第一个**1**定义其为第一个变量，**5**是这个变量的默认值，**0**是最小值，**10**是最大值，而之后的**1**是单次改变的增量。**slider description**将被展示给用户。  
  **扩展slider选项：**
  * slider**1**:variable_name=**5<0,10,1>slider description**--<font color=#aaaaaa>*REAPER 5.0+*</font>  
  variable_name即前缀被指定为默认值， 这种情况下，slider通过该variable_name（变量名）来改变，而非sliderX。这种定义方式可以与以下任意一种语法结合在一起使用。  
  * slider1:0<**0**,5,**1**{**zerolabel,onelabel,twolabel,threelabel,fourlabel,fivelabel**}>some setting  
  这个变量将会被展示为一个从"zerolabel"到"fivelabel"的选项表。请注意，这些变量应该从0开始设定，并且至少像上面的例子一样有1个增量。  
  * slider1:**/some_path:default_value**:slider description  
   在上面这个例子中，**/some_path**规定了一个在REAPER\Data下的子目录的路径，用于扫描.wav，.txt，.ogg和.raw文件。**default_value**定义了一个默认文件名。如果它被使用，这个脚本通常将在：**@serialize**代码块中使用file_open(slider1)读取选定文件的内容。
   * slider1:0<0,127,1>-**Hidden parameter**  
   你也可以同过在其名称前加上"-"来隐藏slider。这个变量将不会在插件UI中显示，不过仍然自动地发挥作用。 
