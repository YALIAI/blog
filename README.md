# 自动驾驶笔记

Yang Xie 的自动驾驶学习博客。静态 HTML / CSS / JavaScript，可直接由 GitHub Pages 托管，无需构建工具。

## 本地预览

```bash
python3 -m http.server 8000
```

打开 `http://localhost:8000/`。

## 更新内容

- 在 根目录增加 HTML 文章，并在 `index.html` 的“最近的笔记”中加上入口。
- 在 `site.js` 的 `resources` 数组中增加资料。`category` 可选 `datasets`、`perception`、`simulation`、`courses`。
- 修改 `style.css` 调整样式。

## 发布

仓库 Settings → Pages → Build and deployment → Deploy from a branch → `main` / `(root)`。静态文件采用相对路径，也支持 `https://<用户名>.github.io/<仓库名>/` 这样的项目站点地址。

资料介绍是个人学习笔记；模型配置与数据集许可请以链接到的官方页面为准。
