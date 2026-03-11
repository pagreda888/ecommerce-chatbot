# Setup Golden Path

To eliminate the blockers discovered previously, a set of new artifacts and configuration files has been added to make the project deploy out-of-the-box. 

## Artifacts Created
- `Dockerfile`: Explicitly pins the runtime to Node 8 (so developers do not have to downgrade their local machines to run `npm i`).
- `docker-compose.yml`: Brings up both the Node.js application and a PostgreSQL database in a single command, passing the correct environment variables automatically.
- `.env.example`: Documents the required environment variables.
- `db/init.sql`: Includes the full database schema to initialize the `users` and `products` tables, and includes seed data so the application doesn't crash silently on an empty DB.
- Modified `db/queries.js`: Updated to fall back to `process.env` so that database credentials aren't hardcoded.

## Pain Point Resolution Table

| Pain Point # | Artifact that fixes it | Status |
| --- | --- | --- |
| 1 (`[VERSION_HELL]` Node 8) | `Dockerfile` & `docker-compose.yml` | Fixed |
| 2 (`[BROKEN_CMD]`) | `docker-compose.yml` | Fixed (Users just use `docker-compose up` instead) |
| 3 (`[MISSING_DOC]` Postgres) | `docker-compose.yml` & `db/init.sql` | Fixed |
| 4 (`[ENV_GAP]` Hardcoded DB) | `.env.example` & `db/queries.js` | Fixed |
| 5 (`[SILENT_FAIL]` App missing DB) | `docker-compose.yml` (`depends_on`) & `db/init.sql` (Seed Data) | Fixed |

## AI Usage Notes
**Prompt 1:**
"Design and build the artifacts that eliminate every blocker documented in the PAIN_LOG.md file. "

**Prompt 2:**
"Add to the golden path the new step by step instructions to deploy the project using the new artifacts and deploy the project using the given instructions."
Response from AI agent:
"I attempted to run the deployment command (docker-compose up --build -d) from the 

GOLDEN_PATH.md
 instructions, but it failed because the Docker daemon is currently not running on your machine.

The error (failed to connect to the docker API at npipe:////./pipe/docker_engine) indicates that Docker Desktop needs to be started before the environment can be created.

Could you please start Docker Desktop and let me know when it's ready? I'll re-run the deployment instructions for you immediately after!"

**Prompt 3:**
"Docker desktop is ready
Document the failure in the golden path file"


Produce at least two of the following, chosen based on what the project actually needs:
- A Makefile with a make setup target that installs dependencies, copies env files, runs migrations, and seeds data
- A docker-compose.yml that brings up the full stack (app + any backing services) with a single docker compose up
- A setup.sh script that handles runtime version checks, dependency installation, and environment bootstrap with clear error messages on failure
- A .env.example file with every required variable documented inline with comments explaining valid values
- A .nvmrc, .tool-versions, or Dockerfile that pins the required runtime version explicitly

Required Output — GOLDEN_PATH.md including:
- Info of the artifacts created
- A table: Pain Point # → Artifact that fixes it → Status (Fixed / Partial / Out of Scope)

No specific additional human prompts were required, but some manual logic mapping was done to build the fixes:
- Synthesizing the schema for `db/init.sql` manually by reading Postgres queries from `db/queries.js` and reading the previously broken syntax in the original sql file.
- Updating `db/queries.js` so `new Pool(...)` reads from `process.env`.
- Configuring `#Exclude local node_modules` in `docker-compose.yml` volumes array so `node-sass` isn't destroyed by a local host environment mismatch.

## New Step-by-Step Deployment Instructions

With these artifacts in place, a new engineer can spin up the full project simply by running the following commands:

1. **Clone the repository** and navigate to the root directory.
   ```bash
   git clone <repository_url>
   cd ecommerce-chatbot
   ```

2. **Copy the environment configuration** from the example template. This creates a `.env` file that Docker Compose will use for your environment variables.
   *(Optional, as the `docker-compose.yml` already contains default fallbacks that align with the Postgres setup)*
   ```bash
   cp .env.example .env
   ```

3. **Start the application** via Docker Compose. This step will pull the Postgres database image, build the Node 8 app environment, automatically seed the database schema, and map port 3000 to your host machine.
   ```bash
   docker-compose up --build
   ```

4. **Verify it's running** by visiting the endpoints. The backend and the database are fully connected.
   Go to: http://localhost:3000

## What the AI Got Wrong
- **No deployment instructions**: The AI did not provide any deployment instructions using the new artifacts it created. 

- **Docker Daemon Down**: The first deployment attempt failed with the error `unable to get image 'postgres:13': error during connect: Post "http://%2F%2F.%2Fpipe%2Fdocker_engine/...": open //./pipe/docker_engine: The system cannot find the file specified.` This happened because Docker Desktop was not running on the host machine. The issue was resolved by manually starting Docker Desktop before running the deployment commands.
