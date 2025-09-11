---
title: "[Hugging Face] 여러 서버에서 공용 모델 캐시 디렉토리 활용하기"
description: "여러 FastAPI 인스턴스가 같은 Hugging Face 모델 캐시를 공유하도록 설정하는 방법을 정리합니다."
date: 2025-08-21 12:09:00 +0900
categories: ["infrastructure", "ml"]
tags: ["huggingface", "model", "cache", "deployment"]
pin: false
math: false
mermaid: false
---

여러 FastAPI 서버가 동일한 Hugging Face 모델 캐시를 공유하도록 설정하면 배포 속도와 안정성이 크게 향상됩니다.  
이 글에서는 **공용 캐시 디렉토리 설정 → 모델 사전 다운로드 → 오프라인 실행** 순서로 정리합니다.

## 1. 공용 캐시 디렉토리 설정

```bash
export HF_HOME=/mnt/hf_cache
export TRANSFORMERS_CACHE=$HF_HOME/transformers
export HF_HUB_CACHE=$HF_HOME/hub
export HF_DATASETS_CACHE=$HF_HOME/datasets
export TOKENIZERS_PARALLELISM=false
```

## 2. 모델 사전 다운로드

특정 리비전을 고정하여 사전에 모델을 내려받습니다.

```bash
huggingface-cli download Qwen/Qwen2.5-VL-7B   --revision 3f6b9d2   --local-dir /mnt/hf_cache/Qwen2.5-VL-7B   --local-dir-use-symlinks False
```

* `--revision` : 모델 버전을 특정 커밋으로 고정  
* `--local-dir-use-symlinks False` : 심볼릭 링크 대신 실제 파일 저장 (NFS 환경에서 안전)

## 3. FastAPI에서 사용하기

```python
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

## 4. 오프라인 실행

```bash
HF_HOME=/mnt/hf_cache TRANSFORMERS_OFFLINE=1 gunicorn -w 2 -k uvicorn.workers.UvicornWorker server:app
```

* `TRANSFORMERS_OFFLINE=1` : 인터넷 접근 없이 캐시에서만 모델 로드  
* 여러 FastAPI 인스턴스가 같은 `/mnt/hf_cache` 공유 가능

## 5. 권한 설정

모든 서버 계정이 캐시 디렉토리에 접근할 수 있도록 권한을 조정합니다.

```bash
sudo mkdir -p /mnt/hf_cache
sudo chmod -R 775 /mnt/hf_cache
sudo chown -R $USER:$USER /mnt/hf_cache
```

---

## 정리

- Hugging Face 모델을 사전 다운로드해 리비전을 고정하면 배포 시 예측 가능성이 높아집니다.  
- 공용 캐시 디렉토리를 설정하면 여러 서버 인스턴스가 동일한 모델을 공유할 수 있어 효율적입니다.  
- 오프라인 모드(`TRANSFORMERS_OFFLINE=1`)를 사용하면 네트워크 장애와 무관하게 안정적인 서비스가 가능합니다.
