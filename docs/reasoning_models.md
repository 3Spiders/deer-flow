# Reasoning Models Support

This document describes the implementation of reasoning models support in DeerFlow, specifically for the planner_node and reporter_node.

## Overview

Reasoning models like DeepSeek R1 and OpenAI O1-thinking are designed to perform step-by-step reasoning, which can improve the quality of plans and reports. These models are particularly good at:

1. Breaking down complex problems into manageable steps
2. Exploring multiple perspectives before making decisions
3. Providing detailed explanations of their thought process
4. Generating more coherent and comprehensive outputs

## Implementation Details

### Configuration

To use reasoning models, add a `REASONING_MODEL` configuration to your `conf.yaml` file:

```yaml
# DeepSeek R1 Example
REASONING_MODEL:
  base_url: "https://api.deepseek.com"
  model: "deepseek-reasoner"
  api_key: YOUR_API_KEY
  temperature: 0.1
  max_tokens: 4000

# OpenAI O1 Example
REASONING_MODEL:
  model: "o1"
  api_key: YOUR_API_KEY
  temperature: 0.1
  max_tokens: 4000
```

A complete example configuration is available in `conf.reasoning_models.yaml.example`.

### Agent-LLM Mapping

The `AGENT_LLM_MAP` in `src/config/agents.py` has been updated to use reasoning models for the planner_node and reporter_node:

```python
AGENT_LLM_MAP: dict[str, LLMType] = {
    "coordinator": "basic",
    "planner": "reasoning",  # Changed from basic to reasoning
    "researcher": "basic",
    "coder": "basic",
    "reporter": "reasoning",  # Changed from basic to reasoning
    "podcast_script_writer": "basic",
    "ppt_composer": "basic",
    "prose_writer": "basic",
}
```

### Planner Node

The planner_node has been enhanced to better utilize reasoning models:

1. **Detailed Reasoning Instructions**: The model is guided through a comprehensive reasoning process:
   ```
   "Please think step by step to create a comprehensive research plan. 
   First, analyze the task carefully to understand what information is needed. 
   Then, break down the research into logical steps, considering dependencies between steps. 
   For each step, determine whether web search is needed and what specific information to collect. 
   Classify each step as either 'research' (gathering information) or 'processing' (analyzing information)."
   ```

2. **Structured Output Format**: Clear instructions for JSON formatting with triple backticks:
   ```
   "After your reasoning, format your response as a valid JSON object following this structure:
   ```json
   {
     "locale": "en-US",
     "has_enough_context": true/false,
     "thought": "Your detailed reasoning about the plan",
     "title": "Title of the research plan",
     "steps": [
       {
         "need_web_search": true/false,
         "title": "Step title",
         "description": "Detailed description of what to research",
         "step_type": "research" or "processing"
       }
     ]
   }
   ```"
   ```

3. **Robust JSON Extraction**: Enhanced regex patterns to extract JSON from different output formats:
   - First tries to find JSON between ```json``` markers
   - Then looks for JSON between any triple backticks
   - Then searches for JSON object with curly braces containing "steps"
   - Includes JSON cleanup to handle trailing commas and other common issues

### Reporter Node

The reporter_node has been enhanced with a structured reasoning approach:

1. **Step-by-Step Analysis Instructions**:
   ```
   "1. First, carefully analyze all the observations and research findings.
      - What are the key facts and insights from the research?
      - Are there any contradictions or gaps in the information?
      - What are the most significant patterns or trends?

   2. Then, organize the information into a coherent structure:
      - Group related information into logical sections
      - Determine the most effective sequence for presenting information
      - Identify where tables would be useful for comparing data"
   ```

2. **Structured Report Format Guidance**:
   ```
   "4. Finally, write a well-structured report following the format specified above, with:
      - Clear, concise key points at the beginning
      - A logical flow from introduction to detailed analysis
      - Effective use of markdown tables for comparative data
      - Proper citations in the Key Citations section"
   ```

3. **Streaming for Long Outputs**: Implemented streaming to handle the potentially longer and more detailed outputs from reasoning models.

## Model-Specific Considerations

### DeepSeek R1

DeepSeek R1 (deepseek-reasoner) is specifically designed for reasoning tasks and offers:

- Strong step-by-step reasoning capabilities
- Good performance on complex planning tasks
- Ability to handle structured output formats
- Consistent JSON formatting when properly prompted

Configuration notes:
- Requires `base_url: "https://api.deepseek.com"`
- Uses the OpenAI-compatible API format
- Works best with temperature between 0.0-0.2 for structured outputs

### OpenAI O1

OpenAI O1 offers:

- Excellent reasoning capabilities with detailed thought processes
- Strong ability to follow complex instructions
- Good performance on structured output tasks
- Reliable JSON formatting

Configuration notes:
- Uses standard OpenAI API format
- Works well with temperature between 0.0-0.2 for structured outputs
- May require higher max_tokens settings for complex reasoning tasks

## Benefits

Using reasoning models for planning and reporting provides several benefits:

1. **More comprehensive plans**: Reasoning models generate more detailed and well-thought-out research plans
2. **Better analysis**: The step-by-step reasoning process leads to more insightful analysis in reports
3. **Improved coherence**: Reports are more coherent and logically structured
4. **Enhanced connections**: Reasoning models are better at making connections between different pieces of information
5. **Higher quality outputs**: The overall quality of plans and reports is significantly improved

## Limitations

There are some limitations to be aware of:

1. **Longer response times**: Reasoning models may take longer to generate responses
2. **Higher token usage**: The step-by-step reasoning process uses more tokens
3. **JSON extraction challenges**: Extracting structured data from reasoning outputs can be more complex
4. **API costs**: Reasoning models typically cost more per token than basic models
5. **API availability**: Some reasoning models may have limited availability or require waitlist approval

## Troubleshooting

Common issues and solutions:

1. **JSON parsing errors**:
   - Check the model's response format in the logs
   - Adjust the JSON extraction regex patterns if needed
   - Consider modifying the prompt to emphasize proper JSON formatting

2. **Incomplete reasoning**:
   - Increase the max_tokens parameter in the model configuration
   - Adjust the temperature to a lower value (0.0-0.1) for more deterministic outputs

3. **API errors**:
   - Verify API keys are correct and have sufficient credits
   - Check that the model names are specified correctly
   - Ensure you have access to the specified models

## Future Improvements

Potential future improvements include:

1. Adding support for more reasoning models (Claude 3 Opus, Gemini 1.5 Pro, etc.)
2. Implementing more sophisticated JSON extraction techniques
3. Optimizing prompts for different reasoning models
4. Adding a toggle to switch between basic and reasoning modes
5. Implementing fallback mechanisms when reasoning models are unavailable
6. Adding model-specific prompt templates for optimal performance