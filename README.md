# Masked Thought


### 1. Quick Start
#### 1) Installation
```bash
conda create -n mask python=3.10
conda activate mask
pip install -r requirements.txt
pip install torch==2.1.0 xformers torchvision==0.16.0 torchaudio==2.1.0 --index-url https://download.pytorch.org/whl/cu118
# install vllm
export VLLM_VERSION=0.2.2
export PYTHON_VERSION=310
pip install https://github.com/vllm-project/vllm/releases/download/v${VLLM_VERSION}/vllm-${VLLM_VERSION}+cu118-cp${PYTHON_VERSION}-cp${PYTHON_VERSION}-manylinux1_x86_64.whl torch==2.1.0 transformers==4.36.2
```
#### 2) Train
You can start with training GSM8K on Llama-2-7b with the following command, it will take about one hour with 8 NVIDIA
A100 GPUs.
```bash
bash training_scripts/run_llama2_7b_gsm8k_mft.sh
```
#### 3) Evaluation
The evaluation take 1 minute using vllm and 1 single A100 GPU.
```bash
# exp_name is the experiment identifier in your training script.
bash evaluation/run_gen_math_greedy_vllm_1.sh ${exp_name}
python evaluation/get_gsm8k_res.py --model ${exp_name}
```

For training and evaluation on other datasets and models, you can refer to ```./training_scripts```, ```./MetaMath``` and ```./MAmmoTH```.

### 2. Train with your own code
If you'd like incorporate MFT into your own code, just add the following codes before feeding `input_ids` to the model. `MASK_INDEX` can be the `token_id` of a new added token`[mask]` or the`[pad]` in the original tokenizer, depending on your preference.
```python
def mask_target_tokens(input_ids, sources_tokenized, mask_probability, MASK_INDEX, tokenizer):
    masked_input_ids = copy.deepcopy(input_ids)
    for input_id, source_len in zip(masked_input_ids, sources_tokenized["input_ids_lens"]):
        for i in range(source_len, len(input_id)):
            if random.random() < mask_probability:
                input_id[i] = MASK_INDEX
    return masked_input_ids
input_ids = mask_target_tokens(input_ids, sources_tokenized, _mask_probability, MASK_INDEX)
```

