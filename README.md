[README.md](https://github.com/user-attachments/files/24160286/README.md)

MolGPT Fine-tuning for SMILES Generation
# MolGPT Fine-tuning for SMILES Generation

## 1. 项目简介

本项目专注于基于预训练 MolGPT 模型的微调（Fine-tuning），用于生成符合化学规则的 SMILES 分子序列。通过在课程提供的 Dataset3 上进行领域自适应微调（Domain Adaptation），模型能够更好地学习目标化学空间的结构分布，并生成具有合理化学性质的新分子。

本 README 仅描述 MolGPT 微调、生成与评估相关工作。

## 2. 研究目标

使用预训练 MolGPT（GPT-2 架构）作为生成模型，在自定义 SMILES 数据集上进行 Causal Language Modeling 微调，提升生成分子的：

- 合法性（Validity）
- 分子性质合理性

对微调过程和生成结果进行系统分析与可视化。

## 3. 项目目录结构说明

```text
GPT_finetune/
├── data/                       # 预处理后的训练数据
├── Dataset3/                   # 原始课程提供数据集
├── models/
│   ├── MolGPT_finetuned/       # 微调后的 MolGPT 模型及 checkpoint
├── outputs/
│   ├── molgpt_samples.txt      # 生成的 SMILES 序列
│   ├── paper_figures/          # 分子结构图、论文级可视化输出
│   ├── comparison_plots.png    # 性质分布或模型对比图
├── download_gpt2.py            # 预训练模型下载脚本
├── draw_molecule_figure.py     # 分子结构可视化脚本（论文风格）
├── evaluate_rdkit.py           # RDKit 评估（Validity/Uniqueness/Novelty）
├── loss_plot.py                # 训练 loss 曲线绘制
├── run_finetune_molgpt.py      # MolGPT 微调训练主脚本
├── run_generate_molgpt.py      # 分子生成脚本（支持 batch generation）
├── visualize_embedding.py       # Embedding 可视化（PCA）
└── README.md                   # 项目说明文档（本文件）
```

---

## 4. 方法流程（Pipeline）

### 4.1 模型与微调方法

- **模型架构**：MolGPT（GPT-2 Causal LM）
- **预训练数据**：ZINC15（模型原始训练）
- **微调方法**：
   - Full fine-tuning（全参数更新）
   - 目标函数：Causal Language Modeling Loss（Cross-Entropy）
   - 输入输出：SMILES 序列 → 下一个 token 预测

该过程属于监督式领域自适应微调（Supervised Fine-tuning, SFT）。

### 4.2 训练设置

- **Tokenizer**：MolGPT 自带 tokenizer
- **输入长度**：根据训练集 SMILES 长度分布设定

训练过程使用：

- FP16 混合精度
- Gradient Accumulation
- 梯度裁剪（防止不稳定）

训练过程中保存多个 checkpoint，用于分析训练动态。

### 4.3 分子生成

使用 `model.generate()` 进行自回归生成，采用 batch-wise generation 提高生成效率，支持：

- Temperature sampling
- Top-p (nucleus) sampling

输出生成的 SMILES 序列至文本文件。

## 5. 结果评估与分析

### 5.1 RDKit 化学评估

使用 RDKit 对生成分子进行评估，包括：

- Validity：可被 RDKit 成功解析的分子比例
- Uniqueness：生成分子的去重比例
- Novelty：相对于训练集的新颖性

### 5.2 分子性质分析

对生成分子计算并分析以下理化性质：

- 分子量（MW）
- logP
- 拓扑极性表面积（TPSA）

并与训练数据分布进行对比，评估微调是否使生成分子更贴近目标化学空间。

### 5.3 可视化分析

项目包含多种可视化分析模块：

- 训练动态
- Loss 曲线（从多个 checkpoint 汇总）
- Embedding 可视化
- 提取 MolGPT 隐层 embedding
- 使用 PCA 进行降维展示
- 分子结构可视化
- 随机抽取 10 个有效分子
- 显示分子结构、SMILES 及关键分子性质
- 输出为论文级 SVG 图像

## 6. 使用方法

### 6.1 微调训练

```powershell
python run_finetune_molgpt.py
```

### 6.2 分子生成

```powershell
python run_generate_molgpt.py
```

### 6.3 RDKit 评估

```powershell
python evaluate_rdkit.py
```

### 6.4 训练 Loss 可视化

```powershell
python loss_plot.py
```

### 6.5 分子结构可视化

```powershell
python draw_molecule_figure.py
```

## 7. 实验结果总结

- 微调后的 MolGPT 模型在 **Validity** 上显著优于 baseline。
- 生成分子的分子性质分布（logP / TPSA）与训练集更接近。
- Embedding 可视化表明模型在微调后形成了更具结构性的化学语义空间。
- Batch generation 显著提升生成效率（10× 以上）。

---

## 8. 技术栈与依赖

```powershell
pip install -r requirements.txt
```

---

## 9. 总结

本项目系统地完成了 MolGPT 在特定 SMILES 数据集上的微调、生成与评估流程。实验结果表明，经过领域微调后，模型在生成合法分子比例及分子性质分布方面均得到明显改善，验证了预训练语言模型在分子生成任务中的有效性。
