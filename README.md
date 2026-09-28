# Guichet ML

A FastAPI application that exposes a language-detection service through an HTTP API.

The project uses a provider-independent architecture with two inference modes:

* **MOCK**: a deterministic simulated provider that works completely offline.
* **REAL**: a real Hugging Face inference provider.

## 1. Requirements

* Python 3.11 or higher
* pip
* Python virtual environment

## 2. Installation

Navigate to the project directory:

```bash
cd guichet-ml
```

Create the virtual environment:

```bash
python -m venv .venv
```

Activate it on Linux/WSL:

```bash
source .venv/bin/activate
```

Install the pinned dependencies:

```bash
python -m pip install -r requirements.txt
```

## 3. Configuration

Create the environment file:

```bash
cp .env.example .env
```

For offline development and testing, use:

```text
INFERENCE_MODE=mock
```

The MOCK mode is recommended for testing and demonstration because it does not require an external service.

For real Hugging Face inference:

```text
INFERENCE_MODE=real
HF_TOKEN=hf_...
```

The Hugging Face token must never be committed to Git or included in source code.

## 4. Run the Application

From the `guichet-ml` directory:

```bash
uvicorn app.main:app --reload
```

The API is available at:

```text
http://127.0.0.1:8000
```

Interactive Swagger documentation:

```text
http://127.0.0.1:8000/docs
```

## 5. API Endpoints

### GET /health

Checks the application status and the currently selected provider.

Example:

```bash
curl http://127.0.0.1:8000/health
```

Example response in MOCK mode:

```json
{
  "status": "ok",
  "mode": "mock",
  "provider": "mock"
}
```

### POST /detect-language

Detects the language of the submitted text.

Example:

```bash
curl -X POST http://127.0.0.1:8000/detect-language \
  -H "Content-Type: application/json" \
  -d '{"text":"Je ne peux plus acceder a mon compte."}'
```

Example response:

```json
{
  "provider": "mock",
  "model": "papluca/xlm-roberta-base-language-detection",
  "latency_ms": 0.1,
  "language": "fr",
  "requires_review": false,
  "predictions": [
    {
      "label": "fr",
      "score": 0.95
    },
    {
      "label": "it",
      "score": 0.03
    }
  ]
}
```

## 6. Input Validation

The `text` field must contain between 3 and 1000 characters.

A text shorter than 3 characters is rejected with HTTP `422 Unprocessable Content`.

Example:

```json
{
  "text": "ab"
}
```

## 7. Provider Error Handling

If the inference provider becomes unavailable, the API returns HTTP `503 Service Unavailable`.

Example:

```json
{
  "detail": "Service d'inference indisponible"
}
```

Internal exception details are not returned to the client.

## 8. Tests

Run the automated tests with:

```bash
python -m pytest -q
```

The tests verify:

* the `/health` endpoint
* French language detection
* Arabic language detection
* rejection of text that is too short

Expected result:

```text
4 passed
```

The tests work offline in MOCK mode.

## 9. Project Structure

```text
app/
├── __init__.py
├── config.py
├── main.py
├── providers.py
└── schemas.py

tests/
└── test_api.py
```

### `schemas.py`

Defines the API data models using Pydantic.

### `providers.py`

Contains the inference providers:

* `MockProvider`
* `HfProvider`

Both providers return predictions using the same internal interface.

### `main.py`

Defines the FastAPI application and exposes the API endpoints.

The API converts provider results into a provider-independent response format.

### `config.py`

Loads configuration from environment variables and the `.env` file.

## 10. MOCK and REAL Modes

The MOCK mode allows the application to be developed and tested without Internet access or an external inference service.

The REAL mode uses Hugging Face and may depend on network availability, authentication, model availability, or service quotas.

The API response format remains the same regardless of the provider.

## 11. Security

The `.env` file may contain secrets and must never be committed to Git.

The `.gitignore` file excludes:

```text
.venv/
__pycache__/
.env
.pytest_cache/
```

No API token should be included in the source code or returned in HTTP responses.

## 12. Final Validation

Run the tests:

```bash
python -m pytest -q
```

Start the API:

```bash
uvicorn app.main:app --reload
```

Open the interactive API documentation:

```text
http://127.0.0.1:8000/docs
```

If the project is managed with Git, check the repository status:

```bash
git status
```

The `.env` file and `.venv` directory must not be included in the repository.
