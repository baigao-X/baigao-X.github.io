# 工程配置文件<no value>

# 1. 工程配置文件

## 1.1. .sln文件

Project Solution 文件。 解决方案的配置，管理方案中的多个vcxproj
一般没有sln，也可以直接打开vcxproj，也可以重新生成sln
sln里有多个工程，当你移除某个工程时sln会有变化，sln并不是太重要

## 1.2. .vcxproj

xml 格式文本，描述源文件和头文件，编译选项、连接参数等信息

*.vcproj：**VS2008**以及VS2008之前版本的VS工程文件
*.vcxproj：**VS2010**以及VS2010之后版本的VS工程文件

### 1.2.1. vcxproj.filters

筛选器文件，指定哪些是头文件，那些是source文件等等。

## 1.3. vcxproj.user

user是本地化用户配置，允许多个用户使用自己喜好的方式配置这个项目（例如打开项目时候窗体位置等与项目内容无关的配置）

## 1.4. UpgradeLog.htm