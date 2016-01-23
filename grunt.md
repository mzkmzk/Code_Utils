# Grunt

作用: 压缩 编译 单元测试 linting > 自动化

##1. 起步

1. 先确认下npm为最新版本`sudo npm update -g npm`
2. 安装Grunt命令行(CLI)到全局环境`npm install -g grunt-cli`

##2. CLI如何工作的

运行`grunt`,利用node提供的`require()`找到系统安装的Grunt,然后加载,然后传递`Gruntfile`文件的信息

