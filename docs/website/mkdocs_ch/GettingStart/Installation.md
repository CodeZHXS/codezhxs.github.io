> [英文页面](https://squidfunk.github.io/mkdocs-material/getting-started/#with-pip-9x)
> 
> - 文章内缺少引用跳转。
> 
> - 代码块没有同框显示两种代码。
> 
> - 代码块没有额外框。
> 
> - 没有yt和时钟图标。
> 
> - @人似乎不能高亮。
> 
> - 页面底部没有更新时间。
>
> - 部分英文没有翻译。

# 开始

Material for MkDocs 是基于 [MkDocs] 的一个强大的文档框架，它是一个用于项目文档的静态网站生成器。
[^1] 如果您熟悉 python，可以使用 [`pip`][pip] 安装 Material for MkDocs。 如果不熟悉 python，建议使用 docker 安装。



[^1]:
    In 2016, Material for MkDocs started out as a simple theme for MkDocs, but
    over the course of several years, it's now much more than that – with the
    many built-in plugins, settings, and countless customization abilities,
    Material for MkDocs is now one of the simplest and most powerful frameworks
    for creating documentation for your project.

[MkDocs]: https://www.mkdocs.org
[pip]: #with-pip
[docker]: #with-docker

## 安装

### 使用 pip 安装 { #with-pip data-toc-label="with pip" }

Material for MkDocs  已经作为一个 [Python package] 发布了，最好在
, [virtual environment] 下使用 `pip` 安装。 打开终端并使用以下命令安装Material for MkDocs：

=== "Latest"

    ``` sh
    pip install mkdocs-material
    ```

=== "9.x"

````
``` sh
pip install mkdocs-material=="9.*" # (1)!
```

1.  Material for MkDocs uses [semantic versioning][^2], which is why it's a
    good idea to limit upgrades to the current major version.

    This will make sure that you don't accidentally [upgrade to the next
    major version], which may include breaking changes that silently corrupt
    your site. Additionally, you can use `pip freeze` to create a lockfile,
    so builds are reproducible at all times:

    ```
    pip freeze > requirements.txt
    ```

    Now, the lockfile can be used for installation:

    ```
    pip install -r requirements.txt
    ```
````

[^2]:
    Note that improvements of existing features are sometimes released as
    patch releases, like for example improved rendering of content tabs, as
    they're not considered to be new features.

这将自动安装所有依赖的兼容版本：包括 [MkDocs]，[Markdown]， [Pygments] 和 [Python Markdown Extensions]。 Material for MkDocs 会提供这些依赖的最新版本，因此无需单独安装这些软件包。

---

:fontawesome-brands-youtube:{ style="color: #EE0F0F" }
__[How to set up Material for MkDocs]__ by @james-willett – :octicons-clock-24:
15m – 这个视频介绍了如何使用 Material for MkDocs 在 GitHub Pages 上创建和托管文档。

[How to set up Material for MkDocs]: https://www.youtube.com/watch?v=Q-YA_dA8C20

---

__提示__: 如果您没有使用Python的经验，建议阅读
[Using Python's pip to Manage Your Projects' Dependencies]，这篇文章介绍了Python的包管理机制，可以帮助您解决遇到的问题。

[Python package]: https://pypi.org/project/mkdocs-material/
[virtual environment]: https://realpython.com/what-is-pip/#using-pip-in-a-python-virtual-environment
[semantic versioning]: https://semver.org/
[upgrade to the next major version]: upgrade.md
[Markdown]: https://python-markdown.github.io/
[Pygments]: https://pygments.org/
[Python Markdown Extensions]: https://facelessuser.github.io/pymdown-extensions/
[Using Python's pip to Manage Your Projects' Dependencies]: https://realpython.com/what-is-pip/

### 使用 docker 安装

官方的 [Docker image] 已预先安装了所有依赖项。打开终端并使用以下命令拉取该镜像：

=== "Latest"

    ```
    docker pull squidfunk/mkdocs-material
    ```

=== "9.x"

    ```
    docker pull squidfunk/mkdocs-material:9
    ```

安装完成后， `mkdocs` 可执行文件会被添加到环境变量中，通常我们使用的是 `mkdocs serve` 命令。如果您对Docker不熟悉，不用担心，在接下来的部分中我们会详细介绍。

以下插件已加载到Docker镜像中：

- [mkdocs-minify-plugin]
- [mkdocs-redirects]

  [Docker image]: https://hub.docker.com/r/squidfunk/mkdocs-material/
  [mkdocs-minify-plugin]: https://github.com/byrnereese/mkdocs-minify-plugin
  [mkdocs-redirects]: https://github.com/datarobot/mkdocs-redirects

??? question "How to add plugins to the Docker image?"

    Material for MkDocs only bundles selected plugins in order to keep the size
    of the official image small. If the plugin you want to use is not included,
    you can add them easily:
    
    === "Material for MkDocs"
    
        Create a `Dockerfile` and extend the official image:
    
        ``` Dockerfile title="Dockerfile"
        FROM squidfunk/mkdocs-material
        RUN pip install mkdocs-macros-plugin
        RUN pip install mkdocs-glightbox
        ```
    
    === "Insiders"
    
        Clone or fork the Insiders repository, and create a file called
        `user-requirements.txt` in the root of the repository. Then, add the
        plugins that should be installed to the file, e.g.:
    
        ``` txt title="user-requirements.txt"
        mkdocs-macros-plugin
        mkdocs-glightbox
        ```
    
    Next, build the image with the following command:
    
    ```
    docker build -t squidfunk/mkdocs-material .
    ```
    
    The new image will have additional packages installed and can be used
    exactly like the official image.

### 使用 git 安装

可以直接从 [GitHub] 仓库中 `clone` 最新的版本：

```
git clone https://github.com/squidfunk/mkdocs-material.git
```

接下来，使用以下命令安装依赖项：

```
pip install -e mkdocs-material
```

[GitHub]: https://github.com/squidfunk/mkdocs-material