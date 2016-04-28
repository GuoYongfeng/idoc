
## Nodejs环境配置（以Windows为例）

### 下载安装
访问[nodejs](https://nodejs.org/)下载并安装，一路next即可。

### 配置环境变量

```
PATH = C:\Program Files\nodejs;
```
### 配置npm
- 在nodejs的安装目录下新增node_global和node_cache目录，并使用npm配置：

```
$ npm config set prefix "C:\Program Files\nodejs\node_global"
$ npm config set cache "C:\Program Files\nodejs\node_cache"
```

- 再次配置环境变量

```
PATH = C:\Program Files\nodejs\node_global;
NODE_PATH = C:\Program Files\nodejs\node_global\node_modules;
```

这个时候，执行`npm install xxx -g`的时候下载的包都会放到`C:\Program Files\nodejs\node_global`下面。

- 修改配置镜像地址及显示下载速度

```
$ npm config set registry http://registry.npm.taobao.org
$ npm config set loglevel=http
```
