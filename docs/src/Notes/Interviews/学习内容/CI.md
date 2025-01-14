### 什么是 CI

CI（**<font style="color:rgb(37, 43, 58);">Continuous Integration</font>**）：持续集成，是一种软件开发实践，目的是让开发团队能够**快速、可靠地构建和测试代码**。

### 为什么需要 CI

**在进行多人协作的日常工作中，为了保证整个团队的代码质量维持一个比较高的水平，我们往往会在 pre-commit 钩子里配置 lint 校验或者在 CI 中执行校验**，两者的最大不同就是本地运行的 git hooks 可以被手动跳过（--no-verify ），且校验未通过内容只有当前分支的开发者本人可见，并伴随着较长的校验运行等待时间，无法在根本上保障代码质量。

### CI 组成

![alt text](image-3.png)

### Github Actions 概述

<font style="color:rgb(62, 62, 62);">GitHub Actions 是 GitHub 推出的持续集成（Continuous Integration，简称 CI）服务，它提供了整套虚拟服务器环境，它为开发者提供了完整的虚拟服务器环境，用于构建、测试、打包和部署项目。</font>

#### <font style="color:rgb(62, 62, 62);">核心概念</font>

- **Workflows**

<font style="color:rgb(62, 62, 62);">工作流由一个或多个作业组成，可以由事件调度或触发。</font>

- **Event**

<font style="color:rgb(62, 62, 62);">事件，触发工作流的特定动作。例如，向存储库提交 pr 或 pull 请求。</font>

- **Jobs**

<font style="color:rgb(62, 62, 62);">作业，在同一 runners 上执行的一组步骤。</font>

- **Steps**

<font style="color:rgb(62, 62, 62);">步骤，可以在作业中运行命令的单个任务</font>

- **Actions**

<font style="color:rgb(62, 62, 62);">操作，是独立的命令，它们被组合成创建作业的步骤。操作是工作流中最小的可移植构建块。</font>

- **Runners**

<font style="color:rgb(62, 62, 62);">运行器，安装了 GitHub Actions 运行器应用程序的服务器。Github 托管的运行器基于 Ubuntu Linux、Microsoft Windows 和 macOS，工作流中的每个作业都在一个新的虚拟环境中运行。</font>

### GitLab CI 概述

GitLab CI 是 GitLab 提供的原生 CI/CD 功能，与 GitHub Actions 类似，它也是基于 YAML 文件配置的自动化工具，但在功能和灵活性上更加全面。

#### 怎么编写

##### 创建.gitlab-ci.yml 文件

GitLab 会自动检测项目根目录中的 `.gitlab-ci.yml` 文件，并使用其中的配置来执行 CI/CD 流程。具体的实现过程如下：

- **检测 **`**.gitlab-ci.yml**`** 文件**：每当有新的提交或者 merge request 时，GitLab 会扫描项目根目录下是否存在 `.gitlab-ci.yml` 文件。如果存在，GitLab 会读取这个文件。
- **解析 YAML 文件**：GitLab 会解析这个 YAML 文件，提取出各个阶段（`stages`）和任务（`jobs`）。每个任务会根据定义的规则执行，通常包括执行命令、设置缓存、上传构建产物等。
- **触发 CI/CD 流程**：根据 GitLab CI 配置的规则，GitLab 会决定何时触发特定的任务。例如，当推送到 `master` 分支时，可能会触发部署任务；而推送到其他分支时，可能会触发构建和测试任务。
- **执行并反馈结果**：GitLab 会在其 UI 上展示每个任务的执行状态，包括成功、失败和日志输出。执行的每个步骤都会按顺序进行，并可以设置条件（如只有某些条件下才执行任务）。

##### 定义 stages

<font style="color:rgb(25, 27, 31);">stages 定义在 YML 文件的最外层，它的值是一个数组，用于定义一个 pipeline 不同的流程节点。</font>

```yaml
stages:
  - build
  - test
  - deploy
```

##### 编写 job

在 `.gitlab-ci.yml` 文件中，`job` 定义了在每个阶段中要执行的任务。每个任务包含：

- `**script**`：要执行的命令（例如安装依赖、构建代码等）。
- `**stage**`：任务所属的阶段。
- `**only**`**/**`**except**`：控制任务在哪些条件下执行。

```yaml
build:
  stage: build
  script:
    - npm install
    - npm run build
```

##### 条件控制

`only` 和 `except` 用于控制 `job` 任务执行的条件。例如，你可能希望某个任务只在某个分支上执行，或者只在某个标签上触发。

- `**only**`：定义哪些分支、标签或事件触发该任务。
- `**except**`：定义哪些分支、标签或事件不触发该任务。

例如：

```yaml
deploy:
  stage: deploy
  script:
    - ./deploy.sh
  only:
    - master
    - tags
```

这表示，`deploy` 任务只会在 `master` 分支或任何标签的提交时触发

#### **区分不同环境**

1. 分支策略：通过不同的 Git 分支（如 `master`、`staging`、`develop`）来区分环境。
   - `master` 分支可以部署到生产环境。
   - `staging` 分支部署到测试环境。
   - `develop` 分支用于开发环境。
2. 在 `.gitlab-ci.yml` 中可以使用 GitLab 的 `variables` 来区分不同环境。例如，部署时根据不同的环境变量来选择部署到不同的服务器或使用不同的配置。

```yaml
deploy_to_staging:
  stage: deploy
  script:
    - deploy --env staging
  only:
    - staging

deploy_to_production:
  stage: deploy
  script:
    - deploy --env production
  only:
    - master
```

3. 多环境部署细节

在 gitlab 中，可以使用 only 或者配置 environments 来实现多环境。

```yaml
image: node:16

stages:
  - deploy-prod
  - deploy-dev

cache: # 缓存
  paths:
    - node_modules

deploy-job-prod:
  stage: deploy-prod
  script:
    - npm install
    - npm run build
    # 安装并配置 oss
    - wget http://gosspublic.alicdn.com/ossutil/1.7.3/ossutil64 -O /usr/bin/ossutil64
    - chmod 755 /usr/bin/ossutil64
    - ossutil64 config -e $OSS_END_POINT -i $OSS_ACCESS_KEY_ID -k $OSS_ACCESS_KEY_SECRET
    # 部署制品文件到阿里云 oss test-zuo11-com 仓库
    - ossutil64 cp -r -f ./.vitepress/dist oss://test-zuo11-com
  when: manual
  only:
    # main分支
    - main
  artifacts:
    paths:
      - ./.vitepress/dist

deploy-job-dev:
  stage: deploy-dev
  script:
    - npm install
    - npm run build
    # 安装并配置 oss
    - wget http://gosspublic.alicdn.com/ossutil/1.7.3/ossutil64 -O /usr/bin/ossutil64
    - chmod 755 /usr/bin/ossutil64
    - ossutil64 config -e $OSS_END_POINT -i $OSS_ACCESS_KEY_ID -k $OSS_ACCESS_KEY_SECRET
    # 部署制品文件到阿里云 oss test-zuo11-com 仓库
    - ossutil64 cp -r -f ./.vitepress/dist oss://test-zuo11-com-dev
  when: manual
  only:
    # v1.0.0分支
    - v1.0.0
  artifacts:
    paths:
      - ./.vitepress/dist
```

### 两者异同

**相同点**

- <font style="color:rgb(51, 51, 51);">都是通过 YAML 语法配置来实现 CI/CD 流水线的</font>
- <font style="color:rgb(51, 51, 51);">都是在项目更目录下创建一个 yml 文件来实现对流水线的控制的</font>
- <font style="color:rgb(51, 51, 51);">CI/CD 运行的背后都是 Runner 组件来实现的</font>

**不同点**

- GitHub Actions 通过“插件”机制来实现 CI/CD 的编写，在 marketplaces 中有别人发布的 actions，找到合适的直接使用即可；GitLab CI 需要自己根据流程来编写流水线，当然也可以引用一些已经内置的模版（涉及 DevSecOps 的居多，DevOps 是 "Development"（开发）和 "Operations"（运维）的缩写，代表了一种**文化**。）。当然，从 16.0 开始，GitLab 也引入了 component & catalog 这样的功能来简化流水线的编写，提高复用性。
- GitLab CI 在流水线功能方便要比 GitHub Actions 多不少。GitLab CI 有 DAG（有向无环图）、合并结果、多项目等流水线类型，主要是针对不同团队规模、不同场景，此外还有流水线的一些审核和配置规则，这些是 GitHub Actions 不足的地方。

### 不同场景选择

#### 企业为什么更倾向于使用 GitLab CI？

1. **私有化部署的需求**：
   - GitHub Actions 无法在企业内部部署，而 GitLab 提供了一键式私有化部署方案，满足数据隐私和合规需求。
   - 国内网络环境对 GitHub 的访问不稳定，而自建 GitLab 实例可以完全避免这一问题。
2. **更强大的功能支持**<font style="color:rgb(51, 51, 51);">：</font>
   - <font style="color:rgb(51, 51, 51);">GitLab CI 的流水线功能更全面，例如支持 DAG 和多项目流水线，能够更好地适应复杂企业场景。</font>
   - <font style="color:rgb(51, 51, 51);">内置安全扫描和代码审计功能，满足企业对代码质量和安全性的高要求。</font>
3. **资源与成本：**
   - <font style="color:rgb(37, 43, 58);">在</font>`Github Action`<font style="color:rgb(37, 43, 58);">中， </font>**<font style="color:rgb(37, 43, 58);">Job</font>**<font style="color:rgb(37, 43, 58);"> 和 </font>**<font style="color:rgb(37, 43, 58);">Step</font>**<font style="color:rgb(37, 43, 58);"> 以及 </font>**<font style="color:rgb(37, 43, 58);">Workflow</font>**<font style="color:rgb(37, 43, 58);"> 都有资源占用以及时间限制，超出限制就会直接取消运行</font>
   - <font style="color:rgb(37, 43, 58);">GitLab 自托管 Runner 无时间与资源限制，适合高频构建需求的企业。</font>

#### 开源为什么选择 Github Actions

<font style="color:rgb(37, 43, 58);">公共仓库和自托管运行器免费使用 GitHub Actions。 对于私有仓库，每个 GitHub 帐户可获得一定数量的免费记录和存储，具体取决于帐户所使用的产品。 超出包含金额的任何使用量都由支出限制控制。</font>

### Gitlab-ci 坑点

1. **Job 一直挂起，没有 Runner 来处理**

   - <font style="color:rgb(25, 27, 31);">首先考虑的是不是 Runner 没有激活，如果没有那么按上面方式处理</font>

   - <font style="color:rgb(25, 27, 31);">还可能是 tag 没有匹配到，上面说过，Runner 注册时是要填写绑定 tag 的，如果你在 YML 里面编写 Job 没有带上 tag 是不会有自定义 Runner 来处理。解决方法:给 Job 加 tags</font>
   - <font style="color:rgb(25, 27, 31);">最后一种可能：你连续注册了多个 Runner,这些 Runner 冲突了，或者是新注册的 Runner 和旧 Runner 使用了同一个 token,这时候先删掉本地其他旧的 Runner，然后重置 token,并使用更新后的 token 重新注册一个 Runner</font>

<font style="color:rgb(25, 27, 31);"></font>

<font style="color:rgb(37, 43, 58);"></font>

<font style="color:rgb(37, 43, 58);"></font>

<font style="color:rgb(37, 43, 58);"></font>

<font style="color:rgb(37, 43, 58);">参考文献：</font>

[https://juejin.cn/post/7155379650061926437](https://juejin.cn/post/7155379650061926437)

[https://gitlab.cn/docs/jh/ci/quick_start/](https://gitlab.cn/docs/jh/ci/quick_start/)

[https://zhuanlan.zhihu.com/p/184936276](https://zhuanlan.zhihu.com/p/184936276)

[https://juejin.cn/post/7261519520106774588?searchId=202501130049271A828F294ED4D2DF0485#heading-12](https://juejin.cn/post/7261519520106774588?searchId=202501130049271A828F294ED4D2DF0485#heading-12)

[https://juejin.cn/post/7155379650061926437#heading-4](https://juejin.cn/post/7155379650061926437#heading-4)
