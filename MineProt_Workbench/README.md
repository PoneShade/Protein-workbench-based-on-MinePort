![header](imgs/header.jpg)

# MineProt_Workbench

这是一个基于AI蛋白质预测工具Alphafold2.3.2和蛋白质管理平台MineProt搭建的蛋白一个蛋白质工作台项目。用户可按照[Alphafold使用指南](https://github.com/google-deepmind/alphafold/blob/main/README.md)输入蛋白质的fasta文件或者pdb文件建立自己的蛋白质数据库，数据库可通过MineProt搭建于localhost上的图形化界面进行管理，并可进行对模型的可视化操作，具体操作见[MineProt操作手册](https://github.com/huiwenke/MineProt/README.md)


**当前项目版本为V1.0.0demo且只支持Debian和ubuntu系统，其中可能存在这很多不完善的地方，如果有任何问题可以通过[pan1704240016@outlook.com](mailto:pan1704240016@outlook.com)找到我**


![CASP14 predictions](imgs/casp14_predictions.gif)

## 开始安装MineProt-Workbench
**AlphaFold是我们项目的基础，故而需要先进行AlphaFold的装载**
您需要一台运行Linux的机器，AlphaFold不支持其他操作系统。完整安装需要高达3TB的磁盘空间来存储基因数据库（建议使用SSD存储）和一块现代的NVIDIA GPU（内存更大的GPU可以预测更大的蛋白质结构）。

请按照以下步骤操作：

安装Docker。
安装NVIDIA Container Toolkit以支持GPU。
配置以非root用户身份运行Docker。
克隆此存储库并进入其中。

Please follow these steps:

1.  安装 [Docker](https://www.docker.com/).
    *  安装
        [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)
        以支持GPU.
    *   配置
        [非root用户身份运行Docker](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user).

1.  克隆此存储库并进入其中。.

    ```bash
    https://github.com/PoneShade/Protein-workbench-based-on-MinePort.git
    cd ./alphafold
    ```

1. 下载遗传数据库和模型参数：

    *   安装“aria2c”。在大多数Linux发行版上，它可以通过包管理器作为“aria2”包提供（在基于Debian的发行版上可以通过运行“sudo apt-install aria2”来安装）。

    *   请使用脚本“scripts/download_all_data.sh”下载并设置完整的数据库。这可能需要相当长的时间（下载大小为556 GB），因此我们建议在后台运行此脚本：

    ```bash
    scripts/download_all_data.sh <DOWNLOAD_DIR> > download.log 2> download_all.log &
    ```

    *   **注意：下载目录“＜download_DIR＞”不应该是AlphaFold存储库目录中的子目录。**如果是，Docker构建将很慢，因为大型数据库将被复制到Docker构建上下文中。

1.  检查AlphaFold是否能够通过运行以下程序使用GPU：

    ```bash
    docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
    ```

    该命令的输出应该显示GPU的列表。如果没有，请检查您在设置
   [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)
    时是否出现问题或者参考以下文档
    [NVIDIA Docker issue](https://github.com/NVIDIA/nvidia-docker/issues/1447#issuecomment-801479573).

1.  构建Docker镜像：

    ```bash
    docker build -f docker/Dockerfile -t alphafold .
    ```


1.安装 `run_docker.py` 依赖项. 

    ```bash
    pip3 install -r docker/requirements.txt
    ```

1.  请确保输出目录存在（默认值为“\/tmp\/alphalfold”），并且您有足够的权限写入该目录。

1.  运行指向FASTA文件的“Run_docker.py”，该文件包含要预测其结构的蛋白质序列（`--FASTA_paths`参数）。AlphaFold将在`--max_template_date`参数指定的日期之前搜索可用的模板；这可以用于在建模过程中避免使用某些模板`--data_dir`是下载了遗传数据库的目录，而`--output_dir`是输出目录的绝对路径。

    ```bash
    python3 docker/run_docker.py \
      --fasta_paths=your_protein.fasta \
      --max_template_date=2022-01-01 \
      --data_dir=$DOWNLOAD_DIR \
      --output_dir=/home/user/absolute_path_to_the_output_dir
    ```

### 基因数据库：

此步骤需要在您的机器上安装aria2c。

AlphaFold需要多个基因（序列）数据库才能运行：

*   [BFD](https://bfd.mmseqs.com/),
*   [MGnify](https://www.ebi.ac.uk/metagenomics/),
*   [PDB70](http://wwwuser.gwdg.de/~compbiol/data/hhsuite/databases/hhsuite_dbs/),
*   [PDB](https://www.rcsb.org/) (structures in the mmCIF format),
*   [PDB seqres](https://www.rcsb.org/) – only for AlphaFold-Multimer,
*   [UniRef30 (FKA UniClust30)](https://uniclust.mmseqs.com/),
*   [UniProt](https://www.uniprot.org/uniprot/) – only for AlphaFold-Multimer,
*   [UniRef90](https://www.uniprot.org/help/uniref).

我们提供了一个脚本 `scripts/download_all_data.sh`可用于下载和设置所有这些数据库：

*   推荐默认设置：

    ```bash
    scripts/download_all_data.sh <DOWNLOAD_DIR>
    ```

    将下载完整数据库。

*   使用 `reduced_dbs` 参数:

    ```bash
    scripts/download_all_data.sh <DOWNLOAD_DIR> reduced_dbs
    ```

    将下载用于
    `reduced_dbs`数据库预设的数据库的精简版本。在AlphaFold运行过程中，应使用相应的AlphaFold参数 `--db_preset=reduced_dbs` 
    ( 请参阅[AlphaFold参数部分](#running-alphafold) section).

**这样Alphafold便配置完成了，接下来我们将配置MineProt**
  ```bash
    cd path/to/MineProt
    toolkit/setup.sh -p 80 -d ./data
 ```
几分钟后，您可以访问位于http://localhost的MineProt站点。
下面下载MineProt的依赖包
在此之前请检查是否安装了python3，如果没有可以允许如下指令
 ```bash
    sudo apt update
    sudo apt install python3
 ```
之后可以运行检测是否安装成功
 ```bash
    python3 --version
 ```
安装成功后运行如下指令下载MineProt所需要的依赖包
 ```bash
cd toolkit/scripts
pip3 install -r requirements.txt
 ```
待运行成功后MineProt及成功装载！
