# HANDS ON IMPLEMENTATION MODULE

## 6.1. Hands-on Development

Welcome to the final module of this bootcamp!

Now, we're going to guide you through the process of setting up a Retrieval Augmented Generation (RAG) architecture using LLM App, an open-source production framework for building and serving AI applications and LLM-enabled real-time data pipelines. While we're working with this tool, consider starring it on GitHub. It is an effortless way to bookmark it for future and track updates, and it also helps the community discover the resource.

**Link to the GitHub repository:** [LLM App](https://github.com/pathwaycom/llm-app)

By the end of this module, you'll be able to build your LLM application that works with real-time data. This implementation guide is aimed at users of Mac, Linux, and Windows systems.

## 6.2. Pre-Requisites

Before we dive in, let's make sure you have all the necessary prerequisites installed on your computer. Not only are these essential for what we're about to embark on, but they'll also be invaluable tools if you decide to contribute to open-source projects in the future.

### Git, Python, and Pip

- Python 3.10 or above should be installed on your machine. If not, [download here](https://www.python.org/downloads/).
- **Pip:** Comes pre-installed with Python 3.4+. It is the standard package manager for Python. You can check if it's downloaded by typing the below command in your terminal/command prompt.

```bash
pip --version
```

If Pip is not installed, you'll get an error. In that case, you need to download and install Pip to manage project packages.
- **Git should be installed on your machine.** If you've installed XCode (or its Command Line Tools), Git may already be installed. To find out, open a Terminal or Command Prompt, and enter:

```bash
git --version
```

If it's not installed, refer to [this documentation](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) and install it.

### OpenAI API Key

This API key is required if you plan to use OpenAI models for embedding and generation. If you are less confident with using open-source alternatives, using OpenAI APIs is a good starting point.

By default, OpenAI currently offers $5 in free credits for new accounts – i.e. the ones with a new phone number and email ID. These free credits should suffice for building your project.

Going forward, we will use text-embedding-ada-002 for generating the vector embeddings [OpenAI documentation](https://openai.com/blog/new-and-improved-embedding-model) and gpt-3.5-turbo for text generation.

To create a new OpenAI API Key:

1. Log into the [OpenAI website](https://platform.openai.com/login?launch)
2. Navigate to the [API Key Management page](https://platform.openai.com/account/api-keys) to generate your key.

**Note: If you're using Windows OS**

The example ahead only supports Unix-like systems (such as Linux, macOS, BSD). But the good news is that you have an easy fix. If you are a Windows user, you can use [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) or use Docker to run the app in a Docker container. The steps for that are shared ahead.

**What is Docker and how to install it?**

Think of Docker like a shipping container for your app. Just as a shipping container can hold all sorts of goods (clothes, electronics, etc.) and can be transported anywhere in the world, Docker bundles your app and everything it needs to run into a 'container.' This makes it easy to share and run your app on any computer.

Similar to Docker, there is a tool called Conda which is showcased in one of the videos ahead. Conda lets you create separate environments to manage different sets of Python packages, ensuring your code runs the same way on any computer.

You can download Docker [here](https://www.docker.com/get-started).
You can download Conda [here](https://docs.conda.io/en/latest/miniconda.html).

Now that we have the pre-requisites, let's proceed. 😄
