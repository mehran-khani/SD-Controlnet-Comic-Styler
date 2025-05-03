# SD ControlNet Comic Styler

This project leverages Stable Diffusion, ControlNet, and LoRAs to transform input images into a specific retro Western comic book style while preserving the original pose. It uses a two-stage process involving initial low-resolution generation guided by OpenPose and a subsequent high-resolution upscaling stage using ControlNet Tile.

## Project Status

*   **Current Stage:** Functional prototype demonstrating the core pipeline.
*   **Environment:** Developed and tested within a Kaggle Notebook environment.

## Key Features

*   **Pose Preservation:** Uses ControlNet with OpenPose preprocessor to maintain the pose from the input image.
*   **Style Transfer:** Applies specific LoRAs to achieve a targeted artistic style (currently focused on Retro Western Comics).
*   **Tiling Upscaling:** Employs ControlNet Tile for generating higher-resolution images with consistent detail, overcoming limitations of generating large images directly.
*   **Customizable Prompts:** Uses distinct prompts for the initial generation and the tiling upscaling stages for better control.
*   **Modular:** Built using the Hugging Face `diffusers` library, allowing for potential swapping of models or components.

## Technology Stack

*   **Python 3**
*   **PyTorch**
*   **Hugging Face Libraries:**
    *   `diffusers`: For Stable Diffusion pipelines and ControlNet integration.
    *   `transformers`
*   **ControlNet Models:**
    *   `lllyasviel/ControlNet` (OpenPose model for pose detection)
    *   `lllyasviel/control_v11f1e_sd15_tile` (Tile model for upscaling)
*   **Base Model:** `runwayml/stable-diffusion-v1-5`
*   **Auxiliary Libraries:** `Pillow` (PIL), `OpenCV-Python`, `numpy`

## Workflow Overview

1.  **Image Preparation:** The input image is loaded, converted to RGB, and resized to a fixed square dimension (e.g., 512x512), ignoring the original aspect ratio for initial processing compatibility.
2.  **OpenPose Extraction:** An OpenPose map (skeleton, face, hands) is generated from the prepared input image using the `OpenposeDetector`.
3.  **Initial Generation (Low-Res):**
    *   A Stable Diffusion Img2Img pipeline combined with the OpenPose ControlNet is used.
    *   Input: Prepared image, OpenPose map.
    *   Prompts: Combines a `base_prompt` (subject focus), a `style_prompt` (artistic style, LoRA triggers), and a randomly selected `background_prompt`.
    *   LoRAs are loaded and activated (`style` and `detail`).
    *   Output: A low-resolution image (e.g., 512x512) reflecting the pose and desired style.
4.  **Tiling Upscaling (High-Res):**
    *   The low-res output is resized (e.g., 2x) using LANCZOS to create a blurry high-resolution base.
    *   A Stable Diffusion Img2Img pipeline combined with the *Tile* ControlNet is used.
    *   The blurry high-res image is processed in overlapping tiles (e.g., 1024x1024 tiles).
    *   Input for each tile: The corresponding crop from the blurry high-res image (used as both `image` and `control_image` for the Tile model).
    *   Prompts: Uses refined prompts (`positive_prompt_tile`, `negative_prompt_tile`) focusing on detail and style, *omitting* the background description and *adding* negatives specific to tiling artifacts (blurriness, seams, face distortion).
    *   LoRAs are loaded and activated again on this pipeline.
    *   Output: Each processed tile is blended back onto a final canvas, resulting in a high-resolution stylized image.

## LoRAs Used

This implementation currently utilizes the following LoRAs:

1.  **Western Comics Style:**
    *   **Purpose:** To impart the core retro comic book aesthetic (limited palette, 1940s feel, specific line art).
    *   **Source:** [https://civitai.com/models/1081588/western-comics-style](https://civitai.com/models/1081588/western-comics-style)
    *   **Trigger Keywords (in `style_prompt`):** `LIMITED PALETTE`, `RETRO COMIC`, `1940S (STYLE)`, `PARTIALLY COLORED`, `WESTERN COMICS (STYLE)`, `NIGHT COMIC` (adjust weights as needed).

2.  **Detail Tweaker LoRA:**
    *   **Purpose:** To enhance fine details and sharpness, often counteracting some softness from diffusion models.
    *   **Source:** [https://civitai.com/models/58390/detail-tweaker-lora-lora](https://civitai.com/models/58390/detail-tweaker-lora-lora)
    *   **Activation:** Loaded with the adapter name `detail` and activated alongside the style LoRA. May not require specific trigger words but influences the overall detail level.

## Future Work & Experimentation

This project is a starting point, and there are many avenues for improvement and exploration:

*   **LoRA Exploration:**
    *   Experiment with different style LoRAs to achieve varied artistic outputs.
    *   Try merging the current 'Western Comics Style' LoRA with others (e.g., character-specific, background-specific, or different detail enhancers) using tools like SuperMerger.
    *   Fine-tune LoRA weights (`adapter_weights`) for both generation stages.
*   **Upscaling Enhancement:**
    *   Refine `positive_prompt_tile` and `negative_prompt_tile` further to combat artifacts and improve detail quality.
    *   Experiment with different `tile_size`, `tile_stride`, and `overlap` values.
    *   Adjust `strength_tile`, `guidance_scale_tile`, and `num_inference_steps_tile` for the upscaling pipeline.
    *   Explore alternative upscaling methods (e.g., integrating dedicated ESRGAN/Real-ESRGAN models before or after the ControlNet Tile stage, or using different ControlNet upscaling models).
*   **Prompt Engineering:** Improve background generation, explore negative prompt optimization.
*   **Input Handling:** Implement aspect ratio preservation during initial resizing.
*   **User Interface:** Develop a Gradio or Streamlit interface for easier use.

## Next Steps: Hugging Face Space

The plan is to create a public demonstration of this pipeline on **Hugging Face Spaces**. The code specifically tailored for the Space (e.g., Gradio interface, dependency handling) will be **committed to a separate GitHub repository**.

## License
MIT License 
