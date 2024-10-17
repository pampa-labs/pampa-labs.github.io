# Exploring OpenAI's Swarm: A New Frontier in Agent Orchestration

After the public release of OpenAI's Swarm framework, I decided to dive in and explore its potential. In this post, I'll share my experience migrating a real-world application from LangGraph to Swarm, and discuss the opportunities this new framework presents.

## From LangGraph to Swarm: A 1-Hour Migration

To test and understand how Swarm works, I adapted the core of an internal app that we use at Pampa to manage our daily tasks, such as tracking expenses, ordering food, and booking meetings, among other features.

Our current solution is implemented using LangGraph, and I was pleasantly surprised to find that migrating to Swarm took less than one hour.

Swarm's focus on ergonomics and simplicity made the transition relatively smooth. Here's a quick overview of the key changes I made:

### 1. Main Agent Definition

In Swarm, agents are defined with their own instructions and available functions. This structure is similar to our previous implementation, but with some syntactic differences. We managed to reduce our implementation from 89 lines to just 36 lines, avoiding the need to define the Graph and States (nodes, edges, entry point, etc.).

Here's a snippet of how I defined an agent in Swarm:

```python
from swarm import Agent, Swarm

AGENT_SYSTEM_PROMPT = """
# Task
Tu nombre es Gabriela y sos un asistente de IA diseñado para ayudar al equipo de Pampa Labs.
                      
# Guidelines                   
Tus respuestas deben ser:

1. Amigables y accesibles, usando un tono cálido
2. Concisas y al grano, evitando verbosidad innecesaria
3. Útiles e informativas, proporcionando información precisa
4. Respetuosas de la privacidad del usuario y los límites éticos

Solo puedes ayudar usando las herramientas disponibles y con pedidos que vengan de miembros del equipo. Todo lo que no se pueda responder usando las herramientas, debes decir que no puedes ayudar y disculparte.
"""

class GabyAgent:
    def __init__(
        self,
        prompt=AGENT_SYSTEM_PROMPT,
        model_name: str = "gpt-4o",
        tools: List[Callable] = [],
    ):
        self.agent = Agent(
            name="Main Agent",
            instructions=prompt,
            functions=tools,
            model=model_name
        )
        self.client = Swarm()
    
    def invoke(self, id, messages, debug=False):
        response = self.client.run(
            agent=self.agent,
            messages=messages,
            debug=debug,
            context_variables={"id": id}
        )
        return response.messages[-1]["content"]
```

### 2. Tools and Functions

Swarm's approach to defining functions is simpler and more flexible than LangGraph's, relying mainly on LLM function calls. Since Pydantic models couldn't be used for function parameters as in our current solution, I adapted the tools to use docstrings for OpenAI API calls and passed the arguments explicitly to the function.

```python
def set_meal_tool(context_variables, meal: str, date: str):
    """
    Sets the meal plan for a specific date.

    Args:
        meal (str): The name of the meal.
        date (str): The date of the meal plan.
    """
    return f"Meal plan created and sent to the provider: {meal} for {date} by team member {context_variables['id']}"
```

As you can notice, the function simply returns a string or a type that has a **str** representation, without needing to manage complex messages—everything happens under the hood.

```python
def get_expenses_tool(context_variables):
    """
    Retrieves all pending expenses.
    """
    # Mock implementation for testing purposes
    mock_expenses = {
        "user1": {
            'expenses': [
                {
                    "expense_type": "food",
                    "date": "2023-05-01",
                    "total_value": 50.0,
                    "state": "pending"
                },
                {
                    "expense_type": "coffee",
                    "date": "2023-05-02",
                    "total_value": 5.0,
                    "state": "pending"
                }
            ]
        }
    }
    # Functions called by an Agent should return a type that has __str__
    return f"Expenses retrieved: {mock_expenses[context_variables['id']]['expenses']}"
```
Here's an example of how to use these tools with the GabyAgent:

```python
gaby_agent = GabyAgent(tools=[set_meal_tool, get_expenses_tool])
response_content = gaby_agent.invoke(
    id="user1",
    messages=[
        {"role": "user", "content": "I want to set the meal plan for the date 2024-05-01. The meal is lasagna."},
        {"role": "user", "content": "I want to get the expenses"}
    ],
    debug=False
)
```

Output:
¡Listo! He establecido el plan de comidas para el 1 de mayo de 2024 con lasaña. Además, aquí tienes tus gastos pendientes:

1. **Comida**: $50.00 (Fecha: 2023-05-01)
2. **Café**: $5.00 (Fecha: 2023-05-02)

Si necesitas algo más, no dudes en avisarme. 😊

### 3. Context Variables for State Management

Context variables enable state management across agent interactions, allowing us to pass configuration variables like `id` for use throughout the interactions:

```python
response = self.client.run(
    agent=self.agent,
    messages=messages,
    debug=debug,
    context_variables={"id": id}
)
```

Currently, we are using checkpoints to save memory between calls in LangGraph. In Swarm, I couldn't find a built-in memory management feature, so if we needed to manage it, we would have to implement our own solution to maintain conversation history.

### 4. Streaming with Swarm

Swarm supports streaming responses, which can be particularly useful for providing a more responsive user experience. Here's an example of how to use streaming with Swarm:

```python
def stream(self, id, messages, debug=False):
    stream = self.client.run(
        agent=self.agent,
        messages=messages,
        debug=debug,
        context_variables={"id": id},
        stream=True
    )
    for chunk in stream:
        yield chunk
```

### 5. Handoffs Between Agents

I was pleasantly surprised by how quickly I could iterate and perform handoffs between agents. The process for defining handoffs is quite similar to Crewai's approach.

In our use case, I was able to efficiently split tasks across different agents to divide responsibilities, such as creating a meal agent and an expenses agent. I didn't need to modify the tools—only define the transfer functions.

```python
meal_agent: Agent = Agent(
    name="Meal Agent",
    instructions="""
    Eres un experto en planificar pedidos de comidas para el Equipo, tus respuesta deben ser claras, directas y en argentino.
    """,
    functions=[set_meal_tool],
    model="gpt-4o"
)

def transfer_to_meal_agent() -> Agent:
    """
    Transfer control to the Meal Agent for handling meal-related tasks.
    """
    return meal_agent

expenses_agent: Agent = Agent(
    name="Expenses Agent",  
    instructions="""
    Eres un experto en gestionar los gastos del Equipo. Tus respuestas deben ser en argentino.
    """,
    functions=[get_expenses_tool],
    model="gpt-4o"
)

def transfer_to_expenses_agent() -> Agent:
    """
    Transfer control to the Expenses Agent for handling expenses-related tasks.
    """
    return expenses_agent

gaby_agent = GabyAgent(tools=[transfer_to_meal_agent, transfer_to_expenses_agent])
```

## Potential and Focus of Swarm

Swarm, as an experimental framework, shows great promise with its focused approach to agent orchestration:

### 1. Simplicity and Core Focus

Swarm's design prioritizes ease of use, quick prototyping, and focuses on core functionalities, allowing developers to build custom solutions without unnecessary complexity, making it lightweight and easy to adapt.

### 2. Flexibility

The framework allows for rapid definition and orchestration of multiple agents with different roles and capabilities.

### 3. Experimental Nature

As an educational resource, Swarm explores new patterns in multi-agent systems, encouraging developers to learn about and experiment with agent orchestration.

### 4. OpenAI Integration

Swarm is designed to work seamlessly with OpenAI's models and APIs, making it an attractive option for developers already working within the OpenAI ecosystem.

## Conclusion

OpenAI's Swarm offers a promising approach to agent orchestration with a focus on simplicity and flexibility. It is ideal for prototyping and building multi-agent systems, with potential for integration with OpenAI's models.

As Swarm evolves, it will be interesting to see how it addresses limitations and expands capabilities. For now, it provides a fresh perspective on agent design, especially for those already working with OpenAI's ecosystem.

Have you tried experimenting with Swarm or a similar framework?

