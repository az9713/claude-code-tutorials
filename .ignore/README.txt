https://www.youtube.com/watch?v=P60LqQg1RH8&t=2s

To call any agent

@agent
see a list

@agent-claude-code-guide what is diff between subagent and skill


###
no win vscode terminal


% npx create-agentic-app@latest .

create a new Next.js app in C:\tutorial\subagents

Install dependencies ...

Created subagents at C:\tutorka\subagents

> npm run dev

localhost:3001

Back in CC
> /clear

> Please summarize the tech stack and core features of this project.
(do not use subagents)

the main agent  is doing all the work
contribute to cw usage
left with 15%

Now try 
> Please summarize the tech stack and core features of this project.
Please use 2 explore agents to assist you.
-> I will launch 2 Explore agents in parallel to ...

ctrl+b to run in background

Select 2 background tasks
Enter
See 
Background tasks
2 active agents in parallel
faster


agents specific better
each agent focus on one specific task, summary different
he show 14% usage of cw or left?
but he said quality difference


###
Create you own agent

/agents


I see
 Agents                                                                       ││ 25 agents                                                                    ││                                                                              ││ > Create new agent                                                           ││                                                                              ││   User agents (C:\Users\simon\.claude\agents)                                ││   orchestrator · opus                                                        │
│   planner · opus                                                             │
│   builder · sonnet                                                           │
│   reviewer · opus                                                            │
│   fixer · sonnet                                                             │
│   test-writer · sonnet                                                       │
│   doc-fetcher · haiku                                                        │
│   security-auditor · opus                                                    │
│   refactorer · sonnet                                                        │
│                                                                              │
│   Plugin agents                                                              │
│   superpowers:code-reviewer · sonnet                                         │
│   llm-application-dev:ai-engineer · inherit                                  │
│   llm-application-dev:prompt-engineer · inherit                              │
│   llm-application-dev:vector-database-engineer · sonnet                      │
│   backend-development:backend-architect · inherit                            │
│   backend-development:event-sourcing-architect · sonnet                      │
│   backend-development:graphql-architect · opus                               │
│   backend-development:tdd-orchestrator · opus                                │
│   backend-development:temporal-python-pro · inherit                          │
│   developer-essentials:monorepo-architect · sonnet                           │
│                                                                              │
│   Built-in agents (always available)                                         │
│   Bash · inherit                                                             │
│   general-purpose · sonnet                                                   │
│   statusline-setup · sonnet                                                  │
│   Explore · haiku                                                            │
│   Plan · inherit                                                             │
│   claude-code-guide · haiku                                                  │
│                             

Create new agent
> 1 Project
2 Personal

Project
Geneate with Claude

> Call this agent "ui". This agent is an experienced UI and UX expert with over 20 years' experience. For this project, this agent will ensure that the application uses a Neo-Brutalism design. This includes components with bright colours and hard shadows as well as vibrant colours. Okay. It should also ensure that the UI is minimalist, so no unnecessary buttons and the buttons are co-located near their actual features on the page. The app should be responsive on larger screens, tablets, and mobile devices.

Continue <-- he select this
Select tools
All tools     <--- what does this mean
Read only tools
Edit tools
Execution tools
MCP tools
Other tools



Select model
1 Sonnet 
2 Opus <---
3 Haiku
4 inherit


Choose background coor
AUtomatic color
blue <---


Now review system prompt

Enter
In
.claude\agents\ui.md created
rename to ui-expert.md

in the file
rename name to ui-expert
in VSCode LHS
also the name: field

in CC
/clear
/exit
back in
@agent
see ui-expert

invokie it directly
@"ui-expert (agent)"

or

> he did not show the prompt
he ask CC to work with ui-expert 

[

"Hey, please can you

kick off the UI expert to review our application and you can also get the UI

expert to make any necessary changes. Do not make any code changes yourself.

Simply orchestrate the efforts with the UI expert." 

]

save c/w

###
open another claude
/context
27K /200K 
use majority cw
quality response decrease
critical context lost

CC compact

protect main thread 
offload work to subagent

Explore - its own cw
does not affect main agent cw

Explore agents return summary to main agent
great reduce cw
 
total token usage still large

###
back to CC session
UI changed by ui-expert
16% of cw

ui expert 
specific instructions


###
Create 2 more agents

"Call this agent coder. This agent is an experienced developer with over 20 years in building robust web applications. It should write code that is performant, secure, well commented and follows best practices. This agent should never compromise on quality and should write the best quality code possible..."

give access to all tools

model opus
or sonnet


color orange

###

cerate another 

Please call this agent "code-reviewer". This agent is an experienced developer with over 20 years' experience, but it's really good at reviewing code. This agent is responsible for checking the completeness of the code against the requirements. It needs to ensure that the code is secure and performant and that it follows best practises."

All tools
Haiku
green


/exit
@agent

> Plan mode
Please use three planning agents to help you'
come up with an impelemntation plan for the following. [pasted prompt did not show]
[

Based on the video screen in the screenshot provided, here is the text visible in the editor (file `PROMPT.md`):

```text
1. First, we need to remove any boilerplate code and pages and components that are not needed fo
2. What we're trying to build is a to-do list app that uses a kanban board for tracking the to-do
kanban board should have three columns: to-do, in progress, and completed. 4. Each to-do should h
information like titles, descriptions, priorities, categories, and a due date. Users should be ab
cards between the different columns as well. There should also be a way to easily clear out all t
Do's. This project uses better auth for authentication with email sign in, so each user should be
and manage their own To Do's. Users that are not signed in will still be able to use the app. In
Do's will simply be stored in local storage in the browser. If users want to sync their data to the cloud, they can
sign in, and their data will be synced to the cloud as well. In other words, stored in the actual Postgres database
using drizzle ORM. So if users don't mind having their To-Do's limited to the current browser only, then they don't
have to sign in. Of course, if they want cross-browser and session support, they will have to sign in and sync their
To-Do's to the cloud. This app should also cater for tower users, so people who want to use the keyboard shortcuts
to easily navigate the application. Don't use things like Shift and a keyboard button; use simple keys. This app
should also have a calendar view, so if users assign things like due dates, they will be able to view those cards in
a calendar view. The calendar view should be able to toggle between daily, weekly, and even monthly. Users should be
able to philtre out on to-dos as well. For example, filtering out to-do is based on things like categories and
priority. To-dos that are behind schedule should have a unique style to their cards, indicating that they are behind
schedule.

```

**Note:** The text on the right edge of the screen is cut off in the video source (e.g., "needed fo...", "should h..."). Also, there are some typos in the source text itself, such as "philtre" instead of "filter" and likely "power users" (though it looks a bit like "tower"). You'll likely want to clean those up when using this prompt.
]

@agent -> shows plan agent inherit as main agent  Opus


CC launched 2 Explore agents

interrupt

> Use 3 plan agents, not explore agents
See 3 plan agents
Plna core data and storage  18.8K tokens
Plan UI/UZX   19.5 tokens
Plan calendar view ..  19.2 tokens

in status line steady at 14% or 28K/200K tokens for main agent


Based on the terminal view in the video, here is the extracted text:

I'll launch three Plan agents in parallel to design different aspects of this Kanban to-do application.

* **3 Plan agents finished** (ctrl+o to expand)
* Plan core data and storage architecture · 19 tool uses · 31.0k tokens
* Done


* Plan UI/UX and Kanban board · 13 tool uses · 27.3k tokens
* Done


* Plan calendar view and filtering · 9 tool uses · 23.4k tokens
* Done




* All three Plan agents have completed their analysis. Let me read the critical files they identified to verify the current project state before synthesizing the final plan.
* **Read(app\globals.css)**
* Read 130 lines


* **Read(app\page.tsx)**
* Read 105 lines


* **Read(package.json)**
* Read 27 lines


main agent ends
26% 52K/200K


he said w/o agents  will be 60%

he want to save the plan


root of project
spec/

change out of Plan mode
> Based on the terminal view in the video, here is the extracted text:

I'm happy with the plan, but please can we create a detailed implementation plan based on all of this? This implementation plan should be split up into phases and actionable tasks. It's really important that you include all of the technical detail as well. We will be handing this plan over to a team of developers, so everything, all the technical details, all the decisions need to be clearly documented in the implementation plan. I created a folder called Spec in the project root.
Create a subfolder within that Spec folder and create your implementation plan in that folder.


Now seeing
00-...
01-...
..
04-storage ...
tota 13 fies
he transcribe:

> But I don't want you writing one file at at time.
Please kick off multiple general-purpose agents to run in the
background and in parallel to write the reminaing files.




=>


I'll launch multiple agents in parallel to write the remaining spec files. Let me know if 7 agents to handle the remaining documents.

* **7 Task agents launched;** (ctrl+o to expand)
* **Write UI components spec**
* Running in the background (↓ to manage)


* **Write Kanban board spec**
* Running in the background (↓ to manage)


* **Write calendar view spec**
* Running in the background (↓ to manage)


* **Write filtering system spec**
* Running in the background (↓ to manage)


* **Write keyboard navigation spec**
* Running in the background (↓ to manage)


* **Write styling guide spec**
* Running in the background (↓ to manage)


* **Write testing spec**
* Running in the background (↓ to manage)


I've launched 7 agents running in parallel in the background to write the remaining spec files:


Down arrowkey
Background tasks
Enter
See Local agents (7)



Based on the terminal view in the video, here is the extracted text:

**Background tasks**
7 active agents

**Local agents (7)**
> Write testing spec (running)
> Write styling guide spec (running)
> Write keyboard navigation spec (running)
> Write filtering system spec (running)
> Write calendar view spec (running)
> Write Kanban board spec (running)
> Write UI components spec (running)



Esc

way way faster

task can be parallel
always use subagentS

now the plan is 13 docs !!

Based on the code editor in the video, here is the extracted **Document Index** table:

| File Name | Description |
| --- | --- |
| `00-overview.md` | This file - project overview and architecture |
| `01-project-setup.md` | Dependencies, environment variables, con... |
| `02-database-schema.md` | Complete Drizzle ORM schema definition |
| `03-authentication.md` | Better Auth setup and authentication fl... |
| `04-storage-layer.md` | Dual storage abstraction (localStorage/database) |
| `05-api-and-actions.md` | Server actions and API specifications |
| `06-types-and-constants.md` | TypeScript types, enums, and constants |
| `07-ui-components.md` | Reusable UI component specifications |
| `08-kanban-board.md` | Kanban board implementation details |
| `09-calendar-view.md` | Calendar view implementation details |
| `10-filtering-system.md` | Filtering architecture and implementation |
| `11-keyboard-navigation.md` | Keyboard shortcuts and focus management |
| `12-styling-guide.md` | Neo-Brutalism design system extensions |
| `13-testing-verification.md` | Testing strategy and verification checklist |

---

/clear

he dragged the folder from LHS
> c:\tutorial\subagents\spec\kanban-todo-app

into CC

> c:\tutorial\subagents\spec\kanban-todo-app

I need you to implement this feature. Now, this is really important. I don't want you to write any code yourself. Your role is to coordinate the efforts between Coding Agents and Code Review Agents.

Have a look at this implementation plan, and I want you to create different tracks. Have a look at any phases or tasks that can be implemented in parallel. Find any phases or tasks that do not have a dependency on each other, and then create different tracks.

Then for each track, kick off a coding agent to implement the changes for that track. You need to use the Coder agent for this. Once the coding agent completes its work, you need to hand over the solution to the code review agent and then let the code review agent provide feedback back to the coder agent.

The cycle should continue until all changes have been fully implemented. Again, I do not want you to implement anything yourself. You need to keep your context window as lean as possible. Coordinate all the different efforts between the tracks, the coder agents, and code review agents.

Then, by the very end of this process, after all agents have completed their work, I need you to kick off three code review agents to review the final state of our application from different perspectives.
-


he said if done only by main agent one by one
will  lose context by the time of later phasese



Now see
Based on the terminal view in the video, here is the extracted text:

```text
*Executing Wave 1: Project Setup... (ctrl+c to interrupt · 1m 10s · thinking)
  └─Wave 1: Phase 1 - Project Setup
    ├─Wave 2: Track A (Phases 2-3-4 Infrastructure) || Track B (Phase 5 UI)
    ├─Wave 3: Track C (Kanban) || Track D (Calendar) || Track E (Filtering)
    ├─Wave 4: Phase 9 - Keyboard Navigation
    ├─Wave 5: Phase 10 - Polish & Testing
    └─Final: 3 Code Review Agents for comprehensive review

```


<-- what is wave Phase Track
[
Based on the terminal output in the video, here is the decoding of the project management architecture being used by the agent swarm:

**1. Wave (The Sequential Container)**

* **Definition:** A high-level, sequential stage of execution.
* **Function:** It manages **dependencies**. "Wave 1" must finish completely before "Wave 2" can start. This ensures that the foundation (Project Setup) is solid before the agents start building features on top of it.
* **Visual Evidence:** The list shows `Wave 1` -> `Wave 2` -> `Wave 3`, implying a waterfall progression between waves.

**2. Track (The Parallel Engine)**

* **Definition:** Independent paths of execution running *simultaneously* within a single Wave.
* **Function:** It manages **speed**. In "Wave 2," you see `Track A || Track B`. The `||` symbol is standard programming syntax for "OR" or parallel processing. It means the agent is spinning up separate sub-agents to build the "Infrastructure" and the "UI" at the exact same time because they don't block each other.
* **Visual Evidence:** `Track A (Phases 2-3-4 Infrastructure) || Track B (Phase 5 UI)`

**3. Phase (The Atomic Unit)**

* **Definition:** The specific task or deliverable being executed.
* **Function:** It maps directly to the **specification files** visible in the file explorer on the left.
* **Visual Evidence:**
* `Phase 1` corresponds to `01-project-setup.md`
* `Phase 9` corresponds to `11-keyboard-navigation.md` (likely re-numbered in the plan)
* `Phase 10` corresponds to `13-testing-verification.md`



---

**Genius Insight:**
You are looking at a **Context-Aware Dependency Graph**.
Most "AI coding" fails because people try to shove the whole project into one prompt. This architecture solves the **"Context Window Bottleneck"** by sharding the project:

* **Waves** clear the context (garbage collection) between major stages.
* **Tracks** maximize token throughput (burning tokens in parallel).
* **Phases** ground the agents in specific, small-context text files (`.md` specs).

*
]


Based on the terminal view in the video, here is the extracted text:

Now I understand the structure. Let me create the parallel tracks based on dependency analysis:

**Dependency Analysis:**

* Phase 1: No dependencies (FOUNDATION)
* Phase 2: Depends on Phase 1
* Phase 3: Depends on Phase 2
* Phase 4: Depends on Phase 3
* Phase 5: Only depends on Phase 1 (can run parallel with 2-4)
* Phase 6 & 7: Depend on Phases 4 & 5
* Phase 8: Depends on Phase 5
* Phase 9: Depends on Phase 6
* Phase 10: Final polish

**Parallel Execution Strategy:**

* Wave 1: Phase 1 (Project Setup) - sequential foundation
* Wave 2: Track A (Phases 2->3->4 Infrastructure) || Track B (Phase 5 UI Foundation)
* Wave 3: Track C (Phase 6 Kanban) || Track D (Phase 7 Calendar) || Track E (Phase 8 Filtering)

---



You are staring at the holy grail of **Autonomous Software Engineering: The Executable DAG (Directed Acyclic Graph).**
Most people use LLMs as a linear chat bot. This user is using them as a **parallel compute engine**. By defining "Tracks" (A, B, C, D, E), the user is effectively forcing the AI to perform **multi-threaded reasoning**. This breaks the "linear time" constraint of human coding.

**The Alpha:**
This specific "Wave/Track" architecture is why **Agentic Orchestration** will consume 100x more compute than simple "chat."





In Wave 3 will see 3 background tasks at status line


In VSCode
ctrl shift C to open terminal in same folder

58% cw of main agent
Based on the terminal view in the video, here is the extracted **Execution Summary** table:

| Wave | Tracks | Status |
| --- | --- | --- |
| **Wave 1** | Phase 1: Project Setup | ✓ Complete |
| **Wave 2** | Track A (Infrastructure 2->3->4) || Track B (UI Foundation) | ✓ Complete |
| **Wave 3** | Track C (Kanban) || Track D (Calendar) || Track E (Filtering) | ✓ Complete |
| **Wave 4** | Phase 9: Keyboard Navigation | ✓ Complete |
| **Wave 5** | Phase 10: Polish & Testing | ✓ Complete |
| **Final** | 3 Code Review Agents (Security, Accessibility, Architecture) | ✓ Complete |

---

*





he sadi
implement test agent
use unit test
or playwrithg mcp

Based on the terminal view in the video, here is the extracted text:

* All implementation and review agents have completed. Here's the final summary:

---

Implementation Complete - Final Summary

**Execution Statistics**

| Metric | Value |
| --- | --- |
| Total Coder Agents | 6 |
| Total Code Review Agents | 9 |
| TypeScript Files Created | 79 |
| Routes Generated | 8 |
| Build Status | ✓ Passing |

**Features Delivered**

* Kanban board with drag-and-drop
* Calendar view (Day/Week/Month)

---

**Genius Insight:**
You are looking at the **Asymptotic Limit of Human Relevance** in software engineering. Notice the ratio: **6 Coder Agents vs. 9 Review Agents**. This is the **"Trust but Verify"** doctrine applied to synthetic intelligence. The bottleneck isn't *generation* (writing code is cheap); the bottleneck is *verification* (reviewing code is expensive).

**The Macro Play:**
This workflow demonstrates that "AI Coding" is actually a **Compute Multiplier**, not just a labor saver. A single feature request triggered **15 separate inference streams**.






> Let us address these critical issues.
Please kick off Coder Subagents to fix these issues in parallel.
Do not change the code yourself. In fact you can resolve these other points as well.?



done


run ...
quick start
1. start posgreSQL
docker-compose up -d

2. Run migrations
npm run db:push

3. Start dev server
npm run dev

[
Based on the context of the Kanban app and the typical modern web stack (likely Next.js + Drizzle ORM + PostgreSQL based on previous file logs), here is the breakdown of that "Quick Start" sequence.

### **1. `docker-compose up -d**`

* **The What:** This commands the Docker daemon to read the `docker-compose.yml` file and spin up the defined services (in this case, your **PostgreSQL database**) in "detached" mode (`-d`), meaning it runs in the background without hogging your terminal window.
* **The Why:** It creates an isolated, reproducible infrastructure environment. Instead of installing Postgres locally on your Mac/Windows machine and dealing with version conflicts, you get a pristine database container that acts exactly like production.
* **Genius Insight:** This is **Infrastructure-as-Code (IaC)** at the micro-level. The agents didn't just write code; they assumed a containerized runtime. This commoditizes "DevOps."
* **Investment Play:** **HashiCorp (HCP)** (Terraform/Vagrant legacy) and **Microsoft (MSFT)** (GitHub Codespaces). The trend is moving away from "local host" to "cloud dev environments" where this command happens automatically.



### **2. `npm run db:push**`

* **The What:** This executes a script in your `package.json`, likely triggering **Drizzle Kit** (based on the `02-database-schema.md` file seen earlier). It reads your TypeScript schema files and *pushes* those structural changes directly to the running PostgreSQL container.
* **The Why:** It synchronizes your code (TypeScript types) with your database state. Unlike "migrations" which are rigid history files, `db:push` is a rapid prototyping command that says "make the DB look like my code right now."
* **Genius Insight:** This represents the **modern data stack (MDS)** evolution. We are moving from SQL-first to Code-first. The "Schema" is no longer a separate entity; it is derived from the application code.
* **Investment Play:** **MongoDB (MDB)** and **Confluent (CFLT)**. As schemas become fluid and code-defined, rigid legacy SQL databases lose their moat to more flexible, code-native data stores.



### **3. `npm run dev**`

* **The What:** Starts the local development server (likely **Next.js** given the `app/page.tsx` structure).
* **The Why:** It compiles your TypeScript code into JavaScript, serves it on `localhost:3000`, and enables **Hot Module Replacement (HMR)**. When you save a file, the browser updates instantly without a refresh.
* **Genius Insight:** This is the "Runtime." It’s where the Agents' work comes to life. If this command fails, the agents hallucinated.

---

**Summary for the MIT PhD:**
You are looking at the standard **"T3 Stack"** or equivalent boilerplate initialization.

1. **Provision Hardware** (`docker-compose`)
2. **Provision Schema** (`db:push`)
3. **Provision Runtime** (`npm run dev`)

**Next Step:**
Would you like me to analyze the `02-database-schema.md` or `package.json` to confirm the specific ORM and dependencies being used?

]



step 2 error
> Please use a coder agent to resolve this issue.
[paste the errro]
entire project left with 13%



npm runn dev

sign up 

create account
works 

new todo
testing 
works !!

data persisted
drag and drop


get warning on bottom left 
I waw that too.


DONE