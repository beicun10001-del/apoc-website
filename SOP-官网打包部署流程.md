# Apoc 官网打包部署 SOP

## 文档信息

| 项目 | 内容 |
|------|------|
| **版本** | v1.0 |
| **更新日期** | 2026-04-09 |
| **适用版本** | Apoc 官网 v5.4.3+ |
| **托管平台** | Vercel |
| **域名** | https://www.apocai.top |

---

## 目录

1. [环境准备](#1-环境准备)
2. [项目结构](#2-项目结构)
3. [资源文件说明](#3-资源文件说明)
4. [完整部署流程](#4-完整部署流程)
5. [版本更新流程](#5-版本更新流程)
6. [Logo 更新流程](#6-logo-更新流程)
7. [GitHub 推送与自动部署](#7-github-推送与自动部署)
8. [故障排除](#8-故障排除)

---

## 1. 环境准备

### 1.1 必需工具

| 工具 | 用途 | 下载地址 |
|------|------|----------|
| Git | 版本控制 | https://git-scm.com/download/win |
| Python（可选） | 图片处理 | https://www.python.org/downloads/ |

### 1.2 Python 图片处理库

```bash
# 安装 Pillow（用于图片格式转换）
pip install Pillow
```

### 1.3 Vercel 账号

- 需要连接 GitHub 仓库：`beicun10001-del/express-js-on-vercel`
- 首次部署后，后续推送将自动触发部署

---

## 2. 项目结构

### 2.1 目录结构

```
C:\Users\北北\Desktop\website\
│
├── index.html              # ⭐ 网站首页（包含下载链接）
├── logo.png                # ⭐ 网站 Logo（1024x1024 RGB PNG）
├── vercel.json             # Vercel 配置文件
│
├── downloads/              # 下载资源目录（可选）
│   └── ...                 # 可放置备用下载文件
│
└── releases/               # 历史版本目录（可选）
    └── ...                 # 可放置历史版本文件
```

### 2.2 关键文件说明

| 文件 | 必须 | 说明 |
|------|------|------|
| `index.html` | ✅ | 网站首页，包含下载链接 |
| `logo.png` | ✅ | 网站 Logo |
| `vercel.json` | ✅ | Vercel 配置（cleanUrls + rewrite） |

---

## 3. 资源文件说明

### 3.1 Logo 要求

| 属性 | 要求 |
|------|------|
| 文件名 | `logo.png` |
| 尺寸 | **1024x1024 像素** |
| 格式 | PNG |
| 颜色模式 | **RGB**（非 RGBA，不能有透明通道） |
| 用途 | 网站导航栏 Logo、Hero 区域大 Logo |

### 3.2 Logo 尺寸转换

如果源图片尺寸不是 1024x1024，需要处理：

```python
# convert_logo.py
from PIL import Image

def convert_logo(source_path: str, output_path: str):
    """转换 logo 为网站所需格式（1024x1024 RGB）"""
    img = Image.open(source_path)

    # 转换为 RGB（去除透明通道）
    if img.mode == 'RGBA':
        # 创建白色背景
        background = Image.new('RGB', img.size, (255, 255, 255))
        background.paste(img, mask=img.split()[3])  # 使用 alpha 作为 mask
        img = background
    elif img.mode != 'RGB':
        img = img.convert('RGB')

    # 调整尺寸为 1024x1024
    img = img.resize((1024, 1024), Image.LANCZOS)

    # 保存
    img.save(output_path, 'PNG', quality=95)
    print(f"[OK] Logo saved: {output_path} (1024x1024 RGB)")

if __name__ == '__main__':
    # 示例：从桌面图片生成网站 logo
    source = r"C:\Users\北北\Desktop\b11d2904c60993a952e32181adf8e7bd.jpg"
    output = r"C:\Users\北北\Desktop\website\logo.png"
    convert_logo(source, output)
```

### 3.3 index.html 下载链接

下载链接位于 `index.html` 第 337 行：

```html
<!-- Windows 下载按钮 -->
<a href="https://github.com/beicun10001-del/Apoc-win/releases/download/v5.4.3/Apoc-Setup-5.4.3.exe"
   class="btn btn-primary" download>
    Windows 下载 (v5.4.3)
</a>

<!-- macOS 下载按钮 -->
<a href="https://harvey-apoc.oss-cn-shenzhen.aliyuncs.com/Apoc.AI.Assistant-5.2.6.dmg"
   class="btn" download>
    macOS 下载 (v5.2.10)
</a>
```

---

## 4. 完整部署流程

### 4.1 流程概览

```
┌─────────────────────────────────────────────────────────┐
│                  官网部署流程                            │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1️⃣  更新 index.html                                   │
│      └── 更新下载链接版本号                              │
│                                                         │
│  2️⃣  更新 logo.png（如需要）                           │
│      ├── 确保 1024x1024 RGB 格式                        │
│      └── 替换 website/logo.png                          │
│                                                         │
│  3️⃣  Git 提交                                          │
│      ├── git add .                                      │
│      ├── git commit -m "v5.4.3: 更新下载链接"           │
│      └── git push origin master                         │
│                                                         │
│  4️⃣  Vercel 自动部署                                    │
│      └── 推送到 GitHub 后自动触发（约 1-2 分钟）          │
│                                                         │
│  5️⃣  验证网站                                          │
│      └── 访问 https://www.apocai.top                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 4.2 手动分步执行

#### 步骤 1：进入项目目录

```powershell
cd C:\Users\北北\Desktop\website
```

#### 步骤 2：更新下载链接

编辑 `index.html`，找到下载按钮，修改版本号：

```html
<!-- 第 337 行左右 -->
<a href="https://github.com/beicun10001-del/Apoc-win/releases/download/v5.4.3/Apoc-Setup-5.4.3.exe"
   class="btn btn-primary" download>
    Windows 下载 (v5.4.3)     <!-- 修改这里的版本号 -->
</a>
```

#### 步骤 3：更新 Logo（如需要）

```python
# Python 脚本处理
from PIL import Image

source = r"C:\Users\北北\Desktop\b11d2904c60993a952e32181adf8e7bd.jpg"
output = r"C:\Users\北北\Desktop\website\logo.png"

img = Image.open(source)
if img.mode == 'RGBA':
    background = Image.new('RGB', img.size, (255, 255, 255))
    background.paste(img, mask=img.split()[3])
    img = background
elif img.mode != 'RGB':
    img = img.convert('RGB')

img = img.resize((1024, 1024), Image.LANCZOS)
img.save(output, 'PNG')
print("[OK] Logo updated!")
```

#### 步骤 4：Git 提交并推送

```bash
# 进入目录
cd C:\Users\北北\Desktop\website

# 添加所有修改
git add .

# 提交（包含版本说明）
git commit -m "v5.4.3: 更新下载链接和Logo

- 更新 Windows 下载链接到 v5.4.3
- 更新 Logo 为新机器人头像样式
- 更新下载按钮显示版本号"

# 推送到 GitHub
git push origin master
```

#### 步骤 5：验证部署

```powershell
# 等待约 1-2 分钟让 Vercel 部署

# 检查网站是否更新
curl -I https://www.apocai.top

# 或者浏览器访问
Start-Process "https://www.apocai.top"
```

---

## 5. 版本更新流程

### 5.1 每次发布新版本时的操作

```
假设新版本号为 v5.5.0，安装包名为 Apoc-Setup-5.5.0.exe
```

#### 5.1.1 先完成 Apoc 安装包发布

按照 `SOP-桌面应用打包流程.md` 完成：
1. PyInstaller 打包
2. Inno Setup 编译
3. 上传到 GitHub Release

#### 5.1.2 更新网站下载链接

编辑 `C:\Users\北北\Desktop\website\index.html`：

```html
<!-- 第 337 行 -->
<a href="https://github.com/beicun10001-del/Apoc-win/releases/download/v5.5.0/Apoc-Setup-5.5.0.exe"
   class="btn btn-primary" download>
    Windows 下载 (v5.5.0)     <!-- 更新版本号 -->
</a>
```

#### 5.1.3 提交网站更新

```bash
cd C:\Users\北北\Desktop\website

git add index.html
git commit -m "v5.5.0: 更新下载链接"
git push origin master
```

#### 5.1.4 验证

约 1-2 分钟后访问 https://www.apocai.top 确认下载链接生效。

---

## 6. Logo 更新流程

### 6.1 流程说明

当需要更换网站 Logo 时：

```
源图片（任意格式）
    ↓
转换为 1024x1024 RGB PNG
    ↓
替换 website/logo.png
    ↓
Git 提交推送
    ↓
Vercel 自动部署
```

### 6.2 完整命令

```python
#!/usr/bin/env python3
# update_website_logo.py
from PIL import Image
import shutil
import os

def update_logo(source_path: str, website_dir: str):
    """更新网站 Logo"""
    output_path = os.path.join(website_dir, "logo.png")

    print(f"Source: {source_path}")
    print(f"Output: {output_path}")

    # 打开源图片
    img = Image.open(source_path)
    print(f"Source size: {img.size}, mode: {img.mode}")

    # 转换为 RGB（去除透明通道）
    if img.mode == 'RGBA':
        print("Converting RGBA to RGB with white background...")
        background = Image.new('RGB', img.size, (255, 255, 255))
        background.paste(img, mask=img.split()[3])
        img = background
    elif img.mode != 'RGB':
        print(f"Converting {img.mode} to RGB...")
        img = img.convert('RGB')

    # 调整尺寸为 1024x1024
    print("Resizing to 1024x1024...")
    img = img.resize((1024, 1024), Image.LANCZOS)

    # 备份原文件
    if os.path.exists(output_path):
        shutil.copy2(output_path, output_path + ".bak")
        print(f"[OK] Backup: {output_path}.bak")

    # 保存
    img.save(output_path, 'PNG', quality=95)
    print(f"[SUCCESS] Logo updated: {output_path}")
    print(f"Final size: {img.size}, mode: {img.mode}")

if __name__ == '__main__':
    # 源图片（可以是任意尺寸和格式）
    source = r"C:\Users\北北\Desktop\b11d2904c60993a952e32181adf8e7bd.jpg"

    # 网站目录
    website = r"C:\Users\北北\Desktop\website"

    update_logo(source, website)
```

### 6.3 验证 Logo

```powershell
# 确认 logo 格式
python -c "from PIL import Image; img=Image.open(r'C:\Users\北北\Desktop\website\logo.png'); print(f'Size: {img.size}, Mode: {img.mode}')"

# 应该输出：Size: (1024, 1024), Mode: RGB
```

---

## 7. GitHub 推送与自动部署

### 7.1 仓库信息

| 属性 | 值 |
|------|------|
| 本地路径 | `C:\Users\北北\Desktop\website` |
| GitHub 仓库 | `beicun10001-del/express-js-on-vercel` |
| 托管平台 | Vercel |
| 网站地址 | https://www.apocai.top |

### 7.2 Git 操作

```bash
# 查看状态
cd C:\Users\北北\Desktop\website
git status

# 添加所有修改
git add .

# 提交
git commit -m "更新说明"

# 推送到 GitHub
git push origin master
```

### 7.3 Vercel 自动部署

推送到 GitHub 后，Vercel 会自动：
1. 拉取最新代码
2. 构建网站
3. 部署到 CDN

**预计时间**：1-2 分钟

### 7.4 手动触发重新部署

如果自动部署未触发：

1. 登录 Vercel 控制台：https://vercel.com
2. 选择项目 `express-js-on-vercel`
3. 点击 **Deployments** → **Redeploy**

---

## 8. 故障排除

### 8.1 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Logo 显示模糊 | 分辨率不足 | 确保源图片至少 1024x1024 |
| Logo 有白色边框 | 透明通道处理不当 | 使用 RGB 模式，白色背景 |
| 下载链接 404 | 未上传到 GitHub Release | 先上传安装包到 Release |
| 网站未更新 | Vercel 缓存 | 等待 2 分钟或手动 Redeploy |
| 部署失败 | Git 推送错误 | 检查 git status 和 git log |

### 8.2 验证下载链接

```powershell
# 检查 GitHub Release 是否存在
$url = "https://github.com/beicun10001-del/Apoc-win/releases/download/v5.4.3/Apoc-Setup-5.4.3.exe"
curl -I $url

# 正常响应应该包含：
# HTTP/2 302 (redirect) 或 HTTP/2 200
```

### 8.3 验证网站 Logo

```powershell
# 下载 logo 并检查
curl -o "$env:TEMP\logo_check.png" "https://www.apocai.top/logo.png"
python -c "from PIL import Image; img=Image.open(r'$env:TEMP\logo_check.png'); print(f'Size: {img.size}, Mode: {img.mode}')"
```

---

## 附录：完整命令清单

```bash
# ============================================
# Apoc 官网完整部署命令
# ============================================

# 1. 进入项目目录
cd C:\Users\北北\Desktop\website

# 2. 更新 Logo（如需要）
python convert_logo.py

# 3. 更新下载链接
# 编辑 index.html 中的版本号

# 4. Git 提交
git add .
git commit -m "v5.4.3: 更新下载链接和Logo"
git push origin master

# 5. 验证部署
# 等待 1-2 分钟后访问 https://www.apocai.top
```

---

## 附录：index.html 关键位置

| 位置 | 内容 | 行号 |
|------|------|------|
| 第 320 行 | 导航栏 Logo | `<img src="logo.png" alt="Apoc">` |
| 第 332 行 | Hero 区域大 Logo | `<img src="logo.png" alt="Apoc Logo" class="hero-logo">` |
| 第 337 行 | Windows 下载链接 | `href=".../v5.4.3/Apoc-Setup-5.4.3.exe"` |
| 第 345 行 | macOS 下载链接 | `href=".../Apoc.AI.Assistant-5.2.6.dmg"` |

---

## 附录：两个 SOP 的关联

```
┌─────────────────────────────────────────────────────────┐
│                     发布完整流程                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────────┐    ┌─────────────────────┐   │
│  │  SOP-桌面应用打包流程  │    │  SOP-官网打包部署流程  │   │
│  │  (Apoc 项目)        │    │  (website 项目)      │   │
│  └──────────┬──────────┘    └──────────┬──────────┘   │
│             │                          │               │
│             ↓                          ↓               │
│  ┌─────────────────────┐    ┌─────────────────────┐   │
│  │  生成安装包           │    │  上传到 GitHub      │   │
│  │  Apoc-Setup-X.X.X.exe│    │  Release           │   │
│  └──────────┬──────────┘    └──────────┬──────────┘   │
│             │                          │               │
│             └──────────┬─────────────┘               │
│                        ↓                               │
│             ┌─────────────────────┐                   │
│             │  更新网站下载链接     │                   │
│             │  index.html        │                   │
│             └──────────┬──────────┘                   │
│                        ↓                               │
│             ┌─────────────────────┐                   │
│             │  推送网站到 GitHub  │                   │
│             │  Vercel 自动部署   │                   │
│             └─────────────────────┘                   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

*文档版本：v1.0 | 更新日期：2026-04-09*
