# 快速使用指南

## 运行分析

```bash
/Users/shulin/Documents/Project/.venv/bin/python /Users/shulin/Documents/Project/roaddata_analysis
```

## 核心参数说明

### 风速统计
| 参数 | 说明 | 示例值 |
|-----|------|--------|
| `wind_speed_original_count` | 原始数据点数 | 45 |
| `wind_speed_outlier_count` | 剔除的异常值 | 0-3 |
| `wind_speed_outlier_ratio` | 异常值比例 | 0-6.67% |
| `wind_speed_mean` | 均值 | 6.12 m/s |
| `wind_speed_p90` | 90百分位 | 7.48 m/s |
| `wind_speed_shapiro_p` | 正态性 p-value | 0.701903 |
| `wind_speed_is_normal` | 是否正态 | True/False |

### 风向圆统计
| 参数 | 说明 | 范围/单位 |
|-----|------|---------|
| `wind_dir_circ_mean` | 圆均值 | 0-360° |
| `wind_dir_circ_std` | 圆标准差 | 弧度 |
| `wind_dir_consistency_R` | 方向一致性 | 0-1（越接近1越集中） |

## 关键输出解释

### 正态性判断
```
✓ Normal (p ≥ 0.05)      → 可以使用参数统计方法
✗ Non-Normal (p < 0.05)   → 考虑使用非参数方法
```

### 风向集中度
```
R > 0.9  → Very High     → 风向极度集中，可靠性高
0.7-0.9  → High          → 风向集中
0.5-0.7  → Moderate      → 风向分散
< 0.5    → Low           → 风向高度分散
```

## 输出文件

- `wind_analysis.png`：完整的分析图表（3列×20行）
  - 左列：风速直方图 + 正态拟合
  - 中列：Q-Q Plot
  - 右列：风向分布（相对角）+ 圆均值标记

## 常见问题

**Q1: 为什么有些 segment 非正态？**
A: 数据中存在异常值或尖峰。通过 IQR 方法已剔除，但某些 segment 剔除后仍保留异常脚尾。

**Q2: 圆统计中 R 值都很高（>0.99）是否正常？**
A: 是的。数据生成过程中风向设置了主导方向分布，这是正确的表现。

**Q3: 可以改变异常值剔除方法吗？**
A: 可以。在主函数中修改参数：
```python
stats_list = calculate_segment_statistics(segments_data, outlier_method='hampel')
```

**Q4: 风向为什么用圆统计而不是线性统计？**
A: 风向是循环数据（0° ≈ 360°），线性统计会在边界处产生错误。例如 10° 和 350° 实际很接近，但线性距离是 340°。

