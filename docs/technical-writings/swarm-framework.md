# OpenAI's Swarm: The Power of Right Abstractions to Build Multi-Agent Systems

After OpenAI announced the public release of Swarm, a library designed to build reliable multi-agent systems, I decided to dive in and explore its potential by migrating a real-world application from LangGraph to Swarm.

## From LangGraph to Swarm: A 1-Hour Migration

At Pampa, we developed and use a productivity app to streamline our daily operations. This app, powered by AI agents and tools, efficiently manages tasks like expense tracking, meal ordering, and meeting scheduling. When OpenAI released Swarm, I saw the opportunity to migrate the core of our solution and discover how well the new library would fit.

I was pleasantly surprised to find that migrating our current solution, implemented using LangGraph, took less than one hour.

Here's a quick overview of the key changes I made:

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
```
Output:
¡Listo! He establecido el plan de comidas para el 1 de mayo de 2024 con lasaña. Además, aquí tienes tus gastos pendientes:

1. **Comida**: $50.00 (Fecha: 2023-05-01)
2. **Café**: $5.00 (Fecha: 2023-05-02)

Si necesitas algo más, no dudes en avisarme. 😊
```
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

### 4. Tracking Requests

As expected of an experimental library, Swarm offers deep debugging logging that helps in understanding agent behavior and interactions, but it lacks built-in tracking or monitoring capabilities.

To track requests by default, our solution uses Langsmith, which provides a detailed structure of the nodes and the actions taken at each step. During this migration, we lost that feature and had to re-implement it using decorators.

### 5. Streaming with Swarm

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

### 6. Handoffs Between Agents

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

## Potential of Swarm

Swarm, as an experimental library, shows great promise with its approach to agent orchestration:

1. Simplicity and Core Focus

The design prioritizes ease of use and quick prototyping, focusing on core functionalities that allow developers to build custom solutions without unnecessary complexity. This makes it lightweight and easy to adapt.

2. Flexibility

It allows for the rapid definition and orchestration of multiple agents with different roles and capabilities.

3. Experimental Nature

As an educational resource, it explores new patterns in multi-agent systems, encouraging developers to learn and experiment with agent orchestration.

4. OpenAI Integration

Designed to integrate seamlessly with OpenAI's models and APIs, it provides a straightforward way for developers to leverage these tools.

## Notebook

For a hands-on experience with the concepts discussed in this article, check out the accompanying Jupyter notebook:

[Swarm Framework Example Notebook](https://github.com/pampa-labs/notebooks/blob/main/swarm_framework_example.ipynb)

This notebook provides a step-by-step guide to implementing a multi-agent system using OpenAI's Swarm framework, demonstrating the concepts and code snippets discussed in this article.

## Conclusion

OpenAI's Swarm stands out as a library developed with well-designed abstractions to build reliable multi-agent systems. It enables developers to create scalable and robust solutions while avoiding unnecessary complexity.

Have you experimented with Swarm or similar frameworks? Share your experiences in the comments below.