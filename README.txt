========================================================================
单数字图像识别系统（简易OCR）—— MATLAB代码使用说明
========================================================================

【文件结构】
matlab_code/
├── main_digit_recognition.m   主程序：识别单张图片并可视化完整处理流程
├── batch_test.m                批量测试脚本：对test_images中全部样本
│                                识别并统计正确率、绘制混淆矩阵
├── preprocessImage.m           预处理函数：灰度化 + 中值滤波降噪
├── binarizeImage.m             二值化函数：Otsu自适应阈值分割
├── segmentDigit.m               分割函数：连通域分析定位数字并裁剪
├── normalizeDigit.m             归一化函数：等比例缩放到32x32
├── loadTemplates.m              加载0-9标准模板图像
├── templateMatch.m              模板匹配函数：归一化相关系数匹配
├── templates/                   0-9标准数字模板图片（32x32二值图）
│   ├── template_0.png ... template_9.png
└── test_images/                 测试样本图片（20张标准样本，覆盖0-9，含噪声/旋转/光照干扰；
                                  另附3张 stress_test*.jpg 高难度压力测试样本，
                                  用于验证系统在重度噪声/大角度旋转下的表现极限）
    ├── test_00_digit0.jpg ... test_19_digit9.jpg   （20张标准测试样本）
    ├── stress_test_digit8.jpg          （压力测试：重度噪声+18°旋转的数字8）
    ├── stress_test2_digit6_rot40.jpg   （压力测试：40°大角度旋转的数字6）
    ├── stress_test3_digit6_flip.jpg    （压力测试：近180°旋转的数字6，视觉上接近9）
    └── labels.csv                （20张标准测试样本的真实标签，供batch_test.m使用）

【运行环境要求】
- MATLAB R2016b 及以上版本（脚本使用了局部函数写法）
- 需要 Image Processing Toolbox（用于 rgb2gray、medfilt2、graythresh、
  imbinarize、bwlabel、regionprops、imresize 等函数）
- 无需 Computer Vision Toolbox / Deep Learning Toolbox

【使用方法】
1. 单张图片识别演示：
   在 MATLAB 中将当前目录切换到 matlab_code 文件夹，运行：
       >> main_digit_recognition
   将自动识别 test_images/test_00_digit0.jpg，并弹出窗口展示：
   原图 -> 灰度图 -> 二值化图 -> 数字定位框 -> 归一化后识别结果，
   同时在命令行窗口输出识别结果与各数字模板的匹配得分。

   若想识别自己的图片，修改脚本开头的 imgPath 变量为你自己的图片路径即可，
   例如：
       imgPath = 'D:\my_digit.jpg';

2. 批量测试与正确率统计：
       >> batch_test
   将自动对 test_images 文件夹内全部20张测试图片进行识别，
   在命令行输出每张图片的真实标签/预测标签/是否正确，
   并在最后给出总体识别正确率与混淆矩阵可视化图。

【算法流程简介】
   原始图像
     -> 灰度化 (rgb2gray)
     -> 中值滤波降噪 (medfilt2, 3x3窗口，去除椒盐噪声)
     -> Otsu自适应阈值二值化 (graythresh + imbinarize)
     -> 连通域分析，取最大连通域作为数字轮廓并裁剪 (bwlabel + regionprops)
     -> 等比例缩放归一化到 32x32 (imresize)
     -> 与0-9共10个标准模板计算归一化相关系数 (NCC)
     -> 取相关系数最大者作为最终识别结果

【模板制作说明】
   templates 文件夹中的 template_0.png ~ template_9.png 是使用标准
   印刷体字体制作的标准数字模板，制作流程（预处理阶段一次性完成，
   不属于每次识别的在线流程）与 normalizeDigit.m 中的归一化规则完全一致：
   先二值化、裁剪外接矩形，再等比例缩放到32x32画布并居中。
   若需要识别特定字体或手写风格的数字，可以自行替换/新增模板图片，
   只需保证模板文件命名规则与放置路径一致即可。

【已知局限性（可通过压力测试样本验证）】
   本系统基于模板匹配，对旋转、大幅形变的适应能力有限。将 main_digit_recognition.m
   中的 imgPath 改为 test_images/stress_test3_digit6_flip.jpg 并运行即可复现：
   数字"6"在旋转约180°后，形状与"9"高度相似，模板匹配会将其误判为"9"。
   这一现象在实验报告"误差分析"部分有详细讨论。

【可能的改进方向（详见实验报告"结论与展望"部分)】
   - 对倾斜角度较大的数字先做角度矫正（如基于最小外接矩形的主轴方向）；
   - 使用多种特征融合（如Hu矩、HOG特征）替代单一模板匹配以提升鲁棒性；
   - 引入多模板（不同字体/手写风格）投票机制，提高对多样化书写风格的适应性。
========================================================================
