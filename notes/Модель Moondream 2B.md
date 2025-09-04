Новая vision модель для эйдж девайсов

Поддерживает структурированные выводы, улучшенное понимание текста, отслеживание взгляда.

▪️ HF: https://huggingface.co/vikhyatk/moondream2

▪️Demo: https://moondream.ai/playground

▪️Github: https://github.com/vikhyat/moondream

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from PIL import Image

model = AutoModelForCausalLM.from_pretrained(
    "vikhyatk/moondream2",
    revision="2025-01-09",
    trust_remote_code=True,
    # Uncomment to run on GPU.
    # device_map={"": "cuda"}
)

# Captioning
print("Short caption:")
print(model.caption(image, length="short")["caption"])

print("\nNormal caption:")
for t in model.caption(image, length="normal", stream=True)["caption"]:
    # Streaming generation example, supported for caption() and detect()
    print(t, end="", flush=True)
print(model.caption(image, length="normal"))

# Visual Querying
print("\nVisual query: 'How many people are in the image?'")
print(model.query(image, "How many people are in the image?")["answer"])

# Object Detection
print("\nObject detection: 'face'")
objects = model.detect(image, "face")["objects"]
print(f"Found {len(objects)} face(s)")

# Pointing
print("\nPointing: 'person'")
points = model.point(image, "person")["points"]
print(f"Found {len(points)} person(s)")


https://huggingface.co/vikhyatk/moondream2
```
`
