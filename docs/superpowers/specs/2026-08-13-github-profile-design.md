# GitHub 个人主页复现设计

## 目标

将当前目录中的 GitHub Profile README 项目复现到公开仓库 `yuanimperialhal/yuanimperialhal`，并使用用户给定的中文介绍替换模板内容。最终 GitHub 个人主页应显示居中的个人介绍，以及根据该账号贡献记录自动生成的贪吃蛇动画。

## 范围

本次只保留原项目的两项核心能力：

1. 在 `README.md` 顶部展示居中的中文个人介绍。
2. 通过 GitHub Actions 生成并展示贡献贪吃蛇动画。

不增加技术徽章、访问量统计、GitHub 数据卡片、社交媒体链接或其他主页模块。

## 文件设计

### `README.md`

使用用户提供的 HTML 和 Markdown 混排内容：

- 居中显示姓名、岗位方向、技术栈和关注领域。
- 在介绍下方展示贡献贪吃蛇动画。
- 动画地址指向当前 Profile 仓库的 `output` 分支，不再引用原模板账号 `seiry`。
- 同时引用亮色与暗色 SVG，使页面随 GitHub 主题切换。

### `.github/workflows/snake.yml`

保留单一工作流，负责：

- 在推送到 `main` 时运行。
- 每天定时运行一次。
- 支持在 GitHub Actions 页面手动运行。
- 使用仓库所有者 `yuanimperialhal` 的贡献数据生成亮色和暗色 SVG。
- 将生成文件发布到 `output` 分支。
- 只授予工作流写入仓库内容所需的 `contents: write` 权限。

## 数据流

1. `main` 分支保存 `README.md` 和工作流。
2. 工作流读取当前仓库所有者的 GitHub 贡献记录。
3. 生成的 SVG 文件写入 `output` 分支。
4. `README.md` 从 `output` 分支加载 SVG。
5. GitHub 将该 README 自动显示在 `yuanimperialhal` 的个人主页。

## 发布方式

- 在当前目录初始化 Git 仓库，默认分支为 `main`。
- 将远端 `origin` 设置为 `https://github.com/yuanimperialhal/yuanimperialhal.git`。
- 提交本次复现内容并推送到远端 `main`。
- 不使用强制推送；如果远端状态在发布前发生变化，则停止并先合并或请用户确认。

## 异常处理

- 如果本机缺少可用的 GitHub HTTPS 凭据，保留完整的本地提交，并让用户完成一次 GitHub 登录后再继续推送。
- 如果 GitHub Actions 因仓库权限设置无法写入 `output` 分支，检查仓库的 Workflow permissions 是否允许读写。
- 如果动画首次生成前暂时显示为空，等待第一次工作流完成后再复查，不能把暂时缺少 `output` 分支误判为 README 配置错误。

## 验证标准

发布前：

- `README.md` 中完整包含用户给定的中文介绍。
- README 不再包含 `seiry`，动画链接只指向 `yuanimperialhal/yuanimperialhal`。
- 工作流 YAML 可被解析，触发分支为 `main`，输出分支为 `output`。
- Git 工作区只包含本次范围内的文件。

发布后：

- `main` 分支可在远端读取。
- GitHub 个人主页显示中文介绍。
- GitHub Actions 成功生成 `output` 分支。
- 亮色和暗色贪吃蛇 SVG 均可访问并正常显示。
