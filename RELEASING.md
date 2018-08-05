发布
=========

 1. 检出 `master` 分支（`git checkout master`）
 2. 从上游拉取最新的修改（`git pull`）
 3. 在 `_changes/` 中为当前日期创建一个新的条目，并写入自上一版本以来的修改。最简单的方法就是复制以前的文件并进行更新。（没错，每个文件的那些双 `---` 行都是必须的）
 4. 提交（`git commit -am "Prepare release 20XX-YY-ZZ"`）
 5. 打标签（`git tag -a 20XX-YY-ZZ -m "20XX-YY-ZZ"`）
 6. 检出 `gh-pages` 分支（`git checkout gh-pages`）
    * 如果你本地上没有 `gh-pages` 分支，请运行 `git checkout -t origin/gh-pages`。
 7. 合并 `master` 分支（`git merge master`）
 8. 推送这两个分支（`git push --all`）
 9. 推送标签（`git push --tags`）
