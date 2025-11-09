⚡️ LLaVA-Mini: Efficient Image and Video Large Multimodal Models with One Vision Token

LaVA-Mini достигает производительности LLaVA-v1.5 с 1 токеном (vs 576), сокращая количество FLOP на 77%, задержку со 100 мс до 40 мс и VRAM с 360 МБ до 0,6 МБ, обеспечивая 3-часовую обработку видео и 10 000 кадров на 24 ГБ GPU 🔥.

Установка: 
```python
conda create -n llavamini python=3.10 -y
conda activate llavamini
pip install -e .
pip install -e ".[train]"
pip install flash-attn --no-build-isolation
```

HF: https://huggingface.co/ICTNLP/llava-mini-llama-3.1-8b

Github: https://github.com/ictnlp/LLaVA-Mini