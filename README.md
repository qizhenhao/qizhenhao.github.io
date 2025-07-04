# 小齐的博客 - 源文件管理

[![GitHub license](https://img.shields.io/github/license/qizhenhao/qizhenhao.github.io?style=flat-square)](./LICENSE)
[![Built with Hexo](https://img.shields.io/badge/Built%20with-Hexo-blue?style=flat-square&logo=hexo)](https://hexo.io)
[![Hosted on GitHub Pages](https://img.shields.io/badge/Hosted%20on-GitHub%20Pages-green?style=flat-square&logo=github)](https://pages.github.com/)
[![GitHub last commit](https://img.shields.io/github/last-commit/qizhenhao/qizhenhao.github.io/source?style=flat-square)](https://github.com/qizhenhao/qizhenhao.github.io/tree/source)

本项目是"小齐的博客"的源文件仓库，使用 [Hexo](https://hexo.io/) 框架构建。`source` 分支用于存放和管理所有的 Markdown 文章、主题配置及项目文件。

通过遵循以下步骤，您可以在任何一台新电脑上快速恢复博客的写作环境。

---

## 环境搭建 (Environment Setup)

### 第一步：安装先决条件

在开始之前，请确保您的电脑上已经安装了以下软件：

1.  **[Git](https://git-scm.com/)**: 用于版本控制和代码同步。
2.  **[Node.js](https://nodejs.org/)**: 建议安装 **LTS (长期支持)** 版本，Hexo 运行在此环境之上。

### 第二步：克隆本项目

打开终端（或 Git Bash），使用以下命令将本仓库的 `source` 分支克隆到本地：

```bash
git clone -b source https://github.com/qizhenhao/qizhenhao.github.io.git 博客文件夹名称
```

> **提示**：将 `博客文件夹名称` 替换为您希望在本地创建的文件夹名字，例如 `my-blog`。

然后进入该文件夹：

```bash
cd 博客文件夹名称
```

### 第三步：安装依赖

在项目文件夹内，运行以下命令来安装所有必需的 Hexo 插件和依赖库：

```bash
npm install
```

> **注意**：如果安装速度过慢或失败，可以尝试切换 npm 镜像源，例如淘宝镜像：
> `npm install --registry=https://registry.npmmirror.com`

### 

### 第四步：拉去子库主题

在Theme是在theme文件夹中的子库。

```shell
git submodule init
git submodule update
# 或者
git submodule update --init
```



至此，您的本地博客环境已完全搭建好！

---

## 日常使用 (Daily Use)

### 本地预览

在进行任何修改或撰写新文章时，可以随时运行以下命令来开启本地服务器，在浏览器中实时预览效果：

```bash
hexo server
# 或者简写为
hexo s
```

启动成功后，在浏览器中访问 `http://localhost:4000` 即可看到您的博客。

### 新建文章

使用以下命令来创建一篇新的博文：

```bash
hexo new "您的文章标题"
```

Hexo 会在 `source/_posts` 目录下生成一个名为 `您的文章标题.md` 的文件，您只需用 Markdown 编辑器打开它并开始写作即可。

### 发布流程

当您完成修改或写完新文章后，需要通过以下两步来完成发布：

1.  **第一步：保存源文件修改**
    将您的所有改动（包括新文章、配置修改等）推送到 `source` 分支，作为云端备份。

    ```bash
    # 添加所有改动到暂存区
    git add .
    
    # 创建一个提交，并写上本次改动的说明
    git commit -m "这里写下您的改动说明，例如：完成《xxx》文章"
    
    # 推送到 GitHub
    git push origin source
    ```

2.  **第二步：部署到网站**
    使用 Hexo 命令将网站发布到 `master` 分支，让所有人都能访问到最新的内容。

    ```bash
    hexo clean && hexo g -d
    # 或者分步执行
    # hexo clean  # 清理缓存
    # hexo generate # 生成静态文件
    # hexo deploy   # 部署
    ```

遵循以上流程，您就可以轻松地在多台设备间同步和管理您的博客了。 