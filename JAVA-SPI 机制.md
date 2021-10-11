# JAVA-SPI 机制
SPI 全称：Service Provider Interface，是Java提供的一套用来被第三方实现或者扩展的接口，它可以用来启用框架扩展和替换组件

## 实现步骤
以实现一个特定的第三方服务为例，并打包成jar包发布，具体实现步骤如下
1. 实现一个服务接口
2. 在`META-INF/services`目录中定义一个以接口类为全名的文件，文件内容是该接口类的具体实现
3. 通过'ServiceLoader.load()' 方法加载对应的接口实现;` ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class)`
