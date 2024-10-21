# OpenAI's Swarm: The Power of Right Abstractions to Build Multi-Agent Systems

After OpenAI announced the public release of Swarm, a library designed to build reliable multi-agent systems, I decided to dive in and explore its potential by integrating it into a real-world application.

## Swarm: A Simple Multi-Agent Solution

I adapted our productivity app at Pampa, which uses a combination of AI agents and tools to streamline various tasks such as expense tracking, meal ordering, and meeting scheduling, all in less than one hour.

Here's a quick overview of the key aspects of the implementation.

### 1. Main Agent Definition

The agent definition in Swarm is conceptually similar to our current solution, but with some key syntactic differences. I was impressed by how concise and expressive the implementation turned out to be - just 36 lines of code. This brevity demonstrates the power of Swarm's well-designed abstractions for defining multi-agent architectures.

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

### 2. Tools

Swarm's approach to defining functions is simple and flexible, relying mainly on LLM function calls.

Since Pydantic models couldn't be used for function parameters, I adapted the tools to use docstrings for OpenAI API calls and passed the arguments explicitly to the function.

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

As you can see, the function simply returns a string or a type that has a **str** representation, without needing to manage complex messages—everything happens under the hood.

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

```¡Listo!
1. **Comida**: $50.00 (Fecha: 2023-05-01)
2. **Café**: $5.00 (Fecha: 2023-05-02)

Si necesitas algo más, no dudes en avisarme. 😊
```

### 3. Context Variables

Context variables enable state management across agent interactions, allowing us to pass configuration variables like `id` for use throughout the interactions:

```python
response = self.client.run(
    agent=self.agent,
    messages=messages,
    debug=debug,
    context_variables={"id": id}
)
```

In Swarm, I couldn't find a built-in memory management feature, so if we needed to manage it, we would have to implement our own solution to maintain conversation history.

### 4. Tracking 

As expected of an experimental library, it offers deep debugging logging that helps in understanding agent behavior and interactions. However, it lacks built-in tracking or monitoring capabilities.

To track requests by default, our solution uses Langsmith, which provides a detailed structure of the nodes and the actions taken at each step. I had to implement this tracking manually using decorators.

### 5. Streaming

The library also supports streaming responses, which can be particularly useful for providing a more responsive user experience.

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

### 6. Handoffs between Agents

I surprised by how quickly I could iterate and perform handoffs between agents. The process for defining handoffs is quite similar to Crewai's approach.

In our use case, I was able to efficiently split tasks across different agents to divide responsibilities, such as creating a meal agent and an expenses agent.

Here's a diagram illustrating the structure of our multi-agent system using Swarm:

![Swarm Multi-Agent System](../assets/swarm-multi-agent-system.png)

As illustrated in the diagram, the beauty of this approach lies in its simplicity. I didn't need to modify the existing tools at all. Instead, I only had to define the transfer functions, which act as bridges between the main agent and the specialized agents. This elegant design allows for seamless task delegation and efficient collaboration between agents, all while maintaining the integrity of the individual tools and their functionalities.

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

Swarm shows great promise with its approach to agent orchestration:

- **Simplicity and Core Focus**: The design prioritizes ease of use and quick prototyping, focusing on core functionalities, allowing developers to build custom solutions without unnecessary complexity. This makes it lightweight and easy to adapt.

- **Flexibility**: It allows for the rapid definition and orchestration of multiple agents with different roles and capabilities.

- **Experimental Nature**: As an educational resource, it explores new patterns in multi-agent systems, encouraging developers to learn and experiment with agent orchestration.

- **OpenAI Integration**: Designed to integrate seamlessly with OpenAI's models and APIs, it provides a straightforward way for developers to leverage these tools.

## Notebook

For a hands-on experience with the concepts discussed in this article, check out the accompanying Jupyter notebook:

[Swarm Framework Example Notebook](https://github.com/lgesuellip/swarm-example/blob/main/gaby_agent_swarm.ipynb)

This notebook provides a step-by-step guide to implementing a multi-agent system using OpenAI's Swarm framework, demonstrating the concepts and code snippets discussed in this article.

## Conclusion

OpenAI's Swarm stands out as a library developed with well-designed abstractions to build reliable multi-agent systems.

Have you experimented with Swarm or similar frameworks? Share your experiences in the comments below.

