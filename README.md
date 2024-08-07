# Huggingface 모델 gguf 모델 포멧으로 변경하는 방법

### GGUF 란?
- GGUF는 GPU 가속을 위한 모델 파일 형식
- 모델을 Hugging Face에서 다운로드한 후, llama.cpp를 사용하여 모델을 GGUF 형식으로 변환하면 GPU 환경에서 효율적으로 모델을 실행할 수 있음
- 모델의 성능을 극대화하고, 실행 속도를 높이며, 메모리 사용을 최적화
- 참고 :  https://huggingface.co/docs/hub/gguf

<br>

### Tutorial 내용 
- LLaMA3 (https://huggingface.co/beomi/Llama-3-Open-Ko-8B) 모델 다운로드 후 GGUF 변경
- GGUF 변경 후 4bit 양자화
- 변경된 gguf 파일을 olama를 통해 Serving

<br><br><br>

# Huggingface에서 모델 다운로드
```bash
# 모델 다운로드
# 출처 : https://huggingface.co/beomi/Llama-3-Open-Ko-8B
git lfs install
git clone https://huggingface.co/beomi/Llama-3-Open-Ko-8B
```

<br><br><br>
# GGUF 포멧으로 변경
```bash

# llama.cpp git clone 후 convert에 필요한 패키지 설치
git clone https://github.com/ggerganov/llama.cpp.git
cd llama.cpp
git pull
pip install -r reqiurement.txt
cd ..

# gguf 포멧으로 모델 변경
# python llama.cpp/convert_hf_to_gguf.py {변경 될 모델이 저장된 경로}
# 실행이 완료 된 후 해당 폴더 안에 ggml-model-f16.gguf 파일이 생성 됨
python llama.cpp/convert_hf_to_gguf.py Llama-3-Open-Ko-8B
```

<br><br><br>

# Ollama 에서 모델 사용
- ModelFile 작성 (아래의 내용을 Copy 후 ModlFile 생성)
```
# FROM {gguf로 변환 된 모델파일 경로}
FROM ggml-model-f16.gguf

TEMPLATE """{{- if .System }}
<s>{{ .System }}</s>
{{- end }}
<s>Human:
{{ .Prompt }}</s>
<s>Assistant:
"""

SYSTEM """You are a helpful a assistant"""

PARAMETER temperature 0
PARAMETER num_predict 3000
PARAMETER num_ctx 4096
PARAMETER stop <s>
PARAMETER stop </s>
```

- Ollama 에서 모델 생성
```bash
# ollama 설치 필수
# ollama create {모델이름지정} -f {앞에서 작성한 모델파일 경로}
ollama create llama3-korean -f ModelFile

# 모델 생성 결과 확인 
ollama list
```