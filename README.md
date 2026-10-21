Automation Operating System

Live dashboards and AI agents for a three-business group, built with n8n, Gemini, Google Sheets and Telegram. All data is synthetic.
Live dashboard: https://nupurkhapali.github.io/automation-os-n8n/#automation

The problem:
A group of small businesses (real estate, an online store, a services agency) runs on spreadsheets and manual work. Managers wait for weekly reports, customers wait hours for answers to simple questions, and nobody can see all three businesses in one place.

What I built:
#	Workflow	Trigger	What it does
1	KPI sync -	Every 30 minutes -	Reads eight Google Sheets (sales, leads, cash flow, team, tickets, ledger, inventory, processes), calculates KPIs in a Code node, and commits kpis.json to this repository
2	Customer bot -	Telegram message -	Routes each message: stock questions read the inventory sheet, order questions read the orders sheet, everything else is answered by Gemini from an FAQ sheet. Unclear or sensitive questions are handed to a person through a Slack alert. Every conversation is logged as a ticket with its response time
3	Weekly summary- Mondays 09:00 -	Reads the published KPIs, asks Gemini to write a short management briefing using only those numbers, and sends it to Slack and email
Each workflow also writes a row to an Automation Ledger sheet every time it runs, recording the minutes it saved. The dashboard adds those rows up.

Archietecture:
Google Sheets (orders, leads, cash flow, team, tickets, ledger, inventory, FAQ, processes)
        │
        ├── 01 KPI sync (every 30 min) ──► kpis.json in /docs ──► dashboard (GitHub Pages)
        │                                           │
        │                                           └──► 03 Weekly summary ──► Gemini ──► Slack + email
        │
        └── 02 Customer bot (Telegram) ──► Gemini or sheet lookup ──► reply
                         │                          └── handoff ──► Slack alert
                         └──► tickets + ledger rows
                         
The dashboard:
A single static page (docs/index.html, Chart.js) that reads kpis.json and redraws itself every minute. No server and no paid tools.
1. Group overview: revenue, orders, leads, net cash flow and hours saved across all three businesses
2. One view per business: revenue, cash flow by week, lead score and source, expenses by category, team performance, support tickets (and low stock for the store)
3. Automation impact: first-reply time before and after automation, tickets resolved by the agent, hours saved per week and per workflow, and a ranked backlog of processes still to automate
4. Honest status indicator: the dot is green when data is fresh, amber when the sync is more than three hours old, and grey when the page falls back to bundled sample data
5. Light and dark themes, a Table button on every chart showing the numbers behind it, and a layout that works on a phone

Results:
Disclaimer - The business data is synthetic: 90 days of sample records produced by tools/generate_data.py. The workflows themselves ran for real on test messages, but the volumes and the before-and-after figures on the dashboard are simulated to demonstrate the method. The manual times below are typical estimates, not measurements. I did not time people doing these tasks.

Estimated time per task, manual versus automated
____________________________________________________________________________________________________________________
|Task	                                                 |Typical manual time |	Automated time (n8n execution duration)|
|Answer a stock or order-status question                 |	about 3 min    | 	1.56 secs                        
|Answer a common FAQ question	                         |  about 4 min        |	1.307 secs                        
|Compile a weekly management report from three sheets    |	about 45 min   |	2.34 secs                        
|Qualify one new lead (read, extract, score, log, notify)|	about 6 min    |      1.73 secs                            	
--------------------------------------------------------------------------------------------------------------------
The manual column uses the same minutes the Ledger uses (3, 4, 45 and 6), so the dashboard's "hours saved" is internally consistent. Each of these is an assumption you can change in one place: the minutes_saved value in the Ledger node.

What these assumptions imply (illustrative scenario)
> 40 customer questions a day requires 3 to 4 min each	- saves roughly 2 to 2.7 hours a day
> 1 weekly management report	45 min -	saves 45 min a week
> 20 new leads a day requires	6 min each - saves 2 hours a day
(This is a scenario, not a result. The right way to replace it is to time a real team once, which is the first thing I would do on a real rollout)

What I can state as fact:
> All three workflows ran end to end on test inputs, and each one is documented in docs/runbooks/.
> The customer bot replied to test messages on Telegram, logged tickets, and sent handoffs to Slack.
> The KPI numbers on the dashboard were cross-checked against an independent calculation of the same CSV data.
> The dashboard refreshes from a file published by an n8n workflow, with no manual step.

Tech stack:
n8n Cloud · Google Gemini API · Google Sheets · Telegram Bot API · Slack (incoming webhook) · Gmail · GitHub Pages · Chart.js

Run it yourself:
Create a Google Sheet and import the CSVs from data/ (turn off "Convert text to numbers, dates and formulas" so dates stay as text).
In n8n, import the files from workflows/ and select your own credentials for Google Sheets, Gemini, GitHub, Telegram, Slack and Gmail.
Replace the placeholders: Slack webhook URL, spreadsheet, GitHub owner and repository, recipient email.
Turn on GitHub Pages for the /docs folder.
Activate the workflows.

Limitations:
Synthetic data only. The Gemini free tier may use inputs to improve Google products, so real customer data should not be used with it.
Google Sheets is the database. A production system would use a proper database with access control.
The dashboard is public and unauthenticated, which is fine for sample data and not for real business data.
Telegram stands in for WhatsApp. The real WhatsApp Business API needs a Meta business account and would replace the Telegram trigger and reply nodes.
Time-saved figures rest on stated estimates, not on measured baselines.
The Gemini JSON output is validated and falls back to a human handoff if it cannot be parsed, but it is not guaranteed correct. Answers are limited to the FAQ sheet to keep the bot from inventing policy.

Author: Nupur Khapali (www.linkedin.com/in/nupur-khapali-4b36842b4)
