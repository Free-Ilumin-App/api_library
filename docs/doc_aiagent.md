# Example 1
```python
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import tool

try:
    llm = ChatGoogleGenerativeAI(
        model="{{model}}",
        temperature={{temperature}},
        google_api_key="{{api_key}}"
    )
    
    tools = []
    
    if 'obter_localizacao_por_ip' in globals():
        @tool
        def get_location_by_ip(ip_address: str):
            """Obtém dados de localização (país, cidade, etc.) a partir de um endereço IP. Use esta ferramenta para obter a localização de um IP.
            Args:
                ip_address (str): O endereço IP para o qual se deseja obter os dados de localização. Ex: '24.48.0.1'
            """
            try:
                result = obter_localizacao_por_ip(ip_address=ip_address)
                return str(result) # Convertendo o resultado para string para o LLM
            except Exception as e:
                return f"Erro ao obter localização para {ip_address}: {e}"
        tools.append(get_location_by_ip)
    
    if tools:
        prompt = ChatPromptTemplate.from_messages([
            ("system", "{{system_prompt}}"),
            ("human", "{input}"),
            ("placeholder", "{agent_scratchpad}")
        ])
        
        agent = create_tool_calling_agent(llm, tools, prompt)
        executor = AgentExecutor(agent=agent, tools=tools)
        
        # Passando o IP obtido do passo anterior para o agente de IA
        user_input_with_ip = f"Localize o IP: {ip_address}"
        result = executor.invoke({"input": user_input_with_ip})
        print(result["output"])
    else:
        print("Nenhuma ferramenta disponível para obter a localização do IP.")
        
except Exception as e:
    print(f"Erro: {e}")
```
