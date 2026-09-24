# Maven + JFrog CLI GitHub Actions 示例

基于测试项目 [gyzong1/maven-example](https://github.com/gyzong1/maven-example)，用 JFrog CLI 完成：

1. **构建并上传**至 Artifactory
2. **搜集并发布** Build Info
3. **扫描** Build（Xray）

示例工作流文件：`.github/workflows/maven.yml`

## 使用方法

将 `maven.yml` 复制到目标仓库：

```text
.github/workflows/maven.yml
```

### Github 仓库配置


| 类型       | 名称                | 说明                                           |
| -------- | ----------------- | -------------------------------------------- |
| Variable | `JF_URL`          | JFrog Platform URL，如 `https://acme.jfrog.io` |
| Secret   | `JF_ACCESS_TOKEN` | 需具备 Deploy / Build Info / Xray Scan 权限       |


### Artifactory 仓库配置

流水线通过 `MAVEN_REPO_RESOLVE` / `MAVEN_REPO_DEPLOY` 指向 Maven 仓库（默认使用同一个 virtual）。请先在 Artifactory 中创建以下 maven 类型仓库：


| 类型      | 示例名称                         | 说明                                                               |
| ------- | ---------------------------- | ---------------------------------------------------------------- |
| Local   | `guoyz-github-maven-local`   | 存放本流水线部署的制品                                                      |
| Remote  | `guoyz-github-maven-remote`  | 代理中央仓库，URL 为 `https://repo1.maven.org/maven2/`                   |
| Virtual | `guoyz-github-maven-virtual` | 聚合上述 local + remote；**Default Deployment Repository** 指向对应 local |


说明：

流水线中使用了 build-scan, 需将相关仓库和 build 加入 **Xray Indexed Resources**，以便 `jf build-scan` 可扫描依赖与制品。

### 可调环境变量


| 变量                       | 默认值                        | 说明                        |
| ------------------------ | -------------------------- | ------------------------- |
| `JFROG_CLI_BUILD_NAME`   | `maven-example`            | Build 名称                  |
| `JFROG_CLI_BUILD_NUMBER` | `${{ github.run_number }}` | Build 编号                  |
| `MAVEN_REPO_RESOLVE`     | `maven-virtual`            | 解析仓库（与项目 `maven.conf` 一致） |
| `MAVEN_REPO_DEPLOY`      | `maven-virtual`            | 部署仓库                      |


## 核心 JFrog CLI 步骤


| 步骤            | 命令                                                |
| ------------- | ------------------------------------------------- |
| 配置 Maven 仓库   | `jf mvn-config`                                   |
| 构建并上传         | `jf mvn clean deploy --build-name/--build-number` |
| 搜集环境信息        | `jf rt build-collect-env`                         |
| 搜集 Git 信息     | `jf rt build-add-git`                             |
| 发布 Build Info | `jf rt build-publish`                             |
| 扫描 Build      | `jf build-scan`                                   |


## 参考链接

- [安装 JFrog CLI](https://docs.jfrog.com/integrations/docs/download-and-install-the-jfrog-cli)
- [JFrog CLI 快速开始](https://docs.jfrog.com/integrations/docs/jfrog-cli-quick-start)
- [JFrog CLI 文档总览](https://docs.jfrog.com/integrations/docs/jfrog-cli)
- [jf mvn 命令说明](https://docs.jfrog.com/artifactory/docs/jf-mvn)

