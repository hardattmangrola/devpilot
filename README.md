# DevPilot

DevPilot is a developer-focused codebase assistant that connects to GitHub, indexes repository source code, and lets users ask questions grounded in that code. It combines retrieval-augmented generation (RAG) with source citations so answers can be traced back to the relevant files and lines.

The project is organized as a Next.js client and a Spring Boot API. PostgreSQL with the pgvector extension stores application data and vector embeddings, while GitHub OAuth provides repository-aware authentication.

## Table of Contents

- [What It Does](#what-it-does)
- [Tech Stack](#tech-stack)
- [Screenshots](#screenshots)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Configuration](#configuration)
- [Local Setup](#local-setup)
- [Useful Commands](#useful-commands)
- [Development Notes](#development-notes)
- [Troubleshooting](#troubleshooting)

## What It Does

- Authenticate users with GitHub OAuth.
- List the repositories available to the authenticated GitHub user.
- Index repository files into searchable code chunks.
- Generate embeddings for indexed code and store them in PostgreSQL with pgvector.
- Retrieve relevant code context for a question.
- Stream AI-generated answers in the chat interface.
- Display citations for the files and line ranges used in an answer.

## Tech Stack

### Core Frameworks and Languages

[![Next.js](https://img.shields.io/badge/Next.js-16.3.2-000000?logo=next.js&logoColor=white)](https://nextjs.org/)
[![React](https://img.shields.io/badge/React-19.2.8-20232A?logo=react&logoColor=61DAFB)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Java](https://img.shields.io/badge/Java-21-ED8B00?logo=openjdk&logoColor=white)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.1.1-6DB33F?logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)

### Database and Storage

[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![pgvector](https://img.shields.io/badge/pgvector-Vector%20Search-336791?logo=postgresql&logoColor=white)](https://github.com/pgvector/pgvector)
[![Hibernate](https://img.shields.io/badge/Hibernate-JPA-59666C?logo=hibernate&logoColor=white)](https://hibernate.org/)

### Artificial Intelligence and RAG

[![Spring AI](https://img.shields.io/badge/Spring%20AI-RAG-6DB33F?logo=spring&logoColor=white)](https://spring.io/projects/spring-ai)
[![OpenAI](https://img.shields.io/badge/OpenAI-Chat%20%26%20Embeddings-412991?logo=openai&logoColor=white)](https://openai.com/)
[![GitHub](https://img.shields.io/badge/GitHub-OAuth-181717?logo=github&logoColor=white)](https://github.com/)

### UI, Styling, and Tooling

[![Tailwind CSS](https://img.shields.io/badge/Tailwind%20CSS-4-06B6D4?logo=tailwindcss&logoColor=white)](https://tailwindcss.com/)
[![shadcn/ui](https://img.shields.io/badge/shadcn%2Fui-Components-000000?logo=shadcnui&logoColor=white)](https://ui.shadcn.com/)
[![TanStack Query](https://img.shields.io/badge/TanStack%20Query-5-FF4154?logo=reactquery&logoColor=white)](https://tanstack.com/query)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Maven](https://img.shields.io/badge/Maven-Wrapper-C71A36?logo=apachemaven&logoColor=white)](https://maven.apache.org/)

The frontend uses Next.js App Router, React, TypeScript, Tailwind CSS, TanStack React Query, Lucide React, React Icons, and Streamdown. The backend uses Spring MVC, Spring Security OAuth 2.0, Spring Data JPA, Spring AI, and Lombok.

## Screenshots

### Landing Page

![DevPilot landing page](asset/landing_page.png)

### Repository Overview

![Repository overview](asset/overview.png)

### Repository Details

![Repository details](asset/repo_page.png)

### Code Chat

![Code chat interface](asset/chat_ui.png)

### RAG Response with Citations

![RAG response with citations](asset/rag_response.png)

### Settings

![Settings page](asset/setting.png)

### Indexed Repository

![Indexed repository view](asset/actual_repo.png)

## Project Structure

```text
.
├── backend/                 Spring Boot API and indexing/RAG services
├── client/                  Next.js web application
├── docker/
│   └── postgres/            PostgreSQL initialization scripts
└── docker-compose.yml       Local PostgreSQL + pgvector service
```

## Prerequisites

Install the following before starting development:

- Git
- Java Development Kit (JDK) `21`
- Node.js with npm
- Docker Desktop with Docker Compose
- A GitHub OAuth application
- An OpenAI API key with access to the configured chat and embedding models

## Configuration

The backend reads configuration from environment variables and uses local development defaults for the database and frontend URLs. Create your own environment configuration before running the application.

### Backend variables

| Variable | Purpose | Local example |
| --- | --- | --- |
| `DB_URL` | PostgreSQL JDBC connection string | `jdbc:postgresql://localhost:5433/devpilot` |
| `DB_USERNAME` | PostgreSQL username | `postgres` |
| `DB_PASSWORD` | PostgreSQL password | `postgres` |
| `OPENAI_API_KEY` | OpenAI API key for chat and embeddings | Set privately |
| `GITHUB_CLIENT_ID` | GitHub OAuth client ID | Set privately |
| `GITHUB_CLIENT_SECRET` | GitHub OAuth client secret | Set privately |
| `FRONTEND_URL` | Frontend origin used by the backend | `http://localhost:3000` |
| `CORS_ALLOWED_ORIGINS` | Allowed browser origins | `http://localhost:3000` |
| `TOKEN_ENCRYPTOR_PASSWORD` | Password used to encrypt stored GitHub tokens | Use a strong local value |
| `TOKEN_ENCRYPTOR_SALT` | Salt used by the token encryptor | Use a unique local value |

The GitHub OAuth callback should be configured as:

```text
http://localhost:8080/login/oauth2/code/github
```

### Frontend variables

Create `client/.env.local` when the backend is not running at its default address:

```env
NEXT_PUBLIC_API_BASE_URL=http://localhost:8080
```

Do not commit API keys, OAuth secrets, or production encryption values. Keep secrets in local environment files or a managed secret store.

## Local Setup

### 1. Start PostgreSQL and pgvector

From the repository root:

```bash
docker compose up -d postgres
```

The database is available at `localhost:5433` with these local defaults:

- Database: `devpilot`
- Username: `postgres`
- Password: `postgres`

The Compose service also runs the initialization script that enables the required PostgreSQL extensions.

### 2. Configure the backend

Set the backend variables in your shell or IDE run configuration. For PowerShell, for example:

```powershell
$env:OPENAI_API_KEY = "your-openai-api-key"
$env:GITHUB_CLIENT_ID = "your-github-client-id"
$env:GITHUB_CLIENT_SECRET = "your-github-client-secret"
$env:TOKEN_ENCRYPTOR_PASSWORD = "a-local-development-password"
$env:TOKEN_ENCRYPTOR_SALT = "a-local-development-salt"
```

Start the backend from `backend/`:

```powershell
cd backend
./mvnw.cmd spring-boot:run
```

On macOS or Linux, use `./mvnw spring-boot:run` instead.

The API is available at `http://localhost:8080`.

### 3. Install and start the frontend

In a second terminal:

```bash
cd client
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in a browser and choose **Continue with GitHub** to begin.

## Useful Commands

### Frontend

```bash
cd client
npm run dev       # Start the development server
npm run lint      # Run ESLint
npm run build     # Create a production build
npm run start     # Serve the production build
```

### Backend

```powershell
cd backend
./mvnw.cmd test
./mvnw.cmd clean package
./mvnw.cmd spring-boot:run
```

### Database

```bash
docker compose ps

docker compose logs -f postgres
docker compose down
```

Use `docker compose down -v` only when you intentionally want to remove the local PostgreSQL data volume.

## Development Notes

- The backend creates or updates the application schema through Hibernate in the current development configuration.
- Vector embeddings use 1536 dimensions, matching the configured `text-embedding-3-small` model.
- Chat replies are streamed from the backend to the client using server-sent events.
- GitHub repository access is scoped through OAuth permissions, so the OAuth application must be configured with the repository scope required by the application.
- For production deployments, replace local defaults, use managed secrets, configure HTTPS callback URLs, and review CORS and session-cookie settings.

## Troubleshooting

### The frontend cannot reach the API

Confirm that the backend is running on port `8080` or set `NEXT_PUBLIC_API_BASE_URL` in `client/.env.local` to the correct API URL.

### Database connection fails

Check that Docker is running and that PostgreSQL is healthy:

```bash
docker compose ps
docker compose logs postgres
```

Also confirm that the backend is using port `5433`, which is the host port mapped by `docker-compose.yml`.

### GitHub login fails

Verify the GitHub OAuth client ID and secret, and make sure the callback URL exactly matches:

```text
http://localhost:8080/login/oauth2/code/github
```

### Indexing or chat fails

Check that `OPENAI_API_KEY` is available to the backend and that PostgreSQL was started with the pgvector extension enabled.
