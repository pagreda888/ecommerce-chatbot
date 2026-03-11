# Pain Log — ecommerce-chatbot

A numbered list of every friction point encountered while attempting a full local setup following **only** the README.

---

1. **[VERSION_HELL]** The README specifies `Node^8`, which is heavily outdated. Running `npm i` on any modern Node.js environment fails because `node-sass@4.9.4` (listed in `devDependencies`) cannot compile its native bindings against newer V8 engines. This is also the root cause of `npm start` failing with `sh: node-sass: command not found` — the package never builds successfully, so no executable is placed in `node_modules/.bin/`.
   - **Severity:** BLOCKER — the very first step (`npm i`) fails on any machine running Node ≥ 16. A developer must discover on their own that they need to downgrade Node (e.g. via NVM) or replace `node-sass` with the modern `sass` package.

2. **[BROKEN_CMD]** The README instructs the user to run `$ npm run compile:sass // watch sass change`. Copy-pasting this line verbatim into most shells (Windows PowerShell, CMD, and even some zsh configurations) will produce a command-line error because `//` is not a valid shell comment — only `#` is. Additionally, the leading `$` is a docs convention that will also cause errors if copied literally.
   - **Severity:** MEDIUM — causes confusion and a failed command for anyone who copy-pastes directly from the README.

3. **[MISSING_DOC]** The application requires a running PostgreSQL database to function, but the README contains absolutely no instructions on how to install PostgreSQL, create the database, or initialize the required schema. A SQL file exists at `db/init.sql`, but the README never references it. A new engineer must discover this file on their own and figure out how to apply it.
   - **Severity:** BLOCKER — without a running, correctly initialized database the application's API routes will all fail.

4. **[ENV_GAP]** The application depends on environment variables and database credentials that are entirely undocumented.
   - `app.js` reads `process.env.PORT` at startup; if `PORT` is undefined the server calls `app.listen(undefined)`, which silently binds to a random port or fails.
   - Database connection credentials (`user: 'jhonatan'`, `password: 'jhonatan'`, `database: 'ebot'`) are **hardcoded** in `db/queries.js` instead of being read from environment variables or a config file.
   - There is no `.env.example` file and no mention of required variables anywhere in the docs. A new engineer must read the source code to discover these settings and either create a matching local Postgres role or change the code.
   - **Severity:** BLOCKER — the app cannot start correctly without knowing the port, and API routes cannot work without the exact Postgres user/password/database.

5. **[SILENT_FAIL]** The `pg` connection `Pool` does not crash the application when the database is unreachable. After working around the previous blockers, the app will happily report `Listening on port 3000`, giving the false impression that everything is fine. However, any HTTP request that hits a database query (`/users`, `/products`, etc.) will return a raw 500 error or trigger an unhandled promise rejection. There is no health-check, no startup connection test, and no user-friendly error message.
   - **Severity:** HIGH — a developer can waste significant time believing the app is running correctly when it is not.

---

## Severity Summary

| Metric | Value |
|---|---|
| **Total friction points found** | **5** |
| **First complete blocker** | Step 1 — `npm i` fails immediately on any modern Node version due to `node-sass@4.9.4` native build failure. |
| **Estimated time lost for a new hire** | **2–4 hours** (time spent downgrading Node / configuring NVM, manually reading source code to find hardcoded DB credentials, reverse-engineering the required SQL schema from raw query strings, and debugging silent 500 errors). |
