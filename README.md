# LLM Project

## API Key Configuration

This project uses the **free Google Gemini API** (`gemini-2.0-flash-exp` model) for all LLM operations. You need to obtain a free API key from [Google AI Studio](https://makersuite.google.com/app/apikey).

Create a `.env` file in the project root with your API key:
```
GOOGLE_API_KEY=your_api_key_here
```

## Required Libraries

The following libraries are used in this project:
- `langchain-community` - Document loaders (PyMuPDFLoader)
- `langchain-google-genai` - Google Gemini LLM integration
- `langchain-core` - Core LangChain functionality (messages, tools)
- `langgraph` - Graph-based agent orchestration
- `PyMuPDF` - PDF document processing
- `python-dotenv` - Environment variable management
- `jupyter` - Notebook environment

## Installation

Install all required dependencies:
```bash
pip install -r requirements.txt
```

## Part 1: Document Extraction & Prompt Engineering with LangChain

**Objective:** Load the PDF document into the LLM context window without RAG.

**Implementation:**
- Used `PyMuPDFLoader` & `PDFPlumberLoader` from `langchain_community.document_loaders` to load the PDF.
- Used only the required pages from the pages [5, 6, 8, 20] from the document.
- Created a `pydantic` model to extract the required fields from the documents.
- Passed the extracted data and the model to the LLM to generate the answer.

**Eplanation:**
For this task based on the document [text](fy2024_analysis_of_revenue_and_expenditure.pdf), the document has lots of tables and some complex layouts. Which `PyMuPDFLoader` is good at, just to test out another loader I have tried `PDFPlumberLoader` and it worked well, however it was not as fast as `PyMuPDFLoader`.

## Part 2: Tool Calling & Reasoning Integration
**Objective:** Extend Part 1 to include LLM reasoning capabilities and external tool usage.

**Implementation:**
- Created a custom `date_to_iso` tool using the `@tool` decorator to convert date strings to ISO format (YYYY-MM-DD)
- Bound the tool to the LLM using `bind_tools()` method
- Extracted date strings from the PDF document (e.g., "16 February 2024", "15 February 2008")
- Used the LLM to invoke the tool and normalize dates automatically
- Implemented reasoning logic to evaluate normalized dates against a reference date (2024-01-01)
- Categorized dates into states: "Expired" (before reference) or "Upcoming" (after reference)
- Demonstrated the LLM's ability to chain tool calls with reasoning to produce structured outputs

**Key Features:**
- Tool definition with Pydantic schema for input validation
- Automatic tool invocation by the LLM based on context
- Multi-step reasoning: tool calling → normalization → evaluation → categorization
- Structured JSON output with original text, normalized date, and status

## Part 3: Multi-Agent Supervisor System

**Objective:** Implement a multi-agent system using LangGraph with a supervisor pattern
### Architecture:
1. **Supervisor Agent**
   - Coordinates the specialized agents
   - Analyzes user queries to determine which agents to invoke
   - Formulates specific sub-queries for each agent
   - Synthesizes responses into a comprehensive final answer
   - Uses `bind_tools()` to access specialized agents as tools

2. **Revenue Agent**
   - Specializes in identifying and extracting government revenue information
   - Implemented as a `@tool` decorated function
   - Uses dedicated LLM instance with system prompt for revenue analysis
   - Extracts actual data and figures from the document

3. **Expenditure Agent**
   - Specializes in analyzing government spending and fund information
   - Implemented as a `@tool` decorated function
   - Uses dedicated LLM instance with system prompt for expenditure analysis
   - Extracts specific fund details and budget figures

### LangGraph Workflow:
```
Supervisor → (decides to call agents) → Tools Node → Supervisor → Final Answer
```

- **Entry Point:** Supervisor node
- **Conditional Routing:** Based on tool_calls presence
- **Tool Execution:** All agent tools executed in parallel when called
- **Loop:** Returns to supervisor after tool execution for synthesis

**Key Implementation Details:**
- Each agent has its own LLM instance for specialized processing
- Supervisor uses `bind_tools()` to make agents available as callable tools
- Tool responses returned as `ToolMessage` objects with proper `tool_call_id` mapping
- System prompts guide each agent's behavior and output format
- No RAG system - full document passed to each agent's context

**Execution:**
The notebook demonstrates the complete workflow by:
1. Displaying all messages exchanged between supervisor and agents
2. Showing tool calls made by the supervisor
3. Presenting the final synthesized answer
