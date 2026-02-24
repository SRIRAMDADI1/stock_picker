## Architecture

┌─────────────────────────────────────────────────────────────────────────────┐
│                           StockPicker Crew (Hierarchical)                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌──────────────┐                                                          │
│   │   Manager    │  (GPT-4o, delegates tasks, goal: pick best company)      │
│   └──────┬───────┘                                                          │
│          │ delegates                                                        │
│          ▼                                                                  │
│   ┌──────────────────────────────────────────────────────────────────────┐ │
│   │  Task 1: find_trending_companies                                       │ │
│   │  Agent: trending_company_finder  │  Tool: SerperDevTool  │  memory ✓  │ │
│   │  Output: TrendingCompanyList → output/trending_companies.json          │ │
│   └──────────────────────────────────┬───────────────────────────────────┘ │
│                                      │ context                              │
│          ┌───────────────────────────┘                                      │
│          ▼                                                                  │
│   ┌──────────────────────────────────────────────────────────────────────┐ │
│   │  Task 2: research_trending_companies                                  │ │
│   │  Agent: financial_researcher  │  Tool: SerperDevTool                  │ │
│   │  Output: TrendingCompanyResearchList → output/research_report.json    │ │
│   └──────────────────────────────────┬───────────────────────────────────┘ │
│                                      │ context                              │
│          ┌───────────────────────────┘                                      │
│          ▼                                                                  │
│   ┌──────────────────────────────────────────────────────────────────────┐ │
│   │  Task 3: pick_best_company                                            │ │
│   │  Agent: stock_picker  │  Tool: PushNotificationTool  │  memory ✓     │ │
│   │  Output: decision report → output/decision.md + push notification     │ │
│   └──────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  Memory: LongTermMemory (SQLite) │ ShortTermMemory (RAG) │ EntityMemory (RAG) │
└─────────────────────────────────────────────────────────────────────────────┘


## Component summary

Layer	Components
Entry	main.py → StockPicker().crew().kickoff(inputs)
Orchestration	CrewAI Crew with Process.hierarchical, manager_agent
Agents	Manager, trending_company_finder, financial_researcher, stock_picker (from agents.yaml)
Tasks	find_trending_companies → research_trending_companies → pick_best_company (from tasks.yaml)
Tools	SerperDevTool (search), PushNotificationTool (Pushover)
Structured output	Pydantic: TrendingCompanyList, TrendingCompanyResearchList
Memory	LongTermMemory (SQLite), ShortTermMemory + EntityMemory (RAG with OpenAI embeddings)
Config	crew.py (crew/agents/tasks), config/agents.yaml, config/tasks.yaml