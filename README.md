# Remote-Agents-with-A2A-Protocol

## Project Goal
To build a decentralized multi-agent system where independent agents running on different servers can discover and communicate with each other using the Agent-to-Agent (A2A) protocol.

## What I Built
I moved beyond monolithic agent applications and designed a remote multi-agent solution. Instead of having all agents running in a single process, I deployed them as distinct services that communicate over HTTP.

### System Components
1. **The Title Agent**: A specialized worker that generates catchy blog headlines.
2. **The Outline Agent**: A worker that takes a title and produces a structured article outline.
3. **The Routing Agent**: The orchestrator that discovers these remote agents and routes user requests to the correct service.

## How I Implemented It
I utilized the Azure AI Agent Service alongside the A2A (Agent-to-Agent) Python SDK to implement the discovery and communication layers.

### Tech Stack & Architecture
- **Protocol**: A2A (Agent-to-Agent)
- **Discovery Mechanism**: AgentCard and AgentSkill
- **Communication**: HTTP via `send_message`

### Code Logic
The core challenge was making the agents "discoverable." To solve this, I defined an `AgentCard` in the server code. This acts as a digital business card, broadcasting the agent's URL, version, and specific skills (like `generate_blog_title`).

For the Routing Agent to talk to the Title Agent, I implemented a client that constructs a JSON payload and sends it asynchronously:

```python
# In the Routing Agent, I construct the payload for the remote worker
payload = {
    'message': {
        'role': 'user',
        'parts': [{'kind': 'text', 'text': task}],
        # ...additional fields...
    }
}
# Then I send it over HTTP and await the response
send_response = await client.send_message(message_request=message_request)
```

To handle the incoming request on the other side, I wrapped the Title Agent in an `AgentExecutor` class. This wrapper manages the task state, processing the request and updating the status until the job is complete.

## Project Structure
The project is structured as a distributed application, with separate directories for each agent server and a central runner script:

```
ai-agents/
└── Labfiles/
    └── 06-build-remote-agents-with-a2a/
        └── python/
            ├── title_agent/       # The Headline Generator
            │   ├── agent.py       # Core logic
            │   ├── agent_executor.py # A2A Task Handler
            │   └── server.py      # Server hosting the AgentCard
            ├── routing_agent/     # The Orchestrator
            ├── client.py          # User Interface
            └── run_all.py         # Utility to launch the swarm
```

## How to Run My Code
To spin up this distributed system, follow these steps:

1. **Clone & Navigate**: Access the A2A specific module in the repository:

    ```bash
    cd ai-agents/Labfiles/06-build-remote-agents-with-a2a/python
    ```

2. **Install Dependencies**: Install the specific A2A SDK along with the Azure AI libraries:

    ```bash
    pip install -r requirements.txt azure-ai-projects azure-ai-agents a2a-sdk
    ```

3. **Launch the Swarm**: Use the utility script to launch all servers and the client simultaneously:

    ```bash
    python run_all.py
    ```

4. **Test It**: Once the servers are active, send a prompt to the Routing Agent:

    ```
    "Create a title and outline for an article about React programming."
    ```

### Outcome
The Routing Agent successfully discovered the remote Title Agent, dispatched the task, and returned a cohesive response containing both a headline and an outline.
