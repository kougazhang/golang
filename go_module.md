### go module 相关命令

**go mod init: 项目初始化**

在当前文件夹下执行:

```shell
go mod init github.com/kgzhang/example
```

`github.com/kgzhang/example` 的作用是作为模块的 `import path`. 这样声明之后, 其他项目导入该模块只需要写为:

```golang
import "github.com/kgzhang/example"
```

如果写为 `go mod init exmaple`, 那么其他项目就无法加载到这个模块.

`go mod init` 执行完毕，就初始化了使用 Go modules 的项目，会多出来一个 go.mod 文件。

**go get: 下载更新包**

- 更新包

  ```go
  // 语法: package@version
  go get -u github.com/kougazhang/requests@v0.0.1rc4
  ```

- 下载包, 默认会下载最新版本.

  ```go
  go get -u github.com/jinzhu/gorm
  ```

- `go mod tidy`

  拉取缺少的模块，移除不用的模块. 

  这个命令如同 python 的 `pip install -r reuqirement.txt`, 搭建项目的运行环境.

- go mod vendor

  将 GOPATH/src/pkg/mod 中的缓存包，复制到项目的 vendor 目录中，即使用每个项目使用自身包的模式，类似之前的 govendor 管理方式

  尽管如此，默认情况下，go build 将忽略该目录的内容。如果想从 vendor/ 目录中建立依赖关系，需要如下参数 build

  ```go
  go build -mod vendor
  ```

### go.mod 文件

`go.mod` 文件记录了当前项目的模块信息，每一行都以一个关键词开头。

#### go.mod 语法

- module： 定义当前项目的模块路径, 即 `go mod init <path>` 命令中的 path.

- go： 标识当前模块的 Go 语言版本，目前来看还只是个标识作用。

- require： 说明 Module 需要什么版本的依赖。

  - `indirect` 注释

  ```go
  module gitee.com/biexiaoyansudian/my.cn
  
  go 1.14
  
  require github.com/jinzhu/gorm v1.9.12 // indirect
  
  //手动注释 indirect 标识表示该模块为间接依赖，也就是在当前应用程序中的 import 语句中，并没有发现这个模块的明确引用，如果没引用，我们提前先拉下来这个包，就会出现该注释，比如直接使用go get拉代码包，而不是 go build 让命令自动根据 go.mod 拉代码包
  ```

- exclude： 用于从使用中排除一个特定的模块版本。在实际的项目中很少被使用，故很少会显式的排除某个包的某个版本，除非我们知道某个版本有严重 bug。比如指令 exclude [github.com/google/uuid](http://github.com/google/uuid) v1.1.0，表示不使用 v1.1.0 版本。

- replace: 替换包的加载路径.

  - 调试第三方包. 把第三方包对应的版本 clone 到本地, 使用 `replace` 把依赖指向本地的第三包, 这样就可以运行本地的第三方包了.

    ```shell
    replace (
        github.com/google/uuid v1.1.1 => ../uuid
    )
    ```

### go.sum 文件

下载依赖包有可能被恶意篡改，以及缓存在本地的依赖包也有被篡改的可能，单单一个 go.mod 文件并不能保证一致性构建，Go 开发团队在引入 go.mod 的同时也引入了 go.sum 文件，用于记录每个依赖包的哈希值（SHA-256 算法），在 build 时，如果本地的依赖包 hash 值与 go.sum 文件中记录得不一致，则会拒绝 build。

### 自己发布一个包

+ go.mod 中的 module 要写成项目的完整路径, 参见上文的 *go mod init: 项目初始化*.

- 合理使用 `git tag`.

  git tag 需要语义化, 遵循一定的规范. 大致规则如下:

  + 小版本更新又分为只更新 patch 和 小版本
    + go get -u 会更新最新的小版本，举个例子，它会将 1.0.0 更新为 1.0.1 或者 1.1.0 这样类似的版本。
    + go get -u=patch 来获取最新的 patch 更新。 举个例子，它会更新到 1.0.1 而不是 1.1.0 版本。
  + 大版本从语义上讲是完全不同的包, 所以不会兼容. 根据语义版本语义，大版本不同于小版本。大版本会破坏向后兼容性。从 Go 模块的角度来看，大版本是一个完全不同的包。一个库的两个不兼容的版本，实质上是两个不同的库。其余的和以前一样，我们推它，把它标记为 v2.0.0 (并可选地创建一个 v2 分支。)

### 使用本地的包

- `git clone xx`, 把远程项目克隆到本地.
- 对克隆到本地的项目执行:  `go mod init <fullPath>`. (参见上文的 *go mod init: 项目初始化*)
-  使用 `replace` , 修改想使用本地包项目的 go.mod 文件, 示例:

```c
// 假设我们本地的 moduledemo 想使用远程项目 mypackage 的本地包
module moduledemo

go 1.14

// 使用 go get 命令后, go.mod 中自动生成的记录.
require "github/xxx/mypackage" v0.0.0

// 使用 replace, 将这个包指向本地的路径 "../mypackage"  
replace "github/xxx/mypackage" => "../mypackage"
```

