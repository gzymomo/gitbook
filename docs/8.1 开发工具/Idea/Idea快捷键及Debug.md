[TOC]

- Ctrl+Alt+O 优化导入的类和包
- CTRL+SHIFT+SPACE 自动补全代码
- Alt+Insert 生成代码(如get,set方法,构造函数等)   或者右键（Generate）
- CTRL+ALT+L  格式化代码
- CTRL+ALT+I  自动缩进
- Ctrl+shift+U 大小写转化
- CTRL+E      最近更改的代码
- CTRL+P   方法参数提示
- CTRL+Q，可以看到当前方法的声明
- Shift+F6  重构-重命名 (包、类、方法、变量、甚至注释等)
- Ctrl+Alt+V 提取变量

- 上一步 / 下一步
`alt + -> / alt + <-`
类似于我们的浏览器的上一页/下一页，切换到光标上一个/下一个移动的位置

- 查看方法调用链
`control + alt + h`
主要用于读代码的时候，查看方法调用关系，或者重构代码的时候，进行风险评估，即谁调用过我(包含直接调用和间接调用)

- 查看方法调用位置
`alt + f7`
和查看方法调用链类似，即谁直接调用过我

- 关键字搜索
contrl + f 当前文件下的文本查询
contrl + shift + f 全局的文本查询
contrl + shift + n 全局的文件查询

# 查询快捷键
Ctrl＋Shift＋Backspace可以跳转到上次编辑的地
CTRL+ALT+ left/right 前后导航编辑过的地方
ALT+7  靠左窗口显示当前文件的结构
Ctrl+F12 浮动显示当前文件的结构
ALT+F7 找到你的函数或者变量或者类的所有引用到的地方
CTRL+ALT+F7  找到你的函数或者变量或者类的所有引用到的地方
Ctrl+Shift+Alt+N 查找类中的方法或变量
双击SHIFT 在项目的所有目录查找文件
Ctrl+N   查找类
Ctrl+Shift+N 查找文件
CTRL+G   定位行
CTRL+F   在当前窗口查找文本
CTRL+SHIFT+F  在指定窗口查找文本
CTRL+R   在 当前窗口替换文本
CTRL+SHIFT+R  在指定窗口替换文本
ALT+SHIFT+C  查找修改的文件
CTRL+E   最近打开的文件
F3   向下查找关键字出现位置
SHIFT+F3  向上一个关键字出现位置
选中文本，按Alt+F3 ，高亮相同文本，F3逐个往下查找相同文本
F4   查找变量来源
CTRL+SHIFT+O  弹出显示查找内容
Ctrl+W 选中代码，连续按会有其他效果
F2 或Shift+F2 高亮错误或警告快速定位
Ctrl+Up/Down 光标跳转到第一行或最后一行下
Ctrl+B 快速打开光标处的类或方法
CTRL+ALT+B  找所有的子类
CTRL+SHIFT+B  找变量的类
Ctrl+Shift+上下键  上下移动代码
Ctrl+Alt+ left/right 返回至上次浏览的位置
Ctrl+X 删除行
Ctrl+D 复制行
Ctrl+/ 或 Ctrl+Shift+/  注释（// 或者/.../ ）
Ctrl+H 显示类结构图
Ctrl+Q 显示注释文档
Alt+F1 查找代码所在位置
Alt+1 快速打开或隐藏工程面板
Alt+ left/right 切换代码视图
ALT+ ↑/↓  在方法间快速移动定位
CTRL+ALT+ left/right 前后导航编辑过的地方
Ctrl＋Shift＋Backspace可以跳转到上次编辑的地
Alt+6    查找TODO

# 其他快捷键
SHIFT+ENTER 另起一行
CTRL+Z   倒退(撤销)
CTRL+SHIFT+Z  向前(取消撤销)
CTRL+ALT+F12  资源管理器打开文件夹
ALT+F1   查找文件所在目录位置
SHIFT+ALT+INSERT 竖编辑模式
CTRL+F4  关闭当前窗口
Ctrl+Alt+V，可以引入变量。例如：new String(); 自动导入变量定义
Ctrl+~，快速切换方案（界面外观、代码风格、快捷键映射等菜单）

## 快速补全行末分号
使用快捷键 Shfit + Ctrl + Enter 轻松实现。

## 粘贴板历史记录
 Shitf + Ctrl + V 就打开粘贴板的历史记录

# 重构
Ctrl+Alt+Shift+T，弹出重构菜单
Shift+F6，重命名
F6，移动
F5，复制
Alt+Delete，安全删除
Ctrl+Alt+N，内联



# Debug 快捷键

F9： 恢复程序
Alt+F10： 显示执行断点

F8： 跳到下一步(按F8 在 Debug 模式下，进入下一步，如果当前行断点是一个方法，则不进入当前方法体内，跳到下一条执行语句。)

F7： 进入到代码(按F7在 Debug 模式下，进入下一步，如果当前行断点是一个方法，则进入当前方法体内，如果该方法体还有方法，则会进入该内嵌的方法中 .)

Alt+shift+F7： 强制进入代码

Shift+F8： 跳到下一个断点(跳出该方法，可以按Shift+F8，在 Debug 模式下，跳回原来地方。)

Atl+F9： 运行到光标处
ctrl+shift+F9： debug运行java类
ctrl+shift+F10： 正常运行java类
Alt+F8： debug时选中查看值