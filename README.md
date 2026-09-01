# Hossam_GPT

Hossam_GPT is an open-source **agentic AI chatbot** built with **Python, FastAPI, LangGraph, LangChain, OpenAI, Tavily, ChromaDB, and SQLite**.

It supports real-time streaming chat, document uploads, retrieval-augmented generation (RAG), web search, conversation memory, and a simple web interface.

## Features

- Chat with an AI agent powered by OpenAI
- Stream responses in real time
- Upload PDF, DOCX, TXT, Markdown, Python, and CSV files
- Use uploaded documents as context through RAG
- Search the web with Tavily for current information
- Store and recall conversation history
- Use a simple FastAPI-based web interface
- Deploy with Docker
- Use GitHub Actions, Amazon ECR, and Amazon EC2 for AWS CI/CD

## Project Overview

This project uses:

- **FastAPI** for the backend server and API endpoints
- **Jinja2** for rendering the frontend UI
- **LangGraph** for agent orchestration and conversation state
- **LangChain** for model integration, tools, messages, and the RAG workflow
- **OpenAI** as the language model provider
- **Tavily** for web search
- **ChromaDB** for vector search over uploaded documents
- **SQLite** for conversation history and persistent checkpoints
- **Docker** for containerized deployment

## Prerequisites

Make sure you have the following installed:

- Python 3.11
- `pip` or Conda
- Git
- An OpenAI API key
- A Tavily API key

Optional deployment requirements:

- Docker
- An AWS account
- An Amazon ECR repository
- An Amazon EC2 instance
- A GitHub Actions self-hosted runner

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/hossamfarhoud/Hossam_GPT.git
```

### 2. Navigate to the project directory

```bash
cd Hossam_GPT
```

### 3. Create a virtual environment

Using Conda:

```bash
conda create -n HossamGPT python=3.11 -y
```

### 4. Activate the virtual environment

```bash
conda activate HossamGPT
```

### 5. Install the dependencies

```bash
pip install -r requirements.txt
```

## Environment Variables

Create a `.env` file in the project root directory:

```env
OPENAI_API_KEY=your_openai_api_key
OPENAI_MODEL=gpt-5-mini

TAVILY_API_KEY=your_tavily_api_key

LANGSMITH_TRACING=false
LANGSMITH_ENDPOINT=https://api.smith.langchain.com
LANGSMITH_API_KEY=your_langsmith_api_key
LANGSMITH_PROJECT=HossamGPT
```

If you do not want to use LangSmith tracing, keep:

```env
LANGSMITH_TRACING=false
```

When tracing is disabled, `LANGSMITH_API_KEY`, `LANGSMITH_ENDPOINT`, and `LANGSMITH_PROJECT` are optional and can be removed from `.env`.

> [!IMPORTANT]
> Never commit `.env` or expose API keys publicly. Make sure `.env` is included in `.gitignore`.

## Run Locally

Start the FastAPI application:

```bash
python app.py
```

Alternatively, if `app.py` exposes a FastAPI instance named `app`, run:

```bash
uvicorn app:app --host 0.0.0.0 --port 8080 --reload
```

Open [http://127.0.0.1:8080](http://127.0.0.1:8080) in your browser.

## Project Structure

```text
Hossam_GPT/
├── app.py                  # FastAPI app and streaming chat endpoints
├── agent.py                # LangGraph agent and tool orchestration
├── database.py             # Conversation history and persistence logic
├── rag.py                  # Document ingestion and RAG logic
├── tools.py                # Web search, memory, and RAG tools
├── requirements.txt        # Python dependencies
├── Dockerfile              # Docker image configuration
├── .dockerignore           # Docker ignore rules
├── .gitignore              # Git ignore rules
├── templates/
│   └── index.html          # Frontend UI
├── uploads/                # Uploaded documents
├── data/                   # SQLite database and application data
└── chroma_db/              # Persistent ChromaDB vector store
```

## Docker Deployment

### 1. Build the Docker image

```bash
docker build -t hossam-gpt .
```

### 2. Run the Docker container

```bash
docker run -d \
  --name hossam-gpt \
  --restart unless-stopped \
  -p 8080:8080 \
  --env-file .env \
  hossam-gpt
```

Open [http://localhost:8080](http://localhost:8080) in your browser.

To preserve uploaded files, SQLite data, and ChromaDB data when the container is replaced, add persistent volume mounts that match the paths used by your application.

## AWS CI/CD Deployment with GitHub Actions

This project can be deployed to AWS using:

- GitHub Actions
- Amazon ECR
- Amazon EC2
- Docker
- A GitHub Actions self-hosted runner

### 1. Create an IAM User

Create an IAM user for CI/CD and grant only the permissions required to authenticate with Amazon ECR and push or pull images.

For initial testing, the AWS-managed `AmazonEC2ContainerRegistryFullAccess` policy can be used. For production, replace it with a least-privilege custom policy scoped to the required ECR repository.

`AmazonEC2FullAccess` is not required when deployment commands run locally on the EC2 self-hosted runner. Add EC2 permissions only if the workflow directly manages EC2 resources.

### 2. Create an ECR Repository

Create an Amazon ECR repository, for example:

```text
hossam-gpt
```

An example full ECR image URI is:

```text
315865595366.dkr.ecr.us-east-1.amazonaws.com/hossam-gpt
```

Store only the repository name in the `ECR_REPO` GitHub variable:

```text
ECR_REPO=hossam-gpt
```

Do not store the full ECR image URI as `ECR_REPO`. If your existing ECR repository is named `bappygpt`, use that exact repository name instead.

### 3. Create an EC2 Instance

Create an Ubuntu EC2 instance. For initial testing, add the following inbound security group rule:

```text
Type: Custom TCP
Port: 8080
Source: your-public-IP/32
```

Avoid exposing port `8080` to `0.0.0.0/0` in production. Use a reverse proxy with HTTPS or restrict the allowed source addresses.

### 4. Install Docker on EC2

Connect to your EC2 instance and update the system:

```bash
sudo apt-get update -y
sudo apt-get upgrade -y
```

Install Docker:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

Add the Ubuntu user to the Docker group:

```bash
sudo usermod -aG docker ubuntu
newgrp docker
```

Verify the installation:

```bash
docker --version
```

### 5. Configure EC2 as a GitHub Self-Hosted Runner

In your GitHub repository, go to:

```text
Settings → Actions → Runners → New self-hosted runner
```

Select Linux and follow the commands generated by GitHub. The registration token is temporary, so use the commands shown in your repository rather than copying them into this README.

After setup, start the runner:

```bash
./run.sh
```

For continuous operation, configure it as a service:

```bash
sudo ./svc.sh install
sudo ./svc.sh start
```

## GitHub Secrets and Variables

Keep sensitive credentials in GitHub **Secrets** and non-sensitive configuration in GitHub **Variables**.

### Repository Secrets

Go to:

```text
GitHub Repository → Settings → Secrets and variables → Actions → Secrets → New repository secret
```

Add:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
OPENAI_API_KEY
TAVILY_API_KEY
LANGSMITH_API_KEY
```

`LANGSMITH_API_KEY` is optional when LangSmith tracing is disabled.

### Repository Variables

Go to:

```text
GitHub Repository → Settings → Secrets and variables → Actions → Variables → New repository variable
```

Add:

```text
AWS_DEFAULT_REGION=us-east-1
ECR_REPO=hossam-gpt
OPENAI_MODEL=gpt-5-mini
LANGSMITH_TRACING=false
LANGSMITH_ENDPOINT=https://api.smith.langchain.com
LANGSMITH_PROJECT=HossamGPT
```

If LangSmith tracing is disabled, the endpoint and project variables are optional.

> [!NOTE]
> The names used in your GitHub Actions workflow must match these secret and variable names. Reference secrets with `${{ secrets.NAME }}` and variables with `${{ vars.NAME }}`.

## GitHub Actions Workflow

Create the workflow file at:

```text
.github/workflows/cicd.yaml
```

The workflow should:

1. Check out the repository.
2. Authenticate with AWS.
3. Log in to Amazon ECR.
4. Build the Docker image.
5. Tag and push the image to ECR.
6. Pull the latest image on the EC2 self-hosted runner.
7. Stop and remove the old application container if it exists.
8. Run the new container with the required secrets, variables, ports, restart policy, and persistent volumes.

Do not print API keys in workflow logs or place secret values directly in the workflow file.

## Usage

After running locally or deploying to AWS:

1. Open the application in your browser.
2. Start chatting with the AI assistant.
3. Upload documents to use as context.
4. Ask questions about the uploaded documents.
5. Ask current-information questions to trigger web search.
6. Continue conversations using saved chat history.

## Example Questions

```text
Summarize the uploaded PDF.
```

```text
Search the web for the latest AI agent news.
```

```text
Based on my uploaded document, what are the key points?
```

```text
Calculate 125 * 48 / 6.
```

## Security and Deployment Notes

- Never commit `.env` to GitHub.
- Store API keys and AWS credentials in GitHub Secrets.
- Do not use Uvicorn's `--reload` option in production.
- Restrict inbound traffic to port `8080`, or place the application behind an HTTPS reverse proxy.
- Use least-privilege IAM permissions for production deployments.
- Configure persistent storage for uploads, SQLite, and ChromaDB before replacing containers.
- Rotate any API key or AWS credential that is accidentally exposed.
- Review the security implications before running a self-hosted runner on a public repository.

## Contributing

Contributions are welcome:

1. Fork the repository.
2. Create a new branch.
3. Make and test your changes.
4. Submit a pull request.

## License

This project is open source. See the repository's `LICENSE` file for the applicable terms.
