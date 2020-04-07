# 简介
Ant的全称是Another Neat Tool，意为另一个好用的工具，是Java中比较常用的项目构建工具。构建过程包括编译、测试和部署等,这里介绍的版本是1.10.1。


# 主要特点

和传统的Make工具相似，能为我们完成工程发布流程中一系列机械工作。并且具有良好的跨平台特性。使用XML来表述构建过程与依赖关系，用task替代shell，语义清晰，便于维护。具有强大的任务系统，便于扩展。其中，task以Java class的形式存在。
为了方便使用，Ant自带了很多默认的task，如：

* echo: 输出信息
* mkdir: 创建文件夹
* exec: 执行shell命令
* delete: 删除文件
* copy: 复制文件

通过组合这些默认task和自己实现的task就能够完成Java项目的构建任务。


# 例子
使用Ant需要编写build.xml来配置任务流程。当然，可以通过-f参数指定其他配置文件作为任务流程描述文件。一个Ant的配置文件如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<project name="helloWorld" basedir="." default="usage">
    <property name="mvn" value="mvn"/>
    <property name="script.lock" value="/build_home/scripts/lock"/>

    <target name="usage" description="Prints out instructions">
        <echo message="使用 'lock' 加锁"/>
        <echo message="使用 'unlock' 解锁"/>
        <antcall target="compile">
            <param name="profile" value="test"/>
        </antcall>
    </target>

    <target name="lock">
        <exec dir="${basedir}" executable="${script.lock}" errorproperty="lock.err"/>
        <fail message="u can use 'ant unlcok to force redeploy'...'">
            <condition>
                <contains string="${lock.err}" substring="locked"/>
            </condition>
        </fail>
    </target>

    <target name="unlock">
        <delete file="${basedir}/.lock"/>
    </target>

    <target name="compile">
        <echo message="编译开始"/>
        <exec dir="${basedir}" executable="${mvn}" failonerror="true">
            <arg line="compile -P ${profile}"/>
        </exec>
        <exec executable="${mvn}" failonerror="true">
            <arg value="war:exploded"/>
        </exec>
    </target>
    
    <!--逻辑判断-->
    <target name="testIf" depends="check" if="flag">
        <echo message="if..."/>
    </target>
    
    <target name="testUnless" depends="check" unless="flag">
        <echo message="unless..."/>
    </target>
    
    <target name="check">
        <condition property="flag">
            <or>
		      <and>
			     <isset property="name"/>
			     <equals arg1="${version}" arg2="1.0" />
		      </and>
		      <available file="/project.version" type="file"/>
            </or>
        </condition>
    </target>
    <!--逻辑判断end-->
   
</project>
```
例子说明：
1. Ant使用顶级元素描述整个工程,使用描述全局属性.
2. 用定义工程中的target以及target间的依赖, 在target中定义task的执行流程.
3. 使用antcall来调用target，通过其子节点param传递参数。

此外，在很多场景下需要用到逻辑判断，如if等。Ant中的if如上面的例子所示，是需要搭配target和condition使用的。上面配置中最后的逻辑判断部分，类似如下伪代码：
```
if (name ！= null && version.equals("1.0")) || fileExist("/project.version") ){
    echo "if..."
}else{
    echo "unless..."
}
```

执行ant [target]即可执行任务流程。

# 注意
1. 使用Ant时，一个常见的需求就是通过命令行给Ant传递参数，可以通过-Dname=value这种形式来传递，在build.xml中通过${name}来引用即可。

2. 对于配置文件中重复出现的元素，可以通过refid引用，减少重复配置。
```
<project>
    <path id="project.class.path">
        <pathelement location="lib/"/>
        <pathelement location="${java.class.path}/"/>
    </path>
    
    <target ...>
        <rmic ...>
            <classpath refid="project.class.path"/>
        </rmic>
    </target>
    
    <target ...>
        <javac ...>
            <classpath refid="project.class.path"/>
        </javac>
    </target>
</project>
```