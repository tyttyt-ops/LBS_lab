# SMPL 线性蒙皮（LBS）可视化实验

姓名：唐悦婷 学号：202311061051 专业：计算机科学与技术（公费师范）

本实验项目完整实现了 SMPL（Skinned Multi-Person Linear Model）的线性蒙皮（Linear Blend Skinning, LBS）流程，并提供了详细的可视化对比。

## 实验任务

本项目完整完成了以下 7 个任务：

### 任务 1：加载 SMPL 并输出基础信息
- 使用 `smplx.create()` 加载 SMPL 模型
- 指定 `model_type='smpl'`, `gender='neutral'`
- 输出模型基础信息：顶点数、面片数、关节数、betas 维度

### 任务 2：可视化模板网格与蒙皮权重
- **单关节权重热力图** (`stage_a_template_weights.png`)：显示特定关节对所有顶点的影响权重
- **全关节主导权重分布图** (`all_joint_weights.png`)：每个面片根据主导影响关节着色

### 任务 3：可视化形状校正与关节回归
- 设置非零形状参数 β 产生体型变化
- 计算形状校正后的网格 `v_shaped`
- 利用 `J_regressor` 回归关节位置
- 可视化结果 (`stage_b_shaped_joints.png`)

### 任务 4：可视化姿态校正
- 设置非零姿态参数 θ（抬手、弯肘等）
- 构造 `pose_feature = R - I`
- 计算 `pose_offsets` 和 `v_posed`
- 可视化姿态偏移大小 (`stage_c_pose_offsets.png`)

### 任务 5：可视化完整 LBS 结果
- 计算关节的全局刚体变换
- 使用 LBS 权重进行加权变换
- 得到最终顶点位置
- 可视化最终姿态 (`stage_d_lbs_result.png`)

### 任务 6：生成总对比图
- 生成 2×2 对比图 (`comparison_grid.png`)，清晰展示四个阶段

### 任务 7：手写 LBS 与官方结果一致性验证
- 计算手写实现与官方前向结果误差
- 平均绝对误差：0.0000000000
- 最大绝对误差：0.0000000000

## 快速开始

### 环境要求

```bash
Python >= 3.8
torch >= 2.0
numpy < 2.0
smplx
matplotlib
scipy
```

### 安装依赖

```bash
# 安装 PyTorch (CPU 版本)
pip install torch --index-url https://download.pytorch.org/whl/cpu

# 安装其他依赖
pip install smplx numpy==1.26.4 matplotlib scipy
```

### 运行实验

```bash
# 使用默认参数运行
python run_lbs_lab.py

# 自定义参数运行
python run_lbs_lab.py --model-dir ./models --out-dir ./outputs --joint-id 18 --num-betas 10
```

### 参数说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `--model-dir` | `./models` | 模型目录，应包含 `smpl/SMPL_NEUTRAL.pkl` |
| `--out-dir` | `./outputs` | 输出图片目录 |
| `--joint-id` | `18` | 要可视化的关节编号（18 = 右肘） |
| `--num-betas` | `10` | 使用的 shape 参数维度 |

## 项目结构

```
lbs/
├── run_lbs_lab.py              # 主脚本：完整的 LBS 实验实现
├── README.md                   # 本文件
├── models/                     # 模型文件目录
│   └── smpl/
│       └── SMPL_NEUTRAL.pkl    # SMPL 中性模型（约 235MB）
└── outputs/                    # 输出结果目录
    ├── stage_a_template_weights.png   # (a) 模板+权重
    ├── stage_b_shaped_joints.png      # (b) 形状混合+关节
    ├── stage_c_pose_offsets.png       # (c) 姿态校正
    ├── stage_d_lbs_result.png         # (d) 最终 LBS
    ├── comparison_grid.png            # 四阶段对比图
    ├── all_joint_weights.png          # 全关节权重分布
    └── summary.txt                    # 模型摘要与误差报告
```

## 实验结果

### 模型基础信息

```
num_vertices: 6890          # 顶点数
num_faces: 13776            # 面片数
num_joints: 24             # 关节数
num_betas: 10              # shape 参数维度
visualized_joint_id: 18    # 可视化关节（右肘）
```

### 一致性验证

```
手写 LBS 与官方 forward 的平均绝对误差: 0.0000000000
手写 LBS 与官方 forward 的最大绝对误差: 0.0000000000
```

手写实现与官方前向结果完全一致！

## 输出文件说明

### 1. stage_a_template_weights.png
- **内容**：模板网格 + 单关节权重热力图
- **含义**：颜色越深，表示该关节（第18号右肘）对顶点影响越强
- **思考**：
  - 为什么一个顶点不只受一个关节影响？
    - 人体皮肤是连续的整体，相邻关节共同影响
    - 保证变形平滑自然，避免撕裂
  - 如果权重全给一个关节，会出现什么效果？
    - 僵硬、机器人效果
    - 关节处会出现不自然的角度
<img width="1077" height="1119" alt="stage_a_template_weights" src="https://github.com/user-attachments/assets/29c77dd1-33df-4030-9b09-9a3655fd928d" />

### 2. stage_b_shaped_joints.png
- **内容**：形状变化后的网格 + 回归出的关节
- **参数**：β₀=2.0（高）、β₁=-1.2（瘦）、β₂=0.8（肌肉）
- **思考**：
  - 为什么关节位置要从形状后的网格回归？
    - 不同体型关节位置不同
    - 胖人关节偏外，瘦人关节偏内
  - v_template 与 v_shaped 的区别？
    - v_template：标准体型模板
    - v_shaped：应用形状参数后的个性化体型
<img width="1077" height="1119" alt="stage_b_shaped_joints" src="https://github.com/user-attachments/assets/f26c7ec8-8a00-46ab-8ad0-1fc143830999" />

### 3. stage_c_pose_offsets.png
- **内容**：姿态校正后的网格，颜色表示偏移大小
- **姿态**：抬手、弯肘、略微弯膝
- **观察**：姿态偏移集中在弯曲部位附近
- **思考**：
  - 为什么 LBS 前要加 pose corrective？
    - 补偿肌肉膨胀效应
    - 弯曲处皮肤会鼓起，需要预变形
  - 如果去掉 pose_offsets，会出现什么问题？
    - 弯曲处皮肤会内陷
    - 不符合真实人体变形
<img width="1077" height="1119" alt="stage_c_pose_offsets" src="https://github.com/user-attachments/assets/87589eba-4b1b-4bb4-8e36-d993cf42d1e4" />

### 4. stage_d_lbs_result.png
- **内容**：最终姿态下的网格 + 关节位置
- **过程**：完整的刚体变换 + 加权蒙皮
- **思考**：
  - J 和 J_transformed 的区别？
    - J：形状后的局部关节位置
    - J_transformed：全局坐标系中的关节位置
  - 为什么最终顶点要写成加权和？
    - 平滑过渡
    - 自然变形
    - 避免接缝
<img width="1077" height="1119" alt="stage_d_lbs_result" src="https://github.com/user-attachments/assets/277603ae-8a3a-40be-b866-ddc9f98fee31" />

### 5. comparison_grid.png
- **内容**：2×2 四阶段对比图
  - (a) Template + LBS Weights
  - (b) Shape Blend + Joint Regression
  - (c) Pose Blend Shapes
  - (d) Final LBS Result
- **作用**：清晰展示 LBS 的每个阶段
<img width="2417" height="2176" alt="comparison_grid" src="https://github.com/user-attachments/assets/e7e4b96c-fd39-4b4c-b672-9d80d9225dbe" />

### 6. all_joint_weights.png（可选）
- **内容**：全关节主导权重分布图
- **含义**：
  - 颜色种类：表示主要受哪个关节控制
  - 颜色明暗：表示该主导权重的强弱
- **作用**：展示 SMPL 模板网格在初始状态下已携带完整的关节影响分布
<img width="1517" height="1559" alt="all_joint_weights" src="https://github.com/user-attachments/assets/476e9cfe-83ff-4183-afc0-d41eceb9e3a7" />

## 技术细节

### LBS 流程

完整的线性蒙皮流程分为四个阶段：

1. **形状混合（Shape Blend）**
   ```python
   v_shaped = v_template + B_S(β)
   ```

2. **关节回归（Joint Regression）**
   ```python
   J = J_regressor @ v_shaped
   ```

3. **姿态校正（Pose Corrective）**
   ```python
   pose_feature = R - I
   pose_offsets = B_P(pose_feature)
   v_posed = v_shaped + pose_offsets
   ```

4. **线性蒙皮（Linear Blend Skinning）**
   ```python
   T = Σ W_j * G_j
   verts = T * v_posed
   ```

### 关键函数

本项目实现了以下关键 SMPL 函数：

- `blend_shapes()` - 形状混合
- `vertices2joints()` - 顶点到关节的回归
- `batch_rodrigues()` - 轴角到旋转矩阵的批量转换
- `batch_rigid_transform()` - 批量刚体变换
- `install_chumpy_pickle_shim()` - 兼容旧版 SMPL 模型文件的加载

## 思考题

### 关于蒙皮权重

**Q: 为什么一个顶点不只受一个关节影响？**
- A: 人体皮肤是连续的整体，相邻关节共同影响才能产生自然的变形效果。如果只受一个关节影响，会导致关节处的不连续变形。

**Q: 如果一个顶点的权重几乎全给了某一个关节，会出现什么效果？**
- A: 会出现僵硬、类似机器人的效果，关节处会呈现不自然的角度变化。

**Q: 如果权重分布很平均，又会出现什么效果？**
- A: 变形不够集中，无法精确控制局部运动，整体变形会显得模糊。

### 关于形状校正

**Q: 为什么关节位置要从形状后的网格回归，而不是固定不变？**
- A: 不同体型的人，关节位置是不同的。例如，肥胖的人关节位置更靠外，瘦削的人关节位置更靠内。

**Q: v_template 与 v_shaped 的差别是什么？**
- A: v_template 是标准体型模板，v_shaped 是应用形状参数（β）后的个性化体型。

### 关于姿态校正

**Q: 为什么 LBS 之前还要加 pose corrective？**
- A: 为了补偿肌肉膨胀效应。当人体关节弯曲时，弯曲处的皮肤会鼓起，LBS 单独无法模拟这种效果，需要姿态校正。

**Q: 如果去掉 pose_offsets，最终人体弯曲处会出现什么问题？**
- A: 弯曲处皮肤会内陷，不符合真实人体的变形特征。

**Q: v_shaped 与 v_posed 的本质区别是什么？**
- A: v_shaped 只考虑了体型变化，v_posed 还加入了姿态校正，为后续的 LBS 做准备。

### 关于线性蒙皮

**Q: J 和 J_transformed 有什么区别？**
- A: J 是形状后的局部关节位置，J_transformed 是经过全局刚体变换后，在世界坐标系中的关节位置。

**Q: 为什么最终顶点要写成加权和，而不是只选择最大权重的关节？**
- A: 加权混合可以保证变形的平滑过渡，避免因硬切换导致的接缝和不自然变形。

## 注意事项

1. **SMPL 模型文件**：由于文件较大（约 235MB），本仓库不包含模型文件。请从 SMPL 官网下载或从其他合法渠道获取。

2. **依赖版本**：注意 PyTorch 和 NumPy 的版本兼容性，建议使用本 README 中指定的版本。

3. **内存要求**：运行实验时需要足够的内存来加载 SMPL 模型和进行计算。

## 参考资料

- [SMPL 官网](http://smpl.is.tue.mpg.de/)
- [SMPL-X Model](https://smpl-x.is.tue.mpg.de/)
- [SMPLify-X 论文](https://smpl-x.is.tue.mpg.de/publications.html)
