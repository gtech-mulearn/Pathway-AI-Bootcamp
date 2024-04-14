## 6.3. Dropbox Retrieval App in 15 Minutes

Let's dive into the setup and operation of the Dropbox AI Chat tool, an innovative solution enabling you to efficiently search through extensive, unstructured documents stored in your Dropbox using advanced AI capabilities.

What you'll be building can be seen below. In this particular showcase, we'll be cloning [this particular repository](https://github.com/pathway-labs/dropbox-ai-chat) to enable you to quickly build a similar application.
![Drop Gif](../images/image2.gif)

# 6.3.1. Building the app without Dockerization

Below, you’ll find a video tutorial by Richard Pelgrim, a Developer Advocate in the stream data processing space, demonstrating how they harnessed the Dropbox document sync application to create an RAG app.

**Link to the Project:** [Dropbox AI Chat Repository](https://github.com/pathway-labs/dropbox-ai-chat) Make sure to star it. ⭐  
**YouTube Video to be Embedded:** [Hands-on Development Video](https://youtu.be/PbSAYHi5gnM)  

*Remember, in the video, Richard uses Conda. If you haven’t used Anaconda or Conda in the past and struggle with it, the next module on Docker should help.*

This app proves invaluable in simplifying the comprehension of the recently released EU AI Act. Feel free to let your creative juices flow to imagine possibilities that you can open with such an application.
Let's analyze a few elements from this video.

**Step 1: Cloning the Repository**
```bash
git clone https://github.com/pathway-labs/dropbox-ai-chat 
cd dropbox-ai-chat
```

**Step 2: Setting up Environment Variables**
Create a .env file in the root directory and populate it with your configurations. Replace {OPENAI_API_KEY} with your actual OpenAI API key.

```env
OPENAI_API_TOKEN={OPENAI_API_KEY}
HOST=0.0.0.0
PORT=8080
EMBEDDER_LOCATOR=text-embedding-ada-002
EMBEDDING_DIMENSION=1536
MODEL_LOCATOR=gpt-3.5-turbo
MAX_TOKENS=200
TEMPERATURE=0.0
DROPBOX_LOCAL_FOLDER_PATH="../../../mnt/c/Users/bumur/Dropbox/documents"
```

Make sure to replace `DROPBOX_LOCAL_FOLDER_PATH` with your local Dropbox folder path and optionally customize other values.

**Step 3 - Optional: Creating a Virtual Environment**
In this case, Richard has used Conda. To create an isolated environment, execute:

```bash
# Setting Up a Virtual Environment
# On macOS and Linux:
# Create and activate a virtual environment
python -m venv pw-env && source pw-env/bin/activate

# On Windows:
# Step 1: Create a new virtual environment in a folder named 'pw-env'
python -m venv pw-env

# Step 2: Activate the virtual environment
pw-env\Scripts\activate
```

**Step 4 - Installing the Dependencies**
```bash
pip install --upgrade -r requirements.txt
```

**Step 5 - Running the App**
Navigate to the root directory and execute `main.py`.
```bash
python main.py
```

**Step 6 - Launching UI with Streamlit**
Run the Streamlit app using the following command:
```bash
streamlit run ui.py
```
Access the UI at http://localhost:8501/ on your browser.

With this, your app should be up and running. 😄

**Connecting the Dots**

If you look closely at the repo and visit the `api.py` file on the repo, you'll be able to connect the dots from what we've learned so far. Here:
- The prompt is processed as embeddings and used as `embedded_query`.
- The data we're getting from our data source (i.e., Dropbox) is converted into smaller chunks with the help of Pathway (pw) and then converted to embeddings and stored in the index.
- Using these, we're creating the augmented prompt with the help of retrieved information and feeding that into GPT-3.5 turbo.
Here `mode="streaming”` implies that the data being ingested will be processed in real time. If you’d like to learn more about switching between batch and streaming, [this link](https://pathway.com/developers/user-guide/getting-started/switch-from-batch-to-streaming) will help. But it’s not immediately necessary. 🙂

```python
# Real-time data coming from external unstructured data sources like a PDF file
input_data = pw.io.fs.read(
dropbox_folder_path,
mode="streaming",
format="binary",
autocommit_duration_ms=50,
)

# Chunk input data into smaller documents
documents = input_data.select(texts=extract_texts(pw.this.data))
documents = documents.select(chunks=chunk_texts(pw.this.texts))
documents = documents.flatten(pw.this.chunks).rename_columns(chunk=pw.this.chunks)

# Compute embeddings for each document using the OpenAI Embeddings API
embedded_data = embeddings(context=documents, data_to_embed=pw.this.chunk)

# Construct an index on the generated embeddings in real-time
index = index_embeddings(embedded_data)

# Generate embeddings for the query from the OpenAI Embeddings API
embedded_query = embeddings(context=query, data_to_embed=pw.this.query)

# Build prompt using indexed data
responses = prompt(index, embedded_query, pw.this.query)

# Feed the prompt to ChatGPT and obtain the generated answer.
response_writer(responses)

# Run the pipeline
pw.run()

```

# 6.3.2. Understanding Docker

This particular page is to help you build the previous file if you're new to Docker and are struggling to install dependencies on your machine.

First off, a quick recap.

Think of Docker like a shipping container for your app. Just as a shipping container can hold all sorts of goods (clothes, electronics, etc.) and can be transported anywhere in the world, Docker bundles your app and everything it needs to run into a 'container.' This makes it easy to share and run your app on any computer.

Given the complexities and manual effort involved in resolving dependency issues in your system, Docker can be a beneficial tool to standardize the development environment among all students.

**Why Use Docker?**
- **Standardized Environment:** Everyone gets the same set of dependencies, reducing "it works on my machine" issues.
- **Isolated:** Doesn't interfere with other projects or system-wide settings.
- **Ease of Use:** Once set up, running the project becomes much simpler.

**Understanding Key Docker Terminologies**
- **Docker Image:** Think of this as a blueprint or a snapshot of a container, including the application and its dependencies. You build an image once and use it to create multiple containers.
- **Docker Container:** A container is a running instance of an image. It's a lightweight, stand-alone, executable software package that includes everything needed to run the code.
- **CMD:** In Docker, the CMD instruction specifies the command that will be executed when the container starts up.
- **Docker Compose:** A tool for defining and running multi-container Docker applications. Using a YAML file (docker-compose.yml), it allows you to specify how different containers interact with each other, making it easier to manage multiple containers as a single service.

Now let's see the step-by-step implementation.

# 6.3.3. Using Docker to Build the App

Welcome to this module on how to use Docker to set up and run the Dropbox AI Chat application. Before we begin, it's essential to ensure that you meet the prerequisites and understand each step thoroughly.

**Basic Prerequisites:**
Ensure you have Docker installed on your machine.
Dropbox must also be installed.

**Step 1: Cloning the Repository**

Open your terminal and run:
```bash
git clone https://github.com/pathway-labs/dropbox-ai-chat
cd dropbox-ai-chat
```

If you have previously cloned an older version of the repository, ensure you're in the correct repository directory and update it using:
```bash
git pull https://github.com/pathway-labs/dropbox-ai-chat
```

What this does: The `git pull` command will update your local repository with the latest changes from the remote repository.

**Step 2: Setting the Environment Variables**

Overview: The .env file sets crucial environment variables for your application. If you're using macOS, the .env file might be hidden by default when viewed through Finder but is visible via Terminal. Regardless of the OS, note that this file plays a pivotal role.

The primary change you'll make in this entire implementation is to the `{PATH_TO_DROPBOX}` variable, which the .env file uses.

Understanding `PATH_TO_DROPBOX` variable used here: This variable defines the relative path from your project to your Dropbox folder. If you want to quickly understand how the relative path works in the context of Linux, you can check this [quick video by Udacity](https://youtu.be/ephId3mYu9o)
or read these comprehensive explanations by [RedHat](https://www.redhat.com/sysadmin/linux-path-absolute-relative)or [Coding Rooms](https://www.codingrooms.com/blog/file-paths)

**Setting the environment variables in MacOS or Linux**

Create an .env file in the project's root directory using touch.
```bash
touch .env
```
Edit the .env file using a text editor like nano or vim.
```bash
nano .env
```
Populate the .env file with the following content, replacing placeholders with actual values. Replace the following while using the environment variables from below: `{OPENAI_API_KEY}` with your OpenAI API key. You can get it [here](https://beta.openai.com/account/api-keys) once you've logged in. And, replace `{REPLACE_WITH_DROPBOX_RELATIVE_PATH}` with the relative path where the Dropbox folder is located.
```env
OPENAI_API_TOKEN={OPENAI_API_KEY}
HOST=api
PORT=8080
EMBEDDER_LOCATOR=text-embedding-ada-002
EMBEDDING_DIMENSION=1536
MODEL_LOCATOR=gpt-3.5-turbo
MAX_TOKENS=200
TEMPERATURE=0.0
DROPBOX_LOCAL_FOLDER_PATH="../../../mnt/c/Users/bumur/Dropbox/documents"
```

Alternative to using export: If .env doesn't work for you, you can set these variables directly in your shell using the command below. However, note that variables set with export (Linux/macOS) or set (Windows, as seen below) last only for the current session. If you want them to persist, you'll need to add them to shell configuration files like adding to .bashrc or .bash_profile for Linux/macOS, or use System Properties on Windows.
```bash
export OPENAI_API_TOKEN={OPENAI_API_KEY}
export EMBEDDER_LOCATOR=text-embedding-ada-002
export EMBEDDING_DIMENSION=1536
export MODEL_LOCATOR=gpt-3.5-turbo
export MAX_TOKENS=200
export TEMPERATURE=0.0
export PATH_TO_DROPBOX={REPLACE_WITH_DROPBOX_RELATIVE_PATH}
```

**Setting the environment variables in Windows:**

Create an .env file using a text editor of your choice and save it as .env like you’d save it as a .pdf.
Populate the .env file as shown above.

Alternative to using export: Use the set command in Command Prompt to set environment variables.
```batch
set OPENAI_API_TOKEN={OPENAI_API_KEY}
set EMBEDDER_LOCATOR=text-embedding-ada-002
set EMBEDDING_DIMENSION=1536
set MODEL_LOCATOR=gpt-3.5-turbo
set MAX_TOKENS=200
set TEMPERATURE=0.0
set PATH_TO_DROPBOX={REPLACE_WITH_DROPBOX_RELATIVE_PATH}
```

**Step 3: Building and Starting Containers**

Now we will build the Docker image and start the containers.

Navigate to the cloned directory dropbox-ai-chat using cd. Then run these commands.
```bash
docker-compose build
docker-compose up
```

Behind the Scenes with Docker:

Dockerfile: This file contains a set of instructions that Docker follows to build an image. It's like a blueprint for your application. Docker reads these instructions and creates a Docker image based on them. This image contains everything your app needs to run. 

Docker-compose: It's a tool for defining and running multi-container Docker applications. In our context, docker-compose uses the docker-compose.yml file to understand how to set up and run the app's services. When you run docker-compose up, it starts the services as defined.

**Step 4: Accessing Applications**

What it does: Opens access to your API and Streamlit UI.

Access Points for your application:

- **API:** [http://localhost:8080/](http://localhost:8080/)
- **Streamlit UI:** [http://localhost:8501/](http://localhost:8501/)

**Step 5: Stopping Containers**

To stop the services and remove the containers, execute:
```bash
docker-compose down
```

By following these steps, you should be able to get both the main application and the Streamlit UI up and running using Docker Compose.

💡An interesting thing to notice Interestingly if you quickly revisit the [LLM Architecture diagram](https://ai-community-iitb-organization.gitbook.io/10-days-llm-bootcamp/retrieval-augmented-generation-and-llm-architecture/llm-architecture-diagram-and-various-steps) we saw earlier, there was a prompt from a Customer Support Executive trying to understand the product release notes made by the development team. With something like the Dropbox app, that problem can be easily addressed.

Now let's look at another example where we use an API as a data source.

[Next Lesson](../Level-6/HANDS-ON-IMPLEMENTATION-MODULE-Part-3.md)📖👣🔜

[Previous Lesson](../Level-6/HANDS-ON-IMPLEMENTATION-MODULE-Part-1.md)🔙📚


