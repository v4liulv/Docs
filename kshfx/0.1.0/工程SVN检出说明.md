# 工程svn检查说明 Project svn checkout explain

### SVN版本信息1.7.0

### SVN检出

#### eclipse
前提条件：eclipse工具并且安装了SVN插件、Maven插件、ScalaIDE插件

第一步： 通过TortoiseSVN工具检出本地文件夹
Svn地址http://dzzw-svn.hnisi.com.cn:8300/svn/ga_code_bmjck/0个人区/刘律/大数据人员关系可视化

第二步: 本地工程文件夹进行
1. Eclipse import
2. 选择Maven下面的Existing Maven projects,点击next
3. 其中Root Directory 选择SVN检查的文件夹路径,自动勾选了全部的maven工程模块
4. 点击Finish, 即可完成，等待Eclipse自动Building maven project.

这样工程即导入成功。可以正常的编写工程代码，提交相关代码等。

注意：如果Eclipse自带的ScalaIDE的插件是2.12或者2.11版本以上,需要修改子模块kshfx_spark的Scala版本为2.10.6.
修改方法：右键工程选择Properties,然后找到ScalaCompiler，点击User Project Settings 打上勾
然后在点击Scala Installation下拉选择2.10.6版本，点击Apply.

这样build工程完成就不报错了

#### idea

**前提**：idea安装Maven插件和SVN插件

步骤：
1. 直接检出项目到本地如：D:\work\kshfx-master-0.1.0

2. idea打开文件即可，idea会自动加载Maven项目

### SVN提交

### 需要忽略文件夹或文件

> * *.class
> * *.log
> * *.jar
> * *.war
> * *.zip
> * *.tar.gz
> * *.rar
> * *.classpath
> * .project
> * .idea/*
> * .svn/*
> * .settings/*
> * target/
> * out/
> * 其他认为不需要提交的部分

### 提交失败处理

#### out of date
处理办法使用Tortoise SVN提交，然后自动update后在commit

#### 代码冲突

如果出现异常尽量使用外部的右键点击文件夹-选择TortoiseSVN进行提交

#### 版本过期
  先update在commit
  
#### 文件相同位置引发的冲突  
* 编辑冲突文件：
    外部右键冲突文件，选择TortoiseSVN-Edit conflicts编辑冲突
    
* 冲突代码合并修改：
   进行冲突代码合并，如果冲突部分您需要修改，最好和别人进行下沟通在进行修改合并
   
* 合并保存提交
   合并完成后，点击Mark as resolved进行保存
    查看文件已经不提示冲突了，点击提交即可。

#### SVN忽略文件夹或文件提交

解决办法：

外部右键点击文件夹-选择TortoiseSVN : Unversion and add..
进行文件夹忽略提交
