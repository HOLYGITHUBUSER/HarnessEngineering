# Git 与产物

- commit 或 push 前先看 `git status --short --branch`。
- 除非工作流明确要求，否则只有用户要求时才 commit。
- commit 前检查 staged changes，避免误带密钥或无关文件。
- 遵守仓库的产物策略；如果项目要求发布产物入库，就要明确加入。
- 不需要 review 但要保留的生成文件，放到约定产物目录或 `backup/`。
- push 后确认本地分支已经和远端同步。
