# API 测试指南

服务运行后，可以通过以下方式测试：

**Curl:**
```bash
curl -X POST -F "file=@./test_dataset/surface/surface57.png" http://<服务器IP>:9000/api/v1/predict
```

**Python 脚本:** (参考 `svm_request测试.py`)
```python
import requests

url = 'http://<服务器IP>:9000/api/v1/predict'
file_path = './test_dataset/surface/surface57.png' # 使用实际路径

try:
    with open(file_path, 'rb') as f:
        files = {'file': f}
        response = requests.post(url, files=files)
        response.raise_for_status() # 检查请求是否成功
        print(response.json())
except FileNotFoundError:
    print(f"Error: File not found at {file_path}")
except requests.exceptions.RequestException as e:
    print(f"Error during request: {e}")
```

**使用 Python 脚本 (通过命令行参数传递图像路径):**

您还可以使用项目提供的 `svm_request_dynamic.py` 或 `svm_request_simplified.py` 脚本来测试 API。这些脚本允许您通过命令行参数直接指定图像文件的路径。

1.  **使用 `svm_request_dynamic.py` (输出详细JSON):**
    此脚本会发送图像到 API 并打印完整的 JSON 响应。

    ```bash
    python svm_request_dynamic.py <你的图像路径>
    ```
    例如:
    ```bash
    python svm_request_dynamic.py ./test_dataset/corona/corona111.png
    ```
    或者使用绝对路径:
    ```bash
    python svm_request_dynamic.py /path/to/your/image.png
    ```

2.  **使用 `svm_request_simplified.py` (输出简化结果):**
    此脚本会发送图像到 API 并仅打印预测的类别和置信度。

    ```bash
    python svm_request_simplified.py <你的图像路径>
    ```
    例如:
    ```bash
    python svm_request_simplified.py ./test_dataset/surface/surface57.png
    ```
    或者使用绝对路径:
    ```bash
    python svm_request_simplified.py /path/to/your/other_image.jpg
    ```

> **注意:**
> - 请将 `<你的图像路径>` 替换为实际的图像文件路径。
> - 确保 API 服务 (`svm_fastapi.py`) 正在运行。
> - 如果脚本与 API 服务不在同一台机器上，请修改脚本中的 `url` 变量，将其中的 `127.0.0.1` 或 `localhost` 替换为 API 服务器的实际 IP 地址。