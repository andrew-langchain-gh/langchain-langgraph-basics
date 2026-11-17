# LangChain and LangGraph Basics

Build a simple travel planner in [agent_basics.ipynb](agent_basics.ipynb) to explore tool use, explicit control flow, parallel searches, and conversation memory.

## Examples

The notebook builds on the same travel planning task:

1. **LangChain flight agent:** use `create_agent` with a Tavily web search tool to research flights.
2. **LangGraph flight agent:** recreate the single-tool agent with explicit state, nodes, edges, and a model/tool loop.
3. **MCP hotel search:** connect to an authenticated Arcade gateway and wrap its Google Hotels search tool.
4. **Parallel travel planner:** prepare shared trip details, search flights and hotels concurrently, and combine both results into a formatted plan.
5. **Multi-turn conversations:** run six independent three-turn conversations for use with a LangSmith conversational evaluator.

LangGraph lets you control the logical flow, including when searches start and when their results are combined:

```text
                        ┌→ find_flights ─┐
START → prepare_trip ────┤               ├→ combine_plan → END
                        └→ find_hotels ──┘
```

## Setup

You need Python 3.13 or later, [uv](https://docs.astral.sh/uv/getting-started/installation/), and an IDE or notebook client that can use a local Python kernel.

From the project directory, install the locked dependencies and create your local configuration:

```bash
uv sync --locked
cp env.example .env
```

If you already have a `.env` file, keep it and update the settings below. Dependencies are declared in `pyproject.toml`; `uv.lock` records their exact resolved versions. The default sync includes the notebook kernel and display dependencies.

Configure these variables in `.env`:

| Variable | Purpose |
| --- | --- |
| `OPENAI_API_KEY` | Model access; the notebook uses `gpt-4.1-mini` |
| `TAVILY_API_KEY` | Web searches for flights |
| `ARCADE_API_KEY` | Authentication for the Arcade MCP gateway |
| `ARCADE_USER_ID` | Arcade user identity, sent in the `Arcade-User-Id` header |
| `LANGSMITH_API_KEY` | LangSmith tracing |
| `LANGSMITH_PROJECT` | Project that receives the notebook traces |
| `LANGSMITH_TRACING` | Set to `true` to enable tracing |

The hotel example uses the [Arcade MCP gateway configured in the notebook](https://api.arcade.dev/mcp/gw_3MCruEccqjnr9QDv3aKDIVb2yXM). Your Arcade credentials must have access to that gateway and its `GoogleHotels.SearchHotels` tool. To use your own gateway, update the URL in the MCP setup cell.

## Run the notebook

Open [agent_basics.ipynb](agent_basics.ipynb), select `.venv/bin/python` as its kernel, and run cells from top to bottom with this project directory as the working directory. The notebook loads `.env` and uses top-level `await` for asynchronous calls.

The examples make live model, web search, and MCP requests and may consume API credits. Flight searches return web listings and snippets; prices and availability need confirmation at the source. The planner does not book travel.

## Conversation examples in LangSmith

The final section wraps the parallel planner in a conversational agent backed by `InMemorySaver`. Each example gets a fresh `thread_id`, reused across its three turns:

1. Request flights and hotels.
2. Ask for different flights and hotels while retaining unchanged trip details.
3. Reply with either “Thanks” or “That's not what I asked for”.

The closing feedback is scripted sample data. Running the loop generates conversations; it does not run an evaluator automatically.

With tracing enabled, open the project specified by `LANGSMITH_PROJECT`. Find the conversations using the `travel-conversation-demo` tag or the printed batch ID, recorded as `conversation_batch` metadata. Turns share `thread_id` metadata so they can be grouped for conversational evaluation.

Transcripts are also stored in `conversation_results`. Rerunning the loop creates six new threads; restarting the kernel clears the in-memory conversation state.
