# 提交到 GNSW/GTLCore

建议标题：`fix: 修复标准超频在多安培输入下的次数预算不足`

本流程准备的是向原仓库提交源码 Pull Request。本文没有创建 Fork、推送分支或发布 PR。

已于 2026-10-31 核对：默认分支为 `gtl-1431-skyblock`，当前提交为 `ae0659e449e70cf24aacbb29c254a5476b47cc70`。当前源码树未发现 CONTRIBUTING、PR 模板或 .github 工作流文件；构建文件配置了 Spotless 及 Mixin 打包检查。

## 1. Fork

登录自己的 GitHub 账号，打开 https://github.com/GNSW/GTLCore/fork ，创建自己的 Fork。

如果已有该仓库的 Fork，使用并更新已有 Fork。后续 clone 地址中的用户名应为自己的 GitHub 用户名。

## 2. 克隆并建立修改分支

在 PowerShell 执行以下命令，将首行用户名替换为自己的值。目标目录需尚未存在；若已克隆，进入已有克隆即可。

```powershell
$githubUser = '你的GitHub用户名'
git clone "https://github.com/$githubUser/GTLCore.git" 'D:\desktop\gtl-ucfe-addon\GTLCore-contribution'
Set-Location -LiteralPath 'D:\desktop\gtl-ucfe-addon\GTLCore-contribution'
git remote add upstream https://github.com/GNSW/GTLCore.git
git fetch upstream
git switch -c fix/standard-overclock-budget upstream/gtl-1431-skyblock
```

从当前开发分支建立修改分支，不从旧发布标签直接开 PR。

## 3. 应用提交用补丁

```powershell
$patchPath = 'C:\Users\yo\AppData\Roaming\.minecraft\versions\GregTech Leisure-1.4.5.0 with add3.1\reference\aggregation-investigation\overclock-fix\gtlcore-upstream-standard-overclock.patch'
git apply --check $patchPath
git apply $patchPath
```

使用这个新补丁：它在之前的源码修补基础上，额外把独立回归检查接入了 Gradle `check`。如果未来上游变更导致 `--check` 失败，先适配冲突，不要强制覆盖整个 Mixin JSON 或 build.gradle。

## 4. 格式化、构建与验证

使用 JDK 17，在克隆仓库根目录执行：

```powershell
java -version
.\gradlew.bat spotlessApply
.\gradlew.bat build
git diff --check
git status --short
git diff
```

`build` 会经过 `check`；补丁新增的 `standardOverclockBudgetTest` 会运行数值回归检查。工程依赖需要正常下载，必要时按错误信息修复构建环境。

构建通过后，在独立测试实例中验证启动无 Mixin 注入错误，并复现：2 并行龙芯、8A MAX 输入，预期耗电约 16.11G EU/t，7.5A MAX，12 tick，每批 4 龙芯。补充特殊超频机器和亚 tick 的回归。

此前已完成 Java 17 目标编译、独立数值检查、字节码静态核对及补丁适用检查；尚未完成完整工程构建或游戏实测。上述步骤是提交前仍需执行的检查，不是已完成声明。

## 5. 提交源码并推送到自己的 Fork

只暂存以下五个文件，核对后提交：

```powershell
git add src/main/java/org/gtlcore/gtlcore/mixin/gtm/api/recipe/RecipeHelperOverclockMixin.java src/main/java/org/gtlcore/gtlcore/utils/StandardOverclockBudget.java src/main/resources/gtlcore.mixin.json src/test/java/org/gtlcore/gtlcore/utils/StandardOverclockBudgetTest.java build.gradle
git diff --cached --stat
git diff --cached
git commit -m "fix: account for actual power in standard overclock budgets"
git push -u origin fix/standard-overclock-budget
```

如本机未配置提交身份，先设置自己的 Git 作者信息；GitHub 推送身份与 Git 提交作者配置是两件事。不要将编译缓存、build 产物、整合包或本地验证日志纳入 PR。

## 6. 在 GitHub 创建 PR

打开自己的 Fork，点击 `Compare & pull request`。也可从原仓库的 Pull requests 页面点击 New pull request，再选择 compare across forks。

- Base repository：`GNSW/GTLCore`
- Base branch：`gtl-1431-skyblock`
- Head repository：自己的 Fork
- Compare branch：`fix/standard-overclock-budget`
- Title：`fix: 修复标准超频在多安培输入下的次数预算不足`
- Body：复制同目录 `PR_BODY.md`，按真实执行结果更新验证部分。

若仍未完成完整构建和实机验证，可先创建 Draft PR，并明确说明待验证项目；验证完成后更新正文并标记 Ready for review。无需为了形式先开 Issue，除非维护者随后要求或你想先讨论该行为是否属于预期规则。

维护者要求修改时，在同一分支修改、提交并 push，原 PR 会自动更新。不要另开一个重复 PR。

参考：

- 原仓库：https://github.com/GNSW/GTLCore
- GitHub 官方贡献流程：https://docs.github.com/en/get-started/exploring-projects-on-github/contributing-to-a-project
