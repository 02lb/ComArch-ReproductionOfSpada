### 23Fall 计算机体系结构课程复现项目

## Spada模拟器相关工作复现

+ 论文的相关模型已经在Github开源：[https://github.com/tsinghua-ideal/spada-sim](https://github.com/tsinghua-ideal/spada-sim)

### 一、基本环境配置

+ 本次复现工作的总体实验环境如下：

| 实验环境     |               |
| ------------ | ------------- |
| 操作系统     | macOS         |
| 集成开发环境 | IntelliJ IDEA |
| 编程语言     | Rust          |

+ SPADA开源模拟器是由Rust语言编写的，所以需要提前配置Rust运行环境。Rust工具链的安装可以使用官网如下给出的指令安装。由于使用的是IDEA作为IDE，只需要安装Rust插件即可。

```bash
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh	
```

+ 使用如下指令可以进行简单的Rust配置验证以及创建一个简单的Rust项目。

```bash
$ rustc -V
$ cargo new demo
$ cd ./demo
$ cargo build 
$ cargo run 	
```

+ 从Github导入SPADA模拟器项目。

```bash
$ git clone https://github.com/tsinghua-ideal/spada-sim.git
```

+ 由于实验中SPADA对稀疏矩阵的解析需要使用到Python库，所以运行以下指令进行配置运行虚拟环境并安装科学计算库。

```bash
$ python3 -m venv spadaenv
$ source spadaenv/bin/activate
$ pip install -U pip numpy scipy
```

+ 使用 Cargo 工具构建 Rust 项目，并且使用了 `--no-default-features` 明确指示不使用任何默认特性。

```bash
$ cargo build --no-default-features
```



### 二、代码分析

+ 在默认使用的参数中使用了如下图所示的默认配置，可以看出使用了 2个每个具有 8 个通道的 MPE ，16个 APE 和一个1.5 MB 的全局缓存。

<img src="https://cdn.jsdelivr.net/gh/02lb/img_picGo@main/img_data/construct.png" style="zoom:50%;" />

+ SPADA模拟器项目的主要代码架构如下所示，相关模块的作用已经在注释中给出。

```
src
|-- main.rs                    % 项目的主文件，模拟主循环
|-- adder_tree.rs              % APE（加法单元）
|-- block_topo_tracker.rs      % 块拓扑跟踪器
|-- colwise_irr_adjust.rs 	    % 列相关不规则调整模块
|-- colwise_reg_adjust.rs      % 列相关规则调整模块
|-- frontend.rs                % 相关配置
|-- gemm.rs                    % GEMM
|-- oracle_rowwise_adjust.rs   % 行调整相关模块
|-- preprocessing.rs           % 预处理模块
|-- py2rust.rs                 % Python加载稀疏矩阵和Rust交互
|-- rowwise_adjust.rs          % 基于行的调整模块
|-- rowwise_perf_adjust.rs     % 基于行的优先调整模块
|-- scheduler.rs               % 任务调度器
|-- simulator.rs               % 主模拟器
|-- storage.rs                 % 存储器
|-- storage_traffic_model.rs   % 存储流量模型
|-- util.rs                    % 工具函数
```



### 三、运行模拟器

+ 最终的实验测试使用的稀疏矩阵可以在[SuiteSparse Matrix](https://sparse.tamu.edu/)（一个用于稀疏矩阵计算的软件库）中进行选择。在实验对Spada模拟器的验证中我使用的是一个线性规划问题的经典稀疏矩阵cari。
+ 下载并导入稀疏矩阵后，激活虚拟环境并使用如下命令行指令运行Spada模拟器。

```bash
(spadaenv) $ ./target/debug/spada-sim accuratesimu spada ss cari config/config_1mb_row1.json	
```

