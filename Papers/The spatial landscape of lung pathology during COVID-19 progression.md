# The *spatial* landscape of lung pathology during COVID-19 progression

---

## 知识点

---

1. IHC（immunohistochemistry）：

   最常见的免疫染色应用

   **原理**：通过利用与生物组织中的抗原特异性结合的**抗体**的原理来选择性地识别组织切片细胞中的**抗原**蛋白。

   **应用**：诊断异常细胞，例如在恶性肿瘤中发现肿瘤细胞。
   
2. **定量限Limit of Quantification，LOQ**

   样品中被测组分能被定量测定的最低浓度或最低量，此时的分析结果应能确保一定的正确度和精密度。

   > The limit of quantitation (LOQ) is set to be the negative probe geomean + some number of standard deviations of the negative probes. This is a confidence threshold rather than a detection threshold. A value below LOQ does not necessarily mean that a target is not expressed, but if the value is >LOQ then we are confident that it is expressed. For CTA we recommend 2.5 as a stringent threshold and 2.0 for a slightly permissive threshold. LOQ is a threshold for high confidence detection.

3.  tukey's test 方法 识别异常值（Q3）

   最大估计值 ：Q1-K(Q3-Q1) 最小估计值 ：Q1+K(Q3-Q1)

   其中 K=1.5 中度异常 K=3 极限异常 数据落在 在 Q1-3(Q3-Q1) / Q1+3(Q3-Q1) 以外 大概率是异常值

---

- 科学问题

  探究COVID-19进程中肺病例的空间图谱。通过靶向36个蛋白质的表达在单细胞精度下来研究人类急性肺损伤（包括SARS-CoV-2的感染）的细胞组成和空间结构。

- 结果：

  这些空间精度的单细胞揭示了感染和损伤肺的紊乱的结构，同时分布着广泛的免疫浸润。

## 方法

---

## 方法

- 样本组成

  23名患者，其中包括在流感（*n* = 2）或细菌感染（*n* = 4），急性细菌性肺炎（*n* = 3）或COVID-19呼吸窘迫综合征（*n* = 10），以及未死于无肺病的个体（*n* = 4）

- GeoMx DSP COVID-19免疫反应panel

  包含大约1850个RNA靶标；分析至多24个ROI

- GeoMx DSP载玻片制备

  1. 蛋白酶K处理组织，暴露RNA靶标，
  2. 组织与RNA探针混合，孵育过夜
  3. 洗涤组织并用组织可视化标记物染色

- GeoMx DSP样品采集

  1. 组织玻片加载到仪器上，扫描可视化整个组织图像
  2. 病理学家从三中类型功能组织（血管区、导气管区、肺泡区）中挑选24个ROI
  3. 每个ROI再基于细胞特异marker的荧光再细分成多个隔室
  4. 然后使用连续的UV光照射每个隔室，以收集不同细胞类型的探针barcodes，依次加入到96孔板中

- Illumina测序 i5 × i7 dual-indexing paired-end

- 数据分析

  1. Illumina的bcl2fastq程序为每个间隔区生成一个fastq文件
  2. 然后GeoMx DnD pipeline 将其转换为dcc格式文件（digital count conversion）
  3. 然后将dcc文件转换成表达矩阵（用户自定义python脚本）
  4. 每个非NTC样本（）至少需要10000个reads
  5. 使用全局Grubb‘s outlier test alpha=0.01监测探针偏差状态
  6. 通过取探针计数的几何平均值，将给定靶标的所有剩余探测计数分解为单个度量
  7. 在取几何平均值之前，对任何产生0个计数的探测添加一个计数1。
  8. 对于每个样本，根据每个池中阴性探针的几何平均值生成一个RNA -探针池特异性的阴性探针归一化因子。
  9. 为了好的数据质量，对于每个ROI我们计算基因计数的75%分位数（对给定基因的所有非离群探针的几何平均值）
  10. 然后检查这些Q3归一化因子的分布，任何ROI大于两倍标准差的被定义为偏差，从log2转换的Q3归一化因子的平均值。
  11. 该标准删除了低于范围的15个ROI和高于范围的1个ROI
  12. 健康肺样本和COVID-19患者样本之间的差异标记丰度采用双面Wald检验，并与Benjamini Hochberg FDR进行多重比较。

- IMC数据预处理

  1. 预处理方法同参考文献26，并作了一下修改
  2. 从**MCD文件**中提取**影像数据**
  3. 采用固定阈值去除热像素（hot pixels），对图像进行放大2次，高斯平滑，并从每幅图像中保存一个方形的500像素的裁剪进行图像分割，作为**HDF5文件**
  4. 用 **ilastik** 对图像中的细胞进行分割，通过人工标记像素为属于三个类别之一:核（由DNA和组蛋白H3通道信号标记的区域）、细胞质（紧邻细胞核周围并与胞质通道信号重叠的区域）和背景（所有通道都是低信号的像素）
  5. Ilastik使用标记的像素来训练随机森林分类器，使用从图像及其导数派生的特征
  6. 使用的特征有高斯的Laplacian、高斯梯度幅值、高斯差分、结构张量特征值和高斯特征值的Hessian，它们都有0.3到10的高斯核宽度
  7. 预测的输出为每个像素的类概率，利用IdentifyPrimaryObjects 模块的CellProfiler对图像进行分割

- 处理过程

  1. 样本准备
  2. 组织染色（IHC）
  3. GeoMx DSP载玻片制备
  4. GeoMx DSP样品采集
  5. 测序

---

## 讨论

- 与健康的肺部相比，在所有疾病中，我们均观察到肺泡腔隙的明显减少，免疫浸润的增加以及细胞凋亡导致的细胞死亡。我们还注意到，尽管中性粒细胞浸润（尽管ARDS或早期COVID-19的患者的肺部浸润比正常肺增加）是细菌性肺炎的标志，并且高度炎症，间质巨噬细胞浸润，补体激活纤维化对于COVID-19晚期的个体的肺尤为重要。