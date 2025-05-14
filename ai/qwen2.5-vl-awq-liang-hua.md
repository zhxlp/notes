# Qwen2.5 VL AWQ量化

### 环境

* python 3.10
* ms-swift 3.4.1
* autoawq 0.2.9

### 步骤

安装环境

```
pip install 'ms-swift[all]' -U
pip install autoawq -U
```

量化

```
CUDA_VISIBLE_DEVICES=7,6,5,4 swift \
    export --model_type qwen2_5_vl --template qwen2_5_vl --model /data/share/models/Qwen--Qwen2.5-VL-72B-Instruct \
    --output_dir /data/share/models/Qwen--Qwen2.5-VL-72B-Instruct-AWQ \
    --dataset 'AI-ModelScope/alpaca-gpt4-data-zh#500' \
              'AI-ModelScope/alpaca-gpt4-data-en#500' \
              'modelscope/coco_2014_caption:validation#500' \
              'swift/VideoChatGPT:Generic#500' \
    --quant_n_samples 256 \
    --quant_batch_size -1 \
    --max_length 8192 \
    --quant_method awq \
    --quant_bits 4 

```

### 参考

{% embed url="https://github.com/modelscope/ms-swift/blob/main/examples/export/quantize/mllm/awq.sh" %}
