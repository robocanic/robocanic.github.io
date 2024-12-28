---
title: Python工程化不完全指南
date: 2024-7-06 21:37:12
tags: Python, 日常开发
categories: 日常开发
---

Python 作为一个弱类型语言，似乎从诞生之初就被很多人觉得这是一个很随便的语言，构建一个可靠的 Python 项目是一个很难的事情。事实也确实如此，随着 AI 的爆发，入坑 Python 的人越来越多，但对 Python 的诟病却只增不减。不过，Python 工程化还是在诟病中曲折前进的，到今天为止，尽管还存在着一些混乱，Python 工程化的标准趋于基本完善。本篇将从“过来人”的视角，由浅入深地讲述 python 工程化的方方面面，提供一个最佳实践供你参考。

# 项目目录结构

Python 作为脚本语言，一个.py 文件能放在任何地方运行，但是**一个清晰的目录结构不仅易于排查问题，更方便别人读懂我们的代码，更加优雅**。Python 由于其应用广泛，不同类型的 Python 项目也有着不同的目录结构规约。这里我选取普通应用工程和数据/AI 工程这两种最为广泛的类型进行说明。

## 普通应用工程

假定我们的项目名为`robocanic-project`，普通应用工程最常用的目录结构风格如下所示：

```Plain Text
 robocanic-project/
 │
 ├── robocanic_project/      # 主包，我们的核心代码都在这里面
 │   ├── __init__.py         # 包初始化文件
 │   ├── module1.py          # 模块1
 │   ├── module2.py          # 模块2
 │   └── main.py             # 主程序入口
 │
 ├── tests/                  # 测试目录
 │   ├── __init__.py
 │   ├── test_module1.py
 │   └── test_module2.py
 │
 ├── requirements.txt        # 依赖文件
 ├── setup.py                # 安装脚本
 ├── README.md               # 项目说明
 └── .gitignore              # Git忽略文件
```

那上面这种还比较宽泛，如果以前后端分离的后端 Web 应用这种比较常见的项目为例，目录结构可以是下面这样的：

```Plain Text
robocanic-project/
├── robocanic_project/                     # 主应用程序目录
│   ├── __init__.py          # 将该目录标识为包，通常初始化 FastAPI 实例
│   ├── main.py              # 主文件，包含应用实例和路由
│   ├── api/                 # API 路由目录
│   │   ├── __init__.py
│   │   ├── v1/              # API 的 v1 版本（可选分版本目录）
│   │   │   ├── __init__.py
│   │   │   ├── routes/   # 具体的 API 端点
│   │   │   │   ├── __init__.py
│   │   │   │   ├── user.py  # 用户相关的 API 路由
│   │   │   │   └── item.py  # 其他模块（如商品）的 API 路由
│   │   │   └── ...          # 更多 API 路由
│   ├── core/                # 核心配置模块
│   │   ├── __init__.py
│   │   ├── config.py        # 配置文件，包含数据库连接、第三方服务配置等
│   │   ├── security.py      # 安全相关的设置，例如 OAuth2、JWT
│   ├── models/              # 数据库模型
│   │   ├── __init__.py
│   │   └── user.py          # 用户模型
│   │   └── item.py          # 其他数据模型
│   ├── crud/                # 增删改查操作的封装
│   │   ├── __init__.py
│   │   ├── user.py          # 用户的 CRUD 操作
│   │   └── item.py          # 其他数据的 CRUD 操作
│   ├── db/                  # 数据库初始化及连接
│   │   ├── __init__.py
│   │   ├── base.py          # 基础的数据库模型
│   │   └── session.py       # 数据库会话（SQLAlchemy 会话）
│   ├── tests/               # 测试目录
│   │   ├── __init__.py
│   │   └── test_user.py     # 用户相关的测试文件
│   │   └── test_item.py     # 其他模块的测试文件
│   ├── utils/               # 工具函数目录
│   │   ├── __init__.py
│   │   └── hashing.py       # 用于密码加密/解密等实用工具
    ├── alembic/             # 数据库迁移工具 Alembic 相关文件（如果使用）
│   │   ├── versions/        # 每个迁移文件都保存在这个目录中
│   │   ├── env.py           # Alembic 环境配置文件
│   │   ├── README           # Alembic 说明
│   │   └── script.py.mako   # Alembic 脚本模板
│   └── ...
├── .env                     # 环境变量文件（敏感信息，不应提交到版本控制）
├── .gitignore               # Git 忽略文件列表
├── requirements.txt         # 依赖包列表（如果使用 pip）
├── setup.py                 # 安装脚本（如果使用setuptools）
├── pyproject.toml           # 项目元数据及依赖管理（如果使用 poetry）
├── README.md                # 项目说明文件
└── Dockerfile               # Docker 配置文件（如果使用容器化）
```

一个以 FastAPI 为基础 Web 框架的项目的目录结构如上所示。其中，除了`api`，`models`，`crud`，`utils`这类见名知意的目录，还有一个不太常见的`alembic`这个目录，alembic 其实是数据库迁移（**Database Migration**）的工具。而数据库迁移就是指在我们的应用开发过程中，当涉及到需要修改 ORM 对应的数据库结构（例如表、字段、索引等）时，引入的一套逐步演进数据库的状态，使其与应用代码保持一致的工具。这套工具一般是半自动或者全自动的，可以大大降低手动操作数据库带来的风险。

## 数据/AI 工程

而对于数据科学，模型训练，科学计算这类项目，则有着另一套完全不一样的目录风格：

```Plain Text
robocanic-project/
├── data/
├── notebooks/
├── scripts/
├── requirements.txt
└── README.md

```

以上是比较简略的目录结构，展开来看：

```Plain Text
robocanic-project/
├── data/
│   ├── raw/                  # 原始数据（未处理的数据）
│   ├── processed/            # 处理后的数据（清洗后的数据、特征工程后的数据）
│   └── external/             # 外部数据源（可能是API或第三方提供的数据）
│
├── notebooks/                # Jupyter Notebooks（探索性分析和初步模型）
│   ├── 01_data_exploration.ipynb  # 数据探索
│   ├── 02_data_cleaning.ipynb     # 数据清理
│   └── 03_model_training.ipynb    # 模型训练
│
├── scripts/                  # 数据处理和模型训练的Python脚本
│   ├── data_preprocessing.py  # 数据预处理脚本
│   ├── train_model.py         # 模型训练脚本
│   └── evaluate_model.py      # 模型评估脚本
│
├── models/                   # 保存的模型（训练好的模型、标记化文件等）
│   └── model_v1.pkl           # 保存的模型文件
│
├── reports/                  # 项目报告与结果展示
│   ├── figures/              # 图表和可视化结果
│   └── final_report.pdf      # 最终报告或结果总结
│
├── requirements.txt          # 项目所需的依赖库列表
├── environment.yml           # Conda 环境配置文件
├── README.md                 # 项目说明文件
└── .gitignore                # Git忽略文件

```

可以看出数据/AI 项目和普通的应用还是有很大区别的。**数据/AI 项目更多是从数据的流向来进行目录结构的划分，从数据采集 → 数据处理 → 模型训练（数据聚合）→ 模型评估（数据评估）→...... 而普通应用项目则更多是按功能模块来划分，比如 MVC 的划分模式，以 APP 为核心的划分模型等。**

总的来说，Python 工程的目录还是比较灵活的，我们在构建我们自己的项目目录时可以不拘泥于固定的目录结构，但也要遵循上面提到的一些基本规律，这样别人上手我们的项目时能够快速读懂我们的项目结构，进而读懂代码。

# 项目环境与依赖管理

接下来要谈到的部分个人认为是 python 中最混乱的部分——依赖管理。但是讲 Python 的依赖管理不能只讲依赖管理，还要连带着**虚拟环境(Virtual Environment)**一起讲。

不同于 Java，Go 这类依赖跟着项目走的做法，Python 项目的依赖是对于当前虚拟环境而言的。也就是说你在命令行中敲下`pip install fastapi`去下载 fastapi 这个包时，这个依赖是对当前环境有效的，如果项目 A 和项目 B 都使用了虚拟环境 E，那项目 A 和项目 B 都可以使用`fastapi`这个依赖，如果有多个项目都需要依赖`fastapi`，那是不是把他们放到一个虚拟化环境里就行？答案是否定的，因为一般来说，一个项目不单单只依赖一个`fastapi`，还有其他的依赖。比如两个项目 A 和 B 依赖了一个共同的包`pymysql`，但是依赖的版本不一样，这样就会发生依赖冲突。因此我们在创建 Python 项目时，一般都会创建一个对应的虚拟环境来防止依赖冲突。所以到这你就会发现虚拟环境其实还是项目层面的，一个项目就对应一个虚拟环境，虚拟环境的这个设计说实话很鸡肋，不仅相同版本的包不能得到复用，还变相地提高了依赖管理的复杂度。

吐槽完之后，我们来看看现阶段环境管理以及依赖管理的最佳实践。这个最佳实践包括不推荐的方式 venv/virtualenv+pip，以及两种推荐的方式：conda+pip，poetry。

## venv/virtualenv+pip（不推荐）

venv 和 virtualenv 都是用来创建 python 虚拟环境的工具，不同的地方在于 venv 是 python3.3 后自带的轻量级虚拟环境创建工具，只能用于 python3；而 virtualenv 则需要另外安装，可以用于 py2 以及 py3，还有一些 venv 没有的功能（个人觉得可有可无）。那对于 python3.3 以后的项目，现在都推荐自带的 venv 了。venv 使用起来也很简单：

1.  `python -m venv venv`命令的第一个 venv，是运行 venv 这个模块来创建一个虚拟环境。第二个 venv 是指虚拟环境的文件夹的名字，这个可以根据需要来修改。命令执行后，会在当前文件夹中，建立一个 venv 目录。 在这个 venv 目录中，会默认放置 python.exe、pip.exe 等与系统目录隔离的可执行文件和依赖。一般 venv 会放置在项目的根目录下。

2.  进入到`venv/Scipts`文件夹，执行`activate`，可以激活当前的虚拟环境; 执行`deactivate`，可以停用当前虚拟环境

3.  删除这个虚拟环境只需要删除掉项目根目录下的 venv 文件夹即可。

venv 没有管理虚拟环境的功能，这是因为虚拟环境的依赖都跟着项目走了（venv 文件夹在项目根目录下）。因此它足够的轻量。

pip 是一个轻量级包管理工具，安装 python 时都会自带 pip。pip 最常用的命令有用于下载依赖的`pip install deps`、`pip install -r requirements`和用于删除依赖的`pip uninstall deps`。很多人在一开始使用 pip 时还觉得简单好用，但在使用多了之后就深恶痛绝。pip 让人深恶痛绝的最主要的两个原因如下：

1. 在命令行执行完`pip install deps`后下载的依赖不会同步到 requirements.txt 中，同样`pip uninstall deps`删除的依赖也不会删除 requirements 中声明的依赖。

2. **`pip uninstall deps`只会删除`deps`这个直接依赖，而不会删除最初下载`deps`这个包引入的间接依赖**

其中，第一点还可以使用`pip freeze > requirements.txt`来完成手动同步，第二点则是吐槽最多的点。这两点经常会引起依赖冲突，生产环境与本地开发环境不一致的情况出现。所以在使用 venv+pip 组合时，得遵循一个开发范式，包括但不限于以下几点

1. 直接依赖需要写入 requirements.txt 中，使用`pip install -r requirements`来下载依赖，摈弃`pip install deps`，保证项目依赖相对清晰

2. 发版时必须执行`pip freeze > requirements.txt`来保证本地与云端环境一致

## conda+pip（推荐用于数据/AI 领域）

conda 不仅是一个包管理工具库，还是一个环境管理工具，广泛用于科学计算领域，conda 命令也比较简单易用，使用 conda 的基本步骤如下：

1. `conda create -n my-ns`创建一个隔离的虚拟环境

2. `conda activate my-ns`，激活虚拟环境

3. `conda install dps`下载依赖

4. `conda remve deps` 卸载依赖，conda 在卸载依赖时会智能地处理包的依赖关系，将间接依赖中无用的包也会一并卸载

5. `conda remove --name my-ns --all`删除虚拟环境

其中，conda 创建的虚拟环境会维护一个与环境名同名的文件目录（类似 venv），但这个文件目录会统一放在一个地方，不会放在项目根目录下，这也让多个使用 conda 的项目“有了共享虚拟环境的可能”。那 conda 既能管理环境，又能管理依赖，是不是就能替代 pip？很遗憾还是不能，主要原因有两点：

1. conda 主要用于数据科学领域，整个环境安装起来就比较重，环境初始时就自带了很多依赖，很多依赖是普通项目不需要的。此外，conda 下载依赖的速度也会慢一些。

2. conda 的包与 pip 安装的包不一样，conda 包是二进制格式的，除了数据科学领域，很多 python 库都会在 PyPI 上发布，但不会在 Anaconda repository 上发布

所以以上两点就限制了 conda 的应用领域，除了数据科学领域，conda 就只能作为一个环境管理的工具。那对于使用 conda 来管理环境的情况，pip 一般也是不可或缺的，使用时的注意点同上面的`venv+pip`

## poetry（推荐）

> 以下代码地址: [https://github.com/robocanic/poetry-fastapi-demo.git](https://github.com/robocanic/poetry-fastapi-demo.git)

上面两种环境与依赖管理的方式都有一个比较棘手的点无法解决，那就是 pip 卸载依赖时无法删除引入的间接依赖，这很可能会导致**依赖臃肿，依赖冲突**等问题。

poetry 是一个 python 依赖管理与环境管理的工具，它能够分析出哪些是直接依赖，哪些是间接依赖，在删除依赖时能够删掉无用的间接依赖，保持项目依赖的清爽。此外，poetry 还有指定模糊依赖，依赖快速升级等实用的功能，这让它在依赖管理方面直接甩了 pip 好几条街。至于环境管理，poetry 则是在 virtualenv 的基础上，增强了一些功能，如自动创建虚拟环境，集中/分离的环境管理等等。

poetry 的命令会相对多一些，但常用的也只有几个，下面从建立一个新项目的流程来熟悉一下如何使用 poetry：

#### 创建虚拟环境

在创建文件夹后，执行`poetry init`，会有一系列互动的 command 来引导你创建`pyproject.toml`:

```YAML
 PS D:\code\python-projects\poetry-fastapi-demo>poetry init
 ​
 This command will guide you through creating your pyproject.toml config.
 ​
 Package name [poetry-fastapi-demo]:  
 Version [0.1.0]:  
 Description []:  A web project template using poetry and fastapi.
 Author [robocanic <robocanic@example.com>, n to skip]:  
 License []:  Apache 2.0
 Compatible Python versions [^3.11]:  ~3.10
 ​
 Would you like to define your main dependencies interactively? (yes/no) [yes] no
 Would you like to define your development dependencies interactively? (yes/no) [yes] no
 Generated file
 ​
 [tool.poetry]
 name = "poetry-fastapi-demo"
 version = "0.1.0"
 description = "A web project template using poetry and fastapi."
 authors = ["robocanic <robocanic@example.com>"]
 license = "Apache 2.0"
 readme = "README.md"
 ​
 [tool.poetry.dependencies]
 python = "~3.10"
 ​
 ​
 [build-system]
 requires = ["poetry-core"]
 build-backend = "poetry.core.masonry.api"
 ​
 ​
 Do you confirm generation? (yes/no) [yes]
```

这里你在命令行输入的在后面都是可以通过修改`pyproject.toml`来同步的，因此一般我们都会快速地跳过这个阶段。生成的`pyproject.toml`如下所示：

```YAML
 [tool.poetry] #项目信息
 name = "poetry-fastapi-demo"
 version = "0.1.0"
 description = "A web project template using poetry and fastapi."
 authors = ["robocanic <robocanic@example.com>"]
 license = "Apache 2.0"
 readme = "README.md"
 ​
 [tool.poetry.dependencies]  #主要依赖
 python = "~3.10"
 ​
 ​
 [build-system]  # 构建工具
 requires = ["poetry-core"]
 build-backend = "poetry.core.masonry.api"
 ​
```

在创建`pyproject.toml`后，在项目根目录下执行`poetry env use python`，会创建一个该项目专有的虚拟环境：

```YAML
 PS D:\code\python-projects\poetry-fastapi-demo> poetry config virtualenvs.in-project true
 PS D:\code\python-projects\poetry-fastapi-demo> poetry env use D:\env\python\python.exe
 Creating virtualenv poetry-fastapi-demo in D:\code\python-projects\poetry-fastapi-demo\.venv
 Using virtualenv: D:\code\python-projects\poetry-fastapi-demo\.venv
```

这里我指定了 poetry 虚拟环境的路径放置在项目根目录的.venv 目录下，我觉得这样的项目组织方式更清晰。`poetry env use python` 这个命令默认使用系统环境变量中的 python，如果你想使用指定版本的 python，可以将 python 换成`path/to/your/py`。

#### 添加依赖

我们可以编辑`pyproject.toml`，添加指定的依赖，再先后执行`poetry lock` 和`poetry install`，`poetry lock`会解析`pyproject.toml`中的主要依赖与开发依赖以及间接依赖，选择合适的依赖版本，记录到 poetry.lock 这个文件中。poetry.lock 的作用类似于 requirements.txt，即“锁定”项目需要的依赖（包括直接依赖和间接依赖）版本。紧接着的`poetry install`就会根据 poetry.lock 中的内容去下载相应版本的依赖。

```Toml
[tool.poetry] # 项目信息
name = "poetry-fastapi-demo"
version = "0.1.0"
description = "A web project template using poetry and fastapi."
authors = ["robocanic <robocanic@example.com>"]
license = "Apache 2.0"
readme = "README.md"

[tool.poetry.dependencies]  # 主要依赖
python = "~3.10"        # python版本>=3.10 且 <3.20
fastapi = "^0.115.0"    # fastapi版本 >=0.115.0 且 <0.116.0
sqlalchemy = "^2.0.36"  # sqlalchemy版本 >=2.0.36 且 < 3.0
pyyaml = "^6.0.2"       # pyyaml版本 >=6.0.2 且 < 7.0.0

[tool.poetry.group.dev.dependencies]  # 开发依赖
pytest = "^8.3.3"       # pytest版本 >=8.3.3 且 < 9.0.0

[build-system]  # 构建工具
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"

```

我们可以指定主要依赖和开发依赖，在发布部署时只打包主要依赖，这样就可以减小包的体积。同时我们可以指定依赖的版本范围，不同的符号有不同的指定范围策略，这一块可以参考 poetry 官方文档：[https://python-poetry.org/docs/dependency-specification/](https://python-poetry.org/docs/dependency-specification/)

此外，我们也可以直接使用命令行`poetry add deps`来下载包，这个命令会自动地更新`pyproject.toml`以及`poetry.lock`文件，保证环境中的依赖与`pyproject.toml`中的的依赖的一致性。

#### 删除依赖

我们可以通过编辑`pyproject.toml`，删除对应的依赖声明，然后执行`poetry lock` 即可，poetry 会自动帮我们把依赖删除。我们也可以使用`poetry remove deps`来删除依赖。

poetry 的这两种删除方式都会解析`deps`的无用的间接依赖，将其一并删除。

#### 删除虚拟环境

执行`poetry env remove poetry-demo`，即可删除虚拟环境。

> 如果在 poetry 的配置中指定了`poetry config virtualenvs.in-project true`，虚拟环境维护的目录就会放在项目的根目录下。类似于 venv 的方式，因此这种情况下删除.venv 文件夹就等同于删除虚拟环境

#### 查看依赖树

poetry 中很实用的一个功能是它可以使用`poetry show --tree`来查看依赖关系，从而在依赖错误时能够更好地定位异常，更快速地解决问题。

![poetry-show--tree](https://image.robocanic.com/notes/image-20241104154436296.png)

#### 导出依赖

理论上来说，有了 poetry 不再需要 pip 了，自然也不需要 requirements.txt 了，但是在后面的项目发布管理中我们会提到，当使用容器化部署时，我们希望我们的镜像尽可能地精简，我们的项目部署能尽可能快速。从精简镜像这个角度来看，poetry 对于我们的项目而言只是开发时依赖，在运行时我们希望只用最简单的依赖就能把项目跑起来。因此我们需要将 poetry.lock 中的依赖转化为 requirements.txt，这样部署时使用`pip install -r requirements.txt`下完依赖之后直接就能 run 起来项目。

poetry 本身也支持导出环境内的依赖，使用`poetry export -f requirements.txt -o requirements.txt`就可以将环境中的所有依赖导出到 requirements.txt 中。

#### 总结

poetry 作为环境管理与依赖管理合二为一的银弹，在体验下来，除了数据科学/AI 领域的项目，我认为 poetry 是开发大多数 python 项目的最佳环境与依赖的管理方式。

# 项目日志

> 以下代码地址: [https://github.com/robocanic/poetry-fastapi-demo.git](https://github.com/robocanic/poetry-fastapi-demo.git)

## 目标

日志的重要性不言而喻，我们这里探讨的“项目日志”，更准确地来说是怎么使用日志能够得到我们想要的效果。通常，我们对于日志会有一些要求：

1. 我们希望我们的日志能够显示出日志时间，日志等级(INFO, WARN, ERROR .....)，日志内容，所在线程，是由哪个组件/py 文件打印的......

2. 对于以上日志信息，不管是自己项目中的还是依赖包中的，都有一个统一的格式打印出来

3. 不仅要在控制台打印，还需要保存到文件中，以便后续排查错误时能够追溯。

4. 更进一步，我们希望 ERROR 级别的日志能够单独放到一个文件中，每天的日志按照日期做一个切割分块......

## 原理

在阐明日志需要达到的效果之后，我们来看看 python 中的 logging 模块，这个模块是 python 自带的，我们可以先来试用一下：

```Python
import logging
logging.info("this is a info log")
logging.warning("this is a warning log")
logging.error("this is a error log")
logging.critical("this is a critical log")
"""
output:
WARNING:root:this is a warning log
ERROR:root:this is a error log
CRITICAL:root:this is a critical log
"""
```

运行这个 python 文件会发现控制台并没有打印出 info 级别的日志，这是因为 logging 模块提供的日志记录函数所使用的日志器设置的日志级别是`WARNING`，因此只有`WARNING`级别的日志记录以及大于它的`ERROR`和`CRITICAL`级别的日志记录被输出了，而小于它的`INFO`级别的日志记录被丢弃了。

我们可以观察每条日志，每个日志都由三部分构成：`日志级别:日志器名称:日志内容`，这是因为 logging 模块提供的日志记录函数所使用的日志器设置的日志格式默认为`%(levelname)s:%(name)s:%(message)s`。

那自然我们会想到，可不可以通过设置 logging 的日志级别与日志格式，来生成我们想要的日志格式？进一步，我们可不可以调整日志输出的位置，同时输出在控制台和文件？

在回答这个问题之前，我们可以来先简单看下 logging 模块涉及到的四个组件

1. 日志器（Logger），提供了我们的代码可以使用的接口，即 log.info，log.warning，log.error 等

2. 处理器（Handler），将 logger 创建的日志记录发送到合适的目的输出

3. 过滤器（Filter），提供了更细粒度的控制工具来决定输出哪条日志记录，丢弃哪条日志记录

4. 格式器（Formatter），决定日志记录的输出格式

这些组件的关系大致如下：日志器（Logger）需要通过处理器（Handler）将日志信息输出到目标位置（文件，控制台，网络等）。而处理器（Handler）可以设置过滤器（Filter）和格式器（Formatter）来对日志进行过滤和格式化，再输出到指定的目标位置。

因此上面的问题也有了答案：通过调整 Logger，Handler，Filter，Formatter 的设置和组合关系，我们可以得到我们想要的日志。

我们可以通过 3 种方式来配置 logging：

1. 使用 Python 代码显式的创建 logger，handle，filter，formatter 并分别调用它们的配置函数

2. 创建一个日志配置文件，然后使用`fileConfig()`函数来读取该文件的内容；

3. 创建一个包含配置信息的 dict，然后把它传递给`dictConfig()`函数；

## 实践

这里我介绍一下第三种，其他两种可以自行研究。第三种我们通常会用一个 yaml/json 文件来描述配置，然后使用`yaml.load`读取日志配置文件，转化成 dict，进而传递给`dictConfig()`，完成 logging 的配置。

yaml 形式的日志配置如下：

```YAML
version: 1
formatters:  # 定义格式器
  default:   # 格式器名
    format: '%(asctime)s - %(name)s - %(levelname)s -%(threadName)s%(thread)d - %(message)s'  # 日志格式：时间 - 日志器名 - 日志等级 - 线程名 线程ID - 日志内容
    datefmt: '%Y-%m-%d %H:%M:%S'   # 时间格式
    use_colors: true   # 使用颜色
handlers:     # 定义处理器
  console:    # 处理器名
    class: logging.StreamHandler  # 使用的handler
    level: INFO    # 日志等级
    formatter: default  # 使用的formatter名字
    stream: ext://sys.stdout  # 输出到控制台
  file:
    class: logging.FileHandler
    level: INFO
    formatter: default
    filename: app.log   # 输出到app.log
loggers:         # 定义日志器
  poetry_fastapi_demo:   # 日志器名称
    level: INFO  # 日志等级
    handlers:    # 使用的handler
      - file
      - console
    propagate: false   # 是否传播到父logger再处理
  poetry_fastapi_demo.routers:  # poetry_fastapi_demo的子logger
    level: INFO
    handlers:
      - console
      - file
    propagate: false   # 是否传播到父logger再处理
  uvicorn:
    level: INFO
    handlers:
      - console
      - file
    propagate: false
root:            # 根日志器，所有没有归属的日志都会由root来处理
  level: INFO
  handlers:
    - console
    - file
```

其中，logger 是有层级结构的，logger 的名称是一个以'.'分割的结构，每个'.'后面的 logger 都是'.'前面的 logger 的 children。而 root 是一个特殊的 logger，它会捕捉没有被其他 logger 处理的日志消息，我们依赖的三方包打印的日志大部分都会由 root logger 捕获到进行处理，因此 root 是很重要的兜底 logger。

在完善好`log.yaml`后，我们做一个验证，`poetry-fastapi-demo`项目结构如下：

```Plain Text
 poetry-fastapi-demo/
 │
 ├── poetry_fastapi_demo/         
 │   ├── models             # 模型
 │   │   ├── __init__.py
 |   |   ├── user.py
 |   ├── routers             # 路由
 |   |   ├── __init__.py
 |   |   ├── user.py
 |   ├── tables              # 增删改查
 |   |   ├── __init__.py
 |   |   ├── user.py
 │   ├── __init__.py    
 │   └── main.py             # 主程序入口
```

`poetry_fastapi_demo/__init__.py`的内容如下，这里我们将 log.yaml 的路径以环境变量传入，然后使用`.dictConfig`完成日志的配置

```Python
import logging.config
import os
import yaml

log_conf_path = os.environ.get("LOG_CONFIG")
with open(log_conf_path, 'r', encoding='utf-8') as f:
    dict_conf = yaml.safe_load(f)
logging.config.dictConfig(dict_conf)

```

`poetry_fastapi_demo/routers/user.py`的内容如下

```Python
import logging

from fastapi import APIRouter

from poetry_fastapi_demo.models import User
from poetry_fastapi_demo.tables import user_table

log = logging.getLogger(__name__)
router = APIRouter()


@router.get("")
def get_all_users():
    return user_table.get_all_users()


@router.get("/{user_id}")
def get_user(user_id: int):
    return user_table.get_user_by_id(user_id)


@router.post("")
def create_user(user: User):
    log.info(f"Creating user: {user}")
    return user_table.add_user(user)


@router.delete("/{user_id}")
def delete_user(user_id: int):
    log.warning(f"Deleting user with id: {user_id}")
    return user_table.delete_user(user_id)


@router.put("")
def update_user(user: User):
    log.warning(f"Updating user: {user}")
    return user_table.update_user(user)

```

运行 main.py（记得设置环境变量），我们可以在控制台中看到日志输出，同时在项目的根目录下也会生成 app.log 日志文件，记录运行的日志。

![app-log](https://image.robocanic.com/notes/app-log.png)

我们发起一个新增 user 的请求：

```Shell
curl --request POST \
  --url http://localhost:8080/api/v1/user \
  --header 'content-type: application/json' \
  --data '{
  "id": 6,
  "name": "robb",
  "email": "robocanic@gmail.com",
 "password": "xclkjva",
  "role": "user"
}'
```

我们可以看到日志显示如下：

![create-user-log](https://image.robocanic.com/notes/create-user-log.png)

到这，基本的日志需求就满足了，我们无需调整代码，仅通过调整配置文件就能适时调整日志。那对于目标中的第 4 点“我们希望 ERROR 级别的日志能够单独放到一个文件中，每天的日志按照日期做一个切割分块”，相信你可以举一反三

# 项目测试

> 以下代码地址: [https://github.com/robocanic/poetry-fastapi-demo.git](https://github.com/robocanic/poetry-fastapi-demo.git)

一个项目可靠的项目离不开完备的测试，测试一般分为单元测试和接口测试，单测针对某个函数或者某个模块，测试返回的参数是否符合预期，而接口测试是针对于 HTTP（RPC）接口，测试的是某一块功能的完备性。

这里我们讨论的仅限于单元测试。在 python 中，单元测试可以使用 unittest 模块和 pytest，uninttest 是 python 自带的测试模块，而 pytest 是一个第三方模块，在兼容 unittest 的基础上，增加了集成测试的功能，方便我们进行批量的单元测试，与 CI 进行结合。因此后文中使用 pytest 来进行讲解。

pytest 可以自动递归指定路径下的所有文件，只要符合`test_`或者`_test`开头的 py 文件都会被识别为测试用例文件。都会在测试计划中执行。

## 实践

在使用 pytest 之前需要使用 pip 或者 poetry 下载，我们可以将 pytest 下载到`dev`分组的依赖，也就是开发时依赖（如`poetry add pytest --group dev`），以便生产部署时精简掉开发时依赖。

项目目录结构如下：

```Plain Text
 poetry-fastapi-demo/
 │
 ├── poetry_fastapi_demo/   # 程序主包    
 │   ├── models/             # 模型
 │   │   ├── __init__.py
 |   |   ├── user.py
 |   ├── routers/             # 路由
 |   |   ├── __init__.py
 |   |   ├── user.py
 |   ├── tables/              # 增删改查
 |   |   ├── __init__.py
 |   |   ├── user.py
 │   ├── __init__.py    
 │   └── main.py             # 主程序入口
 ├── tests/                  # 测试文件
 |   ├── test_user_table.py    # 模型
 ......
```

其中，`poetry_fastapi_demo/tables/user.py`内容如下：

```Python
from typing import Optional, List

from poetry_fastapi_demo.models import User

user_list = [
    User(
        id=1,
        name="Chris",
        email="chris@gmail.com",
        password="pso3sd5xv",
        role="admin"
    ),
    User(
        id=2,
        name="Jon",
        email="jon@gmail.com",
        password="rthdfwe5sdf",
        role="user"
    ),
    User(
        id=3,
        name="Arya",
        email="arya@gmail.com",
        password="ndddrwe42",
        role="user"
    )

]


class UserTable:

    def get_all_users(self) -> List[User]:
        return user_list

    def get_user_by_id(self, user_id: int) -> Optional[User]:
        for user in user_list:
            if user.id == user_id:
                return user
        return None

    def update_user(self, user: User) -> bool:
        for u in user_list:
            if u.id == user.id:
                u.name = user.name
                u.email = user.email
                u.password = user.password
                u.role = user.role
                return True
        return False

    def delete_user(self, user_id: int) -> bool:
        for i, u in enumerate(user_list):
            if u.id == user_id:
                del user_list[i]
                return True
        return False

    def add_user(self, user: User) -> bool:
        for u in user_list:
            if u.id == user.id:
                return False
        user_list.append(user)
        return True


user_table = UserTable()

```

测试案例`test_user_table.py`内容如下：

```Python
from poetry_fastapi_demo.models import User
from poetry_fastapi_demo.tables import user_table


class TestUserTable:

    def test_get_all(self):
        user_table.get_all_users()

    def test_get_by_id(self):
        user = user_table.get_user_by_id(1)
        assert user.name == "Chris"

    def test_update_user(self):
        user = user_table.get_user_by_id(1)
        user.name = "Chris2"
        assert user_table.update_user(user) is True
        assert user_table.get_user_by_id(1).name == "Chris"   # 故意写错

    def test_delete_user(self):
        assert user_table.delete_user(1) is True
        assert user_table.get_user_by_id(1) is None

    def test_add_user(self):
        result = user_table.add_user(User(
            id=4,
            name="robb",
            email="robb@gmail.com",
            password="loqjwnd[as;",
            role="admin"
        ))
        assert result is True
        assert user_table.get_user_by_id(4).name == "robb"



```

然后我们在项目根目录执行`pytest tests/ -v`，输出如下：

![py-test](https://image.robocanic.com/notes/py-test.png)

pytest 会递归地在 tests 文件夹下寻找以`test_`或`_test`开头的 py 文件，在 py 文件中，会寻找以`test_`或`_test`开头的函数或者类，对于我们这个案例来说，一共有 5 个测试用例，在输出中我们也能详细地查看到通过和失败的具体用例，错误的原因，执行的时间等等。此外，pytest 还有其他的功能（可参考[https://learning-pytest.readthedocs.io/zh/latest/index.html](https://learning-pytest.readthedocs.io/zh/latest/index.html)），如测试前和测试后的预处理和后处理，使用插件生成测试报告等，这里就算抛砖引玉给大家。

## CI 集成

按照开发流程来说，我们在本地开发并单测完后，会发布到测试环境中进行进一步地调试验证。有时候我们修改完代码后忘记执行对应的单测就提交了，然后在调试中才发现了错误，然后又回炉重造。为了尽量减少修改小错误反复发布的时间，我们可以将自动化测试集成到 gitlab pipeline（使用参考[https://docs.gitlab.com/ee/ci/pipelines/](https://docs.gitlab.com/ee/ci/pipelines/)）或 github actions(使用参考[https://docs.github.com/zh/actions](https://docs.github.com/zh/actions))。

#### github actions

```YAML
name: Python ci           # workflow名字

on:                            # 触发条件，push或merge到dev中触发
  push:
    branches: [ "dev" ]
  pull_request:
    branches: [ "dev" ]

jobs:
  build:
    runs-on: ubuntu-latest    # 跑在ubuntu虚拟机上
    strategy:                 # 执行策略
      fail-fast: false
      matrix:
        python-version: ["3.10"]

    steps:                    # 执行步骤
    - uses: actions/checkout@v4    # 使用checkout这个已有的action来初始化git环境
    - name: Set up Python ${{ matrix.python-version }}   # 使用setup-python来初始化python环境
      uses: actions/setup-python@v3
      with:
        python-version: ${{ matrix.python-version }}     # 指定python版本
    - name: Set up Poetry                 # 使用setup-poetry这个已有的action来初始化poetry
      uses: Gr1N/setup-poetry@v9
    - name: Install dependencies         # 使用poetry下载依赖，因为是在一个单独的虚拟机/容器中，所以无需创建虚拟环境
      run: |
        poetry config virtualenvs.create false
        poetry lock
        poetry install --all-extras
    - name: Test with pytest            # 使用pytest执行集成测试
      env:
        LOG_CONFIG: ./log.yaml
      run: |
        pytest tests/ -v
```

我们把上面这个 yml 命名为 ci.yml，并放置在项目根目录.github/workflows 目录下，然后 github 会自动检测到 workflow 的存在并按照 yml 中定义的步骤依次执行。然后我们在 dev 分支 push 到远程，然后我们会在 actions 中发现历史执行/正在执行的 workflow。

![github-action](https://image.robocanic.com/notes/github-action.png)

我们点进某一个 workflow 可以看到 job 的执行结果是成功还是失败：

![github-workflow-job](https://image.robocanic.com/notes/github-workflow-job.png)

进一步，我们进入 build 这个 job，我们可以看到集成测试的信息：

![job-test](https://image.robocanic.com/notes/job-test.png)

此外，github actions 还可以探索更多的玩法，比如：当 job 失败了发送邮件到自己的邮箱提示流水线失败了，当 job 失败了回滚到上一个 commit 的状态等等，这些就都留给你去探索了

#### gitlab pipeline

```YAML
stages:
  - automatic_test_dev

automatic_test_job_dev:   # 我们只在测试环境执行自动化测试，生产环境为了加快发布速度，不执行测试
  image: sunpeek/poetry:py3.10-slim    # 使用带poetry的python镜像
  stage: automatic_test_dev
  only:
    - dev                 # 只有dev分支发生变化（merge或push）时触发
  tags:                   # 匹配python标签的runner才运行
    - python
  script:                 # 执行脚本
    - poetry config virtualenvs.create false  # 在容器中，不用创建虚拟环境
    - poetry lock     # 锁定依赖版本
    - poetry install  # 下载依赖
    - pytest tests/   # 执行测试
```

我们将这个 yml 文件命名为.gitlab-ci.yml，然后 push 到 dev 分支，就会触发 pipeline。gitlab pipeline 本质上和 github actions 是一样的，都是声明式的流水线，因此这里就不再赘述，你可以查阅官方文档（[https://docs.gitlab.com/ee/ci/pipelines/](https://docs.gitlab.com/ee/ci/pipelines/)）来组合出你的 pipeline。

# 项目发布管理

当我们项目在本地运行 ok，测试也 ok 后，该考虑发布到各个环境（联调，测试，生产等）。这个过程根据项目的属性分为两类，一类是二方包的发布，另一类则是正经应用发布。

在谈这两类发布之前，我们先来确认发布过程中我们想要达到的目标（要求）：

1. 对于二方包发布，我们希望它在本地就能够发布 beta 版本，以便于快速验证。此外，正式版本的发布，我们希望能够集成到 CI 中。

2. 对于应用发布，我们希望它发布时足够迅速，以便在出现问题时快速推送补丁解决问题。

3. 无论是二方包还是应用发布，我们希望在发布的时候自动执行集成测试（生产除外），生成一份测试报告。

## 二方包发布

二方包的发布根据我们使用的项目环境和依赖管理工具可以划分为两类，第一类是使用 conda+pip 或者 venv+pip 后的发布流程，另一类是使用 poetry 后的发布流程。

### setuptools+twine

发布分为打包和推送两个步骤，setuptools 负责将我们的 python 代码打包，而 twine 则负责将打包后的 pkg 推送到 PyPI 仓库。

setuptools 是很多年来 python 打包的首选工具，我们首先需要执行`pip install setuptools`来下载这个工具。然后在项目根目录下新建一个`setup.py`文件来写打包时的配置，一个 setup.py 如下所示：

```Python
from setuptools import setup, find_packages

with open("README.md", "r", encoding="UTF-8") as fh:
    long_description = fh.read()
version= os.environ.get("GITHUB_REF_NAME", "0.0.1b0") # 包的版本，GITHUB_REF_NAME是github tag name的环境变量，代表代码分支TAG；0.0.1b0 是0.0.1-BETA0的缩写
setup(
    name='poetry_fastapi_demo',  # 包名
    version=version,     # 构建版本
    author='robocanic',  # 作者
    author_email='robocanic@gmail.com',
    description='A web project template using poetry and fastapi.',   # 描述
    long_description=long_description,  # 长的描述
    long_description_content_type="text/markdown",
    url='https://github.com/robocanic/poetry-fastapi-demo',
    packages=find_packages(),  # 打包所有包目录
    classifiers=[
        "Development Status :: 2 - Beta"    # 开发阶段
        "Programming Language :: Python :: 3.10",   # 使用的语言
        "Operating System:: OS Independent",   # 不限定OS
    ],
    install_requires=[     # 该项目的依赖，别人下载我们的包时会自动下载这些依赖
        "fastapi==0.115.0",
        "sqlalchemy==2.0.36",
        "pyyaml==6.0.2",
        "uvicorn=0.32.0"
    ],
    python_requires='>=3.10'    # python要求
)

```

`setup.py`使用代码的方式声明了我们打包的配置，包括构建版本，直接依赖，python 版本要求等等。接下来我们只需要执行`python setup.py sdist`就能将项目打包成.tar.gz 文件，当然我们也可以指定打包格式为`bdist_wheel`，打包成.wheel 文件。打包后的文件默认放在 dist 目录下。

接下来我们需要使用 twine 上传到 PyPI 仓库，twine 也是一个 python 包，在使用之前需要通过`pip install twine`下载。然后执行`twine upload dist/*`就上传到了仓库。

> 如果要上传到私有的 PyPI 仓库，则需要配置上传的用户名和密码，同时在上传时需要指定私有仓库。最便捷的方法是在 root 目录下新增一个.pypirc 文件，文件内容为：

    [distutils]


    index-servers = private-pypi

    [nexus]

repository=https://private-pypi.com/simple/
username=foo
password=bar
然后使用`twine upload -r private-pypi dist/*` 来上传到指定的私有仓库

### poetry

如果你使用了 poetry，那 poetry 就自带了打包和上传的功能。poetry 打包非常简单，只需要执行`poetry build`，它就会参照`pyproject.toml`中的配置进行打包。打包生成的文件默认也会放在 dist 目录下。

然后我们执行`poetry publish`就可以发布到 PyPI 仓库

> 同样，若要使用 poetry 上传到私有的 PyPI 仓库。也需要额外的配置：
> `poetry config repositories.foo-pub [https://pypi.example.org/legacy/](https://pypi.example.org/legacy/)` > `poetry config http-basic.foo-pub <username> <password>` > `poetry publish --repository foo-pub`

### CI 集成

很多时候我们发现了特定版本包的一些问题，我们想要这个特定版本的包对应的源代码，并在此基础上加一个补丁。因此我们需要将我们包的版本和我们代码的特定分支一一对应。通过人工完成这个过程是可行的，我们只需要在每次打包前/后 加一个 commit，推到远程仓库，打一个 tag 就能对应起来。但是这件重复的工作我们时常会忘记，这个重复的工作可以交给一个单独的程序来做吗？答案当然是肯定的，毕竟程序员最习惯的事就是将重复的工作程序化，复杂的工作流程化。gitlab pipeline/github actions 能够通过我们声明的 yml 文件来完成这个过程。

#### gitlab pipeline

一个使用 poetry 的二方包发布的 gitlab pipeline 文件可以如下（集成了单测）：

```YAML
image: sunpeek/poetry:py3.10-slim   # 需要带有poetry的python镜像

stages:
  - automatic_test
  - package_build_push


automatic_test_job:   #
  stage: automatic_test
  only:
    - dev
  tags:
    - golang
  script:
    - poetry config virtualenvs.create false  # 在容器中，不用创建虚拟环境
    - poetry lock     # 锁定依赖版本
    - poetry install  # 下载依赖
    - pytest tests/   # 执行测试


package_build_push_job:
  stage: package_build_push
  only:    # 触发条件：只有打tag之后才会触发该job
    - tags
  script:
    - poetry config repositories.foo-pub https://pypi.example.org/legacy/   # 添加发布仓库
    - poetry config http-basic.foo-pub $FOO_USERNAME $FOO_PASSWORD          # 添加发布仓库的用户名密码，通过流水线环境变量可以设置
    - sed -i "s/^version = \".*\"/version = \"$CI_COMMIT_TAG\"/" pyproject.toml  # 修改构建版本，与TAG一致
    - poetry publish --build  --repository foo-pub   # 打包并推送
```

简单来说，gitlab 在我们打上一个 tag 之后，会起一个 runner 执行这个 yml 文件的 script（具体过程可自行了解）。因此当我们想要发布一个版本时，推到镜像仓库后，打一个 tag，gitlab pipeline 就会进行打包，推送，并且推送到仓库的包的版本和我们的 TAG 是严格对应的。

#### github actions

```YAML
name: Release package           # workflow名字

on:                            # 触发条件，新建tag时触发
  push:
    tags:
      - '*'                  # 任意标签都会触发工作流

jobs:
  pkg:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4    # 使用checkout这个已有的action来初始化git环境
      - name: Set up Python ${{ matrix.python-version }}   # 使用setup-python来初始化python环境
        uses: actions/setup-python@v3
        with:
          python-version: ${{ matrix.python-version }}     # 指定python版本
      - name: Set up Poetry                 # 使用setup-poetry这个已有的action来初始化poetry
        uses: Gr1N/setup-poetry@v9
      - name: Extract version
        id: extract_version
        run: |
          VERSION="${GITHUB_REF#refs/tags/}"
          echo "VERSION=${VERSION}" >> $GITHUB_ENV
      - name: Replace version
        run: |
          sed -i "s/^version = \".*\"/version = \"${{ env.VERSION }}\"/" pyproject.toml
      - name: Build Package        # 打包
        run: |
          poetry config virtualenvs.create false
          poetry build
      - name: Publish Package    # 发布
        env:
          PYPI_API_TOKEN: ${{ secrets.PYPI_API_TOKEN }}    # 提前定义pypi的 api token
        run: |
          poetry config pypi-token.pypi $PYPI_API_TOKEN
          poetry publish


```

#### 总结

poetry 作为集环境管理，依赖管理，打包发布一体的工具，虽然有上手难度，但从长期来看是值得使用的，强烈推荐。gitlab pipeline/github actions 的使用倒是可以看自身需求，如果项目比较小，发布次数少，不要求代码分支与包的版本严格对应，是可以不用的。此外，我们在使用 gitlab pipeline 时也可以适当没那么“严格”，比如，为了图方便我们可以在本地发布非正式版本，而正式版本则走 pipeline，保证正式版本的稳定。

## 应用发布

应用发布即让我们的应用运行在机器上，对外提供服务。应用的运行形态可以基本分为容器/虚拟机，后文中都基于使用更广泛的容器来进行说明。

应用发布大体上分为两个阶段，一个是镜像的打包与推送，另一个是镜像拉取和启动容器。其中，镜像推送，镜像拉取和启动容器对于每一种应用程序都是大差不差的，而镜像打包则是差异比较大的，所以这里我就镜像打包展开来讲。

### 镜像打包

镜像的大小关乎到推送和拉取的效率，因此我们要尽可能地减小镜像大小。python 的基础镜像中，`slim`版本的基础镜像移除了不必要的系统个工具和库，因此基础镜像我们选择`slim`版本（127MB 左右）。

此外，我们如果使用了 poetry 来作为依赖管理的工具，那在打包时就有两种选择：第一种是直接将 poetry 打到基础镜像中，第二种是使用 multi-stage build，即在第一个阶段先下载 poetry，导出依赖到 requirements.txt，然后在第二个阶段使用 pip install 来下载依赖。multi-stage build 虽然最终的镜像里不包含 poetry，但是第一个阶段下载 poetry 和导出依赖都需要时间，而第一种方法中直接将 poetry 打到基础镜像中，镜像大小也只增加了不到 100MB，在可以接受的范围内，因此我更推荐第一种。

确定好镜像策略后，我们来看看 Dockerfile 长啥样（基础镜像的打包就是在 python 基础镜像的上面使用 pip 下载 poetry 即可，不再赘述）：

```Dockerfile
# 包含poetry的python基础镜像
FROM sunpeek/poetry:py3.10-slim
LABEL maintainer="robocanic@gmail.com"
# Keeps Python from generating .pyc files in the container
ENV PYTHONDONTWRITEBYTECODE=1
# Turns off buffering for easier container logging
ENV PYTHONUNBUFFERED=1

# mv source code to a specific directory
WORKDIR /appruntime
COPY . /appruntime

# no need to create virtual env
RUN poetry config virtualenvs.create false
# analyze deps
RUN poetry lock
# Install deps
RUN poetry install

# Creates a non-root user with an explicit UID and adds permission to access the /appruntime folder
RUN adduser -u 5678 --disabled-password --gecos "" appuser && chown -R appuser /appruntime
USER appuser

CMD ["uvicorn", "poetry_fastapi_demo.main:app", "--host=0.0.0.0", "--port=8080", "--log-config=/appruntime/log.yaml"]
```

### CI 集成

如果使用了 gitlab pipeline/github actions，那我们可以将镜像打包，镜像推送都结合到 CI 中

#### gitlab pipeline

```YAML
variables:    # 变量
  IMAGE_NAME: $CI_PROJECT_PATH
  PROJECT_NAME: $CI_PROJECT_NAME


stages:
  - automatic_test_dev
  - package_build_push_dev
  - deploy_dev

automatic_test_job_dev:   # 我们只在联调/测试时执行自动化测试，生产环境为了加快发布速度，不执行测试
  image: sunpeek/poetry:py3.10-slim    # 使用带poetry的python镜像
  stage: automatic_test_dev
  only:
    - dev
  tags:
    - python
  script:
    - poetry config virtualenvs.create false  # 在容器中，不用创建虚拟环境
    - poetry lock     # 锁定依赖版本
    - poetry install  # 下载依赖
    - pytest tests/   # 执行测试

package_build_push_job_dev:
  image: docker:27.3  # 使用docker镜像
  stage: package_build_push_dev
  only:
    - dev
  tags:
    - python
  artifacts:
    paths:
      - BUILDTIME   # 缓存构建时间文件供下一个job使用
  script:
    - date +'%Y%m%d%H%M' > BUILDTIME  # 将构建时间写到文件
    - export IMAGE_TAG=dev-$(cat BUILDTIME)-$CI_COMMIT_SHORT_SHA # 镜像tag  dev-[年月日时分]-[commit id前八位]
    - export IMAGE_NAME=$(echo $IMAGE_NAME | tr 'A-Z' 'a-z')  # 把gitlab的目录和项目名小写作为镜像名
    - docker login -u $REGISTRY_USERNAME -p $REGISTRY_PASSWORD
    - docker build  -t $IMAGE_NAME:$IMAGE_TAG .
    - docker push $IMAGE_NAME:$IMAGE_TAG
```

#### github actions

```YAML
name: Release deploy           # workflow名字

on:                            # 触发条件，新建tag时触发
  push:
    tags:
      - '*'                   # 任意标签都会触发工作流

jobs:
  pkg:
    runs-on: ubuntu-latest
    steps:
      - name: Set up QEMU       # 初始化qemu
        uses: docker/setup-qemu-action@v3
      - name: Set up Docker Buildx   # 初始化buildx
        uses: docker/setup-buildx-action@v3
      - name: Login to Docker Hub   # 登录dockerhub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_PASSWORD }}
      - name: Extract version   # 提取tag
        id: extract_version
        run: |
          VERSION="${GITHUB_REF#refs/tags/}"
          echo "VERSION=${VERSION}" >> $GITHUB_ENV
      - name: Build and push   # 构建镜像并推送
        uses: docker/build-push-action@v6
        with:
          push: true
          tags: ${{ github.repository }}:${{ env.VERSION }}
```

### 总结

如果你使用 docker compose/docker swarm/k8s 等基础设施，你还可以结合 docker/kubectl 或者 docker api/k8s api 来进行 CD，从而实现应用从发布到部署完全流水线化。

应用的发布在不同的团队，不同的公司是有着不同的流程和偏好的，以上只是我提供的一种实现参考，但是无论是什么发布流程，考虑的重点其实是大差不差的：

1. 镜像大小要尽可能压缩

2. 发布要做到可回滚，可灰度

3. 发布流程中的信息要可追溯

4. 发布中用到的组件复用度要高

5. ......

按照这些个思路去编排发布流水线，相信你能得到一个理想中的流水线。
