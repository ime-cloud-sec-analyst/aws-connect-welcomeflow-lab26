# aws-connect-welcomeflow-lab26
Amazon Connect lab: inbound IVR “WelcomeFlow” with Sales / CustomerService / Tech queues, out-of-hours and all-agents-busy prompts, plus phone-number setup screenshots.

Amazon Connect — WelcomeFlow (Lab 26)
A complete, reproducible lab that builds a production-style inbound IVR in Amazon Connect with Sales, Customer Service, and Technical Support queues; friendly prompts for busy, out-of-hours, and invalid input; plus a clean shutdown/cost-control plan.
Table of contents
1.	Why this matters
2.	What you build
3.	Architecture & flow design
4.	Pre-requisites
5.	Step-by-step build
6.	Prompts (copy/paste)
7.	Phone number claiming & association
8.	Testing plan
9.	Operational guidance
10.	Strategies, tips, and must-dos
11.	Challenges & limitations
12.	Security, privacy, and compliance
13.	Cost control & cleanup
14.	Extensibility roadmap
15.	Troubleshooting
16.	Repo structure
17.	Screenshots
18.	License
Why this matters
•	Routes callers to the right expertise (Sales / Support / Tech) quickly.
•	Scales when agents are busy and gracefully handles peaks (“all agents busy”).
•	Communicates availability (“out of hours”) and retries intelligently.
•	Reduces Average Handle Time (AHT) and transfers, increasing CSAT.
•	Is auditable, secure, and cost-effective, and easy to iterate.
This lab gives you a complete working pattern that teams can adopt immediately, then extend (CRM data dips, chat, Lex bots, analytics) as needs grow.
What you build
•	A published WelcomeFlow inbound contact flow.
•	3 working queues: SalesQueue, CustomerService, TechSupport.
•	Main menu (DTMF 1/2/3) with retry on invalid and timeout/default handling.
•	Out-of-Hours announcement using Hours of Operation.
•	All-Agents-Busy branch when transfer capacity is hit.
•	Voice: Neural TTS (e.g., Amy – English UK) with conversational style.
•	A DID number associated to the contact flow for live testing.
•	A clean set of prompts and naming conventions used in the flow.
Architecture & flow design
Inbound Call
   |
   ├─ Set voice (Language: en-GB, Voice: Amy, Neural: Conversational)
   ├─ Play prompt: Welcome Greeting
   ├─ Get customer input (DTMF 1/2/3)
   │    ├─ Pressed 1 → Set working queue: SalesQueue        → Check hours → Transfer to queue
   │    ├─ Pressed 2 → Set working queue: CustomerService   → Check hours → Transfer to queue
   │    ├─ Pressed 3 → Set working queue: TechSupport       → Check hours → Transfer to queue
   │    ├─ Timeout/Default → Play Invalid Option → (loop back to Get customer input, max 1 retry)
   │    └─ Error → Play generic Error message → Disconnect
   |
   ├─ Check hours of operation
   │    ├─ In Hours  → continue to Transfer
   │    ├─ Out of Hours → Play Out_of_Hours option → Disconnect
   │    └─ Error → Play generic Error message → Disconnect
   |
   ├─ Transfer to queue
   │    ├─ At capacity → Play AllAgentsBusy → Disconnect
   │    └─ Error → Play generic Error message → Disconnect
   |
   └─ Disconnect (Termination)

Block naming used in the flow UI (consistently prefixed):
•	Voice_Set
•	Welcome Greeting
•	M01_MainMenu_GetInput (Get customer input)
•	Q01_SetQueue_Sales, CustomerServices_Queue, TechSupport_Queue
•	Out_of_Hours_option
•	E03_AllAgentsBusy
•	E01_InvalidOption
•	M01_Error_message
Pre-requisites
•	Amazon Connect instance created in your preferred region.
•	Permissions to manage Flows, Queues, Phone numbers, Hours of operation.
•	Ability to claim a DID in your region (telephony options depend on region).
•	Optional: Agents created & assigned to queues for live testing.
Step-by-step build
1) Create working queues
19.	Go to Routing → Queues.
20.	Create: SalesQueue (“Sales – L1 voice queue”), CustomerService (“CustomerServiceQueue”), TechSupport (“Technical Services and Support”).
21.	Enable each queue (Status: Enabled).
Tip: Tag queues (e.g., Dept=Sales) for reporting and automation later.
2) Hours of operation
22.	Go to Routing → Hours of operation.
23.	Configure business hours (e.g., 9:00–17:00 local, Mon–Fri).
24.	Name them clearly (e.g., Basic Hours).
3) Build the flow: WelcomeFlow
25.	Go to Routing → Flows → Create flow (Contact flow – inbound).
26.	Add blocks and configure: Set voice (Amy, en-GB, Neural Conversational), Play prompt (Welcome Greeting), Get customer input (DTMF 1/2/3), Set working queue blocks for Sales/CustomerService/TechSupport, Check hours, Transfer to queue, Out_of_Hours_option, E03_AllAgentsBusy, E01_InvalidOption, M01_Error_message, Disconnect.
27.	Wire everything so no outputs are left unconnected.
28.	Publish the flow.
Voice & SSML note: We used Interpret as: Text with Neural Conversational voice for consistent results. To use SSML, set Interpret as SSML and wrap prompts with <speak>...</speak>.
Prompts (copy/paste)
Welcome Greeting
Hello, welcome to VaultIQ Global Solutions. Please hold while we connect you to the next available agent.
Main Menu (M01_MainMenu_GetInput)
For Sales, press 1. For Customer Service, press 2. For Technical Support, press 3.
Invalid Option (E01_InvalidOption)
Sorry, that didn’t come through. Please try again.
All Agents Busy (E03_AllAgentsBusy)
Sorry, all our agents are busy at the moment. Please call back later.
Out of Hours (Out_of_Hours_option)
Sorry, we are closed now. Please call back during working hours.
Generic Error (M01_Error_message)
We’re sorry—an error occurred. Please try again later. Goodbye.
Phone number claiming & association
29.	Go to Channels → Phone numbers → Claim a number.
30.	Choose Voice, DID, your Country (e.g., UK), and pick a number.
31.	In number details, add a clear Description, and attach Contact flow / IVR = WelcomeFlow.
32.	Save. You should now see the number listed and associated with WelcomeFlow.
Testing plan
33.	Call the DID from a mobile/desk phone.
34.	Validate greeting, menu routing (1/2/3), timeout/default → invalid option, out-of-hours prompt, all-agents-busy scenario.
35.	Observe Agent CCP and Contact metrics to confirm queue routing.
Operational guidance
•	Use clear naming conventions (Qxx, Mxx, Exx).
•	Duplicate flows for changes and publish after testing.
•	Enable Contact Trace Records (CTR) to S3 & CloudWatch logs.
•	Centralize prompts in Prompts library for recorded audio later.
•	Maintain Holiday schedules and special branching.
Strategies, tips, and must-dos
•	Connect every branch: Success, Error, Default, Timeout, At capacity, Out of Hours.
•	Keep menus short—3–5 options max.
•	Retry once on invalid input; avoid loops.
•	Use neutral, empathetic language; consistent brand voice.
•	Document choices in this README for audit and handover.
•	Tag resources for reporting and IaC later.
•	Apply cost guardrails (see Cost control & cleanup).
Challenges & limitations
•	SSML behavior varies by voice—test prompts live.
•	At capacity depends on agent availability/concurrency; simulate properly.
•	Regional telephony options vary.
•	Unwired error branches block publishing.
Security, privacy, and compliance
•	Apply least-privilege IAM by role (admin/supervisor/agent).
•	If recording/streaming media, store to S3 with KMS and define retention.
•	Scrub or mask PII in prompts and logs; use PCI-scoped flows for payments.
•	Comply with local telecom regulations for number usage.
Cost control & cleanup
•	Numbers: monthly + per-minute usage; release numbers when done (Channels → Phone numbers → Release).
•	Limit test calls; disable call recording when not needed.
•	Disable or delete the Connect instance when finished (after exporting data you need).
•	Deactivate unused agents.
Extensibility roadmap
•	Amazon Lex bot for natural language self-service.
•	Lambda data dips (CRM lookup) for VIP routing/authentication.
•	Chat & email channels; Task flows for follow-ups.
•	Quick connects for warm transfers between skills.
•	Contact Lens for sentiment & transcription analytics.
•	S3 audio prompts with professional voiceover.
Troubleshooting
•	“Connections missing”: open each block and wire all outputs.
•	No audio/wrong voice: ensure Set voice runs before prompts and “Interpret as” is correct.
•	All agents busy doesn’t trigger: ensure no available agents or adjust concurrency.
•	Out of hours not firing: verify Hours of operation resource and timezone.
•	Publishing fails: open errors panel and fix per-block issues.
Repo structure
/
├─ README.md                  # this document
├─ flows/
│  └─ WelcomeFlow.json        # (optional) exported flow JSON
├─ prompts/
│  ├─ greeting.txt
│  ├─ main_menu.txt
│  ├─ invalid_option.txt
│  ├─ all_agents_busy.txt
│  └─ out_of_hours.txt
└─ screenshots/
   ├─ PublP26-WelcomeFlow-Published.png
   ├─ Phones-ClaimedNumbers.png
   ├─ Queues-Overview.png
   └─ ... (additional step-by-step captures)

Screenshots
Insert your captured screenshots (flow published, queues, phone numbers, etc.) into this section as you document runs and changes.
License
MIT.

Author & Maintainers

Project: Amazon Connect – WelcomeFlow IVR (Lab 26)
Status: Published (working IVR with Sales, Customer Service, and Tech Support queues)

Author:
• Dr. Ime Ben
• GitHub: https://github.com/ime-cloud-sec-analyst
• Organization: VaultIQ Global Solutions Ltd.
• Title/Role: Cloud/Contact Center Engineer or Cloud Security Analyst]
• Email: info@vaultiqglobalsolutions.com
• LinkedIn: www.linkedin.com/in/ime-ben-8aa008362
• Location / Time Zone: London, United Kingdom
• AWS Account Alias (lab): imeben
