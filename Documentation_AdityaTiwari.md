# Work summary – Aditya Tiwari

**Task 1 – QA of the RealWorld (Conduit) app.** The hosted demo (demo.realworld.io) was down, so I set up the stack locally: a FastAPI + PostgreSQL backend and the React-Redux frontend. I tested it through manual exploratory testing, scripted Playwright flows (sign-up, login, editor, delete, logout) and direct API calls with the browser DevTools open. I found **11 bugs** (2 Critical, 3 High, 3 Medium, 3 Low) and logged each with steps, expected vs actual, severity and suspected cause.

The two critical bugs make the app unusable straight from a fresh clone: the stylesheet comes from a dead host, and the API URL is hardcoded to a production server that no longer exists. The **root-cause analysis** covers the error message "0 incorrect email or password". The backend returns `errors` as a list, but the frontend's `ListErrors` component expects the spec's `{field: [messages]}` object and calls `Object.keys` on it, so it prints the array index. My suggested fix is to correct the backend shape, make the component defensive, and add a contract test so this kind of mismatch can't come back. I also checked for stored XSS, and the article body is sanitized correctly.

**Task 2 – n8n workflow.** Every hour (or on a webhook for manual testing) the workflow searches GitHub for AI-agent repos created in the last 7 days. It removes forks and archived repos, keeps the top 5 by stars and fetches the README of the #1 repo, which is a second, dependent API call. It then cleans that README into a short summary. A conditional branch chooses between a "breakout" message (when the top repo has more than 1,000 stars) and a "quiet week" one, and the digest is posted to Discord.

Reliability features:
- retries on every external call;
- an error output on the search call that logs the failure and sends an alert;
- a fallback that still sends the digest if the README fetch fails;
- handling for an empty search result;
- trimming of the message to Discord's 2,000-character limit.

I tested the Code-node logic in Node.js against live GitHub data. That run returned 5,275 matching repos; the #1 had 1,310 stars, so the breakout branch fired. I also tested the fallback, empty-result and error paths.

**Bonus – Uptime monitor.** Every 5 minutes it pings the target URL with retries and measures the response time. It sends a Discord alert when the status is not 200 or the host can't be reached at all. It also keeps running counters in workflow static data and posts a daily 09:00 summary with uptime %, average response time and slowest response time, then resets the counters.

**Design choices.** I used Code nodes only where the logic was real (filtering, text cleaning, stats) and built-in nodes (IF, HTTP Request, Discord) for everything else. I chose the GitHub API because it needs no key, has live data and lets two calls chain naturally. I chose Discord because a webhook needs no bot setup.
