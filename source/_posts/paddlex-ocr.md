---
title: PaddleX OCR 使用
date: 2025-08-31 10:58:57
categories: OCR
tags:
- paddlex
- ocr
permalink: paddlex-ocr
---
共尝试了两套环境：
- 环境1：非docker环境，使用conda安装依赖包，运行在GPU上
- 环境2：docker环境，使用PaddleX官方提供的docker镜像，运行在CPU上
<!--more-->

## 环境1：
### 一、检查环境与依赖
#### 1.查看python、cuda、cuDNN版本
```shell
# 查看python版本
python -V

# 查看cuda版本
nvcc -V

or

conda list | grep cudatoolkit

# 查看cuDNN版本
cat /usr/include/cudnn_version.h | grep CUDNN_MAJOR -A 2

or

conda list | grep cudnn 
```

#### 2.搜索版本
```shell
conda search cudatoolkit
conda search cudnn
```

#### 3.安装指定版本cuda和cuDNN
```shell
conda install cudatoolkit==xx.xx 
conda install cudnn==xx.xx
```

### 二、使用conda创建虚拟环境
#### 1.升级conda版本
```shell
conda update -n base -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main conda
```

#### 2.查看已创建的虚拟环境
```shell
conda env list
```

#### 3.创建虚拟环境
```shell
conda create -n paddlex python=3.10 -y
```

#### 4.激活虚拟环境
```shell
conda activate paddlex
```

#### 5.查看当前环境下已安装的包
```shell
conda list
```

#### 6.删除环境
```shell
conda remove -n paddlex --all
```

### 三、安装paddle相关依赖
#### 1.安装paddlepaddle
```shell
# CPU 版本
python -m pip install paddlepaddle==3.0.0 -i https://www.paddlepaddle.org.cn/packages/stable/cpu/

# GPU 版本，需显卡驱动程序版本 ≥450.80.02（Linux）或 ≥452.39（Windows）
python -m pip install paddlepaddle-gpu==3.0.0 -i https://www.paddlepaddle.org.cn/packages/stable/cu118/

# GPU 版本，需显卡驱动程序版本 ≥550.54.14（Linux）或 ≥550.54.14（Windows）
 python -m pip install paddlepaddle-gpu==3.0.0 -i https://www.paddlepaddle.org.cn/packages/stable/cu126/
```

#### 2.安装paddlex和paddle ocr
```shell
pip install paddlex==3.2.0
pip install "paddlex[ocr]==3.2.0"
```

#### 3.安装高性能推理插件
```shell
# gpu版本
paddlex --install hpi-gpu

# cpu版本
paddlex --install hpi-cpu
```

#### 4.安装项目相关依赖
```shell
conda install fastapi==0.116.1
conda install gunicorn==23.0.0
conda install uvicorn[standard]==0.35.0
...
```

### 四、配置
### 1.获取paddlex pipeline配置文件
```shell
paddlex --get_pipeline_config OCR
```

### 2.修改配置文件模板
```yaml

pipeline_name: OCR

text_type: general

use_doc_preprocessor: False
use_textline_orientation: False
use_hpip: False
device: cpu

SubPipelines:
  DocPreprocessor:
    pipeline_name: doc_preprocessor
    use_doc_orientation_classify: False
    use_doc_unwarping: False
    SubModules:
      DocOrientationClassify:
        module_name: doc_text_orientation
        model_name: PP-LCNet_x1_0_doc_ori
        model_dir: null
      DocUnwarping:
        module_name: image_unwarping
        model_name: UVDoc
        model_dir: null

SubModules:
  TextDetection:
    module_name: text_detection
    model_name: PP-OCRv5_mobile_det
    model_dir: null
    limit_side_len: 640
    limit_type: max
    max_side_limit: 4000
    thresh: 0.3
    box_thresh: 0.6
    unclip_ratio: 1.5
  TextLineOrientation:
    module_name: textline_orientation
    model_name: PP-LCNet_x1_0_textline_ori 
    model_dir: null
    batch_size: 1
  TextRecognition:
    module_name: text_recognition
    model_name: PP-OCRv5_mobile_rec
    model_dir: null
    batch_size: 1
    score_thresh: 0.0
```

### 3.动态生成配置文件
```python
def create_ocr_config():
    with open(config_template, 'r', encoding='utf-8') as f:
        config = yaml.safe_load(f)

    # 检测GPU可用性
    try:
        import paddle
        has_gpu = (
                getattr(paddle.device, "is_compiled_with_cuda", paddle.is_compiled_with_cuda)()
                and paddle.device.cuda.device_count() > 0
        )
    except Exception as e:
        logger.warning(f"GPU 检测失败，使用 CPU。原因: {e}")
        has_gpu = False

    config['device'] = 'gpu' if has_gpu else 'cpu'
    logger.info(f"OCR 将使用设备: {config['device']}")

    # 更新模型路径为绝对路径
    config['SubModules']['TextDetection']['model_dir'] = str(detection_model_dir)
    config['SubModules']['TextRecognition']['model_dir'] = str(recognition_model_dir)

    # 保存临时配置文件
    temp_config_path = Path(__file__).parent / "OCR_temp.yaml"
    with open(temp_config_path, 'w', encoding='utf-8') as f:
        yaml.dump(config, f, default_flow_style=False)

    return temp_config_path
```

## 二、docker下使用
### 1.编写Dockerfile文件
```dockerfile
FROM ccr-2vdh3abv-pub.cnc.bj.baidubce.com/paddlex/paddlex:paddlex3.1.2-paddlepaddle3.0.0-cpu

# copy files
COPY pyproject.toml /app/

# Install uv.
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/

WORKDIR /app

RUN uv pip install . --system --no-cache --python $(which python3)

# 安装 hpi-cpu（利用缓存机制，只在依赖变动时重新执行）
RUN paddlex --install hpi-cpu

# Copy the application into the container.
COPY . .

EXPOSE 17654

CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "17654"]
```

### 2.生成镜像
```shell
docker buildx build --platform=linux/amd64 -t ocr_test:1.0.2 .
```

### 3.运行容器
```shell
docker container run -d -p 17654:17654 -v /root/log:/app/src/log -v /root/benchmark:/app/scripts/benchmark --name ocrTest ocr_test:1.0.2
```

### 4.进入容器查看
```shell
docker exec -it [containerId] bash
```

## 三、配置开机自启动
### 1.创建配置文件
```shell
touch /etc/systemd/system/paddle-ocr.service
```

### 2.编辑服务文件
```ini
[Unit]
Description=Paddle OCR Instance Startup
After=network.target

[Service]
Type=simple
ExecStart=/root/ocr/paddle_ocr/scripts/instance_start.sh
WorkingDirectory=/root/ocr/paddle_ocr/scripts
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```

### 3.重启
```shell
systemctl daemon-reload
```

### 4.设置开机启动
```shell
systemctl enable paddle-ocr.service
```

### 5.立即启动并查看日志
```shell
sudo systemctl start paddle-ocr.service
sudo systemctl status paddle-ocr.service
journalctl -u paddle-ocr.service -f
```

## 四、onnx模型转换
### 1.获取onnx模型
```shell
source .venv/bin/activate
python3 -m ensurepip --upgrade

paddlex --install paddle2onnx
```

### 2.转换模型
```shell
paddlex --paddle2onnx --paddle_model_dir ./models/PP-OCRv5_mobile_det_infer --onnx_model_dir ./onnx --opset_version 7
paddlex --paddle2onnx --paddle_model_dir ./models/PP-OCRv5_mobile_rec_infer --onnx_model_dir ./onnx --opset_version 7
```

## 五、监控
### 1.实时查看显卡情况
```shell
watch -n 1 nvidia-smi
```

## 六、问题
### 1.在gpu上运行paddlex报错：“[Hint: 'cudaErrorInitializationError'. The API call failed because the CUDA driver and runtime could not be initialized. ]”
解释：使用多进程时，运行异常

解决：将所有与paddlex相关的模块的import放在函数内


参考：
- [PaddleX OCR 文档](https://paddlepaddle.github.io/PaddleX/latest/pipeline_deploy/high_performance_inference.html)
- [PaddlePaddle问题解决](https://blog.csdn.net/qq_45779334/article/details/122024343)
