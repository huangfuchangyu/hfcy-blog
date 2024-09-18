## `Monorepo`



#### 演进：

* 一阶段：单仓库应用

一个Git维护项目代码，随着迭代业务复杂度提升，代码越来越复杂，构建效率降低，导致单体巨石应用，这种管理方式称为`Monolith`

* 二阶段：多仓库多模块应用

将项目拆解成多个业务模块，并在多个Git仓库管理，模块解耦，降低巨石应用的复杂度，每个Git仓库模块单独开发管理，这种管理方式随着模块仓库数量提升，跨仓库代码分享困难，模块依赖管理复杂，工程共享配置复杂,这种管理方式称为`MultiRepo`

* 三阶段：单仓库多模块应用

将多个项目集成到一个仓库下，共享工程配置，方便共享模块代码，这种管理方式称为`MonoRepo`





#### 差异对比

| 场景       | MultiRepo                                                    | MonoRepo                                                     |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 代码可见性 | :grinning: 代码隔离，开发者只需要关注自己的部分<br /> :joy_cat:包管理按照个字owner划分，出现问题需要到依赖包中进行判断解决 | :grinning: 一个仓库能看到所有代码，更利于团队协作<br /> :joy_cat: 增加了非owner修改代码的风险 |
| 依赖管理   | :joy_cat: 多个仓库都有自己的 node_modules ，存在依赖重复和版本问题 | :grinning: 多个项目代码在同一仓库，相同版本依赖提升到顶      |
| 代码权限   | :grinning: 各个项目单独仓库，不会出现相互影响与误改情况      | :joy_cat: 多个项目代码在同一仓库，没有权限控制               |
| 开发迭代   | :grinning: 仓库体积小，模块划分清晰，可维护性强<br /> :joy_cat: 多仓库依赖时，需要手动npm link 操作复杂<br /> :joy_cat: 依赖管理复杂，多个依赖可能出现不同版本与重复安装，npm link 会出现不同项目依赖冲突 | :grinning: 多个项目同一个仓库, 可看到相关项目全貌，编码方便<br /> :grinning: 代码复用高<br /> :joy_cat: 多项目在同一仓库 ，体积大<br /> :grinning: 调试方便，可以使用工具自动npm link ，直接使用最新版本依赖，简化操作 |
| 工程配置   | :joy_cat: 各项目构建 打包 代码校验都各自维护，配置差异会导致构建差异 | :grinning: 各项目在同一仓库，工程配置相同                    |
| 构建部署   | :joy_cat: 各项目存在依赖，部署需要依次操作，效率低           | :grinning: 构建性Monorepo工具可以配置依赖项目的构建优先级，可以实现依次命令完成所有的部署 |





#### 踩坑



###### 幽灵依赖

* 问题描述：

`npm / yarn` 安装依赖时，会存在依赖提升，某个项目使用的依赖，没有在其 `package.json` 中声明，也可以直接使用，这种现象称为 '幽灵依赖' , 随着项目的迭代， 这个依赖不被使用时，不再被安装， 使用幽灵依赖的项目， 会因为无法找到依赖而报错

* 解决方案：

基于 `npm / yarn ` 的 Monorepo 方案， 依然会存在 ’幽灵依赖‘ 的问题， 我们可以通过 `pnpm` 彻底解决这个问题



###### 依赖安装耗时长

* 问题描述： 

Monorepo 中每个项目都有自己的 `package.json` 依赖列表, 随着 MonoRepo 中依赖数量的增多，每次安装依赖的耗时会很长

* 解决方案：

相同版本依赖提升到 `Monorepo` 根目录下，减少冗余依赖， 使用 pnpm按需安装及依赖缓存



###### 构建打包耗时长

* 问题描述: 

多个项目构建任务存在依赖时，通常是串行构建或者全量构建，导致构建时间长

* 解决方案: 

增量构建，而非全量构建; 也可以将串行构建，优化成并行构建





#### Lerna

###### Lerna 是什么

* Lerna是Babel 为实现 Monorepo 开发的工具，适用于管理依赖关系和发布
* Lerna 优化了多包工作流，解决了多包依赖，手动发版和版本维护问题
* Lerna 不提供构建，屙屎等任务，工程能力较弱，项目中通常需要基于它进行顶层能力的封装



###### Lerna 主要做三件事

* 为单个包或多个包运行命令（lerna run）
* 管理依赖项（lerna bootstrap）
* 发布依赖包，版本管理，生成变更日志（lerna publish）



###### lerna 能解决什么问题

* 代码共享，调试便捷： 一个依赖包更新，其他依赖此包的项目无需安装最新版本， lerna 自动 link
* 安装依赖，减少冗余: 多个包都是用相同版本的依赖包时， lerna 优先将依赖安装在根目录
* 规范版本管理: lerna 通过git 监测代码变动，自动发版， 更新版本号
* 自动生成发版日志: 使用插件，根据 Git commit 记录， 自动生成 changeLog



###### 自动检测发布， 判断逻辑

1. 校验本地是否存在没被 `commit` 的内容
2. 判断当前分支是否正常
3. 判断当前分支是否在 `remote` 存在
4. 判断当前分支是否存在 `lerna.json` 允许的 `allowBranch` 设置中
5. 判断当前分支提交是否落后于 remote





###### Lerna 工作模式

* 固定模式 （Fixed）
* 独立模式（Independent）



**固定模式 Locked mode**

> Lerna 把多个软件包当做一个整体工程，每次发布对所有的软件包版本统一升级（版本一致）， 无论是否有修改

> 项目初始化时， `lerna init` 默认是 Locked mode

```json
{
  "version": "0.0.0"
}
```





**独立模式 Independent mode **

> Lerna 单独管理每个软件包的版本号，每次执行发布指令，Git 检查文件变动，只发版本有调整的软件包

> 初始化项目时， `lerna init --independent`

``` json
{
  "version": "independent"
}
```





###### Lerna 常用指令

1. 初始化

``` shell
lerna init
```

执行成功后，目录下将会生成这样的目录结构

``` json
- packages(目录)
- lerna.json(配置文件)
- package.json(工程描述文件)
```

``` json
{
  "version": "0.0.0",
  "useWorkspaces": true,
  "packages": [
    "packages/*"
  ]
}
```

需要在项目根目录下的`package.json` 中设置 `"private": true`

``` json
{
  "name": "xxxx",
  "version": "0.0.1",
  "description": "",
  "main": "index.js",
  "private": true,
  "scripts": {
    "test": "echo "Error: no test specified" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "lerna": "^6.4.1"
  },
  "workspaces": [
    "packages/*"
  ]
}
```



2. 创建 package:  `lerna create` 

``` shell
// lerna create <name> [locatioin]

lerna create package2 packages/pwd1
```

在 packages/pwd1 目录下，生成 package2 依赖包

执行 `lerna init `后 ，默认的 lerna workspace 是 `packages/*` 需要手动修改 `package.json` 中的 `workspaces` 再执行指令生成特定目录下的 package 



3. package 添加依赖

安装的依赖， 如果是本地包， lerna 会自动 npm link 到本地包

``` shell
# 安装依赖 默认为 dependencies
lerna add my-model
lerna add my-model --dev  # devDependencies
lerna add my-model --peer # peerDependencies
lerna add my-model[@version] # 安装指定版本
lerna add my-model --scope=module-2    # 给指定包安装依赖
lerna add my-model packages/prefix-*  # 前缀为 xxx 的包安装依赖

```



4. 给所有的package 安装依赖 `bootstrap`

``` shell
# 在根目录执行，安装所有依赖
lerna bootstrap
# 将自动为每个包执行 npm install 和 npm link 操作
```

关于冗余依赖

* Npm 场景下 `lerna bootstrap` 会安装冗余依赖，对于多个package共用的依赖，每个目录都安装
* Yarn 会安装hosit依赖包，相同版本的依赖，安装在根目录，不需要关心

npm 场景下冗余依赖解决方案

* 方案一：  `lerna bootstrap --hoist`
* 方案二：  `lerna.json/command.bootstrap.hoist = true`



5. package shell指令： exec

``` bash
# 删除所有包内lib目录
lerna exec -- rm - rf lib

# 删除 xx 包下的依赖
lerna exec --scope=xx -- yarn remove yyy
```



6. 执行script指令： run 

``` shell
# 所有依赖执行 package.json 文件 script 中的指令
lerna run xxx

# 指定包执行package.json 中的 script 指令
lerna run --scope-my-component xxx
```





7. 清除所有 package 下的依赖：  clean

   ``` shell
   lerna clean
   ```

   



8. 发布软件包，自动监测: publish

``` shell
lerna publish
```

> 1. 运行 lerna updated 来决定哪一个包需要被 publish
> 2. 如有必要 将会更新 lerna.json 中的 version
> 3. 将所有更新过的包中的 package.json 的 version 字段更新
> 4. 将所有更新过的包中的依赖更新
> 5. 为新版本创建一个 git commit 或tag
> 6. 将包 publish 到 npm上





9. 查看自上次发布的变更

``` shell
# 查看上次 release tag 以来修改的包的差异
lerna diff

# 查看自上次 release tag 以来有修改的包名
lerna changed
```



10. 导入已有包

``` shell
lerna import [npm 包本地路径]
```



11. 列出所有包

``` shell
lerna list
```



######  yarn / npm + workspace

Yarn 1.x 及以上版本 ，新增 workspace 能力， 不借助 lerna , 也可以提供原生的 monorepo 支持, 需要在 根目录下 package.json 中 声明 workspace

``` json
{
  private: true, // 必须是私有项目
  workspaces: ["project1", "project2/*"]
}
```



**yarn workspace 对比 lerna**

* yarn workspace 更突出对依赖的管理， 依赖提升到根目录下 node_modules 下，安装更快， 体积更小
* lerna 更突出工作流方面, 使用 lerna 命令来优化多个包管理，如 依赖发包 ， 版本管理， 批量执行脚本





**能力分工：** 依赖管理由 yarn workspace 处理， lerna 承担依赖发布能力



**操作步骤：**

1. 配置lerna 使用 yarn 管理依赖： lerna.json 中配置 `npmClient: yarn`
2. 配置 lerna 启用 yarn workspaces: 
   * 配置`lerna.json/useWorkspace = true`
   * 配置根目录 `package.json/workspaces = ["packages/*"]`, 此时  `lerna.json` 中配置的 packages 将不会生效
   * 配置 `package.json/private = true`

> 上述三点配置项需同时配置，否则 lerna 报错
>
> 此时执行 lerna bootstrap 相当于执行 yarn install 等同于 lerna bootstrap --npm-client yarn --use-workspaces
>
> 由于 yarn 会自动 hosit 依赖包 无需再 lerna bootstrap 时增加参数 --hoist 

不需要发包的项目 配置 `package.json/private = true`







#### Lerna + pnpm + workspace

pnpm是 npm / yarn 衍生而来 ，解决了 npm / yarn 内部潜在风险 ， 极大提升了安装速度,  pnpm 内部使用 基于内容寻址的文件系统来管理磁盘上的依赖， 减少依赖的安装,  `node_modules/.pnpm`为虚拟存储目录, 该目录通过 `<package-name>@<version>` 来实现相同模块不同版本之间的隔离和复用，由于只会根据项目中的依赖生成， 所以并不存在提升。

**CAS 内容寻址存储**  是一种存储信息的方式， 根据内容而不是位置进行检索信息的存储方式

**virtual store 虚拟存储**  指向存储的链接目录，所有直接和间接依赖项都链接到此目录中， 项目当中的 .pnpm 目录 



**Pnpm 相比于 npm  /  yarn 优势如下**

1. 依赖安装速度快： 缓存中有的依赖，直接硬链接到项目的 node_module 中， 减少了 大量的 IO操作
2. 磁盘利用率高: 软 、硬 链接方式， 同一版本的依赖公用一个磁盘空间, 不同版本的依赖 ， 只额外存储 diff内容
3. 无幽灵依赖：  node_modules 目录结构 与 package.json 依赖列表一致 





#### 选型建议

对于轻量级 `Monorepo` 项目，初期可以选择 `Lerna + pnpm + workspace + lerna-changelog` 解决了依赖管理，发版等问题， 随着项目的迭代与依赖关系的复杂，可以平滑的接入 `NX` 来提升构建打包效率







#### 补充：  NX

Nx 是 Nrwl 团队开发的 ， 同时在维护 Lerna ， Nx 可以在 Lerna 5.1 及以上版本可用

构建加速思路： 

* 缓存: 通过缓存 及 远程缓存 减少构建时间
* 增量构建： 最小范围构建，非全量构建
* 并行构建： Nx 自动分析项目的关联关系， 对这些任务进行排序以最大化并行
* 分布式构建： 结合 Nx Cloud 构建任务自动分布在 CI 代理中



Lerna 擅长管理依赖关系和发布, 但扩展基于 Lerna 的 Monorepos 很快就会变得痛苦 ， 因为 Lerna 很慢,  这就是 NX 闪光点