
# Tutorial: Processing Model Coordinate Outputs

**Note**: For complete action space parsing, please refer to [OSWorld `uitars_agent.py`](https://github.com/xlang-ai/OSWorld/blob/main/mm_agents/uitars_agent.py). This tutorial assumes that we have already obtained the raw coordinate output from the model and will process it to determine the actual position in the image that the model intends to click.

---

## Steps:

1. **Visualize the Model's Output Coordinates**  
   Use the code provided below to process the model's output and visualize the coordinates on the image.

## Code Example
```python
# Assume model output
model_raw_response = """Thought: xxx
Action: click(start_box='(197,525)')"""

# Please use re to parse the coordinate values
model_output_width = 197
model_output_height = 525

from PIL import Image
import matplotlib.pyplot as plt

import json
import base64
from io import BytesIO
from PIL import Image

import math

IMAGE_FACTOR = 28
MIN_PIXELS = 100 * 28 * 28
MAX_PIXELS = 16384 * 28 * 28
MAX_RATIO = 200

VIDEO_MIN_PIXELS = 128 * 28 * 28
VIDEO_MAX_PIXELS = 768 * 28 * 28
FRAME_FACTOR = 2
FPS = 2.0
FPS_MIN_FRAMES = 4
FPS_MAX_FRAMES = 768

def round_by_factor(number: int, factor: int) -> int:
    """Returns the closest integer to 'number' that is divisible by 'factor'."""
    return round(number / factor) * factor

def ceil_by_factor(number: int, factor: int) -> int:
    """Returns the smallest integer greater than or equal to 'number' that is divisible by 'factor'."""
    return math.ceil(number / factor) * factor

def floor_by_factor(number: int, factor: int) -> int:
    """Returns the largest integer less than or equal to 'number' that is divisible by 'factor'."""
    return math.floor(number / factor) * factor

def smart_resize(
    height: int, width: int, factor: int = IMAGE_FACTOR, min_pixels: int = MIN_PIXELS, max_pixels: int = MAX_PIXELS
) -> tuple[int, int]:
    """
    Rescales the image so that the following conditions are met:

    1. Both dimensions (height and width) are divisible by 'factor'.

    2. The total number of pixels is within the range ['min_pixels', 'max_pixels'].

    3. The aspect ratio of the image is maintained as closely as possible.
    """
    if max(height, width) / min(height, width) > MAX_RATIO:
        raise ValueError(
            f"absolute aspect ratio must be smaller than {MAX_RATIO}, got {max(height, width) / min(height, width)}"
        )
    h_bar = max(factor, round_by_factor(height, factor))
    w_bar = max(factor, round_by_factor(width, factor))
    if h_bar * w_bar > max_pixels:
        beta = math.sqrt((height * width) / max_pixels)
        h_bar = floor_by_factor(height / beta, factor)
        w_bar = floor_by_factor(width / beta, factor)
    elif h_bar * w_bar < min_pixels:
        beta = math.sqrt(min_pixels / (height * width))
        h_bar = ceil_by_factor(height * beta, factor)
        w_bar = ceil_by_factor(width * beta, factor)
    return h_bar, w_bar

# Open the image
img = Image.open('./data/coordinate_process_image.png')
width, height = img.size
print(f'Original resolution: {width}, {height}')
# Calculate the new dimensions
new_height, new_width = smart_resize(height, width)
new_coordinate = (int(model_output_width/new_width * width), int(model_output_height/new_height * height))
print(f'Resized resolution: {new_width}, {new_height}')
print(new_coordinate)

# Display the image
plt.imshow(img)
plt.scatter([new_coordinate[0]], [new_coordinate[1]], c='red', s=50)  # Mark the point with a red dot
plt.title('Visualize Coordinate')
plt.axis('off')  # Set to 'off' to hide the axes
plt.savefig('./data/coordinate_process_image_som.png', dpi=350)
```

2. The output SOM image should look like this:

![Output SOM Image](./data/coordinate_process_image_som.png)
