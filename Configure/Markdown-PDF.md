---
title: "Linux中的Markdown-PDF配置"
author: "xht03"
date: 2025-03-17 11:00:00
tags:
- notes
categories:
- Configuration
header-includes:
- \usepackage{xeCJK}
---

### 安装
```bash
sudo apt install pandoc textlive-xetex texlive-lang-chinese
```

### 使用
在 Markdown 文件中 yaml 头中添加如下配置：
```yaml
header-includes:
- \usepackage{xeCJK}
```

然后使用以下命令将 Markdown 文件转换为 PDF：
```bash
pandoc demo.md -o demo.pdf --pdf-engine=xelatex
```

如果报错，可以通过 `--verbose` 参数查看详细信息：
```bash
pandoc demo.md -o demo.pdf --pdf-engine=xelatex --verbose
```

### 常见报错

1. latex 公式必须在 `$...$` 或 `$$...$$` 中，且 `$` 之后不能紧跟空格或中文标点，`$`和公式之间不能有空格。