---
title: "[etc] Hugging Face 모델 캐시 여러군데에서 쓰는 방법"
description: "여러 fastapi 서버 인스턴스가 같은 hugging face 모델 캐시 디렉토리를 공유하는 방법"
date: 2025-08-21 12:09:00 +0900
categories: [ "etc", "else" ]
tags: [ "huggingface", "model" ]
pin: false
math: false
mermaid: false
---

리비전 고정, 캐시 디렉토리 공유, 오프라인 실행을 통해 안정적인 배포가 가능한데 그 방법에 대해 설명합니다.

# 공용 캐시 디렉토리 설정

``` bash
export HF_HOME=/mnt/hf_cache
export TRANSFORMERS_CACHE=$HF_HOME/transformers
export HF_HUB_CACHE=$HF_HOME/hub
export HF_DATASETS_CACHE=$HF_HOME/datasets
export TOKENIZERS_PARALLELISM=false
```

# 모델 사전 다운로드

배포 전에 원하는 모델을 특정 리비전(커밋 해시)으로 다운로드합니다:

``` bash
huggingface-cli download Qwen/Qwen2.5-VL-7B   --revision 3f6b9d2   --local-dir /mnt/hf_cache/Qwen2.5-VL-7B   --local-dir-use-symlinks False
```

-   `--revision` : 모델 버전을 고정 (예: 특정 커밋 해시)
-   `--local-dir-use-symlinks False` : 심볼릭 링크 대신 실제 파일 저장
    (NFS 환경에서 안전)

# 사용

``` python
from fastapi import FastAPI
from transformers import AutoModelForCausalLM, AutoTokenizer

app = FastAPI()

MODEL_DIR = "/mnt/hf_cache/Qwen2.5-VL-7B"
REVISION = "3f6b9d2"  # 고정된 리비전

@app.on_event("startup")
def load_model():
    global model, tokenizer
    tokenizer = AutoTokenizer.from_pretrained(
        MODEL_DIR, revision=REVISION, local_files_only=True
    )
    model = AutoModelForCausalLM.from_pretrained(
        MODEL_DIR, revision=REVISION, local_files_only=True, device_map="auto"
    )
    print("[INFO] Model loaded from shared cache")

@app.get("/infer")
def infer(prompt: str):
    inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
    outputs = model.generate(**inputs, max_length=128)
    return {"result": tokenizer.decode(outputs[0], skip_special_tokens=True)}
```

# 실행

``` bash
HF_HOME=/mnt/hf_cache TRANSFORMERS_OFFLINE=1 gunicorn -w 2 -k uvicorn.workers.UvicornWorker server:app
```

-   `TRANSFORMERS_OFFLINE=1` : 인터넷 접근 없이 캐시에서만 모델 로드
-   여러 FastAPI 인스턴스가 같은 `/mnt/hf_cache` 공유

# 권한 설정

모든 서버 계정이 캐시 디렉토리에 접근할 수 있도록 권한을 설정합니다:

``` bash
sudo mkdir -p /mnt/hf_cache
sudo chmod -R 775 /mnt/hf_cache
sudo chown -R $USER:$USER /mnt/hf_cache
```

