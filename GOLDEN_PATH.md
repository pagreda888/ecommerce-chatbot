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

## Prompts & Modifications
No specific additional human prompts were required, but some manual logic mapping was done to build the fixes:
- Synthesizing the schema for `db/init.sql` manually by reading Postgres queries from `db/queries.js` and reading the previously broken syntax in the original sql file.
- Updating `db/queries.js` so `new Pool(...)` reads from `process.env`.
- Configuring `#Exclude local node_modules` in `docker-compose.yml` volumes array so `node-sass` isn't destroyed by a local host environment mismatch.

## Step-by-Step Deployment Instructions

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

## What went wrong

- **Docker Daemon Down**: The first deployment attempt failed with the error `unable to get image 'postgres:13': error during connect: Post "http://%2F%2F.%2Fpipe%2Fdocker_engine/...": open //./pipe/docker_engine: The system cannot find the file specified.` This happened because Docker Desktop was not running on the host machine. The issue was resolved by manually starting Docker Desktop before running the deployment commands.
