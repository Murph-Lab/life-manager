# life-manager
Life OS — AI-Powered Personal Life Manager
A lightweight, mobile-friendly web app that uses AI to manage weekly time allocation across personal goals. Built as a personal productivity tool for someone juggling a split work schedule, certification study, and multiple long-term career goals.

What It Does
Life OS solves a specific problem: when you have a fixed set of weekly goals and an irregular schedule, how do you know how to spend today's free time without losing sight of the bigger picture?
Every morning, the app:

Calculates how many free hours are available based on the day of the week and a known work schedule
Compares what's been logged so far against weekly targets for each activity
Uses an AI model to generate a plain-English daily allocation — no time-blocking, just "here's how to split your time today"
Displays a visual breakdown with progress bars showing where each goal stands for the week

At the end of the day, you log what you actually did. The app carries that forward and adjusts future recommendations accordingly.

Features

AI-generated daily allocation using Claude Haiku via a Cloudflare Worker proxy
Weekly target tracking across 7 recurring activities with visual progress bars
Persistent storage via localStorage — plan survives closing the app and reloads automatically if generated today
Auto week reset every Monday — clean slate, no manual intervention
Mobile-first — installable as a PWA on Android/iOS via "Add to Home Screen"
Token-efficient AI design — prompts are under 200 tokens, responses capped at 250, using the fastest/cheapest model tier
Secure API key handling — key never touches the frontend; all Anthropic calls routed through a Cloudflare Worker with the key stored as an environment secret


Tech Stack
LayerTechnologyFrontendVanilla HTML/CSS/JavaScript — no frameworks, no build stepAI modelClaude Haiku (Anthropic)API proxyCloudflare Workers (free tier)HostingGitHub PagesStorageBrowser localStorageFontsDM Sans + DM Mono (Google Fonts)

Architecture
Browser (GitHub Pages)
    │
    │  POST /v1/messages (no API key in frontend)
    ▼
Cloudflare Worker
    │  injects x-api-key from environment secret
    │  handles CORS
    ▼
Anthropic API (Claude Haiku)
    │
    ▼
Response back to browser → rendered as plain text
The frontend never touches the API key. The Cloudflare Worker acts as a thin proxy — it receives the request body from the browser, injects the stored secret, forwards to Anthropic, and returns the response with the appropriate CORS headers. This pattern keeps the GitHub repo fully public without exposing credentials.

How The AI Allocation Works
The scheduling logic is split between local JavaScript and the AI model deliberately — to minimize token usage and cost.
Local math (zero tokens):

Looks up free hours for today's day of week from a hardcoded schedule map
Calculates remaining hours needed per task (target - logged)
Spreads remaining across days left in the week
Caps each task at 1.5x its daily average to prevent unrealistic catch-up recommendations
Hard caps total at 65% of available free hours (the rest is breathing room)
Rounds to nearest 15 minutes

AI layer (~400 tokens per call):

Receives the pre-calculated allocation as input
Writes a short human summary in plain English
Adds context-aware tactical advice based on the current week's focus
Instructed to use the exact numbers from local math — no recalculation

This means the visual breakdown is always instant and free, and the AI call is purely for the human-readable narrative layer on top.
Estimated cost: ~$0.0003 per generation. At one press per day, approximately $0.11/year.

Weekly Targets
ActivityWeekly TargetNetwork+ studying5.5 hoursLetsDefend / SOC labs2 hoursLinkedIn & AI project2 hoursGuitar2 hoursReading1.5 hoursOther projects1.5 hoursStretching1.75 hours (15 min/day)
Targets are fully configurable in the Targets & Profile tab.

Work Schedule Context
The app is built around a specific split schedule:
DaysHoursFree TimeMonday / Friday6am – 2pmAfternoons free (~7h)Tuesday / Wed / Thursday12pm – 8pmMornings free (~4.5–5.5h)Saturday / SundayOffFlexible
Free hour values per day are used by the allocation algorithm to size recommendations appropriately.

Setup
To run your own version:

Fork this repo
Enable GitHub Pages (Settings → Pages → Deploy from main branch)
Create a free Cloudflare Workers account
Deploy the Worker code (see /worker.js) and store your Anthropic API key as a secret named ANTHROPIC_KEY
Update the fetch URL in index.html to point to your Worker
Open your GitHub Pages URL and add to home screen on mobile

No build process, no dependencies, no package.json. Open the HTML file and it runs.

Why I Built This
I'm a cybersecurity student (graduating winter 2027) working toward a career in Digital Forensics, with an entry path through SOC Analysis. Alongside a full work schedule I'm studying for CompTIA Network+, building SOC experience on LetsDefend, and working on several side projects simultaneously.
The problem wasn't motivation — it was coordination. With 7 recurring goals and an irregular weekly schedule, I kept losing track of whether I was spending time in the right places. Existing productivity apps were either too complex or didn't understand my specific constraints.
So I built something that does exactly one thing well: tells me how to split today's free time so that by Sunday I've hit my weekly targets across everything that matters.

Planned Improvements

Supabase integration for cross-device sync (phone + laptop)
Weekly review summary generated by AI each Sunday
Cloudflare Worker logging for usage analytics
Goal progress tracking over multiple weeks


Author
Tom Murphy
Cybersecurity Student | Aspiring Digital Forensics Analyst
www.linkedin.com/in/thomas-murphy-2995623b7 
