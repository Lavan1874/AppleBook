# 引言

​	从AppleBook中一次拷贝较多内容时，会自动在内容末尾附上版权声明，不利于快速摘录至笔记软件。本文介绍了一种利用 *Keyboard Maestro*实现拷贝时无感去除版权声明，且不影响其他快捷键和软件使用的方法，提高了笔记效率。

# 操作步骤

1. 下载并安装 *Keyboard Maestro*（可以跳过新手指引）
   <center><img src="https://raw.githubusercontent.com/Lavan1874/markdownimage/main/kmicon-1.tiff.png" style="zoom:25%;" /><center>

2. 授予其相关权限<br />
   必须授予的为 Accessibility Permission （无障碍权限）<br />
   权限授予情况可以在软件主界面按下`command`+`,`打开偏好设置，再进入 Security 选项卡查看

   <center><img src="https://raw.githubusercontent.com/Lavan1874/markdownimage/main/Security%20tab.png" style="zoom:35%;" /><center>

3. 在软件主界面 Groups 一栏中点击下方`+`以新建一个分组，可将其命名为 *AppleBook* 或者任何你喜欢的名字

4. 在其命名输入框下方找到`Available in all applications.`选项，点击它并在新出现的选项列表中选择`Available in these applications:`

5. 点击下方绿色`+`，在新出现的选项列表中选择`图书`<br />
   （若没有显示图书选项，点击`Other...`选项手动选择）
   <center><img src="https://raw.githubusercontent.com/Lavan1874/markdownimage/main/Groups_AppleBook_%E5%9B%BE%E4%B9%A6.png" style="zoom:28%;" /><center>

6. 在中间 Macros 栏目下方点击`+`以新建一个宏，并可在右侧将其命名为 *Copy and remove the copyright statement（拷贝并移除版权声明）*或者任何你喜欢的名字

7. 在其命名输入框下方找到`New Trigger`选项，点击它并在新出现的选项列表中选择`Hot Key Trigger`

8. 点击下方新出现的斜体 *Click* ，并键入你想要用于触发此功能的快捷键（本文以 `command`+`C` 为例）；点击右侧`is pressed`选项，在新出现的选项栏中选择你希望的触发方式（本文以`is tapped`为例）

9. 找到并点击下方被虚线框住的`No Action`，此时左侧会从下方弹出可供选择的 Actions 选项

10. 点击最左侧 Categories 栏中的`Clipboard`，在 Actions 复选栏中双击`Copy`<br />
    此时可在右侧看到原被虚线框住的`No Action`被新出现的被实线框住的`Copy`取代

11. 点击最左侧 Categories 栏中的`Execute`，在 Actions 复选栏中双击`Execute an AppleScript`<br />
    此时在右侧原有实线`Copy`框下方出现一新`Execute AppleScript`框，并且光标会自动定位到输入框中

    <center><img src="https://raw.githubusercontent.com/Lavan1874/markdownimage/main/No%20Action_%20Copy_Execute%20an%20AppleScript.png" style="zoom:28%;" /><center>

12. 将以下内容**完整粘贴**入输入框中

    ```
    -- 获取剪贴板的内容
    set clipboardContent to the clipboard
    
    -- 将输入文本按行拆分为列表
    set textList to paragraphs of clipboardContent
    
    -- 设置目标元素"摘录来自"
    set targetElement to "摘录来自"
    
    -- 查找目标元素在列表中的位置
    set targetIndex to 0 (*将目标元素的索引数先设置为0*)
    repeat with i from 1 to count of textList (*用i历遍textList中的所有元素（从1到“textList的数量”*)
    	if item i of textList is targetElement then (*如果第i项命中了设定的目标元素，则...*)
    		set targetIndex to i (*将目标元素的索引数设置为i*)
    		exit repeat
    	end if
    end repeat
    
    -- 移除 目标元素的前一个元素（固定为空行） 和 目标元素之后的所有元素 以及 目标元素本身
    if targetIndex is greater than 2 then
    	set remainingTextList to items 1 through (targetIndex - 2) of textList
    	-- 将“目标元素的前一个元素”之前的所有元素移入一个新list
    	set textList to remainingTextList
    	-- 将新list替代原list
    end if
    
    -- 在元素间添加换行符以恢复原有格式
    set mergedText to "" (*新建一个空的合并文本*)
    repeat with itemValue in textList (*历遍list中的每个元素*)
    	set mergedText to mergedText & itemValue & linefeed (*将每个元素放入合并文本并加上换行*)
    end repeat
    
    -- 移除最后一个换行符以及首尾的引号
    if targetIndex is greater than 2 then
    	set mergedText to text 2 thru -3 of mergedText (*取出合并文本中 第2个字符 到 倒数第3个字符（舍弃 第一个字符（前引号）、倒数第一个字符（最后一个换行符）、倒数第二个字符（后引号））*)
    end if
    
    -- 将调整格式后的文本导入剪切板
    set the clipboard to mergedText
    ```

    <img src="https://raw.githubusercontent.com/Lavan1874/markdownimage/main/Copy_Execute_AppleScript2.png" style="zoom:28%;" />

13. **请注意**：你需要根据系统或图书App使用的语言修改上方代码框中的第8行中引号中的内容，即"摘录来自"（使用简体中文时无需修改）

    ```
    set targetElement to "摘录来自"
    ```

    如：你的系统或图书App使用的语言为`English—英语`时，你需要将 "摘录来自" 替换为 "Excerpt From"

    ```
    set targetElement to "Excerpt From"
    ```

14. 尝试在AppleBook中选择欲拷贝的文字，以你设定的触发方式按下你设定的快捷键，前往其他App中粘贴观察原有的版权声明是否已去除
    （过程中可能会弹出要求授予相关权限，请同意）

# 存在缺点

- 无法解除AppleBook对单次拷贝文字数量的上限，超过上限后会以[…]结尾
- 脚本的实现依赖于找到目标元素：**摘录来自**，若正文中存在单独的一行：**摘录来自**（实际上书籍中很少出现如此情况），则会出现此行以下内容被误删除的错误
- 使用到的软件*Keyboard Maestro*为功能强大的付费软件，请结合使用频率和实际需要决定是否购入正版
- 本人计算机水平十分有限，实现此功能的方式无疑是十分不优雅的，本文仅仅是抛砖引玉，欢迎各位批评指正！





# 感谢

JACK
<br />
ChatGPT
