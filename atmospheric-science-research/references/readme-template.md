# README Template

Write one Chinese `README.md` once every figure is `PASS` or `BLOCKED`. Match the actual project layout; do not restructure the project to fit the template.

```markdown
# 项目说明

简要说明科学目标和主要工作内容。

## 1. 数据简述

- 输入数据路径：
- 数据来源与类型：
- 时间范围：
- 空间范围与分辨率：

## 2. 路径结构和实现功能

- `data_output/fig1/xxx.nc`：实现功能
- `fig1/compute_xxx.py`：实现功能
- `fig1/plot_xxx.py`：实现功能
- `fig1/fig1_xxx.png`、`fig1/fig1_xxx.pdf`：实现功能

## 3. 实现算法与细节

- 采用的假设与默认选择：逐条列出用户确认的选择和可逆的默认设置，算法简单时也要写。
- 算法如无需展开，可写：`本任务算法直接，省略。`
- 若有复杂处理，写清关键公式、统计方法、重网格方案和加速策略。

## 4. 可能存在的问题与风险

- 列出 `BLOCKED` 的图、原因和剩余问题，以及其他未解决的风险。
- 无则写：`目前无明显问题。`

## 5. 图片简述

- 以大气科学论文写作口径，专业、简洁地说明每张图的内容与意义。

## 6. 版本更新

- （更新时填写日期、变更内容、影响文件）
```

On later modification, including Patch Mode, append to「版本更新」and update only the affected path and figure sections.
