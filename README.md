# Automating Speech Audiometry in Pakistani Languages

> A research project to automate speech audiometry (SRT and WRS) using validated Urdu word lists, and to build matching word lists for other Pakistani languages.

**Role:** Project Lead · **Supervisor:** Dr Shibli Nisar · **Status:** Active research (software prototype built; data collection ongoing)

> The source code lives in a private repository because the project is in the ethics-approval and data-collection phase. This repo is a public overview of the project, its design, and my role in it. No participant data or clinical recordings are shared here.

---

## The Problem

Speech audiometry measures how well a person hears *speech*, not just tones. In Pakistan, it has a gap:

- Urdu speakers are routinely tested with **English word lists**, even though English is not the first language of most patients, which can distort results.
- Testing is **manual**: a trained audiologist has to run every step of every session.
- Validated Urdu lists only appeared in **2025**, and other regional languages have none.

## Two Tests

| Test | Question it answers | How it works | Result |
|---|---|---|---|
| **SRT** (Speech Recognition Threshold) | How quiet can speech be before it is lost? | Two-syllable words; the level moves up and down to find the point where ~50% are repeated correctly | A level in dB HL |
| **WRS** (Word Recognition Score) | How clearly is speech understood at a comfortable level? | One-syllable words at a fixed level | Percent correct |

## What We Build On

The project uses the Urdu SRT and WRS word lists published by Tanveer & Rauf (2025) in the *Journal of Speech, Language, and Hearing Research* (30 two-syllable SRT words, 34 one-syllable WRS words, validated with 88 children).
DOI: [10.1044/2025_JSLHR-24-00118](https://doi.org/10.1044/2025_JSLHR-24-00118)

That work leaves open questions: normal-hearing children only, a single female voice, no test-retest data, and no automated procedure. This project addresses the last gap and extends the method to more languages.

---

## Project Structure: Two Streams

### Stream A: Automating the Test
Software that runs SRT and WRS with the Urdu lists and scores the results.

- **Phase 1:** The software plays calibrated recordings and sets levels by rule; the audiologist marks each response. *(Prototype complete.)*
- **Phase 2:** The system captures and scores the patient's response itself, replacing manual marking.
- **Validation:** Pilot comparison against manual testing.

| Manual | Automated |
|---|---|
| Audiologist sets the level | Software sets the level by rule |
| Word plays in headphones | Calibrated word plays |
| Patient repeats the word | Patient responds |
| Audiologist marks right or wrong | Software scores the response |
| Audiologist picks the next level | Software picks the next level |

Same words, same recordings. Only *who runs the steps* changes.

### Stream B: Word Lists for Other Languages
SRT and WRS lists for Punjabi, Pashto, Sindhi, and Balochi, built with the same method used for Urdu:

1. Select words from a language corpus
2. Record one clear native speaker
3. Test familiarity with native listeners
4. Measure each word at many levels
5. Keep the words with steep, similar slopes

Work begins with a feasibility check (is there a corpus, are linguists and a recording space available?), then one language at a time.

---

## Software Overview

**Features of the prototype**

- **SRT** using an adaptive up-down staircase
- **WRS** using fixed-level word lists with a seeded randomizer
- **Two presentation modes:** `AUTO` (adaptive engine adjusts the level) and `MANUAL` (clinician sets the level)
- **Session logging** of every trial for later analysis
- **Clinician-only access** (no open registration)
- **Calibrated audio output** with a hard cap on maximum gain

**Design choices worth noting**

- The audiometric logic is a **pure, framework-free engine**, separate from the UI, audio, and database code, so it can be tested on its own.
- The adaptive staircase is verified with **Monte Carlo simulation**: a simulated listener with a known threshold is run through the engine thousands of times, to check that the procedure recovers the true value and to compare step sizes and stopping rules.
- Unit-tested with Vitest.

**Stack:** TypeScript · Next.js · React · Web Audio API · PostgreSQL · Prisma · Vitest

<img width="1470" height="830" alt="image" src="https://github.com/user-attachments/assets/e2944c1a-24ee-4400-bb0c-9ef34ebf026a" />
<img width="1470" height="828" alt="image" src="https://github.com/user-attachments/assets/13508718-7aed-4a49-9701-9807fd1571bd" />
<img width="1468" height="830" alt="image" src="https://github.com/user-attachments/assets/de3e1641-f20a-4cc0-83cb-15f9b2f3feaa" />




---

## My Role

- Lead the project, which has two streams (test automation and regional-language word lists), and coordinate the team
- Secured faculty supervision and presented the project to the supervisor's MS class
- Reached out to the authors of the source paper and the organisation that built the Urdu lists, and prepared the formal request for the word recordings
- Defined the project scope, phases, and risks (see below)
- Own the clinical and research decisions behind the software: response method, start level, step size, stopping rule, and calibration approach
- Software developed in collaboration with **Waleed Malik** ([GitHub](https://github.com/Waleed-Malik))

## Scope and Limitations

| In scope | Not yet |
|---|---|
| SRT and WRS in quiet | Sentence and noise tests |
| Normal-hearing listeners first | Clinical patients |
| Urdu first, then one more language | |

**Open risks:** recording permissions pending, audiometer access and calibration, ethics approval required before any testing, and the limits of the Urdu lists (children only, one voice, no test-retest data).

## Roadmap

| Phase | Stream A: Automation | Stream B: Languages |
|---|---|---|
| 1. Foundation | Recordings, permissions, protocol | Pick one language, feasibility check |
| 2. Build | Working prototype, calibration | Word selection and recording |
| 3. Test | Pilot against manual testing | Familiarity testing |
| 4. Share | Paper or poster, open code | Slope testing, report |


## References

- Tanveer S, Rauf S. Development of Urdu Speech Audiometry Material for the Deaf and Hard of Hearing Community. *J Speech Lang Hear Res.* 2025;68(6):2900-2914. [doi:10.1044/2025_JSLHR-24-00118](https://doi.org/10.1044/2025_JSLHR-24-00118)
- Ijaz T, Kiyani HS, Saqulain G. *Annals of King Edward Medical University.* 2023;23(4):323-328.

## Contact

**MARYAM ABBASI** · www.linkedin.com/in/maryam-abbasi-49933b361 · mryamabbasiii6@gmail.com
