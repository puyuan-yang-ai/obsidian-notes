请按以下流程处理当前仓库的改动：

1. 运行 git status 和 git diff，总结改动点
2. 生成符合 Conventional Commits 的 commit message（简洁说明“为什么”）
3. 将相关文件 staged（优先精确 add，不要无脑 add .）
4. 执行 git commit
5. 执行 git pull --rebase（如有冲突，提示我并给出解决步骤，不要擅自大改）
6. push 到当前分支对应的远程（origin）
7. 最后输出：commit message、rebase 是否成功、push 的分支名
