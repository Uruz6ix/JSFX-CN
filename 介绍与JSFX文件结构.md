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
* in_pin:**name_1**  
  in_pin:**name_2**  
  out_pin:**none**  
  这些可选描述行定义了每一个JSFX引脚（效果通道）的名称，将在REAPER插件的引脚连接对话框中显示。  
  如果唯一一个被命名的in_pin或out_pin的标签为"none"，REAPER将认定这个效果没有音频输入（或输出），这可以用于一些过程优化。只作用于MIDI的效果应该规定in_pin:none且out_pin:none  
* filename:**0**,**filename.wav**  
  这些描述行用于规定在之后代码中使用的文件的名称。这些定义包括索引**0**和文件名("filename")。索引必须从头开始没有间隔地依次排下，且第一个索引必须为**0**。  
  要使用通常的文件，这些文件应该在REAPER\Data目录中，确保它们可以通过
  **file_open()** 打开，并传递其文件名索引。<font color=#aaaaaa>*此处翻译存疑*</font>  
  你也可以用此描述行规定一个PNG文件。如果你规定了一个PNG文件，它应该在此效果的同一个目录中打开，你可以将其文件名索引作为**gfx_blit()** 函数的变量。<font color=#aaaaaa>*--REAPER 2.018+*</font>    
* options:**option_dependent_syntax**  
  这一行用于规定JSFX选项（用空格来区分不同的选项）：
  * options:gmem=someUniquelyNamespace
  这个选项允许插件分配自己的全局共享缓冲区，详见**gmem[]**   
  * options:want_all_kb
  为这个插件之后的新对象开启"Send all keyboard input to plug-in"（将所有键盘输入发送到插件中）选项，详见**gfx_getchar()**   
  * options:maxmem=XYZ
  使这个插件的最大可用内存限制为指定内存槽。默认大小为8M，最大大小为32M。可以通过 **_memtop()** 检查该脚本的内存是否可用。
  * option:no_meter
  使该插件没有显示仪表 *此处原文为meters，翻译存疑*   
  * option:gfx_idle <font color=#aaaaaa>*--REAPER 6.44+*</font>
  规定后，**@gfx**将被周期性调用（尽管速率可能降低）即使UI关闭。这种情况下，**gfx_ext_flags**将被设定为2.
  * options:gfx_idle_only <font color=#aaaaaa>*--REAPER 6.44+*</font>  
  规定后，**@gfx**将只能被周期性调用，且UI不会被禁用。对于那些没有自定义UI但想从UI线程中完成空闲处理的插件非常有用。 
  * options:gfx_hz=60 <font color=#aaaaaa>*--REAPER 6.44+*</font>  
  规定后，**@gfx**部分将以一个接近指定频率的速率运行（请注意，代码应该使用音频样本统计或者**time_precise()** 来单独获得帧率，而非依赖更新频率）。  
* import filename <font color=#aaaaaa>*--REAPER v4.25+*</font> 
  你可以指定一个文件进行导入（将在JS效果的目录下搜索文件名）。通过这个指令导入文件，将会使所有在导入的文件自身**@init**部分定义的函数在原效果中有效。此外，如果导入的文件实现了其他部分（例如@sample 等），而原文件没有实现这些部分，则将使用导入文件的这些部分。  