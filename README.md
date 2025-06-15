# VEO Sequential Video Generation Pipeline

# VEO Video Pipeline: Sequential Animation Tool for Google Vertex AI

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python Version](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![Code Style: PEP-8](https://img.shields.io/badge/code%20style-PEP--8-orange.svg)](https://www.python.org/dev/peps/pep-0008/)

**Repository:** [https://github.com/chirindaopensource/veo-video-pipeline](https://github.com/chirindaopensource/veo-video-pipeline)  
**Author:** 2025 Craig Chirinda (Open Source Projects)

## Overview

The VEO Video Pipeline is a Python-based tool designed to orchestrate the generation of sequential video animations using Google's state-of-the-art Video Generation Model (VEO) via the Vertex AI API. This project provides a robust, cost-efficient, and extensible framework for transforming a series of text prompts, optionally seeded by an initial reference image, into a cohesive video narrative. The entire pipeline is encapsulated within a single iPython Notebook (`veo2_animation_tool.ipynb`) for ease of use, experimentation, and modification.

This tool is particularly useful for creators, researchers, and developers looking to:
*   Generate short video clips from text prompts.
*   Create longer animated sequences by chaining multiple VEO generations.
*   Leverage an initial image to guide the style and content of the first video segment.
*   Automate the process of using the last frame of a generated segment as the reference for the next, ensuring visual continuity.

The implementation prioritizes robustness through comprehensive error handling, retry mechanisms for API calls, and efficient resource management, including memory-efficient image processing and video stitching.

## Key Features

*   **Sequential Video Generation:** Generates multiple video segments based on an ordered list of prompts.
*   **Image-to-Video & Video-to-Video:**
    *   Initiates the sequence with a user-provided reference image.
    *   Automatically uses the last frame of the previously generated segment as the reference for the next, enabling chained animations.
*   **Google VEO Model Integration:** Utilizes the `video-generation-001` model via the synchronous Vertex AI API.
*   **Robust API Interaction:** Implements `tenacity`-based exponential backoff and retry logic for transient network or API errors.
*   **Cost-Conscious Design:**
    *   Optimized image encoding (JPEG) to reduce payload sizes.
    *   Recommends appropriate video dimensions.
    *   Handles GCS downloads efficiently.
*   **Efficient Video Stitching:** Uses `ffmpeg` (via `subprocess`) for fast, lossless concatenation of video segments.
*   **Configuration Management:** Centralized configuration (`VEOConfig`) for model parameters, API endpoints, and retry settings.
*   **Error Handling:** Custom exceptions (`VEOVideoGenerationError`) for pipeline-specific issues.
*   **Standardized Logging:** Clear and informative logging for monitoring and debugging.
*   **Self-Contained Notebook:** All logic, including helper functions and the main pipeline class, is within `veo2_animation_tool.ipynb`.

## How It Works (High-Level)

1.  **Initialization & Authentication:** Sets up Google Cloud credentials and project details.
2.  **Input Validation:** Checks the validity of the reference image path, scene prompts, and output folder name.
3.  **Initial Reference:** Loads the user-provided reference image, resizes it to VEO-compatible dimensions, and encodes it to Base64.
4.  **Iterative Segment Generation:** For each prompt in the sequence:
    *   Constructs a request payload including the prompt and the current reference image data (either the initial image or the last frame of the previous segment).
    *   Calls the Vertex AI VEO API to generate a video segment.
    *   Handles the API response, which may contain video bytes directly or a GCS URI for download.
    *   Saves the generated segment locally.
    *   If it's not the last prompt, extracts the last frame of the newly generated segment, resizes, and encodes it to Base64 to serve as the reference for the next iteration.
5.  **Video Stitching:** Once all segments are generated, they are concatenated into a single MP4 file using `ffmpeg`.
6.  **Output:** The final stitched video and individual segments are saved to a specified output directory in the user's home folder.

## Prerequisites

Before running the notebook, ensure you have the following:

1.  **Python:** Version 3.8 or higher.
2.  **Google Cloud Platform (GCP) Account:**
    *   A GCP Project with billing enabled.
    *   The **Vertex AI API** enabled for your project.
3.  **`gcloud` CLI:** Google Cloud Command Line Interface installed and configured.
    *   Authenticate by running: `gcloud auth application-default login`
4.  **`ffmpeg`:** The `ffmpeg` command-line tool must be installed and accessible in your system's PATH.
    *   Download from [ffmpeg.org](https://ffmpeg.org/download.html).
5.  **Jupyter Notebook or JupyterLab:** To run the `.ipynb` file.

## Installation & Setup

1.  **Clone the Repository:**
    ```bash
    git clone https://github.com/chirindaopensource/veo-video-pipeline.git
    cd veo-video-pipeline
    ```

2.  **Create a Virtual Environment (Recommended):**
    ```bash
    python -m venv veo_env
    source veo_env/bin/activate  # On Windows: veo_env\Scripts\activate
    ```

3.  **Install Python Dependencies:**
    The necessary Python packages are listed in the import section of the notebook. You can install them using pip:
    ```bash
    pip install google-cloud-aiplatform google-cloud-storage opencv-python Pillow requests tenacity numpy jupyterlab
    ```
    (Consider creating a `requirements.txt` file for easier dependency management in future project iterations).

4.  **Verify `ffmpeg` Installation:**
    Open your terminal and type:
    ```bash
    ffmpeg -version
    ```
    If it's installed correctly, you'll see version information. If not, please install it and ensure it's in your system PATH.

5.  **Set Google Cloud Project ID:**
    The script will attempt to auto-detect your GCP Project ID. However, it's best practice to set it explicitly via an environment variable:
    ```bash
    export GOOGLE_CLOUD_PROJECT="your-gcp-project-id"
    ```
    Alternatively, you can pass it as an argument when calling `create_veo_video_pipeline` within the notebook.

## Configuration

The primary configuration happens within the `veo2_animation_tool.ipynb` notebook, specifically in the example usage block (`if __name__ == "__main__":` or a similar cell designed for execution).

Key parameters to configure:

*   `reference_image_path` (str): Absolute or relative path to your initial reference image (e.g., `.jpg`, `.png`). A dummy image will be created if not found, but using your own is recommended.
*   `scene_prompts` (List[str]): A list of text prompts, where each prompt describes a scene. The videos will be generated in the order of these prompts.
*   `output_folder_name` (str): Name of the folder to be created in your user's home directory to store generated segments and the final video.
*   `project_id` (Optional[str]): Your Google Cloud Project ID. If `None`, the pipeline attempts to auto-detect it or use the `GOOGLE_CLOUD_PROJECT` environment variable.

Advanced configuration options (e.g., video length, FPS, API endpoints) are managed within the `VEOConfig` class and can be modified directly in the notebook if needed, though the defaults are generally sensible.

## Usage

1.  **Launch JupyterLab or Jupyter Notebook:**
    Navigate to the cloned repository directory in your terminal and run:
    ```bash
    jupyter lab
    # or
    # jupyter notebook
    ```

2.  **Open the Notebook:**
    In the Jupyter interface, open `veo2_animation_tool.ipynb`.

3.  **Configure Parameters:**
    Locate the main execution cell (typically at the end of the notebook, often guarded by an `if __name__ == "__main__":` equivalent for notebooks or a clearly marked "Run" cell).
    Modify the `example_params` dictionary with your desired `reference_image_path`, `scene_prompts`, and `output_folder_name`.

    ```python
    # Example configuration within the notebook:
    example_params = {
        "reference_image_path": "path/to/your/image.jpg",
        "scene_prompts": [
            "A futuristic cityscape at dusk, neon lights reflecting on wet streets.",
            "A sleek flying car zips through the city canyons.",
            "The car lands on a skyscraper rooftop, overlooking the sprawling metropolis."
        ],
        "output_folder_name": "my_veo_animation",
        # "project_id": "your-gcp-project-id"  # Optional: uncomment and set if needed
    }
    ```

4.  **Run the Notebook:**
    Execute the cells in the notebook sequentially, or use "Run All Cells." The video generation process can take several minutes per segment, depending on the VEO model's load and video length. Monitor the logging output for progress.

5.  **Locate Output:**
    Upon successful completion, the individual video segments and the final stitched video (`final_stitched_video.mp4`) will be saved in `~/output_folder_name/` (e.g., `~/my_veo_animation/`).

## Core Components (within `veo2_animation_tool.ipynb`)

*   **`VEOConfig` (class):** A centralized configuration class holding constants like API model ID, default video parameters, retryable status codes, etc.
*   **`VEOVideoGenerationError` (class):** Custom exception for pipeline-specific errors.
*   **`VEOVideoPipeline` (class):** The main class orchestrating the video generation workflow.
    *   `__init__(...)`: Initializes authentication and GCP settings.
    *   `_initialize_authentication()`: Handles Google Cloud ADC.
    *   `_validate_inputs(...)`: Validates user-provided parameters.
    *   `_load_and_encode_image(...)`: Loads, resizes (preserving aspect ratio for VEO's recommended dimensions), and Base64 encodes images.
    *   `_generate_video_segment(...)`: Makes the API call to Vertex AI VEO, with retries.
    *   `_download_video_from_gcs(...)`: Downloads video if API returns a GCS URI.
    *   `_extract_last_frame(...)`: Uses OpenCV to get the last frame of a video.
    *   `_frame_to_base64(...)`: Converts an OpenCV frame to a Base64 encoded string for the next API call.
    *   `_stitch_videos_ffmpeg(...)`: Concatenates video segments using `ffmpeg`.
    *   `generate_sequential_videos(...)`: The main public method that executes the entire pipeline.
*   **`create_veo_video_pipeline(...)` (function):** A factory function providing a simpler interface to instantiate and run the `VEOVideoPipeline`.
*   **Helper Functions:** Such as `_is_retryable_http_error` for custom retry logic.

## Cost Considerations

Generating videos with Google's VEO model via Vertex AI incurs costs. Please refer to the [Vertex AI pricing page](https://cloud.google.com/vertex-ai/pricing) for the most up-to-date information on "Generative AI models" or "Multimodal models."

This pipeline uses the synchronous API, which might have different pricing characteristics than batch prediction. Costs are typically based on the duration of the video generated.

This tool attempts to be cost-efficient by:
*   Using recommended image dimensions to avoid unnecessary upscaling/downscaling by the model.
*   Encoding images to JPEG with reasonable quality to reduce request payload sizes.
*   Performing frame extraction and video stitching locally, avoiding additional cloud service costs for these tasks.

**Monitor your GCP billing dashboard regularly when using this tool.**

## Troubleshooting

*   **Authentication Errors:**
    *   Ensure you've run `gcloud auth application-default login`.
    *   Verify the `GOOGLE_CLOUD_PROJECT` environment variable is set correctly or passed to the pipeline.
    *   Check if the Vertex AI API is enabled in your GCP project.
*   **`ffmpeg: command not found`:**
    *   `ffmpeg` is not installed or not in your system's PATH. Revisit the installation steps.
*   **API Quota Errors (e.g., HTTP 429):**
    *   You might be hitting API rate limits or project quotas. The retry logic will attempt to handle transient 429s, but persistent issues may require requesting a quota increase from Google Cloud.
*   **`FileNotFoundError` for reference image:**
    *   Double-check the `reference_image_path` provided.
*   **Long Generation Times:**
    *   Video generation is computationally intensive. Longer videos or higher FPS settings will take more time. The default is 4 seconds at 24 FPS.
*   **Permission Denied (Output Directory):**
    *   Ensure your user has write permissions to their home directory, where the output folder is created.

## Contributing

Contributions are welcome! If you have improvements, bug fixes, or new features you'd like to add, please follow these steps:

1.  **Fork the repository.**
2.  **Create a new branch** for your feature or fix: `git checkout -b feature/your-feature-name` or `git checkout -b fix/your-bug-fix`.
3.  **Make your changes** within the `veo2_animation_tool.ipynb` notebook. Ensure your code adheres to PEP-8 standards where applicable and is well-commented.
4.  **Test your changes thoroughly.**
5.  **Commit your changes:** `git commit -m "Add concise description of your changes"`.
6.  **Push to your forked repository:** `git push origin feature/your-feature-name`.
7.  **Open a Pull Request** to the `main` branch of `chirindaopensource/veo-video-pipeline`. Provide a clear description of your changes and why they are needed.

Please also feel free to open an issue for bug reports or feature requests.

## License

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.
