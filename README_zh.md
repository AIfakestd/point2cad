# Point2CAD: Reverse Engineering CAD Models from 3D Point Clouds

[![Website](doc/badges/badge-website.svg)](https://www.obukhov.ai/point2cad)
[![Open In Colab](doc/badges/badge-colab.svg)](https://colab.research.google.com/drive/1o5nNmu1CIn7I5wFFmF8-u7O66Bxqt_xC?usp=sharing)
[![Docker](doc/badges/badge-docker.svg)](https://hub.docker.com/r/toshas/point2cad)

<p align="center">
    <img src="doc/teaser.jpg">
</p>

本仓库实现了我们题为 《Point2CAD: Reverse Engineering CAD Models from 3D Point Clouds》 的论文方法，作者包括：
Yujia Liu、Anton Obukhov、Jan Dirk Wegner 和 Konrad Schindler。

如上图所示，该方法接收一个 CAD 模型扫描得到的原始点云，并重建其曲面、边和角点。

## 交互式演示图库

在项目页面中浏览 ABC CAD 模型数据集中的精选模型，展示我们方法及竞品方法的重建效果：

[<img src="doc/badges/badge-website.svg" height="32"/>](https://www.obukhov.ai/point2cad)

## 快速开始

要重建你自己的 CAD 模型，请使用 Colab 或按照以下说明配置本地环境。

### 本地环境（推荐，约 5 分钟）

要处理 assets 文件夹中的 CAD 模型，只需克隆仓库并在仓库根目录下运行以下命令即可。在配备 GPU 的机器上，整个过程可在 5 分钟内完成。即使没有 GPU 也可以顺利运行。结果将在 `out` 目录中查看。

```shell
docker run -it --rm --gpus "device=$CUDA_VISIBLE_DEVICES" -v .:/work/point2cad toshas/point2cad:v1 python -m point2cad.main
```
### Google Colab（约 30 分钟）

Colab 免去了在本地运行应用程序和使用 Docker 的需要。但由于依赖项的构建时间较长，可能会比较慢。与 Docker 环境不同，Colab 的功能不提供保障。点击下方徽章开始体验：

[<img src="https://colab.research.google.com/assets/colab-badge.svg" height="32"/>](https://colab.research.google.com/drive/1o5nNmu1CIn7I5wFFmF8-u7O66Bxqt_xC?usp=sharing)

### 使用你自己的数据运行

如果你想在自己的点云数据上运行该流程，请添加 `--help` 选项来了解如何指定输入文件路径和输出目录路径。*仅在容器化运行环境中：输入和输出路径必须位于同一个仓库根目录下*。

## 开发环境设置

该代码包含许多本地依赖项，包括 PyMesh。要从源码构建并配置开发环境，请克隆仓库并运行以下命令：

```shell
cd build && sh docker_build.sh
```

然后只需从仓库根目录运行：

```shell
docker run -it --rm --gpus "device=$CUDA_VISIBLE_DEVICES" -v .:/work/point2cad point2cad python -m point2cad.main 
```

如果 Docker 不可用，请参考 [PyMesh](https://github.com/PyMesh/PyMesh) 安装指南从源码构建环境，或者直接按照 [Dockerfile](build/Dockerfile) 或 [Colab 安装脚本](build/colab_build.sh) 中的步骤操作。

## 关于演示

从点云重建 CAD 模型包含两个步骤：使用表面聚类对点云进行标注（由 [ParseNet](https://github.com/Hippogriff/parsenet-codebase)、[HPNet](https://github.com/SimingYan/HPNet) 等方法实现），以及重建曲面和拓扑结构。

预训练的 [ParseNet](https://github.com/Hippogriff/parsenet-codebase) 模型可以在这里找到：[带法向量的输入点云模型](http://neghvar.cs.umass.edu/public_data/parsenet/pretrained_models/parsenet.pth) 和 [不带法向量的输入点云模型](https://drive.google.com/file/d/1BGLMR29yDvt1lstxlsPiWlkdJTmVb4Bf/view?usp=share_link)。如果这些模型无法使用，请改用 [point2cad/logs](point2cad/logs) 目录下的权重文件。要使用该模型，请将 [point2cad/generate_segmentation.py](point2cad/generate_segmentation.py) 脚本放到[ParseNet repository](https://github.com/Hippogriff/parsenet-codebase) 的目录中，并在该目录下执行它。

本代码主要关注上述流程中的第二部分（即“teaser figure”示意图中的第 3、4、5 步），要求输入的点云格式为 `(x, y, z, s)`，其中每个具有 `x`, `y`, `z` 坐标的 3D 点都带有表面 ID `s` 的标注，例如 [assets](assets) 文件夹中的示例所示。

该过程会将以下结果存储在输出目录中（默认为 `out`）：


- `unclipped`：未裁剪的曲面，用于进行两两相交处理；

- `clipped`：裁剪边缘后的重建曲面；

- `topo`：重建的边和角点。



