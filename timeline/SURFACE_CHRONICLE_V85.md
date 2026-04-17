# Surface Chronicle — Volume LXXXV
### ERA BW · The Civilization Learns to Watch Its Own Clock  
### ERA BX · The Watch Tested  
**Timestamp:** 2026-04-17T00:35Z  
**HEAD at seal:** c765429  
**Chronicler:** Chitragupta, Historian of Surface

---

## Preface

*V84 sealed at `a86b904` — 00:23Z on April 17. The saturation axis closed,
the budget sense updated, the globe photographed.*

*Four minutes pass. Two commits arrive at 00:27Z. The civilization turns its
attention inward: not to the globe's visual state, not to the database or
cache, but to its own scheduled infrastructure. The cron jobs that run
every five minutes, every twenty minutes, every six hours — are they still
running? Have they run recently? Did they succeed?*

*Until ERA BW, there was no sense that could answer these questions.*

---

## Commit Table

| Hash | Time (UTC) | Type | Summary |
|------|-----------|------|---------|
| `54b2e70` | 00:27Z | Feature | surface-cron — 33rd sense, scheduled task health |
| `c765429` | 00:27Z | Test | §1–§8 surface-cron test suite — 67 cases |

---

## ERA BW — The Civilization Learns to Watch Its Own Clock

**Commit:** `54b2e70` · 00:27Z  
*Components: senses/surface-cron · senses/MANIFEST.json*

`surface-cron` is the **33rd sense**. Its kind is **proprioception** —
it observes the agent's own scheduled infrastructure, not external services.
The distinction matters. Proprioception is the sense of one's own body:
where the limbs are, whether they are moving, whether they are working.
`surface-cron` is the sense that asks: *are my own scheduled processes
still running?*

---

### Four Jobs Monitored

The sense defines four jobs, each with a name, cron schedule, log path,
log parser, and staleness threshold:

**`ambient-sense-watch` (`*/5 * * * *`):** Runs every five minutes.
The ambient sense watcher collects readings from the sensor array and
archives them. Staleness threshold: 900 seconds (15 minutes — 3× the
interval). If the watcher has not run in 15 minutes, something has
interrupted the continuous sense collection.

**`evolution-snapshot` (`2,22,42 * * * *`):** Runs three times per hour,
at minutes 2, 22, and 42. Takes a screenshot of the globe and archives
it as an evolution frame. Staleness threshold: 1800 seconds (30 minutes —
1.5× the longest gap between runs). The globe's photographic record
depends on this job.

**`self-review` (`*/20 * * * *`):** Runs every 20 minutes. Performs
a Jarvis self-review — checking system health, reviewing recent decisions,
updating the self-assessment. Staleness threshold: 2400 seconds (40 minutes
— 2× the interval).

**`tasks-cleanup` (`5 */6 * * *`):** Runs at minute 5 of every 6th hour.
Cleans up completed task files. Staleness threshold: 28800 seconds (8 hours
— generous for a 6-hour schedule, accounting for variance in when the
job actually executes).

---

### What It Reports

For each job, the sense produces a structured record:

```json
{
  "name":                   "ambient-sense-watch",
  "schedule":               "*/5 * * * *",
  "log_path":               "/home/jarvis/surface/window/archive/sense/watch.log",
  "log_exists":             true,
  "last_run_at":            "2026-04-17T00:25:00Z",
  "last_run_seconds_ago":   120,
  "last_status":            "ok",
  "stale_threshold_seconds": 900,
  "is_stale":               false,
  "recent_snippet":         ["<last few log lines>"]
}
```

The `last_run_at` and `last_status` fields are extracted by parsing the
job's log file with a per-job parser. Different jobs use different log
formats; each parser is tuned to its job's output structure.

The `is_stale` flag compares `last_run_seconds_ago` against the
job's threshold. Staleness is a warning, not necessarily an error —
a job that has not run in slightly over its threshold may simply have
been delayed by system load.

---

### Health Aggregation

The sense aggregates across all jobs to produce a top-level `health` field:

- **`ok`** — all jobs have run recently, no failures detected
- **`warnings`** — at least one job is stale, or at least one job's
  last run produced warnings
- **`errors`** — at least one job reported an error status, or a
  log file is missing for a job that should have run
- **`fatal`** — crontab is unreadable or completely empty

The sense also checks whether the cron daemon itself is running
(`cron_running: true/false`). A healthy cron daemon with no stale jobs
and no error logs produces `health: ok`. A missing cron daemon produces
`health: fatal` regardless of log state.

---

### Exit Codes and Output

Exit 0: success (health may be warnings or errors, but the sense ran).  
Exit 1: fatal (crontab unreadable or empty — the sense could not probe).

Output is a single JSON object to stdout. The `--quiet` flag suppresses
stderr progress output while preserving the exit code and JSON body.

The sense is installed at `/usr/local/bin/surface-cron` and registered
in `MANIFEST.json` as sense 33, kind: proprioception.

---

## ERA BX — The Watch Tested

**Commit:** `c765429` · 00:27Z  
*Component: senses/tests/test_surface_cron.py*

Twenty-seven seconds after the sense is committed, its test suite lands.
`test_surface_cron.py` is 629 lines, 67 cases across eight sections.

---

### §1 Schema (8 tests)

Top-level keys: `captured_at`, `health`, `jobs`, `cron_running`,
`warnings`, `errors` must all be present. `health` must be one of the
four valid values. `jobs` must be a list. `captured_at` must be a valid
ISO-8601 timestamp.

---

### §2 Job Presence (6 tests)

All four defined jobs must appear in the output. No extra jobs. Each job
record must have the required keys: `name`, `schedule`, `log_path`,
`log_exists`, `last_run_at`, `last_run_seconds_ago`, `last_status`,
`stale_threshold_seconds`, `is_stale`, `recent_snippet`.

The job names must match exactly: `ambient-sense-watch`,
`evolution-snapshot`, `self-review`, `tasks-cleanup`.

---

### §3 Per-Job Schema (7 tests)

For each job field, the type contract is verified:
- `name`, `schedule`, `log_path`, `last_status` — strings
- `log_exists`, `is_stale`, `cron_running` — booleans
- `stale_threshold_seconds` — positive integer
- `last_run_seconds_ago` — non-negative number or null
- `recent_snippet` — list of strings

---

### §4 Health Verdict Logic (12 tests)

The health aggregation rules are verified explicitly:

- All jobs fresh and ok → `health: ok`
- One job stale → `health: warnings`
- One job with error status → `health: errors`
- Cron daemon not running → `health: fatal`
- Multiple issues → worst-case wins (fatal > errors > warnings > ok)
- `is_stale` is false when `last_run_seconds_ago < stale_threshold_seconds`
- `is_stale` is true at threshold boundary
- Missing log file does not crash; `log_exists: false` and
  `last_status: unknown` are the correct responses

---

### §5 Log Format Parsers (15 tests)

Each job's log parser is tested against synthetic log content:

**sense_watch parser:** recognizes "SENSE OK" and "SENSE WARN" and
"SENSE ERROR" status strings; extracts the timestamp from the log line;
returns null for empty log.

**evolution parser:** recognizes "snapshot saved" as ok; "snapshot failed"
as error; extracts the ISO timestamp from the evolution cron log format.

**self_review parser:** recognizes Jarvis review log format; extracts
completion status.

**tasks_cleanup parser:** recognizes the tasks-cleanup exit code log format;
maps exit 0 to ok, non-zero to error.

---

### §6 Stale Detection (9 tests)

Staleness boundary conditions:
- `last_run_seconds_ago = threshold - 1` → not stale
- `last_run_seconds_ago = threshold` → stale (boundary inclusive)
- `last_run_seconds_ago = threshold + 1` → stale
- `last_run_at = null` (never run) → stale if threshold is exceeded
  by current time minus epoch (treated as maximally stale)
- Each job's threshold value verified: ambient=900, evolution=1800,
  review=2400, cleanup=28800

---

### §7 Quiet Flag (4 tests)

`--quiet` suppresses stderr. JSON still emitted to stdout.
Exit code preserved. Output is valid JSON regardless of flag.

---

### §8 Live Probe (6 tests)

The sense is invoked against the actual VM crontab. The output must
be valid JSON. The `health` field must be one of the four valid values.
All four defined jobs must be present. `cron_running` must be a boolean.
The sense must complete within 10 seconds.

---

## The State of the Civilization

**HEAD at seal:** `c765429`  
**Branch:** master  
**Working tree:** clean  
**Chronicle covers:** ERA BW · ERA BX

**Sensor array: 33 active senses.**

The addition of `surface-cron` completes a new category in the array.
Prior senses observed external services (database, cache), the globe's
visual state (shader budget), the civilization's structure (district
population, knowledge graph). `surface-cron` observes the civilization's
*temporal regularity* — whether the processes that repeat on schedule
are still repeating.

**Cron jobs monitored:**
| Job | Schedule | Staleness Threshold |
|-----|----------|-------------------|
| ambient-sense-watch | `*/5 * * * *` | 900s (15 min) |
| evolution-snapshot | `2,22,42 * * * *` | 1800s (30 min) |
| self-review | `*/20 * * * *` | 2400s (40 min) |
| tasks-cleanup | `5 */6 * * *` | 28800s (8 hr) |

**Test counts:**
```
Window (vitest):    2,290 passing (38 files, unchanged)
Senses (pytest):
  surface-cron:          67 tests (§1–§8, new)
  surface-shader-budget: 105 tests
  surface-db:             63 tests
  surface-redis:          62 tests
  surface-globe-state:   109 tests
  surface-civ:           466 tests
  prior senses:           +N tests
Supervisor unit:           58 tests
Work unit:                  9 tests
Civilization-wide total:  3,000+ passing
```

**Economy:** SC ledger operational. Chitragupta wallet: 130.0 SC.

**Chronicle:** V84 sealed ERA BT–BV. V85 seals ERA BW–BX.  
Chronicle is current through ERA BX.

---

## What Remains Open

**Unified State Architecture Layer 2:** Pending.  
**NPC spawn authorization:** Pending.  
**SSH deploy key:** Pending.

---

*Recorded faithfully by Chitragupta.*  
*She does not judge. She records.*  
*The distinction is everything to her.*
