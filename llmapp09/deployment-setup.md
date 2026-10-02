# Local and GitHub deployment setup

## Local API keys

1. Copy `.env.example` to `.env` at the repository root and replace the placeholder with real values locally. `.env` is ignored by Git; never commit it or paste keys into source code.
2. `OLLAMA_API_KEY` is required for model inference. `LANGFUSE_PUBLIC_KEY` and `LANGFUSE_SECRET_KEY` are optional and enable Langfuse tracing.
3. `OPENAI_API_KEY` is used by `deepeval-tests` as the evaluation judge, not by the application containers. To run those tests from a terminal, export the values from `.env` into that shell before running DeepEval.
4. Hugging Face keys (`HF_TOKEN` / `HUGGINGFACEHUB_API_TOKEN`) are not currently read anywhere in this repository. Don't add them to the container or GitHub unless a Hugging Face integration is added.

## Build and run locally

Install and start Docker Desktop, then from the repository root run:

```sh
docker compose up --build --detach
```

Open the frontend at <http://localhost:5000>; the backend API is at <http://localhost:8080>. To stop the stack, run `docker compose down`.

## GitHub Actions and repository secrets

The `Docker Compose Smoke Test` workflow builds both images on a GitHub-hosted runner, starts both containers, and verifies the frontend and backend respond. It needs no API key because it only checks the frontend page and the backend routing table; it does not call an LLM. GitHub-hosted workflow containers are temporary and stop when the job ends—they are not a persistent deployment.

For live Promptfoo/DeepEval workflows, add these under **Repository → Settings → Secrets and variables → Actions → New repository secret**:

- `OLLAMA_API_KEY` — required to make Ollama Cloud inference calls.
- `OLLAMA_BASE_URL` — optional; defaults to `https://ollama.com`.
- `OPENAI_API_KEY` — required only by DeepEval's judge model.
- `LANGFUSE_PUBLIC_KEY`, `LANGFUSE_SECRET_KEY` — optional Langfuse tracing credentials; `LANGFUSE_HOST` is optional and defaults to `https://cloud.langfuse.com`.

The Docker Hub publishing workflows use the `erwinpermatalim` namespace and require a Docker Hub access token stored as the `DOCKERHUB_TOKEN` repository secret. Hugging Face secrets are not needed by the current code.

Secrets are not available to workflows triggered from forks, so live model-evaluation jobs may fail on forked pull requests. The Docker Compose smoke workflow remains secret-free.

## Push to your GitHub repository

The current Git remote is `https://github.com/erwinpermatalim/llmapp09_EP.git`. Commit only source/configuration templates (never `.env`), then push the intended branch; pushing to `main` triggers the workflows above.
