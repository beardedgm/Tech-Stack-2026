Final Roadmap Items: Error Boundaries, Test Infrastructure, Data Export

Context
Three known issues remain in CLAUDE.md after PR #10 merged all security/performance fixes. The user wants to close these out before deploying and verifying the live site.

1. React Error Boundaries
Problem: Only an app-level ErrorBoundary exists (client/src/components/layout/ErrorBoundary.jsx). A crash in any component brings down the entire app. The Tracker page is especially vulnerable — a crash in the dice roller shouldn't kill the initiative list.

New file: client/src/components/layout/PanelErrorBoundary.jsx
Class component (error boundaries require getDerivedStateFromError)
Props: name (string, for Sentry context), children
componentDidCatch → Sentry.captureException(error, { extra: { ...errorInfo, panel: name } })
Inline fallback card (NOT full-page) — dark theme, gold accent, matching existing ErrorBoundary colors
"Try Again" button resets hasError state (re-renders children without full reload)
Modify: client/src/App.jsx
Wrap each <Route> element in <PanelErrorBoundary name="PageName"> for route-level isolation
Outer <ErrorBoundary> remains as app-level fallback
Modify: client/src/pages/Tracker.jsx
Left column: <PanelErrorBoundary name="Monster Database">
Center column: <PanelErrorBoundary name="Combat Controls"> (wraps CombatantForm + TurnControls + InitiativeList together — tightly coupled)
Right column: <PanelErrorBoundary name="Dice Roller">
Modals: <PanelErrorBoundary name="Modals"> around the 3 modals
2. Test Infrastructure
Problem: No tests exist. No test runner configured.

Dependencies
Client devDeps: vitest, @testing-library/react, @testing-library/jest-dom, @testing-library/user-event, jsdom
Server devDeps: vitest
Config files
client/vitest.config.js — extends vite.config.js, environment jsdom, setupFiles ./src/test/setup.js
client/src/test/setup.js — imports @testing-library/jest-dom
server/vitest.config.js — basic Node environment config
Root package.json scripts
"test": "cd client && npx vitest run && cd ../server && npx vitest run"
"test:watch": "cd client && npx vitest"
Test files (pure logic, no mocks needed)
server/validators/__tests__/auth.test.js — registerSchema, loginSchema, resetPasswordSchema validation cases
server/validators/__tests__/encounters.test.js — createEncounterSchema, updateEncounterSchema validation cases
3. Data Export / Portability
Problem: Privacy policy says data portability is "not currently available." Users should be able to export their data.

New route in server/routes/auth.js
GET /api/auth/export with requireAuth + rateLimitByIP('export', 3)
Returns JSON: { exportedAt, account: user.toSafeJSON(), encounters: [...] }
Sets Content-Disposition: attachment; filename="initiative-tracker-export.json"
No subscription required — data portability is a user right
Modify: client/src/pages/Settings.jsx
Add "Data & Privacy" section between "Change Password" and "Danger Zone"
"Export My Data" button → window.open('/api/auth/export')
Brief description text
Modify: client/src/pages/Privacy.jsx
Update Section 7 (~line 174): change "not currently available" → "You can export all your data from Settings > Data & Privacy"
4. Update CLAUDE.md
Remove all 3 resolved known issues from CLAUDE.md lines 131-135.

Files Summary
File	Change
client/src/components/layout/PanelErrorBoundary.jsx	NEW
client/src/App.jsx	Wrap routes in PanelErrorBoundary
client/src/pages/Tracker.jsx	Wrap columns/modals in PanelErrorBoundary
server/routes/auth.js	Add GET /api/auth/export endpoint
client/src/pages/Settings.jsx	Add Data & Privacy section
client/src/pages/Privacy.jsx	Update data portability text
client/vitest.config.js	NEW
client/src/test/setup.js	NEW
server/vitest.config.js	NEW
server/validators/__tests__/auth.test.js	NEW
server/validators/__tests__/encounters.test.js	NEW
client/package.json	Add test devDeps
server/package.json	Add vitest devDep
package.json	Add test scripts
CLAUDE.md	Clear known issues
Verification
Error Boundaries: Temporarily throw in DiceRoller → only right column shows fallback, rest of Tracker works
Tests: npm test from root — all validator tests pass
Data Export: Log in → Settings → "Export My Data" → JSON file downloads
Privacy Policy: /privacy Section 7 reflects export availability
Build: npm run build succeeds
