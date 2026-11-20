# Proj-SRTandWRS-showcase
# Speech Audiometry (Project Name TBD)

A web application to automate speech audiometry testing—Speech Recognition Threshold (SRT) and Word Recognition Score (WRS)—designed for Urdu first, with support for other languages later.

- **SRT (Speech Recognition Threshold)**: Adaptive up-down procedure targeting the presentation level (dB) where ~50% of words are repeated correctly.
- **WRS (Word Recognition Score)**: Fixed-level word list presentation to measure word recognition percentage.
- **Dual Presentation Modes**:
  - `AUTO`: Adaptive engine adjusts presentation level based on patient responses.
  - `MANUAL`: Clinician sets and adjusts presentation levels directly.
- **Persistence**: PostgreSQL database (hosted on Supabase) via Prisma to record finished sessions and individual trial logs.
- **Audio Output**: Web Audio API tone synthesis with digital gain control (placeholder until equipment calibration).

---

## Installation & Setup

> **Warning**: Use test data only until ethics approval is confirmed.

1. **Install dependencies**:
   ```bash
   npm install
   ```

2. **Set up environment variables**:
   Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```
   Configure your Supabase credentials in `.env`:
   - `DATABASE_URL`: Transaction pooler URL (port 6543, with `?pgbouncer=true`) for runtime queries.
   - `DIRECT_URL`: Direct connection URL (port 5432) for running migrations.
   - `NEXT_PUBLIC_SUPABASE_URL`: Supabase project URL (`https://<project-ref>.supabase.co`).
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`: Supabase public anonymous/publishable key only (never use service role key).

3. **Run database migrations**:
   Apply migrations to your PostgreSQL database:
   ```bash
   npx prisma migrate dev
   ```

4. **Verify database connection**:
   Check connectivity and table row counts:
   ```bash
   npm run db:check
   ```

5. **Start the development server**:
   ```bash
   npm run dev
   ```
   Open [http://localhost:3000](http://localhost:3000) in your browser. Available routes include:
   - `/srt`: Speech Recognition Threshold test
   - `/wrs`: Word Recognition Score test
   - `/sessions`: List of saved test sessions
   - `/login`: Clinician authentication form

---

## Clinician Authentication & Account Management

Authentication is managed server-side using Supabase Auth (`@supabase/ssr`). Open public registration is disallowed.

### 1. Creating and Inviting Clinician Accounts
Clinician accounts must be provisioned directly by an administrator via the Supabase dashboard:
1. Navigate to your project in the **Supabase Dashboard**.
2. Open **Authentication** from the sidebar, then select **Users**.
3. Click **Add user** and select either:
   - **Create user**: Manually enter the clinician's email address and assign an initial password.
   - **Invite user**: Send an email invitation link for the clinician to establish credentials.

### 2. Disabling Open Sign-Up
To protect participant hearing records and ensure only authorized clinical staff can log in, public sign-ups must be disabled:
1. In the Supabase Dashboard, go to **Authentication > Providers > Email**.
2. Turn OFF (disable) **"Allow new users to sign up"** (or uncheck "Enable Sign Up").
3. Save the configuration.

> **Security Note**: Never expose or configure a `SUPABASE_SERVICE_ROLE_KEY` in this application. All client and server operations rely strictly on the scoped `NEXT_PUBLIC_SUPABASE_ANON_KEY` combined with user JWT verification.

---

## Deploying to Vercel

Follow these steps when deploying the project to Vercel:

1. **Set Environment Variables**:
   In your Vercel Project Settings under **Environment Variables**, configure the following for both **Production** and **Preview** environments:
   - `DATABASE_URL`: Supabase transaction pooler URL (port 6543, with `?pgbouncer=true`).
   - `DIRECT_URL`: Supabase direct connection URL (port 5432).
   - `NEXT_PUBLIC_SUPABASE_URL`: Supabase project URL (`https://<project-ref>.supabase.co`).
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`: Supabase public anonymous key.

2. **URL-Encode Special Characters in Database Password**:
   If your database password contains special characters (such as `@`, `:`, `/`, `?`, `#`, `[`, `]`, `%`), ensure they are properly URL-encoded (e.g. `%40` for `@`, `%3A` for `:`) in both `DATABASE_URL` and `DIRECT_URL` to prevent connection string parsing failures.

3. **Run Migrations Before First Deploy**:
   Vercel build pipelines do not run database migrations. Run migrations from your local machine against the Supabase database before your initial deployment:
   ```bash
   npx prisma migrate deploy
   ```

4. **Redeploy After Variable Updates**:
   Always redeploy your project in Vercel after modifying environment variables to ensure the new values are applied to runtime instances.

---

## Running Tests

Run the Vitest test suite (single-run mode):
```bash
npm test
```

---

## Running Simulation Experiments

Execute experiments using the `npm run exp` script:

- **Compare step sizes across starting levels**:
  ```bash
  npm run exp experiments/compare-step-sizes.ts
  ```

- **Trace an individual staircase session trial by trial**:
  ```bash
  npm run exp experiments/trace-session.ts
  ```

- **Compare stopping rules and reversal limits**:
  ```bash
  npm run exp experiments/compare-stopping-rules.ts
  ```

---

## Folder Layout

- **`src/engine/`**: Pure audiometric logic with no React, DOM, audio, database, or network code.
  - `staircase.ts`: Adaptive up-down staircase implementation for SRT.
  - `wrs.ts`: Word Recognition Score test runner and seeded list randomizer.
  - `session.ts`: Session state machine supporting both `auto` and `manual` modes.
  - `simulation.ts`: Seeded Mulberry32 PRNG and logistic `SimulatedListener` for Monte Carlo verification.
- **`src/audio/`**: Web Audio API playback.
  - `player.ts`: Generates tone bursts and calculates linear gain via `dbToGain` with strict maximum gain clamping.
- **`src/config/`**: System configuration.
  - `calibration.ts`: Transducer calibration offsets and reference level definitions (placeholder values).
- **`src/data/`**: Data assets.
  - `placeholder-words.ts`: Hardcoded placeholder word identifier lists (`word-01` ... `word-34`).
- **`src/app/`**: Next.js App Router.
  - `page.tsx`: Home landing page.
  - `srt/page.tsx`: Interactive SRT test interface.
  - `wrs/page.tsx`: Interactive WRS test interface.
  - `sessions/page.tsx`: Saved sessions history table.
  - `api/sessions/`: REST route handlers for saving and querying test sessions.
- **`src/lib/`**: Shared utilities.
  - `prisma.ts`: Prisma Client singleton.
  - `validation.ts`: Request validation for session persistence.
- **`experiments/`**: Simulation scripts for evaluating clinical step sizes, stopping rules, and staircase dynamics.

---

## Open Decisions

The following items are placeholders or pending clinical specification:

1. **Threshold formula**: Exact clinical calculation formula for threshold estimation (particularly for manual presentation mode).
2. **Start level**: Clinical default starting level in dB.
3. **Step size**: Default presentation step size (dB) per trial.
4. **Stopping rule**: Target stopping criteria (reversal count vs. maximum trial budget).
5. **Discarded reversals**: Number of initial reversal turnarounds excluded from threshold averaging.
6. **Audible range**: Minimum and maximum allowable presentation levels in dB.
7. **Calibration method**: Method and equipment standards (e.g. artificial ear/coupler and sound level meter) for calibrating digital output to true dB HL.
8. **Meaning of "33 words" in SRT notes**: Clarification needed on the source and role of the "33 words" referenced in project notes.
9. **Project name**: Final project title to replace the working placeholder.
