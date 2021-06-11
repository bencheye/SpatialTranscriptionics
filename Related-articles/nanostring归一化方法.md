# nanoString归一化方法

> 标准化的目的是调整技术变量，如不同的ROI/AOI表面积和组织mRNA质量，使有意义的生物学和统计发现。

 ## 基础知识

- 背景: 

  由唯一的负探针捕获的脱靶计数的测量;这些探针没有针对已知的人类转录本;在CTA分析中，每个ROI/AOI将多个唯一的阴性探针浓缩为一个NegProbe计数(多个阴性探针的几何平均值)。

- 信号强度：通过高表达的内源基因捕获的目标计数的测量

-  Cancer Transcriptome Atlas (CTA) 

-  signal-to-background ratio (SNR) 信噪比

- 我们将一个SNR > 2的基因定义为“检测的基因”，而SNR为3的基因也是合理的。

- 通过这个热图可以看出被检测基因在AOI中比例的不同

- 基于AOI类型和组织ID知道大概多少基因在背景以上

- 一些aoi的高于背景数的比率特别低，把它们从研究中剔除可能会更好

- 热图中存在低信噪比的小条带基因。

- 顶上是统计每个基因SNR>0的ROI比例

![image-20210509145327677](C:\Users\benche\AppData\Roaming\Typora\typora-user-images\image-20210509145327677.png)

## AOI QC

-  QC

  识别应该删除的低表现的aoi

  当探针非特异性地结合到核苷酸、蛋白质和ECM材料时，CTA数据的**背景噪声**就出现了。为了测量背景，CTA面板有80+阴性对照探针，以人类基因组中不存在的序列为目标。这些探测器一起提供了对每个片段的背景水平的准确估计。当CTA数据被处理为基因水平测量时，这些探针用几何平均值平均，以创建单个测量，即NegProbe。

- log2转换的阴性探针counts的AOI分布数目图  (note the use of a Log2 x-axis).

  ![image-20210509155551273](C:\Users\benche\AppData\Roaming\Typora\typora-user-images\image-20210509155551273.png)

- 在GeoMx数据中，中位数计数是适用的，但我们通常将四分位数3计数(Q3)作为信号强度的更高表示度量。Q3是AOI的第75百分位基因计数值;换句话说，Q3是一个单基因计数(或两个基因计数的平均值，基于总基因的奇数/偶数)。

- **Q3低于背景需要删除**

- note the use of a Log2 

  ![image-20210509194018018](C:\Users\benche\AppData\Roaming\Typora\typora-user-images\image-20210509194018018.png)

- AOI 的Q3 counts应该大于2或者4（Log2）
- 如果有多个pool的数据，每个pool需要单独分析

## gene QC

- 一般的方法是检测每个基因，看其研究范围内的信号是否高于背景

- 是用signal to noise来计算比例，SNR>1说明该基因表达量比本底高，SNR<1说明该基因表达比本底低。如果这个基因在90%的ROI中SNR<1，说明这个基因是弱表达，那么我们就把它去掉

- LOQ表示noise

- geomean+2.5 *sd

- (geomean probe in all segments) / (geomean probes within target) <= 0.1

- (Fail Grubbs outlier test in) >= 20% segments

- 作为阐释基因质量控制这一概念的一个基本启发，让我们定义一个定量限制(LOQ)，低于LOQ的基因将被排除。例如，我们可以将LOQ定义为NegProbe背景值之上的2个标准差;我们可以选择或多或少保守一点。在这一点上，我们需要定义一个基因的AOIs应该低于LOQ的百分比，在它被排除之前。

- GEOMEAN PROBE IN ALL SEGMENTS / GEOMEAN PROBES WITHIN TARGET

  > This is a filter for catching dropout probes (low outliers). It excludes probes for which the average counts across all segments are <=10% (or the value you enter) of the counts for all probes to that gene. This test catches probes that didn’t actually make it into the final probe pool (which should be consistent across experiments) and removes probes that perform poorly relative to other probes to the same target
  >
  > Example: if four of five probes to GeneA yield 100 counts per segment on average and the fifth probe yields <10 counts on average, then the fifth probe will be called a global outlier (removed from ALL segments).
  >
  > 删除一个基因多个探针中不充分测序

- FAILS GRUBBS OUTLIER TEST （离群值检验）

  > A Grubbs outlier test is performed on the probe counts to a given target in a given segment (5 values if there are 5 probes to the target). This is done for every target in every segment. If a particular probe is called an outlier in >=20% of segments, then it will be called a global outlier (removed from ALL segments). This test removes probes that are outliers in a user-defined proportion of segments from the entire dataset.

- 前两个probe QC是针对多探针的，那么如果一个基因对应单个probe，是否需要过滤

- LoQ = nGeoMean(negProbe)*SD(negProbe) *( 2 or 2.5)

-  removed probes that failed two-tailed outlier test in >50% of ROIs

   Grubbs’s test：要求数据符合正太分布

   removed probes that fell below background in >99% of ROIs

  LOQ = 2 SD*geomean of negative probes

- ![image-20210509201344710](C:\Users\benche\AppData\Roaming\Typora\typora-user-images\image-20210509201344710.png)

- 基于我们选择的% AOI阈值，我们将从我们的大多数分析中滤除低信号基因。（20%）

## Q3normalzation

- 缩放数据，使他们有相同的Q3值，
- 将每个AOI的所有基因除以它们各自的Q3计数，
- 将所有AOIs中的所有基因乘以一个常数，这个常数定义为所有AOIs的Q3计数的几何平均值。

