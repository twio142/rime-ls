# rime-ls

> A language server that provides input method functionality using librime.

为 rime 输入法核心库 [librime](https://github.com/rime/librime) 的主要功能实现 LSP 协议，
从而将编辑器的代码补全功能当作输入法使用。

项目还处在**早期阶段**

目标是提供 rime + LSP 的通用解决方案, 在不同编辑器内实现与其他 rime 前端类似的输入体验，
解决 vim 编辑模式下的输入法冲突和切换问题。

主要使用场景:

- 在 vim 编辑模式下写 markdown, $\LaTeX$ 等文档（更像输入法）
- 在编程时输入特殊的变量名或者写注释（更像代码补全）

## Features

- 用 rime 能输入的东西按理说都能输入 ( 汉字, 标点, emoji ...)
- 支持按数字选择补全项
- 支持候选词翻页
- 多种触发方式
  - 默认开启, 随时补全, 用快捷键控制关闭 (大量输入)
  - 平时关闭, 检测到配置的特殊字符或光标前有非英文字符时触发补全 (少量输入)
- 可以按配置其他 rime 输入法的方式去配置 (只有能影响候选项的配置是有用的)
- 可以同步系统中已有 rime 输入法的词频
- 无需图形界面，可在远程登录服务器时使用
- 可以通过 TCP 远程使用 (无任何加密，谨慎使用) (since v0.1.3)

效果展示：

https://user-images.githubusercontent.com/14821247/213079440-f0ab2ddd-5e44-4e41-bd85-81da2bd2957f.mp4

更多编辑器（nvim, helix, zed, qt-creator）下的展示见 [Showcase](https://github.com/wlh320/rime-ls/wiki/%E6%A1%88%E4%BE%8B%E5%B1%95%E7%A4%BA-Showcase)

## Usage

> [!WARNING]
> 第一次启动时 rime 需要做大量工作, 可能会很慢

### 直接使用

1. 下载 Release / 自己从源码编译 / 包管理器安装
2. 创建一个专属 rime-ls 的 rime 配置目录
3. 配置 LSP 客戶端, 例如:
   - [Neovim](doc/nvim.md)
   - [Vim + coc.nvim](doc/vim.md)
   - [Vscode](doc/vscode.md)
   - [Helix](doc/helix.md)
   - [Zed](doc/zed.md)
4. 像配置其他 Rime 输入法一样在 rime-ls 的用户配置目录进行配置
5. 輸入拼音, 就可以看到补全提示
6. 可以通过修改 rime-ls 的配置项控制补全行为

### 通过容器使用

1. 下载或自行编译 rime-ls 的 docker 镜像，例如 `docker pull ghcr.io/wlh320/rime-ls:master`
2. 根据使用需求参考 docker-compose.yaml 以合适的参数运行容器
3. 其他同上。注意配置客户端时需要使用容器内部的路径

> [!TIP]
> 容器中不包含 Rime 内置输入方案的数据。使用时可以根据需要映射到 `/usr/share/rime-data` 目录，
> 或者使用 [rime-ice](https://github.com/iDvel/rime-ice/releases/tag/nightly)
> 等不依赖 Rime 内置输入方案的方案。

## Configuration

所有可配置项及其默认值（以 json 为例）:

```jsonc
{
  "initializationOptions": {
    "enabled": true, // 是否启用
    "shared_data_dir": "/usr/share/rime-data", // 指定 rime 共享文件夹
    "user_data_dir": "~/.local/share/rime-ls", // 指定 rime 用户文件夹，最好别与其他 rime 前端共用
    "log_dir": "~/.local/share/rime-ls", // 指定 rime 日志文件夹
    "max_candidates": 9, // [v0.2.0 后不再有用] 与 rime 的候选数量配置最好保持一致
    "trigger_characters": [], // 为空表示全局开启，否则列表内字符后面的内容才会触发补全
    "schema_trigger_character": "&", // [since v0.2.0] 当输入此字符串时请求补全会触发 “方案选单”
    "paging_characters": [",", ".", "-", "="], // [since v0.2.4] 输入这些符号会强制触发一次补全，可用于翻页 见 issue #13
    "max_tokens": 0, // [since v0.2.0] 大于 0 表示会在删除到这个字符个数的时候，重建所有候选词，而不使用删除字符操作，见 pr #7
    "always_incomplete": false, // [since v0.2.0] true 强制补全永远刷新整个列表，而不是使用过滤，见 pr #7
    "preselect_first": false, // [since v0.2.3] 是否默认选择第一个候选项
    "long_filter_text": false, // [since v0.3.0] 使用更长的 filter_text，某些编辑器如 helix/zed 连续补全需要设置 true
    "show_filter_text_in_label": false, // [since v0.3.0] 在候选项的 label 中显示 filter_text，某些编辑器如 zed 需要设置 true
    "show_order_in_label": true // [since v0.4.0] 在候选项的 label 中显示数字
  }
}
```

## Build

### Ubuntu

1. 配置 Rust 环境, 安装额外依赖 `clang` 和 `librime-dev`
2. 编译
   - `librime >= 1.6` => `cargo build --release`
   - `librime < 1.6` => `cargo build --release --features=no_log_dir`

### ArchLinux

可以通过我在 AUR 上打的包 [rime-ls](https://aur.archlinux.org/packages/rime-ls) 编译安装

手动从源码编译与上面类似，其他 linux 发行版也差不多

### NixOS

本项目已收录至 [nixpkgs](https://github.com/NixOS/nixpkgs/blob/master/pkgs/by-name/ri/rime-ls/package.nix)

编辑器配置 `shared_data_dir` 时，需要根据 `rime-data` 的管理方式进行对应设置：

1. 如不使用 Nix 安装 `rime-data`（eg. 使用 Git 仓库管理）：

   同其他发行版，`shared_data_dir` 设置为 `<rime-data-root-dir>/share/rime-data` 即可（注意 NixOS 不提供 `/usr/share` 路径，请将 `rime-data` 保存至其他可写入路径下）。

2. 如使用 Nix 安装 `rime-data`:

   可以在 NixOS configuration 中设置 `RIME_DATA_DIR` 环境变量：
   
   ```Nix
   {
     environment.variables."RIME_DATA_DIR" = "${pkgs.rime-data}/share/rime-data";
   }
   ```
   
   或在 NixOS configuration 中创建固定路径到 `rime-data` 的软链接：

   ```Nix
   { pkgs, ... }:
   {
     # symlink to `/var/lib/rime-ls/rime-data`
     systemd.tmpfiles.rules = [
       # ...
       "L+ /var/lib/rime-ls/rime-data - - - - ${pkgs.rime-data}/share/rime-data"
     ];
     # OR symlink to `/etc/rime-ls/rime-data`
     environment.etc."rime-ls/rime-data" = {
       enable = true;
       mode = "symlink";
       source = "${pkgs.rime-data}/share/rime-data";
     };
   }
   ```

   之后在编辑器配置中，设置 `shared_data_dir` 为 `${RIME_DATA_DIR}` 或以上设置的软链接路径。
   
   如需使用自定义的 `rime-data` 配置，可以通过 Nix expression 打包并安装，替换上述 `${pkgs.rime-data}` 为对应包名即可。

   TIP: Nixpkgs/NUR 现提供了 `rime-ice` `rime-zhwiki` 等第三方配置，但暂无自动合并配置的解决方案，建议有需求的用户自行维护。

### Windows

1. 配置 Rust 环境, 安装额外依赖 `clang` 和 `librime`
2. 通过 librime 的 [Release](https://github.com/rime/librime/releases/) 下载 windows 版本，例如 `rime-xxxx-Windows-msvc-x64.7z`，解压至某个目录
3. 设置环境变量以便编译时找到 librime 的相关文件，在 powershell 下可以：
   ```powershell
   $env:LIBRIME_LIB_DIR="C:\解压出来的目录\dist\lib" # 找库文件
   $env:LIBRIME_INCLUDE_DIR="C:\解压出来的目录\dist\include" # 找头文件
   $env:LIB="C:\解压出来的目录\dist\lib" # 链接时找 lib 文件
   ```
4. 编译 `cargo build --release`

### macOS

1. 安装 [鼠须管输入法](https://github.com/rime/squirrel)
2. 安装 [librime](https://github.com/rime/librime)
   从此项目中的 [Release](https://github.com/rime/librime/releases/)
   下载最新的 MacOS 相关的压缩包，解压缩后将 include 文件夹以及 lib 文件夹下的内容分别复制到 `/usr/local/include` 和 `/usr/local/lib`;
3. 设置环境变量以便编译时找到 librime 的相关文件（参考[相关 issue](https://github.com/wlh320/rime-ls/issues/24)）

   ```bash
   # 用于编译
   export LIBRIME_LIB_DIR=/usr/local/lib
   export LIBRIME_INCLUDE_DIR=/usr/local/include
   # 用于运行
   export DYLD_LIBRARY_PATH=/usr/local/lib  # 最好放在~/.zshrc中，记得修改~/.zshrc 后，source ~/.zshrc
   ```

   lib 库文件后面调用的时候应该还需要到 MacOS 的“安全性与隐私”中进行授权，所以建议调用解压后 bin 文件下的可执行文件来提前触发授权。

4. 编译，参考 [Ubuntu](#Ubuntu)
5. 配置，`shared_data_dir` 应该设置为 `/Library/Input Methods/Squirrel.app/Contents/SharedSupport`

## 个人词库同步

> [!WARNING] > **不推荐**与系统中的已有 rime 输入法共用一个用户目录，避免数据库上锁无法使用。
> 使用前备份自己的数据, 避免因作者对 rime API 理解不到位可能造成的数据损失。

可以通过 rime 的 sync 功能将系统中已安装的 rime 输入法的词库同步过来。

如需同步词库, 可以在 rime-ls 自己的用户目录下的 `installation.yaml`
添加`sync_dir: "/<existing user data dir>/sync"` 配置项。

通过 LSP 的 `workspace/executeCommand` 手動調用 `rime-ls.sync_user_data` 的命令进行同步 (since v0.1.2)

## FAQ

1. 为什么默认补全繁体中文？怎么修改候选个数？

   答：这部分由 Rime 负责，参考 [Rime 的帮助文档](https://rime.im/docs/)。推荐对 Rime 有初步了解后再使用本软件。

2. 某个编辑器不能用/不好用

   答：由于 LSP 客户端在补全上的实现都很不一致（影响用户体验的差异主要有：怎样寻找单词边界、怎样过滤候选项），
   不能保证每个编辑器默认就有很好的使用体验，可能需要配置或者插件支持。

## Contributions

欢迎为本项目贡献力量，你可以：

- 发现并修改代码里的 bug
- 为某个编辑器实现相关插件
- 提供某个编辑器的更好用的用户配置
- 提供专为 rime-ls 设计的 rime 输入方案
- 帮助实现更好的 CI/CD

## TODO

- [x] 實現更多 librime 的功能
  - [x] 按数字键选择候选项
  - [x] 与 rime API 同步翻页
  - [x] 与 rime API 同步提交
  - [x] 输入标点符号
  - [x] 输入方案选择
- [x] 实现更友好的触发条件
  - [x] ~~计划实现光标前面有汉字就开启, 但发现不同编辑器行为不一致, 搁置~~ 多加了一次正则匹配解决了, 不知道性能如何
- [ ] 读 LSP 文档, 继续提升补全的使用体验
- [x] 參數可配置 (用户目录, 触发条件, 候选数量)
- [x] ~~實現一個更好的 librime 的 rust wrapper 庫~~ 目前功能比较固定，没必要了
- [ ] 測試其他 LSP clients
- [x] 测试不同操作系统和 librime 版本
- [ ] 测试与不同 rime 配置的兼容性
- [x] 配置 GitHub CI，编译各个平台的 Release
- [ ] 各种编辑器插件 (help wanted)

## Known Issues

- [ ] #31 补全时吞符号，暂时可以通过在输入方案设置所有符号直接上屏进行规避
- [ ] #20 用数字选词以后还需要一次确认，这是 LSP 协议本身带来的限制，需要编辑器插件配合提升体验
- [ ] 调用 sync_user_data 后偶尔内存占用大，这是 rime 已知问题 https://github.com/rime/librime/issues/440

## Credits

受到以下项目启发

- [ds-pinyin-lsp](https://github.com/iamcco/ds-pinyin-lsp)
- [cmp-rime](https://github.com/Ninlives/cmp-rime)
- [librime-sys](https://github.com/lotem/librime-sys)
- [tower-lsp-boilerplate](https://github.com/IWANABETHATGUY/tower-lsp-boilerplate)
