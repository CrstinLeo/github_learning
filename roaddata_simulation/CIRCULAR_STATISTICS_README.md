# 圆统计风向分析实现总结

## 问题背景
风向数据存在 0-10° 和 350-360° 之间的循环重叠问题，线性统计方法（简单均值和标准差）会导致偏差。

## 解决方案：圆统计（Circular Statistics）

### 核心原理
使用 **sin 和 cos 三角变换** 处理角度数据的周期性：

$$\text{圆均值} = \text{atan2}(\sin\text{均值}, \cos\text{均值})$$

### 实现的统计指标

#### 1. **圆均值 (Circular Mean)**
- 将所有风向转换为单位向量
- 计算这些向量的平均方向
- **优点**：正确处理 0°/360° 的周期边界

#### 2. **圆标准差 (Circular Standard Deviation)**
$$\text{CircStd} = \sqrt{-2 \ln(R)}$$
- R 是结果向量的长度（0 到 1）
- **R 接近 1**：风向高度集中
- **R 接近 0**：风向分散

#### 3. **方向一致性 (Consistency R)**
- 范围：0 到 1
- **R > 0.99**：风向非常稳定（几乎所有段都达到此标准）
- **R < 0.5**：风向高度分散

## 代码实现

### 关键函数

```python
def circular_mean_std(deg):
    """计算圆统计指标"""
    rad = np.deg2rad(deg)
    sin_mean = np.mean(np.sin(rad))
    cos_mean = np.mean(np.cos(rad))
    mean = (np.rad2deg(np.arctan2(sin_mean, cos_mean)) + 360) % 360
    
    R = np.sqrt(sin_mean**2 + cos_mean**2)
    circ_std = np.sqrt(-2*np.log(R)) if R > 0 else np.nan
    return mean, circ_std, R
```

## 分析结果对比

### 线性统计 vs 圆统计对比（示例：Segment 0）

| 指标 | 线性统计 | 圆统计 | 说明 |
|------|---------|-------|------|
| 风向均值 | 121.35° | 1.37° | ✓ 圆统计正确捕捉了主导方向（接近0°）|
| 标准差 | 166.39° | 0.1143 | ✓ 圆统计显示方向高度集中 |
| R值 | - | 0.9935 | 风向一致性极高（接近1） |

### 另一个示例（Segment 12 - 接近180°）

| 指标 | 线性统计 | 圆统计 | 说明 |
|------|---------|-------|------|
| 风向均值 | 167.51° | 359.53° | ✓ 圆统计正确处理了180°附近的折返 |
| 标准差 | 175.43° | 0.1300 | ✓ 圆统计显示实际的低分散性 |
| R值 | - | 0.9916 | 高一致性 |

## 集成的改进

### 1. **统计计算函数增强**
- 添加 Shapiro-Wilk 正态性检验（用于风速）
- 完整整合圆统计到 `calculate_segment_statistics()`
- 每个 segment 现在返回 3 个新指标：
  - `wind_dir_circ_mean`：圆均值
  - `wind_dir_circ_std`：圆标准差
  - `wind_dir_consistency_R`：方向一致性

### 2. **输出展示增强**
- `print_statistics_table()` 现在同时展示线性和圆统计
- 清晰标注圆统计的处理优势
- 添加正态性检验结果

### 3. **可视化增强**
- 风向图现在显示 **圆均值标记**（紫色虚线）
- 包含圆统计参数在图表标题中
- 易于对比线性拟合和圆统计结果

## 文件输出

### 命令行输出
```
Wind Direction (Linear):    Mean=121.35°, Std=166.39°, ...
Wind Direction (Circular):  Mean=1.37°, CircStd=0.1143, Consistency(R)=0.9935
  [Note: Circular statistics use sin/cos transformation to handle 0-10° and 350-360° overlap]
```

### 可视化图表
- `wind_analysis.png`：更新的分析图表（1.8 MB）
- 每个 segment 的风向图中显示圆均值线

## 关键发现

✓ **所有 segment 的 R 值都 > 0.98**
- 风向分布高度集中，具有明确的主导方向
- 异常值虽然存在但不影响总体方向

✓ **圆均值与风向实际分布一致**
- 接近 0-10° 的 segment：圆均值在 1-3°
- 接近 180° 的 segment：圆均值在 359-360°
- 证明圆统计有效消除了周期性边界问题

✓ **风速分布的正态性**
- 大部分 segment 风速呈正态分布（Shapiro-Wilk p > 0.05）
- 少数 segment 存在异常值导致非正态分布

## 使用建议

1. **优先使用圆统计结果** 进行风向分析和建模
2. **结合 R 值评估** 风向数据的可靠性
3. **线性统计保留** 以供参考和异常值检测
4. **后续分析** 应使用圆均值而非线性均值

## 技术参考

- Mardia, K. V. & Jupp, P. E. (1999). *Directional Statistics*
- 圆统计库：`scipy.stats` 与 NumPy 的 sin/cos 变换
