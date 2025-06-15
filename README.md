# VEO Sequential Video Generation Pipeline

A professional, production-ready Python implementation for generating sequential videos using Google's VEO-2 (Video Generation) model via Vertex AI. This pipeline enables the creation of coherent, multi-scene video sequences by chaining together individual video segments, using the last frame of each segment as the reference image for the next.

## 🎯 Overview

The VEO Sequential Video Generation Pipeline orchestrates the complete workflow of creating cinematic video sequences from a single reference image and a series of scene prompts. Each generated video segment seamlessly transitions into the next, creating a cohesive narrative flow ideal for storytelling, product demonstrations, or creative content generation.

### Key Features

- **Sequential Video Chaining**: Automatically extracts the last frame from each segment to seed the next generation
- **Robust Error Handling**: Comprehensive exception handling with custom error types and detailed logging
- **Production-Grade Authentication**: Seamless integration with Google Cloud Authentication (ADC)
- **Efficient Video Processing**: Memory-optimized frame extraction and FFmpeg-based video stitching
- **Configurable Parameters**: Centralized configuration with sensible defaults for video dimensions, quality, and API settings
- **Comprehensive Retry Logic**: Exponential backoff retry strategy for transient API failures
- **Cross-Platform Compatibility**: Works on Windows, macOS, and Linux environments

## 🏗️ Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│  Reference      │    │  VEO API         │    │  Video          │
│  Image          │───▶│  Generation      │───▶│  Segments       │
│                 │    │                  │    │                 │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │                        │
                                ▼                        ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│  Frame          │◀───│  Last Frame      │    │  FFmpeg         │
│  Extraction     │    │  Extraction      │    │  Stitching      │
│                 │    │                  │    │                 │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

## 📋 Prerequisites

### System Requirements

- Python 3.8 or higher
- FFmpeg installed and available in system PATH
- Google Cloud Project with Vertex AI API enabled
- Sufficient disk space for video processing (minimum 1GB recommended)

### Authentication Setup

1. **Install Google Cloud CLI**:
   ```bash
   # macOS
   brew install google-cloud-sdk
   
   # Ubuntu/Debian
   sudo apt-get update && sudo apt-get install google-cloud-cli
   
   # Windows: Download from https://cloud.google.com/sdk/docs/install
   ```

2. **Authenticate with Google Cloud**:
   ```bash
   gcloud auth application-default login
   gcloud config set project YOUR_PROJECT_ID
   ```

3. **Enable Required APIs**:
   ```bash
   gcloud services enable aiplatform.googleapis.com
   gcloud services enable storage.googleapis.com
   ```

## 🚀 Installation

### Option 1: Direct Installation

```bash
# Clone the repository
git clone https://github.com/chirindaopensource/veo-video-pipeline.git
cd veo-video-pipeline

# Install dependencies
pip install -r requirements.txt
```

### Option 2: Virtual Environment (Recommended)

```bash
# Create and activate virtual environment
python -m venv veo-pipeline-env

# Activate environment
# On Windows:
veo-pipeline-env\Scripts\activate
# On macOS/Linux:
source veo-pipeline-env/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### Dependencies

```text
google-cloud-aiplatform>=1.34.0
google-cloud-storage>=2.10.0
opencv-python>=4.8.0
Pillow>=10.0.0
requests>=2.31.0
tenacity>=8.2.0
numpy>=1.24.0
```

## 📖 Usage
### Advanced Usage with Custom Pipeline

```python
from veo_pipeline import VEOVideoPipeline, VEOConfig
from pathlib import Path

# Initialize pipeline with custom configuration
pipeline = VEOVideoPipeline(
    project_id="your-project-id",
    location="us-central1"
)

# Generate with custom parameters
output_path = pipeline.generate_sequential_videos(
    reference_image_path="reference.jpg",
    scene_prompts=[
        "Professional product showcase with dramatic lighting",
        "Smooth 360-degree rotation revealing product details",
        "Close-up shots highlighting key features",
        "Final hero shot with brand elements"
    ],
    output_folder_name="product_demo_2024"
)
```

### Environment Configuration

Set environment variables for seamless operation:

```bash
export GOOGLE_CLOUD_PROJECT="your-project-id"
export GOOGLE_APPLICATION_CREDENTIALS="path/to/service-account.json"  # Optional
export VEO_DEFAULT_LOCATION="us-central1"  # Optional
```

## ⚙️ Configuration

### VEOConfig Class

The pipeline uses a centralized configuration class for easy customization:

```python
class VEOConfig:
    # API Configuration
    MODEL_ID = "video-generation-001"
    API_TIMEOUT_SECONDS = 900
    
    # Video Parameters
    DEFAULT_VIDEO_LENGTH = "4s"
    DEFAULT_FPS = 24
    LANDSCAPE_DIMS = (1024, 576)
    PORTRAIT_DIMS = (576, 1024)
    
    # Quality Settings
    JPEG_ENCODING_QUALITY = 95
    
    # Retry Configuration
    RETRYABLE_STATUS_CODES = {429, 500, 502, 503, 504}
```

### Custom Configuration

```python
# Override default settings
VEOConfig.DEFAULT_VIDEO_LENGTH = "8s"
VEOConfig.DEFAULT_FPS = 30
VEOConfig.API_TIMEOUT_SECONDS = 1800
```

## 🧪 Testing

### Run Example Pipeline

```bash
python veo_pipeline.py
```

This will create a test video sequence using a generated reference image and sample prompts.

### Unit Tests

```bash
python -m pytest tests/
```

### Integration Tests

```bash
python -m pytest tests/integration/ --slow
```

## 🔧 Troubleshooting

### Common Issues

1. **Authentication Errors**
   ```
   DefaultCredentialsError: Could not automatically determine credentials
   ```
   **Solution**: Run `gcloud auth application-default login`

2. **FFmpeg Not Found**
   ```
   FileNotFoundError: ffmpeg not found
   ```
   **Solution**: Install FFmpeg and add to PATH
   - macOS: `brew install ffmpeg`
   - Ubuntu: `sudo apt update && sudo apt install ffmpeg`
   - Windows: Download from https://ffmpeg.org/

3. **API Quota Exceeded**
   ```
   HTTPError: 429 Too Many Requests
   ```
   **Solution**: The pipeline automatically retries with exponential backoff. Check your API quotas in the Google Cloud Console.

4. **Video Generation Timeout**
   ```
   VEOVideoGenerationError: Video generation failed
   ```
   **Solution**: Increase `API_TIMEOUT_SECONDS` in VEOConfig or simplify prompts.

### Debug Mode

Enable detailed logging:

```python
import logging
logging.getLogger().setLevel(logging.DEBUG)
```

## 📊 Performance Considerations

### Resource Usage

- **Memory**: ~2-4GB RAM during processing
- **Storage**: ~100-500MB per video segment
- **Network**: ~10-50MB per API request
- **Processing Time**: 2-5 minutes per segment

### Optimization Tips

1. **Batch Processing**: Process multiple sequences in parallel
2. **Image Preprocessing**: Resize images to optimal dimensions beforehand
3. **Prompt Engineering**: Use clear, specific prompts for better results
4. **Caching**: Implement local caching for frequently used reference images

## 🤝 Contributing

We welcome contributions! Please follow these guidelines:

1. **Code Style**: Adhere to PEP 8 standards
2. **Type Annotations**: Include type hints for all functions
3. **Documentation**: Update docstrings and README for new features
4. **Testing**: Add unit tests for new functionality
5. **Error Handling**: Maintain comprehensive error handling


### Development Setup

```bash
# Clone repository
git clone https://github.com/chirindaopensource/veo-video-pipeline.git
cd veo-video-pipeline

# Install development dependencies
pip install -r requirements-dev.txt

# Install pre-commit hooks
pre-commit install

# Run tests
python -m pytest
```

### Code Style

This project follows PEP 8 standards with the following tools:
- **Black**: Code formatting
- **isort**: Import sorting
- **flake8**: Linting
- **mypy**: Type checking

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- **Documentation**: N/A
- **Issues**: [GitHub Issues](https://github.com/chirindaopensource/veo-video-pipeline/issues)
- **Discussions**: [GitHub Discussions](https://github.com/chirindaopensource/veo-video-pipeline/discussions)
- **Email**: N/A

## 🙏 Acknowledgments

- Google Cloud Vertex AI team for the VEO model
- OpenCV community for computer vision tools
- FFmpeg project for video processing capabilities
- The Python packaging community for excellent tooling

## 📚 Related Projects

- [Google Cloud Vertex AI Python SDK](https://github.com/googleapis/python-aiplatform)
- [OpenCV Python](https://github.com/opencv/opencv-python)
- [Pillow (PIL Fork)](https://github.com/python-pillow/Pillow)

---

**Built with ❤️ by CS Chirinda**

*For questions or support, please open an issue*
