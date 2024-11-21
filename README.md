# Image Generator

## 项目简介

Image Generator 是一个基于 Vue.js 的图片生成和编辑工具，允许用户上传、编辑和下载图片。该项目使用了 `vue-cropperjs` 进行图片裁剪，并支持从剪贴板粘贴、上传本地图片或输入远程图片地址。

## 项目结构

- `src/` - 源代码目录
  - `components/` - 包含主要的 Vue 组件
    - `ImageEditor.vue` - 图片编辑器组件
    - `ImageGenerator.vue` - 图片生成器组件
  - `App.vue` - 主应用组件
  - `main.js` - 应用入口文件
- `public/` - 公共资源目录
- `README.md` - 项目说明文件
- `package.json` - 项目依赖和脚本配置

## 项目运行

### 1. 安装依赖

在项目根目录下运行以下命令以安装项目依赖：

```bash
yarn install
```

### 2. 启动开发服务器

使用以下命令启动开发服务器，项目将在本地进行热重载：

```bash
yarn serve
```

打开浏览器并访问 `http://localhost:8080`，即可查看项目。

### 3. 构建生产版本

如果你想要构建项目的生产版本，可以使用以下命令：

```bash
yarn build
```

构建完成后，生成的文件将位于 `dist/` 目录中。

### 4. 代码检查和修复

使用以下命令对代码进行检查和修复：

```bash
yarn lint
```

## 部署项目

要将项目部署到远程服务器，您可以按照以下步骤操作：

1. **构建项目**：首先，确保您已构建项目的生产版本（如上所述）。
2. **上传构建文件**：将 `dist/` 目录中的所有文件上传到您的服务器。您可以使用 FTP、SCP 或其他文件传输工具。
3. **配置服务器**：确保您的服务器配置正确，以便能够提供静态文件。您可以使用 Nginx、Apache 或其他 Web 服务器。
4. **访问项目**：在浏览器中输入您的服务器地址，您应该能够看到项目运行。

## 依赖

该项目依赖以下库：

- `vue` - Vue.js 框架
- `vue-cropperjs` - 图片裁剪组件
- `view-design` - UI 组件库
- `cropperjs` - 图片裁剪库

## 许可证

该项目使用 MIT 许可证，详细信息请查看 LICENSE 文件。

---

如需更多信息或帮助，请查看 [Vue.js 文档](https://vuejs.org/) 或 [vue-cropperjs 文档](https://github.com/xyxiao001/vue-cropperjs)。
