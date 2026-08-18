# 每日作业报告



## 1. 本日问题

- 里程碑：day-02
- 学生或小组：为ai发电
- 使用者：设施维护团队
- 真实输入：Kaggle Concrete Crack Images
- 需要的输出：每张图 crack/no_crack 二分类
- 与使用者最相关的错误：漏检裂缝（FN）
- 本日产品边界：只用于安排人工复核，不替代现场检查

## 2. 真实数据或真实课程输入

- 所有者/发布者：arunrk7
- 标题：Surface Crack Detection
- 原始 URL：https://www.kaggle.com/datasets/arunrk7/surface-crack-detection
- 许可标签或使用许可：Data files © Original Authors
- 下载/取得日期：2026-08-18
- 预期文件与结构：data/raw/Positive 与 data/raw/Negative
- 检查命令：python train.py --check-data
- 实际检查结果：REAL DATA CHECK PASSED，counts {Negative: 20000, Positive: 20000}
- 已知缺失、偏差或限制：
  - 来源单一：图片都来自同一混凝土表面类型和相近拍摄条件（光照、相机、背景），换到其它工地/混凝土材质可能表现变差。
  - 相似图块泄漏风险：数据里存在同一表面相邻裁剪的相似图块，随机划分可能让训练和测试看到近重复内容，成绩可能偏乐观。
  - 检查器能力有限：--check-data 只验证数量和目录结构，不能证明数据没有偏差、标注完全正确、许可适合所有用途。



## 3. 可复现运行

```powershell
# 当前目录
cd "D:\ai-summer-camp-2026-main\ai-summer-camp-2026-main\student-work\day-02-concrete"

# 安装
python -m pip install -r requirements.txt

# 数据检查
python train.py --check-data

# 主程序（多数类基线）
python train.py

# 主程序（SmallCNN 候选）
python train.py --model cnn

# 测试
python -m unittest discover -s tests -v
```

关键预期输出与实际输出位置：

- `python train.py --check-data` → `REAL DATA CHECK PASSED`，counts `{Negative: 20000, Positive: 20000}`
- `python train.py` → 生成 `runs\baseline.json` 与 `runs\baseline-errors.png`，打印基线 `evaluation`
- `python train.py --model cnn` → 生成 `runs\cnn.json` 与 `runs\cnn-errors.png`，打印候选 `evaluation` 与 `train_loss`
- `python -m unittest discover -s tests -v` → 3 个测试 `ok`

## 4. 基线与候选

### 简单基线

- 方法：总是预测训练集多数类别（本日训练集多数类是 crack），完全不读取图像内容。
- 为什么足够简单：只统计训练集标签，不做任何图像处理，是最低比较起点。
- 命令：`python train.py`（默认 `--model baseline`）
- 结果：accuracy 0.5，crack_precision 0.5，crack_recall 1.0，漏检 FN 0，误报 FP 150，混淆矩阵 [[0, 150], [0, 150]]

### 候选方法

- 学生完成的核心改动：在 `models.py` 的 `SmallCNN` 里实现 7 层网络：`Conv2d(3,8,k=3,p=1)+ReLU+MaxPool2d(2)` → `Conv2d(8,16,k=3,p=1)+ReLU+MaxPool2d(2)` → `Flatten` → `Linear(16*16*16,2)`，`forward` 返回 `self.net(images)`。
- 保持不变的数据、划分、指标或参数：seed=2026、max_per_class=600、75/25 划分、同一批测试图、同一评估指标。
- 命令：`python train.py --model cnn`
- 结果：train_loss 0.682→0.619；accuracy 0.8133，crack_precision 0.8917，crack_recall 0.7133，漏检 FN 43，误报 FP 13，混淆矩阵 [[137, 13], [43, 107]]

| 项目 | 基线 | 候选 | 含义 |
| --- | ---: | ---: | --- |
| 主指标 accuracy | 0.5 | 0.8133 | 候选整体正确率更高 |
| 重要错误（漏检裂缝 FN） | 0 | 43 | 基线不漏裂缝但把每张图都标成 crack，没有筛选价值；候选漏 43 张裂缝，必须交给人工复核 |

## 5. 一个真实失败案例

- 样本位置/编号：data\raw\Positive\09570.jpg
- 真实结果：crack（有裂缝）
- 系统输出：no_crack（被漏检）
- 可以观察到什么：裂缝细、浅，与背景对比度弱，无明显阴影、污渍或划痕干扰
- 说明的限制：模型对细、浅、低对比度的裂缝容易漏检
- 不能证明什么：一个样本不能证明模型整体失败，也不能说明其它 42 个漏检都是同一原因
- 下一项最小检查：统计全部 43 个漏检样本的裂缝宽度与对比度，看是否都集中在细、浅、低对比度的裂缝

## 6. 智能体与学生工作边界

- 智能体提出/生成/修改了什么：读取课程文件并给出 180 分钟计划；指导安装 Python 3.12 与依赖（含 matplotlib 3.11.1 降级到 3.10.9）；指导复制 starter、下载数据、运行数据检查、基线和 CNN；诊断并解释三个环境问题（python 是 Microsoft Store 占位程序、matplotlib 正则引擎 bug、文件未保存）；整理基线与候选的对比数字、报告各节填写清单和失败案例模板。未替学生修改任何代码文件，models.py 由学生自己填写。
- 学生怎样核对文件、来源、输出、测试和 diff：用 Get-Location / Get-ChildItem 确认终端位于副本目录；用 `python train.py --check-data` 核对 REAL DATA CHECK PASSED 与两个 20000；用 `python -m unittest discover -s tests -v` 核对 3 个测试 ok；运行 `python train.py` 与 `python train.py --model cnn`，读取 runs 下两份 JSON 的数字；打开 `data\raw\Positive\09570.jpg` 原图观察裂缝特征。提交前还将运行 `git status` 与 `git diff --check` 核对暂存内容。
- 学生修改或拒绝了什么建议：跳过了第 1 步复述问题、第 5 步的两个预测问题、第 7 步的比较表，后在报告阶段补齐；
- 每名成员能独立解释的代码或证据：models.py 的 7 层网络与形状变化；train.py 的 balanced_split_indices、majority_baseline、confusion_counts；比较表数字；失败案例

## 7. 结论与限制

在相同数据、划分和指标下，候选 SmallCNN 的 accuracy 从基线的 0.5 提高到 0.8133，误报 FP 从 150 降到 13，说明它比「总猜多数类」更有筛选价值。但候选漏检了 43 张真实裂缝（crack_recall = 0.7133），因此不能单独用它判断「无裂缝」。数据来自单一来源和相近拍摄条件，且存在相似图块，随机划分可能因数据泄漏高估成绩。本结论只基于每类 600 张、2 个 epoch、固定 seed 与划分的小规模实验。输出只能用于安排人工复核，不能替代现场检查、工程师判断或安全决策。下一步最小检查是统计 43 个漏检样本的裂缝特征，验证「细、浅、低对比度裂缝易漏检」的假设。

## 8. 提交复核

- [x] README 从新环境可以开始运行
- [x] 数据检查、测试和主程序重新运行
- [x] 报告数字与保存输出一致
- [x] `presentation.pptx` 在 3 分钟内讲完
- [x] `submission.json` 路径正确
- [x] 无密钥、大数据、私人信息、虚拟环境或缓存
- [x] GitHub 网页复查并邮件发送 URL

