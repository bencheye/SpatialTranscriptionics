# nanoString数据预处理说明

## 文件类型

## fastq原始测序数据处理成dcc格式

> **GeoMxNGSPipeline**软件

- Input

  1. fastq文件
  2. ini的配置文件
     1. [processing] 包含处理的参数
     2. AOI_List 包含AOI名称和坐标
     3. Target olig的barcode序列和对应的**靶点号**

- Output

  dcc文件

   	1. 靶点号和count
   	2. 扫描的属性

---

## dcc格式质控和归一化：

> **GeomxTools** --- R包（R4.1版本）

### readNanoStringGeomxSet 

> 直接打包QC和预处理过程

- 参数

  dccFiles： DCC文件路径，

  pkcFiles： 探针注释信息，包含probeID，基因名，pkc模块，阴性探针标记

  phenoDataFile： 表型meatdata.xls, 包含样本、ROI、分割等信息

  phenoDataSheet： xls的sheet名

  phenoDataDccColName： sheet中样本的表名（列名）

- Output---S4的对象

  - 表达矩阵（标准化后）

    expMat <- demoData@assayData$exprs

    行为基因，列为样本名（每个ROI ID名称）

  - 样本的ROI矩阵

    roiMat <- demoData@phenoData@data

    行为样本名（每个ROI ID名称），列为ROI相关信息
  
- 计算背景

  ​	阴性探针表达强度（多个探针取几何平均值）

- 计算表达强度-Signal strength

  - 高表达的内源性基因捕获的靶细胞计数的测量方法

- 信噪比（SNR）signal-background-ratio

  - SNR>2 OR SNR>3

- AOI QC

  - Q3(一个AOI区域75%)

- gene QC

  ​	LOQ