# Gemini API Complete Guide

A comprehensive guide to Google's Gemini API (2025 version) with examples, best practices, and detailed explanations optimized for LLM understanding.

## Table of Contents

1. [Overview](#overview)
2. [Models and Pricing](#models-and-pricing)
3. [Getting Started](#getting-started)
4. [Core Features](#core-features)
   - [Basic Text Generation](#basic-text-generation)
   - [System Instructions](#system-instructions)
   - [Multi-turn Conversations](#multi-turn-conversations)
5. [Advanced Features](#advanced-features)
   - [Thinking (Reasoning)](#thinking-reasoning)
   - [Function Calling](#function-calling)
   - [Structured Output](#structured-output)
   - [Long Context](#long-context)
   - [Grounding with Google Search](#grounding-with-google-search)
   - [URL Context Tool](#url-context-tool)
   - [Code Execution](#code-execution)
6. [Working with Media](#working-with-media)
   - [Files API](#files-api)
   - [Image Understanding](#image-understanding)
   - [Document Processing](#document-processing)
   - [Video and Audio](#video-and-audio)
7. [Embeddings](#embeddings)
8. [Optimization Techniques](#optimization-techniques)
   - [Context Caching](#context-caching)
   - [Token Management](#token-management)
9. [Prompt Engineering](#prompt-engineering)
10. [Best Practices](#best-practices)

## Overview

The Gemini API has been completely redesigned. The new API uses a different structure and client initialization:

```python
from google import genai

# New client initialization
client = genai.Client(api_key="YOUR_API_KEY")

# Generate content
response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents="Explain how AI works in a few words",
)
print(response.text)
```

## Models and Pricing

### Available Models

#### Gemini 2.5 Pro Preview
- **Model ID**: `gemini-2.5-pro-preview-05-06`
- **Context Window**: 1,048,576 tokens input, 65,536 tokens output
- **Best For**: Complex reasoning, coding, STEM problems, large dataset analysis
- **Features**: Thinking mode, structured output, caching, function calling, code execution, search grounding

**Pricing (per 1M tokens)**:
- Input: $1.25 (≤200k tokens), $2.50 (>200k tokens)
- Output: $10.00 (≤200k tokens), $15.00 (>200k tokens)
- Thinking Tokens: Included in output pricing
- Context Caching: $0.31-$0.625 + $4.50/hour storage

#### Gemini 2.5 Flash Preview
- **Model ID**: `gemini-2.5-flash-preview-05-20`
- **Context Window**: 1,048,576 tokens input, 65,536 tokens output
- **Best For**: Fast responses, cost efficiency, adaptive thinking
- **Features**: Thinking budgets, all Pro features

**Pricing (per 1M tokens)**:
- Input: $0.15 (text/image/video), $1.00 (audio)
- Output: $0.60 (non-thinking), $3.50 (thinking)
- Context Caching: $0.0375-$0.25 + $1.00/hour storage

#### Other Models
- **Gemini 2.0 Flash**: Next-gen features, realtime streaming
- **Gemini 2.0 Flash-Lite**: Cost efficiency, low latency
- **Gemini 1.5 Pro/Flash**: Legacy models with proven performance

### Model Variants

```python
# Specialized variants
models = {
    "text-generation": "gemini-2.5-flash-preview-05-20",
    "native-audio": "gemini-2.5-flash-preview-native-audio-dialog",
    "text-to-speech": "gemini-2.5-flash-preview-tts",
    "image-generation": "gemini-2.0-flash-preview-image-generation",
    "video-generation": "veo-2.0-generate-001",
    "embeddings": "gemini-embedding-exp-03-07",
    "live-api": "gemini-2.0-flash-live-001"
}
```

## Getting Started

### Installation and Setup

```python
# Install the SDK
# pip install -q -U "google-genai>=1.0.0"

from google import genai
from google.genai import types

# Initialize client
client = genai.Client(api_key="YOUR_API_KEY")
```

### Basic Configuration

```python
# Configure generation parameters
response = client.models.generate_content(
    model="gemini-2.5-flash-preview-05-20",
    contents="Your prompt here",
    config=types.GenerateContentConfig(
        temperature=0.7,        # 0-2, controls randomness
        top_p=0.95,            # Nucleus sampling
        top_k=40,              # Top-k sampling
        max_output_tokens=8192, # Maximum response length
        stop_sequences=["END"], # Stop generation triggers
        response_modalities=["TEXT"],  # Output format
        response_mime_type="text/plain"  # MIME type
    )
)
```

## Core Features

### Basic Text Generation

```python
# Simple text generation
response = client.models.generate_content(
    model="gemini-2.5-flash-preview-05-20",
    contents="Write a haiku about programming"
)
print(response.text)

# With multiple inputs
response = client.models.generate_content(
    model="gemini-2.5-flash-preview-05-20",
    contents=[
        "Context: Python is a programming language",
        "Task: Explain Python's main advantages"
    ]
)
```

### System Instructions

```python
# Set behavior guidelines
response = client.models.generate_content(
    model="gemini-2.5-flash-preview-05-20",
    contents="What's the weather like?",
    config=types.GenerateContentConfig(
        system_instruction="You are a helpful weather assistant. Always mention that you cannot access real-time data."
    )
)
```

### Multi-turn Conversations

```python
# Create a chat session
chat = client.chats.create(
    model="gemini-2.5-flash-preview-05-20",
    config=types.GenerateContentConfig(
        system_instruction="You are a helpful coding assistant"
    )
)

# Send messages
response = chat.send_message("What is a Python decorator?")
print(response.text)

response = chat.send_message("Can you show me an example?")
print(response.text)

# Get conversation history
for message in chat.get_history():
    print(f"{message.role}: {message.parts[0].text}")
```

#### Streaming Responses

```python
# Stream responses for better UX
response = chat.send_message_stream("Explain quantum computing")
for chunk in response:
    print(chunk.text, end="")
```

## Advanced Features

### Thinking (Reasoning)

The Gemini 2.5 series includes "thinking" capabilities for complex reasoning tasks.

#### Basic Thinking Usage

```python
# Thinking is enabled by default for 2.5 models
response = client.models.generate_content(
    model="gemini-2.5-flash-preview-05-20",
    contents="What is the sum of the first 50 prime numbers?"
)
print(response.text)
```

#### Thought Summaries

```python
# Enable thought summaries to see reasoning process
response = client.models.generate_content(
    model="gemini-2.5-flash-preview-05-20",
    contents="Solve: If 5 machines make 5 widgets in 5 minutes, how many widgets do 100 machines make in 100 minutes?",
    config=types.GenerateContentConfig(
        thinking_config=types.ThinkingConfig(
            include_thoughts=True
        )
    )
)

# Extract thoughts and answer
for part in response.candidates[0].content.parts:
    if part.text:
        if part.thought:
            print("Reasoning:", part.text)
        else:
            print("Answer:", part.text)
```

#### Thinking Budget (Flash only)

```python
# Control thinking tokens (0-24576)
response = client.models.generate_content(
    model="gemini-2.5-flash-preview-05-20",
    contents="List 3 physics discoveries",
    config=types.GenerateContentConfig(
        thinking_config=types.ThinkingConfig(
            thinking_budget=1024  # Limit reasoning tokens
        )
    )
)

# Check token usage
print("Thinking tokens:", response.usage_metadata.thoughts_token_count)
print("Output tokens:", response.usage_metadata.candidates_token_count)
```

#### Streaming with Thinking

```python
# Stream thoughts during generation
thoughts = ""
answer = ""

for chunk in client.models.generate_content_stream(
    model="gemini-2.5-flash-preview-05-20",
    contents="Complex logic puzzle here...",
    config=types.GenerateContentConfig(
        thinking_config=types.ThinkingConfig(
            include_thoughts=True
        )
    )
):
    for part in chunk.candidates[0].content.parts:
        if part.text:
            if part.thought:
                thoughts += part.text
                print("Thinking:", part.text)
            else:
                answer += part.text
                print("Answer:", part.text)
```

### Function Calling

Function calling allows models to interact with external tools and APIs.

#### Basic Function Calling

```python
# Step 1: Define function declaration
get_weather_declaration = {
    "name": "get_weather",
    "description": "Get current weather for a location",
    "parameters": {
        "type": "object",
        "properties": {
            "location": {
                "type": "string",
                "description": "City and state, e.g., 'San Francisco, CA'"
            },
            "unit": {
                "type": "string",
                "enum": ["celsius", "fahrenheit"],
                "description": "Temperature unit"
            }
        },
        "required": ["location"]
    }
}

# Step 2: Configure tools
tools = types.Tool(function_declarations=[get_weather_declaration])
config = types.GenerateContentConfig(tools=[tools])

# Step 3: Send request
response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents="What's the weather in Tokyo?",
    config=config
)

# Step 4: Check for function call
if response.candidates[0].content.parts[0].function_call:
    function_call = response.candidates[0].content.parts[0].function_call
    print(f"Function: {function_call.name}")
    print(f"Arguments: {function_call.args}")
    
    # Step 5: Execute function (your code)
    weather_result = {"temperature": 22, "condition": "sunny"}
    
    # Step 6: Send result back
    function_response = types.Part.from_function_response(
        name=function_call.name,
        response={"result": weather_result}
    )
    
    final_response = client.models.generate_content(
        model="gemini-2.0-flash",
        contents=[
            types.Content(role="user", parts=[types.Part(text="What's the weather in Tokyo?")]),
            types.Content(role="model", parts=[types.Part(function_call=function_call)]),
            types.Content(role="user", parts=[function_response])
        ],
        config=config
    )
    print(final_response.text)
```

#### Parallel Function Calling

```python
# Define multiple functions
functions = [
    {
        "name": "turn_on_lights",
        "description": "Turn lights on/off",
        "parameters": {
            "type": "object",
            "properties": {
                "on": {"type": "boolean"}
            },
            "required": ["on"]
        }
    },
    {
        "name": "set_temperature",
        "description": "Set room temperature",
        "parameters": {
            "type": "object",
            "properties": {
                "celsius": {"type": "number"}
            },
            "required": ["celsius"]
        }
    }
]

tools = types.Tool(function_declarations=functions)
response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents="Turn on the lights and set temperature to 22 degrees",
    config=types.GenerateContentConfig(tools=[tools])
)

# Handle multiple function calls
for part in response.candidates[0].content.parts:
    if part.function_call:
        print(f"{part.function_call.name}({part.function_call.args})")
```

#### Function Calling Modes

```python
# Control function calling behavior
config = types.GenerateContentConfig(
    tools=[tools],
    tool_config=types.ToolConfig(
        function_calling_config=types.FunctionCallingConfig(
            mode="ANY",  # AUTO (default), ANY, NONE
            allowed_function_names=["get_weather"]  # Optional restriction
        )
    )
)
```

#### Automatic Function Calling (Python only)

```python
# Define Python function with type hints
def get_temperature(location: str, unit: str = "celsius") -> dict:
    """Get current temperature for a location.
    
    Args:
        location: City and state, e.g., 'Tokyo, Japan'
        unit: Temperature unit (celsius or fahrenheit)
    
    Returns:
        Dictionary with temperature and unit
    """
    # Mock implementation
    return {"temperature": 22, "unit": unit, "location": location}

# Use function directly
response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents="What's the temperature in Paris?",
    config=types.GenerateContentConfig(
        tools=[get_temperature]  # Pass function directly
    )
)
print(response.text)

# Disable automatic calling if needed
config = types.GenerateContentConfig(
    tools=[get_temperature],
    automatic_function_calling=types.AutomaticFunctionCallingConfig(
        disable=True
    )
)
```

### Structured Output

Generate JSON or enum outputs with guaranteed schema compliance.

#### JSON Generation

```python
# Method 1: Schema configuration (recommended)
from pydantic import BaseModel

class Recipe(BaseModel):
    recipe_name: str
    ingredients: list[str]
    cooking_time: int
    difficulty: str

response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents="Give me a recipe for chocolate chip cookies",
    config={
        "response_mime_type": "application/json",
        "response_schema": Recipe,
    }
)

# Access as JSON string
print(response.text)

# Access as parsed objects
recipe = response.parsed
print(recipe.recipe_name)
```

#### Complex Schema Example

```python
class Address(BaseModel):
    street: str
    city: str
    country: str

class Person(BaseModel):
    name: str
    age: int
    email: str
    address: Address
    hobbies: list[str]

response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents="Generate a fictional person's profile",
    config={
        "response_mime_type": "application/json",
        "response_schema": Person,
    }
)

person = response.parsed
print(f"{person.name} lives in {person.address.city}")
```

#### Enum Generation

```python
import enum

class Sentiment(enum.Enum):
    POSITIVE = "positive"
    NEGATIVE = "negative"
    NEUTRAL = "neutral"

response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents="Analyze sentiment: 'This product exceeded my expectations!'",
    config={
        "response_mime_type": "text/x.enum",
        "response_schema": Sentiment,
    }
)
print(response.text)  # "positive"
```

#### Lists and Complex Structures

```python
from typing import List, Optional

class TaskItem(BaseModel):
    task: str
    priority: str
    due_date: Optional[str] = None
    completed: bool = False

response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents="Create a todo list for planning a birthday party",
    config={
        "response_mime_type": "application/json",
        "response_schema": List[TaskItem],
    }
)

tasks = response.parsed
for task in tasks:
    print(f"- [{task.priority}] {task.task}")
```

### Long Context

Gemini models support up to 1M+ tokens, enabling processing of entire books, codebases, or hours of video.

#### Basic Long Context Usage

```python
# Process large documents
with open("long_document.txt", "r") as f:
    long_text = f.read()  # Up to ~4M characters

response = client.models.generate_content(
    model="gemini-2.5-flash-preview-05-20",
    contents=[
        long_text,
        "Summarize the key points of this document"
    ]
)
```

#### Context Window Information

```python
# Check model limits
model_info = client.models.get(model="gemini-2.5-flash-preview-05-20")
print(f"Input limit: {model_info.input_token_limit:,} tokens")
print(f"Output limit: {model_info.output_token_limit:,} tokens")
```

### Grounding with Google Search

Enhance responses with real-time web information.

#### Search as a Tool (Gemini 2.0+)

```python
# Configure Google Search tool
google_search_tool = Tool(
    google_search=GoogleSearch()
)

response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents="When is the next total solar eclipse in the United States?",
    config=GenerateContentConfig(
        tools=[google_search_tool],
        response_modalities=["TEXT"],
    )
)

print(response.text)

# Get search metadata
if response.candidates[0].grounding_metadata:
    metadata = response.candidates[0].grounding_metadata
    print("Search queries:", metadata.web_search_queries)
    print("Sources:", metadata.grounding_chunks)
```

#### Dynamic Retrieval (Gemini 1.5 only)

```python
# Control when to use search
response = client.models.generate_content(
    model="gemini-1.5-flash",
    contents="Who won the latest Nobel Prize in Physics?",
    config=types.GenerateContentConfig(
        tools=[types.Tool(
            google_search_retrieval=types.GoogleSearchRetrieval(
                dynamic_retrieval_config=types.DynamicRetrievalConfig(
                    mode=types.DynamicRetrievalConfigMode.MODE_DYNAMIC,
                    dynamic_threshold=0.7  # 0-1, higher = more selective
                )
            )
        )]
    )
)
```

### URL Context Tool

Process and analyze web content directly from URLs.

```python
# Configure URL context tool
url_context_tool = Tool(
    url_context=types.UrlContext
)

response = client.models.generate_content(
    model="gemini-2.5-flash-preview-05-20",
    contents="Compare recipes from https://example1.com and https://example2.com",
    config=GenerateContentConfig(
        tools=[url_context_tool],
        response_modalities=["TEXT"],
    )
)

# Combined with Google Search
response = client.models.generate_content(
    model="gemini-2.5-flash-preview-05-20",
    contents="Find the latest AI news and summarize from the top sources",
    config=GenerateContentConfig(
        tools=[url_context_tool, google_search_tool],
        response_modalities=["TEXT"],
    )
)
```

### Code Execution

Execute Python code within the model's environment.

#### Basic Code Execution

```python
# Enable code execution
response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents="Calculate the factorial of 20",
    config=types.GenerateContentConfig(
        tools=[types.Tool(code_execution=types.ToolCodeExecution)]
    ),
)

# Process response parts
for part in response.candidates[0].content.parts:
    if part.text:
        print("Text:", part.text)
    elif part.executable_code:
        print("Code:", part.executable_code.code)
    elif part.code_execution_result:
        print("Result:", part.code_execution_result.output)
```

#### Code Execution with I/O

```python
# Upload CSV for analysis
csv_file = client.files.upload(file="data.csv")

response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents=[
        csv_file,
        "Analyze this data and create a visualization"
    ],
    config=types.GenerateContentConfig(
        tools=[types.Tool(code_execution=types.ToolCodeExecution)]
    ),
)

# Model can read files and generate matplotlib plots
# Plots are returned as inline images
```

#### Available Libraries

The code execution environment includes:
- NumPy, Pandas, SciPy, scikit-learn
- Matplotlib, Seaborn (for visualization)
- TensorFlow, OpenCV
- And many more standard data science libraries

**Limitations**:
- 30-second execution timeout
- No custom library installation
- No network access
- matplotlib only for graphics

## Working with Media

### Files API

Upload and manage files for use with the Gemini API.

#### Basic File Upload

```python
# Upload a file
myfile = client.files.upload(file="path/to/image.jpg")
print(f"Uploaded: {myfile.name}")

# Use in generation
response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents=[myfile, "Describe this image"]
)
```

#### File Management

```python
# Get file metadata
file_info = client.files.get(name=myfile.name)
print(f"Size: {file_info.size_bytes}")
print(f"MIME: {file_info.mime_type}")
print(f"State: {file_info.state}")

# List all files
for f in client.files.list():
    print(f" - {f.name} ({f.display_name})")

# Delete file
client.files.delete(name=myfile.name)
```

#### File Upload Limits
- Maximum file size: 2GB
- Total storage: 20GB per project
- Retention: 48 hours (auto-deleted)
- No download capability via API

### Image Understanding

Process and analyze images with advanced capabilities.

#### Basic Image Analysis

```python
# From file upload
image_file = client.files.upload(file="photo.jpg")
response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents=[image_file, "What's in this image?"]
)

# From URL (inline)
import requests
image_url = "https://example.com/image.jpg"
image_bytes = requests.get(image_url).content
image_part = types.Part.from_bytes(
    data=image_bytes,
    mime_type="image/jpeg"
)

response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents=["Describe this image", image_part]
)
```

#### Object Detection (Bounding Boxes)

```python
# Get bounding boxes for objects
prompt = """Detect all prominent items in the image. 
The box_2d should be [ymin, xmin, ymax, xmax] normalized to 0-1000."""

response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents=[image_file, prompt]
)

# Convert normalized coordinates to pixels
def denormalize_bbox(bbox, img_width, img_height):
    ymin, xmin, ymax, xmax = bbox
    return [
        int(ymin * img_height / 1000),
        int(xmin * img_width / 1000),
        int(ymax * img_height / 1000),
        int(xmax * img_width / 1000)
    ]
```

#### Image Segmentation

```python
# Get segmentation masks
prompt = """Give the segmentation masks for the wooden and glass items.
Output a JSON list where each entry contains:
- "box_2d": bounding box
- "mask": base64 encoded PNG mask
- "label": descriptive label"""

response = client.models.generate_content(
    model="gemini-2.5-flash-preview-05-20",
    contents=[image_file, prompt],
    config={
        "response_mime_type": "application/json"
    }
)

# Process masks
import base64
import json
from PIL import Image
import io

masks = json.loads(response.text)
for mask_data in masks:
    # Decode base64 mask
    mask_bytes = base64.b64decode(mask_data["mask"])
    mask_img = Image.open(io.BytesIO(mask_bytes))
    # Process mask...
```

#### Multiple Images

```python
# Compare images
image1 = client.files.upload(file="before.jpg")
image2 = client.files.upload(file="after.jpg")

response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents=[
        "What changed between these images?",
        image1,
        image2
    ]
)
```

### Document Processing

Process PDFs and documents with native understanding.

#### PDF Processing

```python
# Upload PDF (<20MB: inline, >20MB: use Files API)
pdf_file = client.files.upload(file="document.pdf")

response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents=[pdf_file, "Summarize this document"]
)

# From URL
import httpx
pdf_url = "https://example.com/paper.pdf"
pdf_data = httpx.get(pdf_url).content

response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents=[
        types.Part.from_bytes(
            data=pdf_data,
            mime_type='application/pdf'
        ),
        "Extract key findings from this research paper"
    ]
)
```

#### Multiple PDFs

```python
# Compare documents
pdf1 = client.files.upload(file="report_2023.pdf")
pdf2 = client.files.upload(file="report_2024.pdf")

response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents=[
        pdf1, 
        pdf2, 
        "Compare the main differences between these reports in a table"
    ]
)
```

#### PDF Limitations
- Maximum 1,000 pages
- Each page = 258 tokens
- Supports text extraction, charts, tables
- Best with proper orientation and clear text

### Video and Audio

Process multimedia content with native understanding.

#### Video Processing

```python
# Upload and process video
video_file = client.files.upload(file="video.mp4")

# Wait for processing
import time
while video_file.state.name == 'PROCESSING':
    time.sleep(5)
    video_file = client.files.get(name=video_file.name)

response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents=[
        video_file,
        "Describe what happens in this video"
    ]
)
```

#### Audio Processing

```python
# Process audio file
audio_file = client.files.upload(file="podcast.mp3")

response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents=[
        audio_file,
        "Transcribe and summarize the key points"
    ]
)
```

#### Token Rates
- Video: 263 tokens per second
- Audio: 32 tokens per second

## Embeddings

Generate semantic embeddings for text similarity and search.

### Basic Embeddings

```python
# Generate single embedding
result = client.models.embed_content(
    model="gemini-embedding-exp-03-07",
    contents="What is the meaning of life?"
)
embedding = result.embeddings[0]
print(f"Embedding dimension: {len(embedding.values)}")  # 768

# Batch embeddings
result = client.models.embed_content(
    model="gemini-embedding-exp-03-07",
    contents=[
        "First text",
        "Second text",
        "Third text"
    ]
)
for i, embedding in enumerate(result.embeddings):
    print(f"Text {i}: {len(embedding.values)} dimensions")
```

### Task Types

Optimize embeddings for specific use cases:

```python
# Semantic similarity
result = client.models.embed_content(
    model="gemini-embedding-exp-03-07",
    contents="Compare this text",
    config=types.EmbedContentConfig(
        task_type="SEMANTIC_SIMILARITY"
    )
)

# Document retrieval
doc_embedding = client.models.embed_content(
    model="gemini-embedding-exp-03-07",
    contents="Long document text...",
    config=types.EmbedContentConfig(
        task_type="RETRIEVAL_DOCUMENT"
    )
)

query_embedding = client.models.embed_content(
    model="gemini-embedding-exp-03-07",
    contents="Search query",
    config=types.EmbedContentConfig(
        task_type="RETRIEVAL_QUERY"
    )
)
```

Available task types:
- `SEMANTIC_SIMILARITY`: Text similarity comparison
- `CLASSIFICATION`: Text categorization
- `CLUSTERING`: Grouping similar texts
- `RETRIEVAL_DOCUMENT`: Indexing documents
- `RETRIEVAL_QUERY`: Search queries
- `QUESTION_ANSWERING`: Q&A systems
- `FACT_VERIFICATION`: Fact checking
- `CODE_RETRIEVAL_QUERY`: Code search

### Truncated Embeddings

```python
# Reduce dimensionality for efficiency
result = client.models.embed_content(
    model="text-embedding-004",
    contents="Text to embed",
    config=types.EmbedContentConfig(
        output_dimensionality=256  # Default: 768
    )
)
```

## Optimization Techniques

### Context Caching

Save costs and reduce latency by caching repeated content.

#### Implicit Caching (Automatic)

Enabled by default for Gemini 2.5 models:
- No setup required
- Automatic cost savings on cache hits
- Minimum 1,024 tokens (Flash) or 2,048 tokens (Pro)

```python
# Check implicit cache usage
response = client.models.generate_content(
    model="gemini-2.5-flash-preview-05-20",
    contents="Your content here"
)
print(f"Cached tokens: {response.usage_metadata.cached_content_token_count}")
```

#### Explicit Caching

For guaranteed cost savings with large, reused contexts:

```python
# Create cache
video_file = client.files.upload(file="tutorial_video.mp4")
cache = client.caches.create(
    model="gemini-2.5-flash-001",  # Must use explicit version
    config=types.CreateCachedContentConfig(
        display_name="Tutorial Video Cache",
        system_instruction="You are a video content analyzer",
        contents=[video_file],
        ttl="3600s"  # 1 hour
    )
)

# Use cache
response = client.models.generate_content(
    model="gemini-2.5-flash-001",
    contents="What programming concepts are explained?",
    config=types.GenerateContentConfig(
        cached_content=cache.name
    )
)

# Check savings
print(f"Cached tokens: {response.usage_metadata.cached_content_token_count}")
print(f"New tokens: {response.usage_metadata.prompt_token_count}")
```

#### Cache Management

```python
# List caches
for cache in client.caches.list():
    print(f"{cache.display_name}: expires {cache.expire_time}")

# Update TTL
client.caches.update(
    name=cache.name,
    config=types.UpdateCachedContentConfig(
        ttl='7200s'  # 2 hours
    )
)

# Update expiry time
import datetime
expire_time = datetime.datetime.now(datetime.timezone.utc) + datetime.timedelta(hours=3)
client.caches.update(
    name=cache.name,
    config=types.UpdateCachedContentConfig(
        expire_time=expire_time
    )
)

# Delete cache
client.caches.delete(cache.name)
```

### Token Management

Monitor and optimize token usage for cost control.

#### Count Tokens

```python
# Count before sending
token_count = client.models.count_tokens(
    model="gemini-2.0-flash",
    contents="Your prompt here"
)
print(f"Input tokens: {token_count.total_tokens}")

# Multimodal content
image_file = client.files.upload(file="image.jpg")
token_count = client.models.count_tokens(
    model="gemini-2.0-flash",
    contents=["Describe this image", image_file]
)
print(f"Total tokens: {token_count.total_tokens}")
```

#### Token Rates

**Text**: ~4 characters = 1 token
**Images**: 
- ≤384x384 pixels: 258 tokens
- Larger: tiled into 768x768 chunks, 258 tokens each

**Media**:
- Video: 263 tokens/second
- Audio: 32 tokens/second  
- PDF: 258 tokens/page

## Prompt Engineering

### Core Principles

#### 1. Clear Instructions
```python
# ❌ Vague
"Tell me about dogs"

# ✅ Specific
"Write a 200-word educational summary about Golden Retrievers, covering their temperament, care needs, and suitability as family pets"
```

#### 2. Few-Shot Examples
```python
prompt = """
Classify the sentiment as positive, negative, or neutral:

Example 1:
Text: "This product exceeded all my expectations!"
Sentiment: positive

Example 2:
Text: "The service was okay, nothing special."
Sentiment: neutral

Example 3:  
Text: "I'm extremely disappointed with this purchase."
Sentiment: negative

Now classify:
Text: "The quality is decent for the price."
Sentiment:"""
```

#### 3. Structured Formats
```python
# Use prefixes for clarity
prompt = """
Context: You are a technical documentation writer.
Task: Create a README file for a Python project.
Requirements:
- Include installation instructions
- Add usage examples
- List dependencies
Format: Markdown

Project details:
Name: DataAnalyzer
Purpose: Statistical analysis toolkit
Dependencies: pandas, numpy, matplotlib

Output:"""
```

#### 4. Step-by-Step Reasoning
```python
# Complex problems benefit from explicit steps
prompt = """
Solve this problem step by step:

A store offers a 20% discount on all items. If you buy 3 items
priced at $50, $30, and $70, and there's an additional 10% off
for purchases over $100, what's the final price?

Step 1: Calculate the original total
Step 2: Apply the 20% discount
Step 3: Check if eligible for additional discount
Step 4: Calculate final price

Show your work for each step."""
```

### Advanced Techniques

#### Chain of Thought
```python
# Enable reasoning for complex tasks
response = client.models.generate_content(
    model="gemini-2.5-flash-preview-05-20",
    contents="""
    A farmer has 17 sheep. All but 9 die. How many are left?
    Think step by step before answering.
    """
)
```

#### Role-Based Prompting
```python
config = types.GenerateContentConfig(
    system_instruction="""You are an experienced data scientist with expertise in machine learning.
    You explain complex concepts in simple terms and always provide practical examples."""
)

response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents="Explain gradient descent",
    config=config
)
```

#### Output Formatting
```python
# Control output structure
prompt = """
Analyze this product review and provide:

Review: "The laptop is fast but the battery life is disappointing. 
Great screen quality though!"

Format your response as:
**Pros:**
- [list items]

**Cons:**
- [list items]

**Overall Rating:** [1-5 stars]
**Summary:** [one sentence]
"""
```

### Multimodal Prompting

#### Image + Text
```python
# Place image before text for single images
response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents=[
        image_file,  # Image first
        "Create a recipe based on the ingredients shown"
    ]
)

# Be specific about image regions
response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents=[
        image_file,
        "Focus on the items in the upper left corner. What are they?"
    ]
)
```

#### Multiple Images
```python
# Order matters for comparison
response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents=[
        "Compare these UI designs:",
        types.Part(text="Design A:"), image1,
        types.Part(text="Design B:"), image2,
        "Which has better accessibility?"
    ]
)
```

## Best Practices

### 1. Model Selection
- **Use 2.5 Flash** for: Fast responses, high volume, cost efficiency
- **Use 2.5 Pro** for: Complex reasoning, STEM problems, code generation
- **Use specialized variants** for: Audio (native-audio), TTS, image generation

### 2. Error Handling
```python
try:
    response = client.models.generate_content(
        model="gemini-2.0-flash",
        contents=prompt
    )
except Exception as e:
    print(f"Error: {e}")
    # Implement retry logic or fallback
```

### 3. Token Optimization
- Place large, reusable content at prompt beginning
- Use caching for repeated contexts
- Monitor usage_metadata for cost tracking
- Consider truncated embeddings for scale

### 4. Safety and Ethics
- Always validate generated content
- Implement content filtering as needed
- Respect copyright (max 15-word quotes)
- Follow Google's AI principles

### 5. Performance Tips
- Use streaming for better UX
- Batch operations when possible
- Leverage parallel function calling
- Optimize file sizes before upload

### 6. Prompt Engineering
- Start with zero-shot, add examples if needed
- Use system instructions for consistent behavior
- Break complex tasks into steps
- Specify output format explicitly

### Common Pitfalls to Avoid

1. **Over-relying on model knowledge**: Use grounding for current events
2. **Ignoring token limits**: Always check model constraints
3. **Poor error handling**: Implement robust retry logic
4. **Inefficient caching**: Cache large, frequently-used content
5. **Vague prompts**: Be specific and provide context

## Migration Notes

If migrating from older Gemini versions:
- Client initialization changed: `genai.Client()` not `genai.configure()`
- New `contents` parameter instead of `prompt`
- Thinking mode enabled by default on 2.5 models
- Search is now a tool, not a retrieval config
- Many new features: thinking budgets, URL context, native audio

