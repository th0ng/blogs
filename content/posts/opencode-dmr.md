+++
title = "Docker Model Runner with OpenCode"
date = "2025-11-29T17:44:17+01:00"
#dateFormat = "2006-01-02" # This value can be configured for per-post date formatting
author = "Hoang Danh Thong"
cover = ""
tags = ["ai", "agent", "opensource"]
keywords = ["", ""]
description = "Create a local coding agent using OpenCode and Docker Model Runner."
showFullContent = false
readingTime = false
hideComments = true
+++

OpenCode is awesome! It is a TUI AI coding agents which uses the [AI SDK](https://ai-sdk.dev/) to support for 75+ LLM providers and also supports running **local model**.

Docker Model Runner was launched with the purpose of making it **simpler** and **faster** to run and test AI models locally. It's built on top of `llama.cpp` and is accessible through the OpenAI API, which can be used by OpenCode. So, let's go ahead and create an AI agent for our agentic workflow that run completely local!

## 1. Get started with Docker Model Runner (DMR)

First, follow the [instructions](https://docs.docker.com/ai/model-runner/get-started/) to enable docker model runner and start using pulled models.

- To check for available local models:
```bash
docker model list
```

- To list running models:
```bash
docker model ps
```

#### Enable host-side TCP support:

Let's enable host-side TCP support to prevent this from happening:
![Host-side TCP support not enabled](/img/opencode-dmr/host-side-tcp.png)

If you are using Docker Desktop, just simply select **Enable host-side TCP support** in Docker Desktop settings AI section.

# TODO: how to to it using docker cli???

Now, you can try calling the endpoint to list models:
```bash
curl http://localhost:12434/engines/llama.cpp/v1/models
```

## 2. Using DMR models in OpenCode

OpenCode can be configured using a JSON config file, and the global OpenCode config lives in `~/.config/opencode/opencode.json`. Per project configuration is also possible simply by adding a `opencode.json` in your project directory and its settings will merge or override the global settings.

Go ahead and create the config file if it's not there, and let's add the DMR model to the configuration so that it could be used by OpenCode.

```json
{
  "$schema": "https://opencode.ai/config.json",
}
```

The config file has a schema that's defined in ['opencode.ai/config.json'](https://opencode.ai/config.json), it helps the editor to validate and autocomplete based on the schema.

DMR models can be added under `provider`, I am using `ai/qwen3-coder` as an example:
```json
{
    "dmr": {
        "npm": "@ai-sdk/openai-compatible",
            "name": "Docker Model Runner",
            "options": {
                "baseURL": "http://localhost:12434/engines/llama.cpp/v1"
            },
            "models": {
                "ai/qwen3-coder:latest": {
                    "name": "Qwen3 Coder"
                }
            }
    }
}
```

Multiple models could be added to the list, and to set the default model to be used, use:

```json
{
 "model": "dmr/${your_default_model}"
}
```

***Notes*** For the `baseURL`, 'llama.cpp' can be left out.

Our config file should now look something like this:
```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "dmr": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Docker Model Runner",
      "options": {
        "baseURL": "http://localhost:12334/engines/llama.cpp/v1"
      },
      "models": {
        "ai/qwen3-coder:latest": {
          "name": "Qwen3 Coder"
        }
      }
    }
  },
  "model": "dmr/ai/qwen3-coder:latest"
}
```

And that's it, simple, right?
![Working DMR in OpenCode](/img/opencode-dmr/working-dmr.png)

## 3. Wait...

If you are working with a large code-base, you will often run into this: **the request exceeds the available context size** isue, and listing the available models using `docker model list` results in:
![result of model list](/img/opencode-dmr/model-ps.png)

As you can see, there's no `CONTEXT` column listed, but it can actually be configured using:
```bash
docker model configure --context-size={your_new_context_size} {your_model}
```

Try increasing the context size of the model to match your need and also take into account of your hardware specs too.

