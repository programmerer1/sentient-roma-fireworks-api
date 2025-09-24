# sentient-roma-fireworks-api

**This is an experimental guide for integrating the Fireworks API into https://github.com/sentient-agi/ROMA**

1. Step One. Add the Fireworks API key to .env:

```bash
FIREWORKS_AI_API_KEY=your_fireworks_ai_api_key_here
```

2. Step Two. Make changes in the sentient.yaml file:

```bash
llm:
  provider: "fireworks_ai"
  api_key: "${FIREWORKS_AI_API_KEY}"
```

3. Step Three. Open the file **ROMA/src/sentientresearchagent/config/config.py** and find the following line:

```bash
valid_providers = ['openai', 'anthropic', 'azure', 'custom', 'openrouter']
```

Add fireworks_ai to this array as a valid provider:

```bash
valid_providers = ['openai', 'anthropic', 'azure', 'custom', 'openrouter', 'fireworks_ai']
```

4. Step Four. In the file **ROMA/src/sentientresearchagent/hierarchical_agent_framework/agent_configs/agents.yaml** replace all models with those supported by FIREWORKS. Make sure that model_id always starts with fireworks_ai, and the provider must be litellm. Here’s an example of the correct configuration:
```bash
  - name: "BusinessIdeaGenerator"
    type: "executor"
    adapter_class: "ExecutorAdapter"
    description: "Writes detailed business idea"
    model:
      provider: "litellm"
      model_id: "fireworks_ai/accounts/sentientfoundation/models/dobby-unhinged-llama-3-3-70b-new"
    prompt_source: "prompts.executor_prompts.BUSINESS_IDEA_GENERATOR_EXECUTOR_SYSTEM_MESSAGE"
    registration:
      action_keys:
        - action_verb: "execute"
          task_type: "SEARCH"
      named_keys: ["BusinessIdeaGenerator"]
    enabled: true
```

**For clarity, I have also uploaded agents.yaml to this repository.**
**Another important note**. Some core agents send additional parameters or have hardcoded model names or API URLs in their source code, which can break our requests. Therefore, we need to disable them.

OpenAICustomSearcher, GeminiCustomSearcher, and ExaComprehensiveSearcher do not allow the use of FIREWORKS, so we disable them by setting enabled: false in the file **ROMA/src/sentientresearchagent/hierarchical_agent_framework/agent_configs/agents.yaml**:
```bash
  - name: "OpenAICustomSearcher"
    type: "custom_search"
    adapter_class: "OpenAICustomSearchAdapter"
    description: "Direct API-based search using OpenAI's search capabilities"
    adapter_params:
      # model_id: "gpt-4o"
      search_context_size: "high"  # Options: "medium", "high", "low" (works with both OpenAI and OpenRouter)
      # Optional: use OpenRouter instead of OpenAI API (requires OPENROUTER_API_KEY)
      use_openrouter: true
      model_id: "fireworks_ai/accounts/fireworks/models/gpt-oss-120b"  # When using OpenRouter
    registration:
      action_keys:
        - action_verb: "execute"
          task_type: "SEARCH"
      named_keys: ["OpenAICustomSearcher", "default_openai_searcher"]
    enabled: false

  - name: "GeminiCustomSearcher"
    type: "custom_search"
    adapter_class: "GeminiCustomSearchAdapter"
    description: "Direct API-based search using Google Gemini's search capabilities"
    adapter_params:
      model_id: "fireworks_ai/accounts/fireworks/models/llama4-maverick-instruct-basic"
    registration:
      action_keys:
        - action_verb: "execute"
          task_type: "SEARCH"
      named_keys: ["GeminiCustomSearcher", "default_gemini_searcher", "gemini_searcher"]
    enabled: false

  - name: "ExaComprehensiveSearcher"
    type: "custom_search"
    adapter_class: "ExaCustomSearchAdapter"
    description: "Comprehensive search using Exa API with LiteLLM processing - prioritizes reliable sources"
    adapter_params:
      model_id: "fireworks_ai/accounts/fireworks/models/llama4-maverick-instruct-basic"  # Can be any LiteLLM-supported model
      num_results: 5      # Number of Exa search results to retrieve
      # include_domains: ["en.wikipedia.org", "imdb.com"]
    registration:
      action_keys:
        - action_verb: "execute"
          task_type: "SEARCH"
      named_keys: ["ExaComprehensiveSearcher", "exa_searcher", "comprehensive_searcher"]
    enabled: false
```

The OpenAICustomSearcher agent is built into almost all profiles by default, so we need to go into the following files
**ROMA/src/sentientresearchagent/hierarchical_agent_framework/agent_configs/profiles/deep_research_agent.yaml
ROMA/src/sentientresearchagent/hierarchical_agent_framework/agent_configs/profiles/crypto_analytics_agent.yaml
ROMA/src/sentientresearchagent/hierarchical_agent_framework/agent_configs/profiles/general_agent.yaml
ROMA/src/sentientresearchagent/hierarchical_agent_framework/agent_configs/profiles/opensourcegeneralagent.yaml**

In each of these files, find the following line:

```bash
  executor_adapter_names:
    SEARCH: "OpenAICustomSearcher"
```

Replace the agent with this one, another suitable one, or custom (by the way, I recently shared a guide on how to add custom agent: https://github.com/programmerer1/sentient-roma-custom-agent-guide):

```bash
  executor_adapter_names:
    SEARCH: "SmartWebSearcher"
```

The BasicReasoningExecutor agent sends additional parameters that are not supported by Fireworks, so go to the file and disable these parameters by using the # sign:

```bash
  - name: "BasicReasoningExecutor"
    type: "executor"
    adapter_class: "ExecutorAdapter"
    description: "Performs Analysis and Synthesis of information"
    model:
      provider: "litellm"
      model_id: "fireworks_ai/accounts/fireworks/models/gpt-oss-120b"  # Switched from slow o3 model for faster execution
    # agno_params:
    #   reasoning: true
    prompt_source: "prompts.executor_prompts.REASONING_EXECUTOR_SYSTEM_MESSAGE"
    tools: 
    #  - name: "PythonTools"
    #    params:
    #      save_and_run: false  # Disable automatic execution of Python code
    #  - "ReasoningTools"
    registration:
      action_keys:
        - action_verb: "execute"
          task_type: "THINK"
      named_keys: ["BasicReasoningExecutor"]
    enabled: true
```

After applying the changes, restart the containers:

```bash
docker compose -f docker-compose.yml down
docker compose -f docker-compose.yml up -d
```

Finish. I hope this guide will be useful to you.
