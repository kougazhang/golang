Dep 是类似于 npm 的包管理工具.

## 安装

 `brew install dep`

## 使用

1. 创建一个项目, (也是需要将项目放到 `$GOPATH/src` 下的)

```
$ mkdir -p $GOPATH/src/github.com/me/example
$ cd $GOPATH/src/github.com/me/example
```

2. 将初始化项目

```
$ dep init
$ ls
Gopkg.toml Gopkg.lock vendor/
```

3. 各个文件的作用

   + Gopkg.toml 用于调整 Gopkg.lock 已同步的一些包，如版本高了，在这个文件设置对应版本，运行命令后，就同步到 Gopkg.lock。 有其它包管理工具的注意了 跟 composer 的 composer.json 和 npm 的 package.json 不一样，它不是主要依赖次文件同步管理包，这个文件只是作为 调整，调整，调整！相信您已经记住了。
   + Gopkg.lock 根据项目代码里面的 import 和 Gopkg.toml 文件，获取相关依赖，最后写入到 项目 vendor / 目录下。

4. 更新配置
   Go 语言讲求没用到的东西，就不应该存在在代码里面，所以 dep 包管理工具和其它语言的包管理工具不一样，它根据您项目里用到什么第三方包，来生成对应的依赖配置文件 Gopkg.lock，如果配置文件的软件包存在问题，则通过 Gopkg.toml 文件调整，比如第三方软件包版本高了，需要指定某个版本，那么在 Gopkg.toml 文件写入相关配置，然后 通过 dep ensure -update -v 更新即可，加 -v 可以看到运行过程信息。

5. 其他命令
    + dep ensure 同步 Gopkg.toml 文件修改的配置，加上 -update 则是即同步也更新各个依赖。
    + dep ensure -add github.com/XXX/XX 添加新依赖项，比如官方文档的例子：
    ```
    $ dep ensure -add github.com/pkg/errors
    ```
    成功的话，会更新 Gopkg.lock 文件和 vendor / 目录，并会写入配置约束 github.com/pkg/errors 到 Gopkg.toml 文件。如果你代码里面没有用这个第三方依赖包（import），它还会报告一个警告：
    ```
    "github.com/pkg/errors" is not imported by your project, and has been temporarily added to Gopkg.lock and vendor/.
    If you run "dep ensure" again before actually importing it, it will disappear from Gopkg.lock and vendor/.
    ```
    所以，我觉得没有必要通过这命令来添加依赖，直接在代码里面 import github.com/pkg/errors，再运行 dep ensure 岂不美哉？！

