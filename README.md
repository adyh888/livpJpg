# LIVP 图片提取工具

一个基于 uni-app 开发的 LIVP 文件图片提取工具，支持将 iPhone 拍摄的 LIVP 文件中的图片（包括 HEIC 格式）提取并转换为 JPG 格式，支持批量处理和下载。

## 功能特点

- 支持选择多个 LIVP 文件进行批量处理
- 自动提取 LIVP 文件中的 JPG 和 HEIC 图片
- 将 HEIC 格式图片自动转换为 JPG 格式
- 提供图片预览功能
- 支持单张图片下载和批量打包下载

## 技术栈

- 框架：uni-app（支持 Vue 3）
- 依赖库：
  - `jszip`：用于解析 LIVP 文件（本质是 ZIP 压缩包）
  - `heic-to`：用于将 HEIC 格式图片转换为 JPG

## 使用方法

1. 点击"点击选择 .livp 文件"区域
2. 选择一个或多个 LIVP 文件
3. 等待文件解析和转换完成
4. 预览提取出的图片
5. 可选择单张下载或点击"下载全部图片"打包下载

## 项目结构

```
livpJpg/
├── pages/
│   └── index/
│       └── index.vue    # 主页面，包含所有功能实现
├── App.vue              # 应用入口组件
├── main.js              # 应用入口文件
├── manifest.json        # 应用配置文件
├── pages.json           # 页面路由配置
├── package.json         # 项目依赖配置
└── index.html           # HTML 入口文件
```

## 安装与运行

1. 克隆仓库

```bash
git clone https://github.com/adyh888/livpJpg.git
```

2. 安装依赖

```bash
npm install
```

3. 运行项目（根据 uni-app 开发工具选择对应命令）

```bash
# 例如使用 HBuilderX 运行到浏览器
```

## 注意事项

- 处理大量或大型 LIVP 文件时可能需要较长时间，请耐心等待
- HEIC 转换功能依赖浏览器环境支持，部分旧浏览器可能存在兼容性问题
- 批量下载功能会将所有图片打包为 ZIP 文件

## 许可证
