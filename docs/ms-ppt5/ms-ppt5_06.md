# 第六章：工作流程

在本章中，我们将讨论 Puppet 中的工作流程。我们将介绍什么是好的技术工作流程，如何将其应用到 Puppet 上，以及如何使用**Puppet 开发工具包**（**PDK**）来改进我们的工作流程。我们将探讨一个好的工作流程的以下特性：易用性、快速反馈、入职容易性和质量控制。我们将使用 Puppet Git 仓库提供一个基本的 Puppet 工作流程，该流程可以调整以适应任何管理系统。我们还将探索 Puppet 发布的新的 PDK，它可以改进我们的工作流程。

本章将涵盖以下主题：

+   Puppet 工作流程

+   设计 Puppet 工作流程

+   使用 PDK

# Puppet 工作流程

工作流程是一系列处理流程，从启动到完成。随着组织中 Puppet 环境变得更加复杂，一个值得信赖的共享工作流程将使得共享工作变得更加容易。Puppet 工作流程应该允许我们访问代码、编辑代码、测试代码，并最终将代码部署回 Puppet Master。虽然这不是强制要求，但强烈建议组织或团队采用共享工作流程。共享工作流程有几个主要好处，如下所示：

+   可衡量的易用性

+   快速反馈

+   入职容易性

+   质量控制

# 易用性

设计并开始工作流程的主要原因是为了提供易用性。团队应该围绕他们的代码库设计工作流程，使他们能够了解如何检索特定的代码、如何编辑这些代码以及新的编辑会带来哪些影响。工作流程还提供了一种标准化的方式来打包代码，以便现有代码库使用。工作流程中的每个步骤应该是清晰、简洁、易于沟通和可重复的。重要的是，团队中的每个人不仅要了解工作流程的运作方式，还要理解每个步骤存在的原因，这样他们才能在组织中发生变化时进行故障排除并为工作流程做出贡献。

相对于个性化的工作流程，共享工作流程的主要好处之一是能够衡量工作流程对组织的影响。为了衡量我们的工作流程，我们首先将标准和非标准的工作单位分开。我们对代码所做的编辑通常在大小和复杂性上有所不同，且不容易用标准单位衡量。另一方面，代码通常是以相同的方式进行检出、测试和部署的，这使我们能够很好地估算出完成工作流程所需的时间，减去代码编辑的部分。

如果我们的工作流需要大约 30 秒来克隆代码库，编辑代码的时间不确定，5 分钟来运行测试，另加 30 秒来在我们的环境中部署代码，那么我们的工作流，进行一次测试时，约需 6 分钟。如果我们的团队有 8 名成员，每人平均每天执行这个工作流 10 次，那么我们的工作流实际上每周占用了大约 8 小时的团队工作时间（*8 x 10 x 6* = *480* 分钟，或 8 小时）。将测试时间减半，可以使我们团队在工作流上花费的总时间减少大约 3 1/3 小时每周。因此，由于工作流中可以节省的这部分可量化时间，团队应尽可能考虑优化工作流。

通常，你不需要比估算执行工作流中标准功能所需时间更详细的估算，但你需要知道哪些部分可能会被多次执行。在使用 Puppet 时，用户很可能会写代码、推送代码并进行测试，而不是从服务器拉取代码。你可以分别检查工作流的每个部分，尝试改进其中的一部分，但你也应该考虑到修改对工作流其他部分的影响。

# 快速反馈

一个好的工作流应该为用户提供持续的反馈。每个步骤都应该有明确的定义，并具有严格的通过或失败标准。例如，Git 会在检测到问题时发出警告，比如无法拉取代码或推送代码到原始仓库。我们可以通过扩展 Git 提交钩子（无论是服务器端还是客户端）来进行检查，确保代码在被接纳到组织的 Git 服务器之前是处于正确状态的。在我们的测试标准中运行 Puppet 时，我们期望它能进行干净且幂等的运行。Puppet 目录不应生成失败的资源，也不应在每次 Puppet 运行时管理相同的资源。

使用 Puppet 解决问题所需的时间会随着工作流对工程师提供的反馈量增加而缩短。如果你在一个需要将代码推送到 Puppet Master 环境并在真实代理上进行测试的工作流中，简单地运行 `puppet parser validate` 可以节省大量时间。解析器验证会快速告诉你 Puppet 代码是否可以编译，而不是告诉你它将会做什么。这个简单的命令可以减少我们在代码上执行 `git commit`、推送到 Git 仓库、部署到环境、登录测试机并等待 Puppet 代理触发目录错误的次数。我们甚至可以确保每次提交前都会运行这个命令，使用 Git 的预提交钩子。自动化测试工具，如 RSpec 和 Beaker，可以扩展这种方法，并且与 CI/CD 管道（将在下一章讨论）结合使用，可以为代码开发者提供更快速的反馈。

# 容易上手

一个良好的工作流自然能够促进新成员加入项目的便利性，无论是开源项目还是组织的一部分。一个简单的工具套件和指南对新成员来说是无价的，能帮助他们克服首次提交的难关。即使是一个简单的`README`，如果维护得当，也能发挥很大作用。将新成员引入项目是昂贵的，而高质量的工作流可以减少新成员所花费的时间。加入新项目成员还需要现有项目成员提供一些信息和时间。如果你的项目是一个持续开发的工作，成员流动很可能会发生，因此，在工作流中节省现有成员的时间，并缩短新成员达到有效性所需的时间，应当是优先事项。

# 质量控制

一个好的工作流应始终致力于减少错误并提高代码质量。工作流中的每一个内置安全机制都可以让团队更快速地在复杂功能上进行迭代。简单的措施，比如防止直接推送到生产分支并将生产环境基于语义版本化的代码，可以实现快速开发，而不必担心破坏关键基础设施。

以下列出了一些围绕安全性和稳定性设计的工作流改进示例：

+   防止直接向控制仓库的生产环境推送代码

+   防止直接向单个模块的主节点推送代码

+   在将所有清单推送回源仓库之前，运行 Puppet 解析器验证所有清单

+   在将代码合并到主分支或类似生产环境的控制仓库分支之前，进行代码审查

+   自动化测试

# 设计 Puppet 工作流

自 Puppet 开始以来，代码管理经历了很多变化。即使是整体工作流也发生了巨大变化。本节将帮助你了解 Puppet 代码管理的一些历史，面临的一些挑战，最重要的是，一些设计和使用强大 Puppet 工作流的解决方案。

最初，我们直接将 Puppet 清单写入磁盘。我们通过 SSH 登录到 Puppet Master 并直接编辑清单，将大部分代码视为远程机器的配置文件。这种模式需要为应用到代理的代码提供自定义备份和恢复，并且不提供便捷的回滚功能。如果部署出现问题，你不得不手动从备份中提取代码片段并将其部署到系统中。一些社区成员开始将 Puppet 代码存储在 Git 中。随着组织中独立仓库数量的增加，手动一个个引入 Git 仓库变得更加麻烦，一些社区开源项目也开始形成，专注于 Git 代码的分阶段管理。

# Puppet 工作流的组成部分

虽然 r10k 不是唯一的 Puppet 代码管理工具，但它已成为企业组织中部署的标准代码管理工具。我们将把工作分解为任务和仓库，如下所示：

+   仓库：

    +   控制仓库

    +   模块仓库

+   任务：

    +   克隆

    +   创建新分支

    +   编辑相关代码

    +   添加并提交

    +   推送

    +   Puppet 登录并部署

    +   分类

    +   测试（自动或手动）

# 仓库

代码管理要求所有代码都存储在 Git 中。将代码拆分到多个仓库并将其置于主分支上，可以引用不同版本的代码。每个模块应独立存放在单独的仓库中，以便进行版本控制和治理。`Puppetfile` 将通过 Puppet Forge 或指向您自己本地 Git 实例来调用这些仓库。

# 控制仓库

如前一章所述，我们的控制仓库不过是一个 Git 代码仓库。与其打交道时需要特别注意的一点是，分支名称对应于 Puppet 环境。如果您创建一个名为 `feature` 的 Git 分支并部署代码，Puppet Master 将把代码部署到 `/etc/puppetlabs/code/environments/feature`。通常，主分支会被另一个名为 `production` 的保护分支取代，以便代理默认将代码提交到生产分支。

# 模块仓库

模块仓库是标准的 Git 仓库。通常，我们希望保护主分支，避免其接收直接提交。组件模块的贡献者应通过提交拉取请求（pull request）来提交代码，并在将代码合并到主分支之前进行代码审查。主分支应该始终是模块的功能性版本，尽管它不需要是一个准备好部署到生产环境的版本。将主分支视为稳定代码，可以让非生产环境可靠地指向所有仓库的主分支，以在开发过程中获取最新的接受代码。在部署到生产环境时，我们将使用 Git 标签来创建一个版本，比如 1.2.0。然后，我们可以将最新的代码部署到非生产环境，并正式将代码接受到生产环境中。

# 任务

在基于 Code Manager 或 r10k 的系统中，工作流的主要驱动因素是 Git 工作流。Git 工作流有多种模型，比如 GitHub flow 和 Git flow，但本书的主要重点不是 Git，因此我们将从一组最基本的命令和流程开始。最有效的开始方式是使用我们的控制仓库提供的临时环境。在这个工作流中，我们假设已经在现场实现了 Git 解决方案，或者由托管服务提供商提供，并且 Puppet Master 正在使用 Code Manager 部署环境。

工作流的第一步是识别需要更改的组件。在这个工作流示例中，我们假设正在对一个组件模块和嵌入在控制仓库中的配置文件进行更改。在手动测试阶段，我们会包含修复步骤，包含新的代码部署和新的推送到 Git 仓库。

# 克隆并编辑组件仓库

首先，我们将克隆组件模块，切换到新特性分支，并对仓库中的文件进行编辑。我们将确保在开发过程中使用 Git 分支，这样可以将我们的代码推送到上游 Git 仓库，而不影响原始代码。我们将以在现有模块的单独分支上创建新的代码快照来结束这一步，以便我们可以独立地测试这段代码。这一系列步骤是以下操作的通用工作流程：

1.  为单个模块制作上游仓库的副本（`git clone`/`pull`）

1.  创建一个与主分支分开的模块分支（`git checkout`）

1.  对代码进行任何和所有编辑（选择的 IDE）

1.  创建代码当前状态的快照（`git add` 和 `commit`）

1.  将快照推送回上游仓库（`git push`）

实际操作中的代码如下：

```
# Clone the remote git repository for the module. You can skip this step if the
# repository is already present on your local system
git clone git@gitserver.com:puppet/module.git

# If the repository is already local on the system, we'll just want to update our
# local master branch
git pull origin master

# Check out a new environment based on the existing master branch, which is the
# default branch of a git repository, and the branch we should start on on a clone.
git checkout -b new_feature

# We'll edit some files to add new features

# Adding new paramters to init
vim manifests/init.pp - Adding new parameters to init
# Adding a new feature to config
vim manifests/config.pp
# Ensuring the new feature is represented in the deployed template
vim templates/file.epp

# Add all edited files to git staging, assuming you're at the base of the repository
git add .

# Add an atomic commit, not only describing what the commit is, but why it was done
git commit -m 'Added new code to support feature requested by client'

# Push this code back to the origin repository as the new branch
git push origin new_feature
```

我们的编辑现在已经在上游仓库的`new_feature`分支中。主分支将继续作为其他人进一步开发的参考点，并用于在暂存环境中的测试。为了开始测试这段代码，我们将创建一个新的 Puppet 环境，专门用于测试和对这段代码进行迭代。

# 克隆控制仓库

第一步和上一步一样：克隆 Git 仓库。关于 Puppet 环境需要记住的一点是，这个仓库的分支对应一个 Puppet 环境。大多数 Puppet 用户没有主环境，而是使用 Puppet 默认放置节点的生产环境。如果你的组织有任何生产环境之前的环境（许多组织都有），你需要确保在创建新分支之前，先从现有分支开始。`git checkout -b`命令会创建一个新分支，起始于你当前所在的分支。以下是创建新环境的步骤，模仿现有环境：

1.  从上游仓库克隆控制仓库的副本（`git clone`）。

1.  检出你希望基于其编写新代码的环境（`git checkout`）。

1.  检出一个新分支，基于当前分支（`git checkout -b`）：

```
# This step is not needed if the repository is already on the local file system
git clone git@gitserver.com:puppet/control-repo.git

# We'll assume integration is the pre-production branch used by the organization
# to stage changes before moving into production-like branches
# Remember, there usually is no master branch in a control repository, so we want
# to target a specific branch to work against.
git checkout integration

# If this repo has been freshly cloned, git pull shouldn't provide any new updates,
# but it's safe to run either way. If the repository has already been cloned in the
# past, you definitely want to run this command to pull the latest commits from 
# upstream.
git pull origin integration

# We'll perform a second checkout, with the -b flag to indicate a new branch based on the existing branch
git checkout -b new_feature

```

就像我们为组件模块仓库所采取的步骤一样，这一系列命令确保我们拥有最新的提交的集成分支的本地仓库副本，并且我们基于现有代码启动了一个新分支。我们已经准备好直接在控制仓库中编辑文件，例如 `Puppetfile`、`hieradata` 以及嵌入的 `roles` 和 `profiles`（如果你将它们保存在控制仓库中，而不是作为独立的仓库）。一旦我们拿到代码，我们将编辑相关文件，创建新提交，将代码推送回源仓库，并部署环境。

# 编辑控制仓库

一旦我们进入目标环境的本地副本，就可以开始对代码进行更改了。我们通常会创建这些额外的短期环境，以便通过简单的命令部署新代码。我们有一些文件需要关注，因为我们将控制仓库视为整个环境的配置文件。`Puppetfile` 用于管理依赖项，包括任何组件模块（来自 Forge 或你自己的环境）。`roles` 和 `profiles` 通常也保存在控制仓库中，代码可以直接在这些环境中进行编辑。对控制仓库进行更改的工作流程如下：

1.  编辑文件（使用你选择的 IDE）。

1.  创建当前代码状态的快照（`git add` 和 `commit`）。

1.  将环境推送回远程仓库（`git push`）：

```
# Edit our files

# Change the branch of the component module to new_feature
vim Puppetfile

mod 'module',
 git => git@gitserver.com:puppet/module.git,
 branch => 'new_feature'

# Make a change in the profile that utilizes the component modules
vim site/profiles/manifests/baseline.pp

# Add our new changes, to be staged for a commit
git add .

# Commit our changes
git commit -m 'Supporting new Feature to support <effort>'
```

```

# Push our code back to the control repository as a new branch intended to be
# realized as a new environment on the Puppet Master
git push origin new_feature
```

此时，我们已经编辑了控制仓库中的模块和文件并推送回源仓库。接下来，我们将部署前面代码中创建的分支，并调整我们的配置以使用模块更改。除非你已经设置了 Git 钩子或 CI/CD 解决方案，否则你还需要在 Puppet Master 上触发环境部署。

# 在 Puppet Master 上部署新环境

Puppet 提供了 PE 客户端工具，如 第五章 *管理代码* 中所述，专门用于部署代码。如果这些工具在你的工作站上不可用，你也可以登录到 Puppet Master，那里已经可以使用它们。假设你正在使用 Code Manager，无论是在本地工作站还是远程服务器上，接下来的步骤都是相同的。

1.  从 Puppet Enterprise 获取登录令牌（`puppet-access login`）。

1.  从上游仓库分支部署环境（`puppet-code deploy`）：

```
# If PE Client Tools are not installed locally, the Puppet Master comes with them
# installed by default. We'll assume that the PE client tools are not already
# installed and log in to the Puppet Master
ssh user@puppet.org.net

# Generate an authorization token to allow your PE Console user to deploy code
puppet-access login

# Use our access token to deploy our new environment. Notice the -w flag, which
# triggers the client tools to wait and give you a pass or fail message on the
# status of the deployment.
puppet-code deploy new_feature -w
```

现在我们的代码已作为一个全新的环境部署在 Puppet Master 上。我们仍然缺少一个步骤，就是对我们的测试系统进行分类并确保它被放置在正确的环境中。对于 Puppet Enterprise 用户，你可以使用 PE 控制台中的节点分类器组来同时分类和声明一个环境。要创建一个新的节点组，选择一个环境，勾选环境组框，命名并点击创建。进入你的新环境组，将测试节点固定到该组，并向分类页面添加任何相关的类。

你也可以通过 `manifests/site.pp` 在控制库中进行分类，方法如下：

```
node 'test.node' {
  include relevant_role_or_profile
  include new_feature
}
```

通过 Hiera 进行分类的代码如下：

```
# data/host/test.node.yaml
---
classification:
  - relevant_role_or_profile
  - new_feature

# manifests/site.pp

# Notice the lack of a node group around the include statement
include $::classification
```

Puppet 用户常用的分类方法有多种，但没有自动化测试的话，我们需要做一些分类并运行代理来检查测试结果。

# 测试更改

在你的测试节点正确连接到环境组后，你可以登录到节点并触发代理运行，使用 puppet `agent -t`。或者，你也可以通过 PE 控制台运行 Puppet 代理并查看日志。如果没有看到任何变化，可能有以下几种原因：

+   代理已经运行，在你在控制台中对节点进行分类和运行 Puppet 代理之间。

+   漏掉了一个步骤，代码未正确部署。

+   你的代码没有在系统上触发任何新的更改，你应该修改系统以查看 Puppet 是否会修正该更改。

确保检查你的更改所针对的资源，查看代理是否已经部署了新的更改。你可能还需要验证代码部署是否已正确完成，并确保你已将代码推送回 Git 库。如果你的代码没有触发系统上的任何更改，或者触发了不期望的更改，你可以执行以下更短的工作流程，直到代码正确解决：

1.  在目标库中编辑代码：控制库或模块库（使用你选择的 IDE）

1.  创建代码的快照（`git add` 和 `commit`）

1.  将代码推送回远程库（`git push`）

1.  重新部署环境（`puppet-access login` 和 `puppet-code deploy`）

1.  在测试机器上触发代理运行（`puppet agent` 或 PE 控制台）

1.  检查目标系统上的变化

1.  重复直到达到预期状态：

```
# Start in the repository with the change. This could be a component module
# or the control repository. We're assuming each repository is still on the
# branch from the last step, and no pulls or branch changes are necessary.

# Edit the file with the targeted changes
vim manifests/manifest.pp

# Add the file to the git staging area
git add manifests/manifest.pp

# Commit the file to the repository
git commit -m 'Fixing specific bug'

# Push the repository back to upstream origin
git push origin new_feature

# From the Puppet Master, or a workstation with PE Client Tools

# Log in with RBAC
puppet-access login

# Deploy the environment
puppet-code deploy new_feature -w

# On the test node

# Run the agent, observe the results
puppet agent -t

# Repeat as necessary until issues are solved
```

一旦我们的代码达到预期状态，我们就可以开始将它放回 Puppet Master 上的长期环境中。模块应该将其代码合并回主分支，并且对控制库的更改需要与长期存在的分支合并。

# 合并分支

在之前的步骤中，我们将工作代码隔离到了一个功能分支和一个短期的、非生产环境中。虽然团队和组织应当选择一些合并的保护措施和策略，例如同行代码审查和自动化测试，但本节将重点介绍将分支合并到 master 或长生命周期分支的步骤。企业和开源的基于 Web 的 Git 解决方案通常包含一些额外的控制，来指示谁可以合并到仓库，以及合并到哪些分支。一般的最佳实践是允许进行同行代码审查，审查者可以将代码接受到长生命周期分支或 master 分支中。通过命令行合并代码是一个简单的过程，步骤如下：

1.  切换到你想要合并的分支（`git checkout`）

1.  将另一个分支合并到当前分支（`git merge`）

1.  将合并后的分支推送到上游仓库（`git push`）：

```
# Many Enterprise-focused git repositories have built in merge features, that ar
# likely more robust and easier to use than a simple git merge. If you have an in
# house git solution, follow the program documentation on a merge request

# On Module
# We'll change to target branch, in this case master
$ git checkout master

$ git merge feature_branch
Updating 0b3d899..227a02e
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```

```

# Push the branch to upstream repository so Puppet can find it.
$ git push origin master
```

在控制仓库中进行合并有时会遇到麻烦，因为 `Puppetfile` 在不同版本间（有意）不同。我们的 `生产型` 分支应该使用 Git 标签来声明要部署的代码的目标版本，并将其推广到各个环境中。我们的 `非生产型` 环境通常指向每个模块的 master 分支，提供最新的稳定代码供环境测试和开发。合并的方式与组件模块相同；只需确保不要将 `生产型` 分支上的 `Puppetfile` 覆盖为 `非生产型` 分支中的、控制较少的 `Puppetfile`。生产分支应参考 Git 标签来部署代码。

# Git 标签与版本控制

Git 标签用于创建代码的一个永久状态，并将其与现有分支区分开。标签不是用来迭代的，而是作为代码状态的时间标记。这使得标签非常适合用于 Puppet 代码的发布版本控制。我们可以从任何分支创建标签，但 master 是最常用的创建发布标签的分支。我们可以简单地在模块仓库中使用 `git tag` 命令来创建一个带有语义版本号的快照，并将其推送到 origin 仓库，供 r10k 或 Code Manager 调用。Git 标签的工作流也很简洁，步骤如下：

1.  检出目标分支（通常是 master）以便打标签（`git checkout`）

1.  对代码进行版本控制（`git tag`）

1.  推送标签到远程仓库（`git push`）：

```
rary at Ryans-MBP in ~/workspace/packt/module (master)
$ git tag 'v1.4'

$ git tag -l
v1.4

$ git push origin v1.4
```

在我们的模块成功版本控制后，我们可以编辑 `生产型` 的 Puppetfile 来使用我们的标签，而不是指向某个特定的开发分支或 master 分支：

```
# Production-like branch, tagged with a solid version number
forge https://forge.puppetlabs.com

mod 'module',
 git => 'git@gitserver.com:puppet/module.git',
 tag => 'v1.4'
```

这是 Puppet 工作流的简化版本，但仍有改进的空间。Puppet 最近发布了一款名为 PDK 的工具，旨在帮助将高质量的 Puppet 工具整合进你的工作流中。

# 使用 PDK

一个好的工作流程应当提供易用性、快速反馈、便捷的上手过程和质量控制。PDK 旨在提高这一领域的生产力。PDK 中的许多工具已经存在了一段时间，但它们通常很难使用且配置复杂，尤其是在工作站开发环境下。

# PDK

Puppet 在其官网上免费提供 PDK，并且为每个主要操作系统提供了相应的版本。它使用完全隔离的环境提供 Puppet 二进制文件和 RubyGems，这使得开发变得更加简便。PDK 中包含的工具，从版本 1.5.0 开始，如下所示：

+   创建新的 Puppet 工件：

    +   模块

    +   类

    +   已定义类型

    +   任务

    +   Puppet Ruby 提供程序

+   PDK 验证——简单的健康检查：

    +   Puppet 解析器验证（Puppet 语法）

    +   Puppet 风格检查（Puppet 风格）

    +   Puppet 元数据语法

    +   Puppet 元数据样式

    +   RuboCop（Ruby 风格）

+   PDK 单元测试（Puppet RSpec—单元测试）

# 创建新的 Puppet 工件

PDK 允许用户使用最佳实践创建新的工件。每个 `pdk new` 命令都会构建一个已经按照 Puppet 结构化的工件。这些工件旨在符合 Puppet 的最佳实践。如果你是第一次在隔离的环境中测试 PDK，从新模块开始是最简单的方法。

# pdk new 命令

命令`pdk new module`会引导用户进入一个提示，要求用户指定 Puppet Forge 用户名、作者全名、模块许可证以及支持的操作系统。如果你没有 Forge 用户名或模块许可证，可以随便输入任何值。输入提示后，你会找到一个新目录，其中包含代码。如果你希望将这些代码推送到上游仓库，可以在命令行中按以下步骤操作：

```
# From directory pdk new module was run in, enter the module, create a
# git repository and add all files to staging
$ cd module
$ git init
$ git add .

# Initial Commit is a good common message as a starting point
$ git commit -m 'Initial Commit'

# Add the upstream remote
$ git remote add origin git@gitserver.com:puppet/module.git

# Push to master and begin regular module development workflow
$ git push origin master
```

如果你正在使用一个之前创建的模块，可以使用 `pdk convert` 命令将模板中缺失的任何项加入到现有模块中。默认情况下，PDK 部署位于 [`github.com/puppetlabs/pdk-templates`](https://github.com/puppetlabs/pdk-templates) 的模板。如果你需要修改这里找到的任何文件，可以从官方仓库克隆 `pdk-templates` 的副本，并将其推送到一个中心 Git 仓库。你需要使用 `pdk convert --template-url <https>` 来选择新的模板并将其部署到现有模块中。`--template-url flag` 命令还将把新 URL 设置为工作站上的默认 URL。

你可以随意制作这个模板的副本，因为 Puppet 提供的模板相当全面，并且有较强的主观看法。它甚至包括一些启动 CI/CD 系统的方法，例如 `gitlab-ci`。你可以删除不使用系统的相关文件，并确保模板提供的内容对你的组织是有意义的。

模板仓库为 PDK 提供了三个目录和配置文件，如下所示：

+   `moduleroot`：此目录中的 Ruby 模板将覆盖现有文件。若您需要强制使用某个特定文件（例如 CI/CD 流水线文件），这非常有用。

+   `moduleroot_init`：此目录中的 Ruby 模板不会覆盖现有文件。这对于起始文件（如模块模板）非常适用。

+   `object_templates`：Ruby 模板，决定在执行诸如 `pdk new class` 等命令时文件的输出。

+   `config_defaults.yaml`：为 PDK 模板中的所有 Ruby 模板提供默认值和变量。

一旦获得新的模块模板，您可以开始在模块内创建 Puppet 代码清单。通过在新模块内使用 `pdk new class`，我们可以开始创建清单。该命令会按照自动加载布局创建清单，因此运行 `pdk new class server::main` 会在 `manifests/server/main.pp` 位置创建一个文件。使用默认模板创建的类将是一个空的、无参数的类，文件顶部会有 Puppet 字符串风格的文档注释。`pdk new defined_type` 命令会创建一个类似的文件，但会使用定义声明而不是类声明：

```
$ pdk new class config
pdk (INFO): Creating '/Users/rary/workspace/packt/module/manifests/config.pp' from template.
pdk (INFO): Creating '/Users/rary/workspace/packt/module/spec/classes/config_spec.rb' from template

# Sample with folders
$ pdk new class server::main
pdk (INFO): Creating '/Users/rary/workspace/packt/module/manifests/server/main.pp' from template.
pdk (INFO): Creating '/Users/rary/workspace/packt/module/spec/classes/server/main_spec.rb' from template.
```

`pdk new task` 命令将基于模板在 `tasks` 目录中创建文件，用于 Puppet 任务。Puppet 任务是通过 Puppet 自动化管理基础设施中临时脚本和命令的一种方式。`pdk new provider` 是用于设计新的自定义 Ruby 提供程序到 Puppet 的实验性功能。

一旦创建并开发了新的对象，PDK 还将提供一套工具集来进行语法和样式检查，使用 `pdk validate`。

# pdk validate 命令

PDK 提供了 `pdk validate` 来检查语法和样式。语法检查确保代码可以编译，并且确保清单文件或 JSON 元数据中没有遗漏逗号或闭括号等内容。语法检查还可以通过 `puppet parser validate` 手动执行。样式检查会检查代码是否符合标准的样式指南。Puppet-lint 用于提供 Puppet 的样式检查，所有规则可以在 [`puppet-lint.com/`](http://puppet-lint.com/) 查找。当模块处于健康状态时，PDK 将会对所有任务显示勾选标记：

```
$ pdk validate
pdk (INFO): Running all available validators...
pdk (INFO): Using Ruby 2.4.4
pdk (INFO): Using Puppet 5.5.1 ![] Checking metadata syntax (metadata.json tasks/*.json).
![] Checking module metadata style (metadata.json).
![] Checking task metadata style (tasks/*.json).
![] Checking Puppet manifest syntax (**/**.pp).
![] Checking Puppet manifest style (**/*.pp).
![] Checking Ruby code style (**/**.rb).

```

无效的 `metadata.json` 文件将阻止将模块上传到 Forge，并且无法运行 RSpec 测试。该文件详细列出了模块的作者信息以及其他信息，如依赖关系和支持的操作系统：

```
#Invalid Metadata.json

$ pdk validate
/opt/puppetlabs/pdk/private/ruby/2.4.4/lib/ruby/gems/2.4.0/gems/pdk-1.5.0/lib/pdk/module/metadata.rb:142:in `validate_name': Invalid 'name' field in metadata.json: Field must be a dash-separated user name and module name. (ArgumentError)
```

`pdk validate` 还会在模块中的每个清单文件上运行 Puppet 解析器验证。在以下示例中，`init.pp` 文件末尾忘记了一个大括号，PDK 提示我们该代码无法编译：

```
# Failed Parser Validation
# Can be ran alone with puppet parser validate

$ pdk validate
pdk (INFO): Running all available validators...
pdk (INFO): Using Ruby 2.4.4
pdk (INFO): Using Puppet 5.5.1
![] Checking metadata syntax (metadata.json tasks/*.json).
![] Checking module metadata style (metadata.json).
![] Checking Puppet manifest syntax (**/**.pp).
![] Checking Ruby code style (**/**.rb).
info: task-metadata-lint: ./: Target does not contain any files to validate (tasks/*.json).
Error: puppet-syntax: manifests/init.pp:9:1: Could not parse for environment production: Syntax error at '}'
```

如果 Puppet 解析器验证通过，`puppet-lint` 将在所有清单上运行。它将根据 Puppet 风格指南打印代码中的错误和警告。在以下示例中，我们对一个清单运行 pdk validate，清单中第 10 行有一行超出了 140 字符，并且第 9 行后有多余的空格：

```
$ pdk validate
pdk (INFO): Running all available validators...
pdk (INFO): Using Ruby 2.4.4
pdk (INFO): Using Puppet 5.5.1
![] Checking metadata syntax (metadata.json tasks/*.json).
![] Checking module metadata style (metadata.json). ![]Checking Puppet manifest syntax (**/**.pp).
![] Checking Puppet manifest style (**/*.pp).
![] Checking Ruby code style (**/**.rb).
info: task-metadata-lint: ./: Target does not contain any files to validate (tasks/*.json).
warning: puppet-lint: manifests/init.pp:10:140: line has more than 140 characters
error: puppet-lint: manifests/init.pp:9:28: trailing whitespace found
```

在某些情况下，我们希望禁用某个警告或错误，而不是打印出来。可以在 [`puppet-lint.com/checks/`](http://puppet-lint.com/checks/) 上找到检查列表，并可以用于禁用单个检查。在以下示例中，注意消息语句后的注释，告诉 lint 忽略 140 字符限制：

```
# A description of what this class does
#
# @summary A short summary of the purpose of this class
#
# @example
# include module
class module {

  notify {'String-trigger':
    message =>'This is the string that never ends. Yes it goes on and on my friends. Some developer just started writing without line breaks not knowing what they do, so this string will go on forever just because...' # lint:ignore:140chars
  }

}
```

如果我们希望在一个清单中忽略多个地方，可以通过将注释放在单独的一行并以 `# lint:endignore` 结束，使用 lint 块 `ignore`。在以下示例中，我们有两个大字符串，在 `puppet-lint` 检查时不会触发警告：

```
class module::strings {

# lint:ignore:140chars
  notify {'Long String A':
    message =>'This is the string that never ends. Yes it goes on and on my friends. Some developer just started writing without line breaks not knowing what they do, so this string will go on forever just because this is the string that never ends...'
  }

  notify {'Long String B':
    message =>'This is another string that never ends. Yes it goes on and on my friends. Some developer just started writing without line breaks not knowing what they do, so this string will go on forever just because this is the string that never ends...'
  }

# lint:endignore

}
```

如果你希望禁用某些检查，可以创建一个 `puppet-lint.rc` 文件。该文件可以放置在 `/etc` 目录下作为全局配置，也可以放置在主目录下的 `.puppet-lint.rc` 文件作为用户配置，或者放置在模块的根目录下作为 `.puppet-lint.rc`。如果你的团队使用本地开发工作站，可以考虑将 `.puppet-lint.rc` 添加到 PDK 模板中，以便在每个仓库中强制执行标准：

```
# Permanently ignore ALL 140 character checks
$ cat puppet-lint.rc
--no-140chars-check
```

最后，任何 Ruby 代码都将通过 RuboCop 进行验证。RuboCop 将检查模块中所有 Ruby 文件的样式。这为自定义 facts、types、providers 以及用 Ruby 编写的任务提供了样式检查：

```
$ pdk validate
pdk (INFO): Running all available validators...
pdk (INFO): Using Ruby 2.4.4
pdk (INFO): Using Puppet 5.5.1
![] Checking metadata syntax (metadata.json tasks/*.json).
![] Checking module metadata style (metadata.json).
![] Checking Puppet manifest syntax (**/**.pp).
![] Checking Puppet manifest style (**/*.pp).
![] Checking Ruby code style (**/**.rb).
info: task-metadata-lint: ./: Target does not contain any files to validate (tasks/*.json).
error: rubocop: spec/classes/config_spec.rb:8:38: unexpected token tRCURLY
(Using Ruby 2.1 parser; configure using `TargetRubyVersion` parameter, under `AllCops`)
```

`pdk validate` 提供了一个快速检查代码样式和语法的工具，但不会检查代码的功能。PDK 还提供了一个开箱即用的 RSpec 测试模板，因此，当使用 `pdk new class` 创建一个新类时，系统会自动创建一个简单的对应 RSpec 测试。

# pdk test unit 命令

使用 `pdk new class` 创建的新清单也会提供一个默认的 RSpec 测试。编写单元测试以确保清单在运行时执行预期的操作。Puppet 提供的默认单元测试确保代码在 `metadata.json` 中列出的每个操作系统上都能成功编译，并使用这些操作系统的默认 facts。这可以扩展以创建更强大的单元测试。在以下示例中，已添加一个检查，表示模块的 `init.pp` 应提供一个名为 `/etc/example` 的文件，而该文件在清单中并未提供：

```
$ pdk test unit
pdk (INFO): Using Ruby 2.4.4
pdk (INFO): Using Puppet 5.5.1
![] Preparing to run the unit tests.
![] Running unit tests.
 Evaluated 45 tests in 2.461011 seconds: 9 failures, 0 pending.
![] Cleaning up after running unit tests.
failed: rspec: ./spec/classes/module_spec.rb:9: expected that the catalogue would contain File[test]
 module on centos-7-x86_64 should contain File[test]
 Failure/Error:

 it { is_expected.to compile }
 it { is_expected.to contain_file('/etc/example') }
 end
 end
```

默认的 PDK 提供的简单测试仅为每个模块提供 `it { is_expected.to compile }` 作为 RSpec 测试。在下一章中，我们将扩展初始的 RSpec 模块，涵盖单元测试并为 Puppet 模块提供一些基本的代码覆盖率测试。

# 摘要

本章开始时，我们详细描述了什么构成一个良好的工作流程。将工作流程与持续集成和持续交付策略结合后，许多工作变得更加容易，这些内容将在下一章中介绍。我们将扩展由 Puppet PDK 构建的 RSpec 测试，并讨论验收测试策略。我们还将介绍一些新的工作流程和工具，以便在开发 Puppet 代码和清单时提供更及时的反馈。
