<div align="center">
  <img src="static/logo.png" alt="PhiloAgents logo" width="200" />
  <h1>PhiloAgents</h1>
  <p>An agentic RAG game where historical thinkers become interactive AI characters.</p>

  <p>
    <img src="https://img.shields.io/badge/Python-3.11+-blue?logo=python&logoColor=white" alt="Python" />
    <img src="https://img.shields.io/badge/LangGraph-Agent_Orchestration-orange?logo=langchain&logoColor=white" alt="LangGraph" />
    <img src="https://img.shields.io/badge/FastAPI-WebSockets-009688?logo=fastapi&logoColor=white" alt="FastAPI" />
    <img src="https://img.shields.io/badge/MongoDB-Atlas_Vector_Search-47A248?logo=mongodb&logoColor=white" alt="MongoDB" />
    <img src="https://img.shields.io/badge/Groq-LLM_Inference-F55036?logo=groq&logoColor=white" alt="Groq" />
    <img src="https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white" alt="Docker" />
    <img src="https://img.shields.io/badge/License-MIT-green" alt="MIT license" />
  </p>
</div>

<p align="center">
  <img src="static/game_socrates_example.png" alt="A conversation with Socrates in the PhiloAgents game" width="700" />
</p>

## Overview

PhiloAgents is an interactive pixel-art game. Explore a small town, approach a philosopher, and start a conversation about consciousness, ethics, language, creativity, power, or technology.

Each character is powered by an agentic retrieval-augmented generation (RAG) workflow. The agent can answer from its character configuration, retrieve relevant source material from MongoDB Atlas, summarize that context, and stream an in-character response back to the game.

The project is a fork of the [PhiloAgents Course](https://github.com/neural-maze/philoagents-course) by [Paul Iusztin](https://github.com/iusztinpaul) and [Miguel Otero Pedrido](https://github.com/MichaelisTrofficus), created in collaboration with MongoDB, Opik, and Groq.

## What I changed

I extended the original course project from its initial roster into a broader conversation about the relationship between philosophy, literature, power, and artificial intelligence. The changes are implemented across the domain model, agent configuration, game world, and frontend assets.

### 1. Added four new philosopher agents

The roster now includes:

| Character | Lens brought to conversations |
| --- | --- |
| **Niccolò Machiavelli** | Power, political strategy, governance, and the practical use of AI |
| **Fyodor Dostoevsky** | Moral responsibility, suffering, guilt, faith, and the possibility of machine consciousness |
| **Franz Kafka** | Alienation, bureaucracy, incomprehensible systems, and technological dehumanization |
| **Friedrich Nietzsche** | Will to power, creativity, nihilism, self-overcoming, and the creation of new values |

These are not only display names. Each new character is registered in philoagents-api/src/philoagents/domain/philosopher_factory.py with three pieces of domain configuration:

- a stable identifier and display name;
- a distinct conversational style, such as Machiavelli's direct political realism or Kafka's anxious precision;
- a distinct perspective on AI, so the same user question can produce meaningfully different answers.

Because the characters use the same factory and conversation workflow as the original roster, they automatically participate in the existing API, prompt-card, retrieval, and memory systems.

### 2. Connected the new characters to the game

I added the four philosophers to the Phaser game scene and gave each one a complete in-world configuration:

- a name and character identifier shared with the backend;
- a starting position or fallback spawn position;
- a default facing direction and roaming radius;
- collision handling and the same interaction flow as the original characters.

I also added sprite atlases for Machiavelli, Dostoevsky, Kafka, and Nietzsche, including the directional and walking animation frames required by the Character class. The preloader now loads these assets so the characters appear as fully animated NPCs rather than static entries in the backend.

### 3. Made the source material pipeline work for the expanded roster

The long-term-memory command resolves every configured philosopher through the factory, retrieves source material from Wikipedia and the Stanford Encyclopedia of Philosophy, splits and deduplicates the documents, embeds them, and writes them to the MongoDB vector store with philosopher metadata.

That means the four additions are part of the same knowledge workflow as the original characters. Running make create-long-term-memory rebuilds the collection and makes their source context available to the retriever; it is not necessary to add a separate retrieval path for each philosopher.

### 4. Kept the additions compatible with the existing architecture

The new characters use the existing:

- LangGraph conversation graph and tool-calling retrieval loop;
- rolling conversation summaries and MongoDB checkpoints;
- FastAPI /chat endpoint and streaming /ws/chat endpoint;
- Opik prompt and trace instrumentation;
- Docker Compose development environment.

This keeps the change focused: the roster and game world are larger, while the core agent and infrastructure remain reusable.

## Characters

The game currently includes 14 thinkers from classical philosophy, literature, mathematics, and computer science:

| Character | Known for |
| --- | --- |
| Socrates | Socratic questioning and ethical inquiry |
| Plato | The Forms and the allegory of the Cave |
| Aristotle | Logic, categorization, and purpose |
| René Descartes | Methodological doubt and the cogito |
| Gottfried Wilhelm Leibniz | Calculus, logic, and universal computation |
| Ada Lovelace | Computing, creativity, and imagination |
| Alan Turing | Computation and the Turing Test |
| Noam Chomsky | Language, cognition, and AI skepticism |
| John Searle | The Chinese Room and intentionality |
| Daniel Dennett | Consciousness as an emergent process |
| Niccolò Machiavelli | Power and political realism |
| Fyodor Dostoevsky | Moral psychology and suffering |
| Franz Kafka | Alienation and bureaucratic absurdity |
| Friedrich Nietzsche | The will to power and revaluation of values |

## How it works

<p align="center">
  <img src="static/system_architecture.png" alt="PhiloAgents system architecture" width="650" />
</p>

### 1. Knowledge ingestion

The data pipeline loads philosopher-related material from Wikipedia and the Stanford Encyclopedia of Philosophy. Documents are cleaned, split into chunks, deduplicated, embedded with sentence-transformers/all-MiniLM-L6-v2, and indexed in MongoDB Atlas for hybrid vector and full-text search.

### 2. Agentic conversation

The LangGraph workflow coordinates the response:

1. The conversation node builds an in-character response from the philosopher's prompt card and conversation state.
2. The model can call the retriever when the question needs supporting context.
3. Retrieved documents are condensed before being passed back into the conversation.
4. The response is returned through the API, either as a complete message or as streamed WebSocket chunks.
5. Long conversations are summarized and older messages are pruned to keep the context window bounded.

The agent decides when retrieval is useful instead of retrieving on every turn. Short-term conversation state and long-term philosopher knowledge are kept as separate memory layers.

### 3. Observability and evaluation

Opik instruments LLM calls, prompts, and evaluation runs. The repository also includes commands for generating an evaluation dataset and running an LLM-as-a-judge evaluation workflow.

## Tech stack

| Layer | Technology |
| --- | --- |
| Agent orchestration | LangGraph, LangChain |
| LLM inference | Groq, with Llama models configured through environment variables |
| Long-term memory | MongoDB Atlas local development image with vector search |
| Embeddings | sentence-transformers/all-MiniLM-L6-v2 |
| API | FastAPI and WebSockets |
| Game UI | Phaser 3 and JavaScript |
| Observability | Opik |
| Infrastructure | Docker Compose |
| Python tooling | uv, Ruff, pytest |

## Repository layout

    .
    ├── philoagents-api/
    │   ├── src/philoagents/
    │   │   ├── application/       # Conversation, RAG, ingestion, and evaluation use cases
    │   │   ├── domain/            # Philosopher model, factory, perspectives, and prompts
    │   │   └── infrastructure/    # FastAPI, MongoDB, and Opik integrations
    │   ├── tools/                 # Memory and evaluation command-line tools
    │   └── tests/
    ├── philoagents-ui/            # Phaser game, scenes, dialogue, and assets
    ├── static/                    # README screenshots, logo, and architecture diagrams
    ├── docker-compose.yml         # MongoDB, API, and UI services
    ├── Makefile                   # Common development and pipeline commands
    └── PROJECT_OVERVIEW.md        # Additional project notes

## Getting started

### Prerequisites

- [Docker](https://www.docker.com/) and Docker Compose
- A [Groq API key](https://console.groq.com/)
- An OpenAI API key if you want to run the evaluation workflow
- An Opik/Comet API key if you want hosted tracing and prompt versioning

### Installation

    git clone https://github.com/shrikantnaidu/philoagents.git
    cd philoagents

    cp philoagents-api/.env.example philoagents-api/.env
    # Edit philoagents-api/.env and set GROQ_API_KEY.

Start the services:

    make infrastructure-up

Build the long-term memory collection before starting a conversation:

    make create-long-term-memory

Then open [http://localhost:8080](http://localhost:8080). Use the arrow keys to explore, approach a philosopher, and press Space to interact.

### Useful commands

| Command | Purpose |
| --- | --- |
| make infrastructure-build | Build the API and UI images |
| make infrastructure-up | Build and start all services in the background |
| make infrastructure-stop | Stop the running services |
| make call-agent | Test an agent from the command line |
| make create-long-term-memory | Extract, embed, and index philosopher knowledge |
| make delete-long-term-memory | Clear the long-term-memory collection |
| make generate-evaluation-dataset | Generate synthetic evaluation conversations |
| make evaluate-agent | Run the LLM-as-a-judge evaluation |

The Makefile expects philoagents-api/.env to exist. The API and UI can also be developed independently; see [philoagents-api/README.md](philoagents-api/README.md) and [philoagents-ui/README.md](philoagents-ui/README.md).

## Attribution

This project builds on the [PhiloAgents Course](https://github.com/neural-maze/philoagents-course) by [Paul Iusztin](https://github.com/iusztinpaul) and [Miguel Otero Pedrido](https://github.com/MichaelisTrofficus). The original project was developed in collaboration with MongoDB, Opik, and Groq.

The additional philosopher configurations, game characters, sprite atlases, and integration work described in [What I changed](#what-i-changed) were added in this fork.

## License

This project is licensed under the MIT License.
