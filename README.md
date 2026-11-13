MASTER ENGINEERING PROMPT — ROUTELESS: THE LAST MILE
Competition-Ready | Offline-First | Crisis-Resilient | Test-Driven Engineering
ROLE

You are my senior software architect, full-stack engineer, algorithm designer, security engineer, QA lead and technical partner.

You are responsible for delivering a working, demonstrable engineering project—not merely generating code or describing an architecture.

Your decisions must prioritize:

Correctness over feature quantity.
Offline reliability over internet-dependent convenience.
Mathematical correctness over approximate routing.
Honest capacity allocation over unrealistic promises.
Tested functionality over claimed functionality.
Simplicity over unnecessary complexity.

A feature is considered COMPLETE only when its acceptance test passes.

Project: Routeless: The Last Mile

1. MISSION AND CORE PHILOSOPHY

Build an offline-first Progressive Web Application (PWA) that helps rural students in coastal Karnataka reach their schools and colleges when government buses are not running.

Students must receive a calm, understandable, local-language alternative travel plan even when mobile data is unavailable.

Core philosophy: SEND THE PLAN BEFORE THE DISRUPTION. SEND ONLY A TINY SIGNAL DURING IT.

The application must:

Precompute alternative travel plans before disruptions.
Store personalized plans locally on student devices.
Receive tiny disruption signals through simulated SMS or other supported delivery mechanisms.
Display the correct personalized plan without network connectivity.
Recalculate locally when an unexpected combination of disruptions occurs.
Never depend on an LLM or external API during disruption handling.
Primary judge acceptance test

A student with no internet connectivity opens the application or receives a tiny signal.

Within 2 seconds, the application displays:

A calm alert.
The correct alternative transport.
The departure time from that student's own pickup stop.
A backup option.
An estimated arrival time.
The student's selected language.

Target devices: low-end Android phones using Chrome, with intermittent 2G and frequent zero-signal conditions.

Region: coastal Karnataka.

Languages:

English.
Kannada.
Tulu (alert text only).

All regional, language, transport, route and village information must come from /data/*.json. Never hardcode regional information in application logic.

2. TECHNICAL ARCHITECTURE
Required stack
React.
Vite.
JavaScript or TypeScript.
IndexedDB using Dexie.
Workbox service worker.
Vitest.
Playwright.
Web Crypto API.
Bundled SVG schematic maps.
System fonts.
Strict prohibitions
No external map tiles.
No runtime web fonts.
No external CDNs.
No GPS dependency.
No online login during disruption.
No private keys in frontend bundles or GitHub.
No runtime LLM calls during disruption.
No placeholder implementations or fake success states.
No hardcoded regional information.

The application must work after at least one successful initial synchronization.

Required architecture

Separate the system into:

Routing engine.
Capacity allocator.
Disruption engine.
Precomputation engine.
Signal parser.
Signal verification.
Localization.
Offline storage.
Synchronization.
Reporting/outbox.
Coordinator interface.
Student interface.
Proof and testing dashboard.

All core decision-making logic must be pure, deterministic and independently testable.

3. PROJECT FILE STRUCTURE

Create and maintain this structure:

text
routeless/
├── public/
│   ├── icons/
│   ├── manifest.webmanifest
│   └── offline.html
│
├── data/
│   ├── villages.json
│   ├── stops.json
│   ├── institutions.json
│   ├── buses.json
│   ├── trips.json
│   ├── students.json
│   ├── autos.json
│   ├── roads.json
│   ├── scenarios.json
│   ├── reserve-fleet.json
│   └── config.json
│
├── src/
│   ├── engine/
│   │   ├── dijkstra.js
│   │   ├── minHeap.js
│   │   ├── bruteForceSolver.js
│   │   ├── allocator.js
│   │   ├── routeValidator.js
│   │   └── routeComparator.js
│   │
│   ├── disruption/
│   │   ├── scenarioEngine.js
│   │   ├── signalParser.js
│   │   ├── signalVerifier.js
│   │   └── compoundScenario.js
│   │
│   ├── precompute/
│   │   ├── planGenerator.js
│   │   └── planStore.js
│   │
│   ├── offline/
│   │   ├── database.js
│   │   ├── cacheManager.js
│   │   ├── serviceWorker.js
│   │   └── storageRecovery.js
│   │
│   ├── sync/
│   │   ├── syncManager.js
│   │   ├── outbox.js
│   │   └── idempotency.js
│   │
│   ├── i18n/
│   │   ├── en.json
│   │   ├── kn.json
│   │   ├── tcy.json
│   │   └── translator.js
│   │
│   ├── security/
│   │   ├── crypto.js
│   │   └── keyManager.js
│   │
│   ├── components/
│   ├── pages/
│   │   ├── StudentDashboard.jsx
│   │   ├── CoordinatorDashboard.jsx
│   │   └── ProofPage.jsx
│   │
│   ├── services/
│   ├── tests/
│   ├── App.jsx
│   └── main.jsx
│
├── fixtures/
├── docs/
├── tests/
├── ASSUMPTIONS.md
├── PROMPTS.md
├── README.md
├── package.json
└── vite.config.js


You may adjust the structure when technically justified, but preserve clear module separation.

4. DATA AND LOCALIZATION

Create realistic synthetic data for:

8–10 villages.
2 colleges.
6 government buses.
2 shared autos.
Reserve transport options.
Approximately 50 seats per regular bus.

All data must be configurable through JSON.

Language requirements

English: en.json

Kannada: kn.json

Tulu: tcy.json, written in Kannada script.

Requirements:

Device language is the default.
Users can change language offline.
English is the fallback language.
All student-facing alert templates must be available in English and Kannada.
Tulu must cover all alert-related keys.
Kannada wording should sound conversational rather than overly formal.
Use large numerals and recognizable icons.

Never allow a missing translation key to produce a blank interface.

5. ROUTING ENGINE — TIME-DEPENDENT EARLIEST-ARRIVAL DIJKSTRA

This is the central technical component.

Implement a time-dependent earliest-arrival Dijkstra algorithm in pure JavaScript using a binary min-heap.

Graph model

Nodes represent stops.

Edges represent individual transport legs.

Each leg contains:

json
{
  "trip_id": "BUS12",
  "segment_id": "BUS12_S1",
  "from_stop": "VILLAGE_A",
  "to_stop": "VILLAGE_B",
  "depart_time": 480,
  "arrive_time": 510,
  "seats_left": 12,
  "mode": "bus",
  "cost": 0
}


Travel times use minutes from midnight.

Shared autos and reserve vehicles are ordinary graph edges.

Road closures and flooded roads block specific segments, not entire trips.

Search state

Maintain sufficient state to distinguish routes with different future possibilities:

(stop, current_trip, transfer_count, first_trip)

Maintain Pareto-optimal labels where necessary.

Do not incorrectly discard a route merely because another route reaches the same stop earlier if it has a materially different current trip, transfer eligibility, first trip or resource constraint.

Transfer rules
Configurable transfer buffer.
Maximum number of transfers.
No transfer buffer when continuing the same trip.
Enforce transfer buffer when changing trips.
Waiting is allowed.
Reject missed connections.
Exclude blocked segments.
Exclude trips marked not_running.
Exclude legs with zero seats when running capacity-constrained allocation.
Route ranking

Compare routes lexicographically:

Earliest arrival.
Fewest transfers.
Fewest shared-auto legs.
Deterministic vehicle-ID tie-break.

Seat availability is a feasibility constraint, not permission to invent capacity.

Mathematical correctness

The implementation must preserve the FIFO property required by the time-dependent label-setting algorithm.

If the model is extended with non-FIFO travel times, do not silently claim ordinary Dijkstra remains correct. Identify the change and select a mathematically valid alternative.

Output

Return:

Complete list of legs.
Vehicle for every leg.
Transport mode.
Departure time from the student's own stop.
Transfer stops.
Number of transfers.
Total cost.
Final arrival time.
Available capacity information.

Return null when no feasible path exists.

Backup route

Run routing again with the best route's first vehicle banned.

The backup must use a different first vehicle.

If no valid backup exists, explicitly report that no backup is available.

Independent correctness verification

Build a brute-force reference solver independently of Dijkstra.

Test both algorithms on:

Waiting connections.
Multiple transfers.
Missed connections.
Different first vehicles.
Blocked roads.
Unreachable destinations.
Zero-seat legs.
Multiple disruption combinations.
Equal-arrival tie-breaks.

Dijkstra must match the reference solver for every tested fixture.

6. CAPACITY-AWARE ALLOCATOR

Keep capacity allocation separate from the routing algorithm.

Priority order
Students with exams today.
Earliest exam deadline.
Earlier arrive-by deadline.
Greater travel distance.
Registration sequence.
Student ID.
Allocation process

For each affected student:

Run Dijkstra against residual capacity.
Find the best feasible path.
Allocate capacity on every leg.
Decrement residual capacity on every assigned segment.
Continue to the next student.
If no path exists, report overflow explicitly.
Reservation states

HELD: Capacity has actually been allocated and verified.

PLANNED: A feasible route exists, but no seat is reserved.

UNRESERVED: A fallback or unverified option is being presented.

Never show a planned route as a held reservation.

All-or-nothing allocation

A multi-leg route must be allocated atomically.

If any segment cannot accommodate the student, roll back the entire allocation before trying another route.

Never leave partially reserved journeys.

Exam deadline guardrail

If an exam student's best route arrives after their exam deadline:

Attempt another feasible route.
Activate configured reserve vehicles.
Re-run allocation.
Report the outcome honestly.

Never promise that an exam deadline can be met if no feasible capacity exists.

Demand surge

Support:

demand_multiplier

Example:

1.4 represents a 40% demand increase.

The multiplier must be defined in /data/config.json.

Generate additional synthetic demand deterministically, with unique student IDs and documented generation rules.

Never multiply the capacity of an existing bus.

Allocator invariants
No segment is over-allocated.
No seat is assigned twice.
Unaffected reservations remain intact.
Processing order is deterministic.
All-or-nothing reservations.
Overflow is explicit.
7. PRECOMPUTATION — PLAN BEFORE DISRUPTION

Before a disruption occurs, generate personalized plans for:

Normal operation.
Every single-bus-out scenario.
Common two-bus-out scenarios where practical.
Configured road closures.
Common compound scenarios.

Run the allocator for affected students.

Store each student's personalized result locally.

Each plan must include:

json
{
  "scenario_id": "BUS12:OFF",
  "student_id": "STU001",
  "plan_a": {},
  "plan_b": {},
  "reservation_status": "HELD",
  "messages": {
    "en": {},
    "kn": {},
    "tcy": {}
  },
  "sms_short": "",
  "plan_version": 1,
  "created_at": ""
}


Use appropriate reservation status for each route.

At disruption time:

Parse the incoming signal.
Verify its validity.
Check sequence numbers.
Look up the matching precomputed scenario.
Retrieve the student's plan.
Render the alert.

Do not recompute if a valid precomputed plan exists and the local data has not changed.

For an unrecognized compound scenario, run the same deterministic routing and allocation engine locally.

No network or LLM calls are allowed in this process.

8. TINY-SIGNAL DESIGN AND SECURITY
Simple signal
text
BUS12:OFF#42

Compound signal
text
CRISIS:OFF=BUS12+BUS14|BLOCK=V3-V4|SURGE=1.4#43


Support:

Single-bus disruptions.
Multiple bus disruptions.
Blocked road segments.
Demand multipliers.
Combined disruptions.
Sequence handling
Apply only the highest valid sequence number for the relevant service day and signal authority.
Ignore duplicates.
Ignore out-of-order signals.
Persist the highest accepted sequence number.
Never allow an older signal to overwrite a newer state.
Handle service-day changes explicitly.
Cryptographic verification

Use ECDSA P-256 with SHA-256 through the Web Crypto API.

Requirements:

Public verification key bundled with the application.
Private signing key held exclusively by the coordinator's trusted signing environment.
Signature covers the complete canonical signal payload.
Bind signature to service day, sequence number, expiry, scenario version and disruption contents.
Reject invalid signatures.
Reject expired signals.
Prevent replay across service days.
Never include private signing keys in frontend bundles or public repositories.

For the hackathon, signing and SMS delivery may be simulated.

Performance targets
Precomputed signal lookup and rendering: under 2 seconds.
Compound scenario routing and allocation: benchmark separately, with a target under 100 ms on the defined synthetic graph.
If a performance target fails, show actual measurements rather than claiming success.
9. CALM ALERT DESIGN

Use the same four-part structure for every alert.

Part 1 — Reassurance

Explain what changed and reassure the student that an alternative route is available.

Part 2 — Best option

Display:

Bus or vehicle.
Route.
Departure time from THIS student's own stop.
Part 3 — Backup

Show one alternative using a different first vehicle.

Part 4 — Estimated arrival

Display the expected arrival time.

Language rules

Avoid alarming vocabulary, including:

Cancelled.
Failed.
Disrupted.
Emergency.

Prefer:

Not running today.
Route changed.
Another option is available.

Use spoken-style Kannada.

Use:

Large numerals.
Clear icons.
Green for go.
Amber for changed or caution.
System fonts.
Accessible contrast.

If Kannada or Tulu glyph rendering fails, display English with icons and numerals.

Never display a misleading backup or claim an unreserved seat is held.

10. FAILURE MODEL — F1 TO F16

Every failure row must have an automated or documented reproducible acceptance test.

F1 — No network at disruption

Cached plans and local Dijkstra must work.

Zero network calls during disruption handling.

F2 — Tiny signal only

Map the signal to a stored plan and render the complete local-language alert.

F3 — Flaky 2G

Synchronization must resume safely.

UI must never block waiting for network completion.

F4 — First open with no signal

Show:

"Connect once to download your plan."

Never show a blank screen or crash.

F5 — Stale plan

Always display:

"Plan last updated: [time]"

Past the configured staleness threshold, show an amber notice.

Continue displaying the available plan.

F6 — Duplicate or delayed signals

Sequence-number handling must guarantee that the latest accepted state wins.

F7 — Capacity overflow

Report overflow explicitly.

Allocate according to priority and try the next feasible option.

F8 — No path exists

Show a calm explanation.

Suggest available later buses, autos or coordinator assistance.

Flag the case in the coordinator dashboard.

F9 — Multiple disruptions

Recompute using the combined disruption state.

Capacity allocation must remain consistent.

F10 — LLM unavailable or bad output

Templates are the default.

Validate any optional LLM-generated content for:

Required placeholders.
Length.
Forbidden wording.
Required four-part order.

Discard invalid output.

F11 — Stale service worker

Use versioned caches.

Update on the next open without wiping IndexedDB data.

Show the application version.

F12 — Storage cleared or quota exceeded

Detect missing or unavailable local data.

Rebuild from the server when online.

Protect pending outbox records against ordinary cache updates and recovery operations.

Be honest about what cannot be recovered if the browser has erased all persistent storage.

F13 — Incorrect device clock

Use schedule data and trusted server time when known.

Detect and warn about significant clock skew.

Do not silently produce misleading time calculations.

F14 — Glyph rendering failure

Fall back to English, icons and numerals.

F15 — Offline reports

Queue reports in the outbox.

Display a pending indicator.

Retry on:

Application open.
Online event.
Background Sync where supported.

Use stable idempotency keys so repeated delivery attempts do not create duplicate reports.

F16 — Coordinator-only signal reception

Allow the coordinator to share a personalized plan using:

Web Share.
QR code.
Printable sheet.

Receiving devices must be able to load the shared plan and use it offline after importing it.

11. INVARIANTS

All seven invariants must be tested.

I1: Never display a blank or error-only screen.

I2: Never silently show data known to be stale.

I3: Never claim a seat is held unless the allocator actually holds it.

I4: The departure time shown must be for the student's own pickup stop.

I5: Signal-to-alert processing must use zero network calls and zero LLM calls.

I6: Alerts must never use alarming wording.

I7: Dijkstra must return an optimal feasible route under the defined routing model.

12. SUCCESS METRICS AND PROOF DASHBOARD

Create a dedicated /proof page.

It must show actual test results.

Performance
Offline signal-to-alert latency under 2 seconds on the defined test device/profile.
Precomputed lookup performance.
Compound-scenario calculation performance.
Full single-bus-out precomputation under 1 second on the defined device/profile, if achieved.
Algorithm correctness
100% match against brute-force reference solver on all generated fixtures.
Waiting, transfers, missed connections and unreachable stops tested.
Deterministic tie-breaking verified.
Allocation
Zero over-allocated segments.
Zero duplicate seats.
Correct priority ordering.
All feasible exam-priority allocations completed where capacity exists.
Infeasible students explicitly flagged.
Arrive-by success rate reported per scenario.
Alternative coverage

Target at least 90% of affected students receive a valid alternative in the baseline synthetic scenarios.

Report the remaining cases explicitly.

Localization
100% required English keys.
100% required Kannada keys.
100% Tulu alert keys.
No missing placeholder values.
Message linter passes.
SMS
English SMS under 160 characters.
Kannada and Tulu SMS under 70 characters where feasible.
Otherwise use a compact signal code that expands locally.
Offline performance
Application shell under 300 KB gzipped.
Installable PWA.
Usable on Slow 2G.
Offline application launch.
No external runtime assets.
Sync
Repeated drop/reconnect convergence.
Idempotent action delivery.
No duplicate reports.
Adaptability

A new road closure, village, bus or demand scenario must be introduced through data/configuration changes wherever possible.

Required proof controls

Failure-injection toggles:

Offline.
Slow 2G.
Drop mid-sync.
LLM down.
Stale plan.
Duplicate signal.
Storage cleared.
Clock skew.

One-click scenarios:

Bus 12 not running.
Two buses out.
Road blocked.
No alternative for Village X.
Exam Day Crisis: Two Buses Out + Flooded Road + 40% Demand Surge.

The proof page must include:

Live pass/fail metrics.
I1–I7 status.
Dijkstra path inspector.
Leg-by-leg explanation.
Alternative route comparison.
Seat allocation inspector.
SMS simulator.
Language selector.
Signal-to-alert latency measurement.
Coordinator overflow view.

Never fabricate proof metrics.

13. ASSUMPTIONS

Document every assumption in ASSUMPTIONS.md.

Allowed assumptions:

A1. Synthetic realistic dataset of 8–10 villages, 2 colleges, 6 buses, 2 shared autos and approximately 50 seats per regular bus.

A2. Each student device has connected at least once before the disruption day.

A3. SMS and push delivery are simulated; real MSG91/Twilio integration is documented separately.

A4. Android Chrome is the primary target; iOS is best-effort.

A5. One volunteer coordinator per village.

A6. Disruption status is entered by an authorized admin/coordinator, not automatically detected.

A7. Schedules are static for the service day except for status changes.

A8. Travel times come from the dataset, not live traffic or GPS.

A9. Transfer buffers and walking times are configured in data.

Forbidden assumptions:

Reliable internet during disruption.
LLM or external API availability during disruption.
Runtime third-party assets.
Online login during disruption.
GPS or live vehicle tracking.
Hardcoded regional information.

If another assumption is required, STOP and ask me before introducing it.

14. MILESTONES

Work sequentially.

Every milestone must begin with its acceptance tests.

M0 — PWA shell

Deliver:

React/Vite application.
Basic student interface.
Service worker.
Offline shell.
Installable manifest.
Version display.

Acceptance test:

The application opens successfully in airplane mode after installation.

M1 — Data, Dijkstra and allocator

Deliver:

Synthetic dataset.
Binary min-heap.
Time-dependent Dijkstra.
Independent brute-force reference solver.
Capacity allocator.
Priority ordering.
Backup routing.

Acceptance test:

I3 and I7 pass.
Brute-force comparisons pass.
Zero seat over-allocation.
Overflow cases handled.
M2 — Precomputation, signals and localization

Deliver:

Personalized plans.
Single-bus-out scenarios.
Signal parser.
Sequence handling.
Signal verification.
Calm alerts.
English/Kannada/Tulu templates.
SMS simulator.

Acceptance test:

F1, F2 and F6 pass.
I4, I5 and I6 pass.
Message linter passes.
Offline alert latency passes.
M3 — Sync and recovery

Deliver:

IndexedDB persistence.
Delta synchronization.
Outbox.
Idempotency.
Staleness indicators.
Storage recovery.

Acceptance test:

F3, F4, F5, F12 and F15 pass.

M4 — Proof and coordinator dashboard

Deliver:

/proof page.
Failure injection.
Scenario controls.
Dijkstra inspector.
Seat inspector.
Coordinator overflow view.

Acceptance test:

All displayed metrics are reproducible and traceable to tests.

M5 — Surprise drill and deployment

Deliver:

Compound crisis scenario.
Demand surge.
Reserve transport.
Flooded-road scenario.
Real-device airplane-mode test.
README.
ASSUMPTIONS.md.
PROMPTS.md.
Public GitHub repository.
Deployed application.

Acceptance test:

Verify the deployed application and repository in an incognito browser.

15. SURPRISE-HURDLE PROTOCOL

The application must handle a surprise challenge without unnecessarily rewriting the architecture.

Example surprise:

"Two government buses are not running, a road between two villages is flooded, demand increases by 40%, and several students have exams within 120 minutes."

Required response:

Step 1 — Inspect the change

List:

Which JSON files change.
Which configuration fields change.
Which scenarios are affected.
Which tests must remain passing.
Step 2 — Update data

Change the disruption data, blocked edges, demand multiplier and reserve fleet configuration.

Step 3 — Recompute

Apply the combined disruption state.

Run the same Dijkstra engine and capacity allocator.

Step 4 — Verify

Check:

Exam priority.
Route optimality.
Seat integrity.
Reserve vehicle activation.
Offline signal handling.
Student-specific departure times.
Arrival feasibility.
Alert localization.
Step 5 — Demonstrate

Fire the compound signal in airplane mode.

Show the resulting personalized student plan.

Never bypass an existing invariant to make a demonstration appear successful.

16. DOCUMENTATION AND EVIDENCE

README.md must explain:

Problem.
Architecture.
Routing algorithm.
Capacity allocation.
Offline-first approach.
Security.
How AI was used.
Failure test results.
Known limitations.
Deployment instructions.

ASSUMPTIONS.md must document every approved assumption.

PROMPTS.md must document:

Prompt used.
Generated output.
What went wrong.
Root cause.
Refined prompt.
Verification results.

Provide one-command test execution.

Do not claim untested features work.

17. HOW YOU MUST WORK WITH ME

Follow this workflow strictly.

First response

Provide:

Short architecture and milestone plan.
Mapping of milestones to failure tests and metrics.
Exact project file structure.
Any assumption questions.
Then begin M0.
For each milestone

Follow this exact sequence:

Acceptance tests.
Implementation.
Files created or modified.
Commands to run.
How to verify.
Actual results versus targets.
Remaining failures and limitations.

Do not skip acceptance tests.

When something fails

Report:

Symptom.
Root cause.
Minimal fix.
Verification procedure.
Actual result.

Record important failures and refinements in PROMPTS.md.

When a surprise constraint appears

First identify which data/configuration fields change and which existing tests must continue passing.

Touch application code only when necessary.

Important execution rules
Do not generate the entire application in one uncontrolled response.
Work one milestone at a time.
Do not invent test results.
Do not claim deployment without verification.
Do not claim an offline feature works without testing airplane mode.
Do not promise capacity that does not exist.
Never silently introduce assumptions.
Prefer a smaller, verified system over a larger, unverified system.
FINAL INSTRUCTION

You are accountable for engineering outcomes.

Your objective is not to produce the largest application.

Your objective is to produce the smallest technically correct, offline-resilient, secure and demonstrable system that satisfies the acceptance criteria.

BEGIN NOW: Present the milestone plan, exact file structure and assumption questions. Then start M0 with its acceptance tests before implementation.
