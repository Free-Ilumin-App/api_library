# AI Agent Tool Documentation

## Overview

The AI Agent tool allows you to create intelligent agents using multiple AI platforms (OpenAI, Gemini, Claude, Groq) that can execute complex tasks with or without custom tools. These agents can be integrated into your automation workflows to provide conversational AI capabilities, tool orchestration, and structured outputs.

## Supported Platforms

- **OpenAI**: GPT-4.1, GPT-4o, o1
- **Google Gemini**: gemini-2.5-flash-lite, gemini-2.5-flash, gemini-2.5-pro
- **Anthropic Claude**: claude-3-5-sonnet-20241022, claude-3-haiku-20240307
- **Groq**: llama-3.1-70b-versatile, mixtral-8x7b-32768, gemma-7b-it

Use default model for Google Gemini: gemini-2.5-flash-lite
Use default model for OpenAI: gpt-4.1-nano-2025-04-14

## Usage Examples

### Example 1: Basic AI Agent (No Tools)

A simple conversational AI agent for general queries and assistance.

```python
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_core.messages import SystemMessage, HumanMessage

try:
    llm = ChatGoogleGenerativeAI(
        model="gemini-2.5-flash-lite",
        temperature=0.7,
        google_api_key="your_api_key"
    )
    
    system_prompt = """You are a helpful business assistant specializing in customer support. 
    You provide professional, concise responses and help users with their inquiries."""
    
    user_message = "How can I track my order status?"
    
    messages = [
        SystemMessage(content=system_prompt),
        HumanMessage(content=user_message)
    ]
    
    response = llm.invoke(messages)
    print(response.content)
    
except Exception as e:
    print(f"Error: {e}")
```

**Use Cases:**
- Customer support chatbots
- General information assistants
- Content generation
- Text analysis and summarization

---

### Example 2: AI Agent with Single Tool

An AI agent that can access one custom tool for specific functionality.

```python
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import tool

try:
    llm = ChatGoogleGenerativeAI(
        model="gemini-2.5-flash-lite",
        temperature=0.7,
        google_api_key="your_api_key"
    )
    
    tools = []
    
    # Check if the IP location function is available in the execution context
    if 'obter_localizacao_por_ip' in globals():
        @tool
        def get_location_by_ip(ip_address: str):
            """Get location data (country, city, etc.) from an IP address.
            
            Args:
                ip_address (str): The IP address to get location data for. Ex: '24.48.0.1'
            """
            try:
                result = obter_localizacao_por_ip(ip_address=ip_address)
                return str(result)
            except Exception as e:
                return f"Error getting location for {ip_address}: {e}"
        
        tools.append(get_location_by_ip)
    
    if tools:
        prompt = ChatPromptTemplate.from_messages([
            ("system", "You are a network security analyst. Use your tools to analyze IP addresses and provide detailed location information."),
            ("human", "{input}"),
            ("placeholder", "{agent_scratchpad}")
        ])
        
        agent = create_tool_calling_agent(llm, tools, prompt)
        executor = AgentExecutor(
            agent=agent, 
            tools=tools,
            verbose=False,
            handle_parsing_errors=True
        )
        
        user_input = "Analyze the location of IP: 8.8.8.8"
        result = executor.invoke({"input": user_input})
        print(result["output"])
    else:
        print("IP location tool not available in execution context.")
        
except Exception as e:
    print(f"Error: {e}")
```

**Use Cases:**
- Network security analysis
- Geolocation services
- Fraud detection systems  
- Content localization

---

### Example 3: AI Agent with Multiple Tools

An AI agent with access to multiple tools for complex task orchestration.

```python
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import tool

try:
    llm = ChatGoogleGenerativeAI(
        model="gemini-2.5-flash-lite",
        temperature=0.3,
        google_api_key="your_api_key"
    )
    
    tools = []
    
    # Collect all available AI tools from execution context
    current_globals = dict(globals())
    excluded_names = [
        'print', 'len', 'str', 'int', 'float', 'dict', 'list', 'tuple', 'set',
        'range', 'enumerate', 'zip', 'map', 'filter', 'sorted', 'reversed',
        'max', 'min', 'sum', 'abs', 'round', 'bool', 'type', 'isinstance',
        'hasattr', 'getattr', 'setattr', 'tool', 'agent', 'llm', 'executor'
    ]
    
    print("Loading available AI tools...")
    
    for name, func in current_globals.items():
        if (callable(func) and 
            not name.startswith('_') and 
            name not in excluded_names and
            not (hasattr(func, '__module__') and 
                 func.__module__ and 
                 ('langchain' in str(func.__module__) or 'builtins' in str(func.__module__)))):
            
            try:
                # Create wrapper function for each AI tool
                def create_wrapper(original_func, original_name):
                    @tool(name=original_name)
                    def ai_tool_wrapper(*args, **kwargs):
                        try:
                            result = original_func(*args, **kwargs)
                            return result
                        except Exception as e:
                            return {"error": f"Error in {original_name}: {str(e)}"}
                    return ai_tool_wrapper
                
                wrapper = create_wrapper(func, name)
                tools.append(wrapper)
                print(f" Tool loaded: {name}")
                
            except Exception as e:
                print(f" Error loading {name}: {e}")
                continue
    
    print(f"Total: {len(tools)} tool(s) available")
    
    if tools:
        prompt = ChatPromptTemplate.from_messages([
            ("system", """You are an intelligent automation assistant with access to custom AI tools. 
            Analyze user requests and use the appropriate tools to complete tasks efficiently.
            Always explain your reasoning and provide clear results."""),
            ("human", "{input}"),
            ("placeholder", "{agent_scratchpad}")
        ])
        
        agent = create_tool_calling_agent(llm, tools, prompt)
        executor = AgentExecutor(
            agent=agent,
            tools=tools,
            verbose=False,
            handle_parsing_errors=True,
            max_iterations=10
        )
        
        user_input = "Process this customer data and send a welcome email"
        result = executor.invoke({"input": user_input})
        print(result["output"])
        
    else:
        print("No tools available. Using direct LLM response.")
        from langchain_core.messages import SystemMessage, HumanMessage
        
        messages = [
            SystemMessage(content="You are a helpful assistant."),
            HumanMessage(content="I need help with automation tasks.")
        ]
        
        response = llm.invoke(messages)
        print(response.content)
        
except Exception as e:
    print(f"Execution error: {e}")
    import traceback
    traceback.print_exc()
```

**Use Cases:**
- Complex business process automation
- Multi-step data processing workflows
- Customer onboarding sequences
- Intelligent task orchestration

---

### Example 4: AI Agent with Structured JSON Output

An AI agent that returns structured JSON responses for integration with other systems.

```python
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_core.messages import SystemMessage, HumanMessage
from langchain_core.output_parsers import JsonOutputParser
from langchain_core.prompts import PromptTemplate
from pydantic import BaseModel, Field
import json

# Define output structure
class CustomerAnalysis(BaseModel):
    customer_id: str = Field(description="Unique customer identifier")
    risk_score: int = Field(description="Risk score from 1-100")
    category: str = Field(description="Customer category: low_risk, medium_risk, high_risk")
    recommendations: list = Field(description="List of recommended actions")
    confidence: float = Field(description="Confidence level 0.0-1.0")
    analysis_date: str = Field(description="Analysis timestamp")

try:
    llm = ChatGoogleGenerativeAI(
        model="gemini-2.5-flash-lite",
        temperature=0.1,  # Low temperature for consistent structured output
        google_api_key="your_api_key"
    )
    
    # Set up JSON output parser
    parser = JsonOutputParser(pydantic_object=CustomerAnalysis)
    
    # Create structured prompt
    prompt = PromptTemplate(
        template="""You are a customer risk analysis AI. Analyze the provided customer data and return a structured JSON response.

Customer Data: {customer_data}

{format_instructions}

Provide a thorough analysis with accurate risk scoring and actionable recommendations.""",
        input_variables=["customer_data"],
        partial_variables={"format_instructions": parser.get_format_instructions()}
    )
    
    # Sample customer data
    customer_data = {
        "id": "CUST_12345",
        "transaction_history": [
            {"amount": 1500.00, "type": "purchase", "date": "2024-01-15"},
            {"amount": 2300.00, "type": "purchase", "date": "2024-01-20"},
            {"amount": 500.00, "type": "refund", "date": "2024-01-25"}
        ],
        "account_age_days": 90,
        "payment_method": "credit_card",
        "geographic_location": "US-CA",
        "previous_disputes": 0
    }
    
    # Format prompt and get response
    formatted_prompt = prompt.format(customer_data=json.dumps(customer_data, indent=2))
    
    messages = [
        SystemMessage(content="You are an expert customer risk analyst. Always respond with valid JSON."),
        HumanMessage(content=formatted_prompt)
    ]
    
    response = llm.invoke(messages)
    
    # Parse JSON response
    try:
        parsed_result = parser.parse(response.content)
        print("=== Customer Risk Analysis ===")
        print(json.dumps(parsed_result, indent=2))
        
        # Access structured data
        if parsed_result.get('risk_score', 0) > 70:
            print(f"\n�  HIGH RISK CUSTOMER: {parsed_result.get('customer_id')}")
            print("Immediate action required!")
            
    except Exception as parse_error:
        print(f"JSON parsing error: {parse_error}")
        print("Raw response:", response.content)
        
except Exception as e:
    print(f"Error: {e}")
```

**Example Output:**
```json
{
  "customer_id": "CUST_12345",
  "risk_score": 25,
  "category": "low_risk",
  "recommendations": [
    "Continue monitoring transaction patterns",
    "Offer premium services",
    "Send satisfaction survey"
  ],
  "confidence": 0.85,
  "analysis_date": "2024-01-29T10:30:00Z"
}
```

**Use Cases:**
- API integrations requiring structured data
- Database operations with consistent formats
- Report generation systems
- Automated decision-making workflows

---

## Advanced Features

### Tool Auto-Discovery

The AI Agent automatically discovers and loads available tools from the execution context:

- **User AI Tools**: Custom functions created by users
- **Platform Integrations**: Pre-built connectors (WhatsApp, Email, etc.)
- **Data Operations**: Database and table management functions
- **Utility Functions**: Date, text, and data processing tools

### Error Handling

Robust error handling ensures reliable execution:

```python
try:
    # Agent execution
    result = executor.invoke({"input": user_message})
    print(result["output"])
except Exception as e:
    print(f"Execution error: {e}")
    # Fallback to direct LLM if agent fails
```

### Memory and Context Management

- **Conversation Memory**: Maintain context across multiple interactions
- **Tool State**: Preserve tool execution results between calls
- **User Context**: Access user-specific data and preferences

## Best Practices

1. **System Prompts**: Write clear, specific instructions for the AI agent
2. **Tool Documentation**: Provide detailed descriptions for custom tools
3. **Error Handling**: Always implement try/catch blocks
4. **Temperature Control**: Use lower temperatures (0.1-0.3) for structured outputs
5. **Validation**: Validate inputs and outputs for production systems
6. **Security**: Never expose API keys or sensitive data in responses

## Integration with AutoPy2

AI Agents integrate seamlessly with the AutoPy2 platform:

- **Workflow Steps**: Use as intelligent workflow components
- **User Tables**: Access and modify user data tables
- **Credentials**: Securely access platform integrations
- **Webhooks**: Respond to external events intelligently
- **Scheduling**: Run periodic AI analysis tasks

## Troubleshooting

### Common Issues

1. **Tool Not Found**: Ensure custom tools are properly loaded in the execution context
2. **API Errors**: Verify API keys and model availability
3. **JSON Parsing**: Use structured prompts for consistent JSON output
4. **Memory Issues**: Monitor token usage for long conversations
5. **Rate Limits**: Implement appropriate delays for API calls

### Debug Mode

Enable verbose logging for troubleshooting:

```python
executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,  # Enable debug output
    handle_parsing_errors=True
)
```

This documentation provides comprehensive guidance for implementing AI agents in your AutoPy2 automations, from simple conversational agents to complex multi-tool orchestration systems.
