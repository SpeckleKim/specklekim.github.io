---
layout: default
title: "Tool Use & Function Calling"
lang: en
---

# Tool Use & Function Calling

## Introduction

Tool use (also called function calling) enables language models to call external functions and APIs during inference. Rather than generating text only, an [LLM](/llm-eng.html) can now invoke tools—search databases, execute code, fetch real-time data, control devices—making models agents capable of interacting with external systems. Tool use is fundamental to building practical AI systems.

## The Tool Use Paradigm

### Traditional LLM Output

[LLM](/llm-eng.html) generates only text:
```
User: "What's the weather in London?"
LLM: "I don't have access to real-time weather data..."
```

### With Tool Use

[LLM](/llm-eng.html) can invoke tools:
```
User: "What's the weather in London?"
LLM: [calls tool] weather_api(location="London")
Tool: Returns {"temp": 15C, "condition": "rainy"}
LLM: "It's 15°C and rainy in London."
```

## Function Calling APIs

### OpenAI

OpenAI's [GPT](/gpt-eng.html) models support function_calling natively:

```python
functions = [
    {
        "name": "get_weather",
        "description": "Get weather for a location",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {"type": "string"}
            }
        }
    }
]

response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "Weather in London?"}],
    functions=functions
)
```

Model returns tool calls in structured format, system executes them.

### Anthropic

Claude supports tool use through the messages API:

```python
tools = [
    {
        "name": "weather",
        "description": "Get weather data",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {"type": "string"}
            }
        }
    }
]

response = client.messages.create(
    model="claude-opus",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "Weather?"}]
)
```

### Open-Source Models

- Gorilla: Specialized in function calling
- LLaMA with Hermes: Function calling fine-tuned
- Mistral: Native tool calling support
- Smaller models less reliable at complex tool use

## Tool Selection and Routing

### Fixed Tool Set

Predefined tools available to model:
- Tools declared upfront
- Model selects which tools to use
- Simple, deterministic

**Limitations**: Model can't call undefined tools, must anticipate all needs.

### Dynamic Tool Discovery

Tools discovered at runtime:
- Describe available tools to model
- Model selects from available set
- More flexible, but model must understand descriptions

**Challenge**: Ensuring model understanding of tool capabilities.

### Hierarchical Tool Selection

Multiple levels of tool selection:
- High-level: "category of tool"
- Low-level: "specific tool within category"
- Reduces decision space at each step

**Example**:
1. Category: "information retrieval"
2. Specific: "search web" vs "query database"

## Tool Composition

### Sequential Tool Calls

One tool calls output feeds into next:

```
User: "Compare prices of iPhone 15 on Amazon vs BestBuy"
1. search_amazon("iPhone 15")
2. search_bestbuy("iPhone 15")
3. LLM compares prices
```

### Parallel Tool Calls

Multiple tools called simultaneously:

```
search_amazon("iPhone 15") ------\
search_bestbuy("iPhone 15") ------> LLM comparison
search_walmart("iPhone 15") -----/
```

Faster than sequential, requires more computation.

### Conditional Tool Calls

Tool selection depends on conditions:

```
if user asks about time:
    call get_current_time()
elif user asks about weather:
    call weather_api()
else:
    call general_knowledge()
```

## Error Handling and Validation

### Tool Failure Handling

What if tool returns error?

```python
try:
    result = weather_api("London")
except ToolException as e:
    # Option 1: Inform LLM of error, let it retry differently
    # Option 2: Automatically retry with different parameters
    # Option 3: Return error to user
```

[LLMs](/llm-eng.html) need graceful error handling; they're prone to invalid tool calls.

### Parameter Validation

Ensure tool parameters are valid:

```python
def get_weather(location: str, units: str = "celsius"):
    if units not in ["celsius", "fahrenheit"]:
        raise ValueError("Invalid units")
    return weather_api(location, units)
```

Prevent passing malformed data to tools.

### Hallucination Prevention

[LLMs](/llm-eng.html) may invent tool names or parameters:

```
User: "What's the weather?"
LLM [hallucinates]: "Calling tool: weather_predictor..."
System: "Tool 'weather_predictor' not found"
LLM: "Let me use weather() instead"
```

Include explicit error messages to guide correction.

## Real-World Tool Integration

### Search Tools

Enable model to search web:
- Search engines (Google, Bing APIs)
- Knowledge bases (Wikipedia, documentation)
- Internal databases

**Example**: "Tell me about recent AI breakthroughs" → searches news + Wikipedia.

### Calculation Tools

Offload math to ensure correctness:
- Calculator
- Algebra solver
- [Statistics](/probability-eng.html) functions

**Example**: "What's 17234 * 8472?" → calculator tool ensures correctness.

### Code Execution

Execute code dynamically:
- Python REPL
- Sandboxed code environment
- Data analysis (pandas, matplotlib)

**Example**: "Plot my sales data" → executes Python with provided data.

### API Integration

Call production APIs:
- CRM (Salesforce, HubSpot)
- Databases (SQL queries)
- SaaS services (Slack, email)

**Challenges**: Authentication, permission boundaries, audit trails.

### File Systems

Read/write files:
- Knowledge base queries
- Document retrieval
- File creation

**Security**: Sandbox file access to prevent unauthorized reads.

## Best Practices

### Clear Tool Descriptions

Descriptions must unambiguously convey purpose:

```python
# Bad: "Get data"
# Good: "Retrieve current weather conditions for a specified city"

# Bad: "Process information"
# Good: "Execute Python code in sandboxed environment"
```

### Minimal Tool Set

Fewer tools = better model performance:
- Reduced confusion
- Faster decision-making
- Easier debugging

Start with essential tools, add more if needed.

### Explicit Tool Constraints

Communicate limitations:
```python
"Tool X works for locations in US only"
"Tool Y has 100 requests/day limit"
```

### Monitor Tool Usage

Log which tools are called:
- Understand model behavior
- Identify misuse (wrong tool chosen)
- Spot [hallucinations](/hallucination-eng.html)

### Graceful Degradation

Model should work even if tools unavailable:
- Tool timeout → return partial answer
- Tool failure → try alternative approach
- No network → use cached knowledge

## Limitations and Challenges

**Accuracy**: [LLMs](/llm-eng.html) can call non-existent tools or use wrong parameters

**Latency**: Tool calls add network latency (100ms-1s per call)

**Cost**: Tool calls consume tokens; expensive at scale

**Security**: Must validate all tool calls, prevent unauthorized operations

**Complexity**: Each tool adds potential points of failure

## Future Directions

- **Auto-tool-discovery**: Models automatically discover available tools
- **Tool learning**: Models improve tool use over time from experience
- **Tool composition**: Complex workflows from simple tools
- **Multimodal tools**: Tools consuming/producing images, audio
- **Tool-specific models**: Fine-tuned models optimized for specific tool use

## Conclusion

Tool use transforms [LLMs](/llm-eng.html) from text generators into agents capable of interacting with real systems. Effective tool use requires clear API design, robust error handling, and thoughtful security boundaries. As tool ecosystems mature, agent systems become increasingly powerful and practical.
