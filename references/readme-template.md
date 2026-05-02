# README Template

Write one Chinese `README.md` after RR reaches `PASS` for all figures. Match the actual project layout — do not restructure to fit the template.

```markdown
# 项目说明

简要说明科学目标和主要工作内容。

## 1. 数据简述

- 输入数据路径：
- 数据来源与类型：
- 时间范围：
- 空间范围与分辨率：

## 2. 路径结构和实现功能

- `data_processed/xxx.nc`：实现功能
- `fig1/compute_xxx.py`：实现功能
- `fig1/plot_xxx.py`：实现功能
- `fig1/fig1_xxx.png`：实现功能

## 3. 实现算法与细节

- 如无需展开，可写：`本任务算法直接，省略。`
- 若有复杂处理，写清关键公式、统计方法、重网格方案、加速策略和重要假设。

## 4. 可能存在的问题与风险

- 无则写：`目前无明显问题。`

## 5. 图片简述

- 以大气科学论文写作口径，专业、简洁地说明每张图的内容与意义。

## 6. 版本更新

- （更新时填写日期、变更内容、影响文件）
```

On later modification, append to「版本更新」and update only affected path/figure sections.
