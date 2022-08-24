### 模块管理

Maven 可以有效管理多个模块，每个模块都可以看作一个独立的 Maven 项目。模块之间共通依赖可以提取到 `parent` 项目。

`parent` 项目的 `<packaging>` 是 `pom` 而不是 `jar`，因为 `parent` 本身不含任何代码。

模块之间可以有依赖关系。

#### 1、管理依赖

当项目模块较多，为了避免引入 jar 版本的混乱，需要通过 `dependencyManagement` 解决。

在项目的根 pom 中使用 `dependencyManagement` 标签定义好所有子模块依赖的 jar 的版本，子模块依赖对应的 jar 时即可不指定版本号。

管理依赖 `dependencyManagement` 只是进行版本管理，并不会将其定义的 jar 导入项目作为项目依赖。