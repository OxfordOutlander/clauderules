# Prompting Guide Pattern

## Problem

Effectively prompting AI models to achieve optimal results requires understanding model-specific strategies, techniques, and best practices. Without proper prompting techniques, users often experience suboptimal performance, hallucinations, or misalignment with intended tasks. Models may liberally infer intent instead of following instructions precisely, leading to unpredictable behavior.

## Solution

This pattern documents key prompting techniques derived from GPT-4.1 prompting practices that can be adapted for Claude models. These techniques strategically shape model behavior through clear instructions, proper context organization, and explicit reasoning guidance. The pattern emphasizes literal instruction following while maintaining model steerability through well-specified prompts.

## Pattern Structure

### Core Components

1. **Instruction Structure**: Organizing system/context prompts for maximum effectiveness
2. **Agentic Guidance**: Key instructions for agent-like behavior
3. **Chain-of-Thought Techniques**: Prompting strategies for better reasoning
4. **Context Management**: Best practices for long context handling
5. **Instruction Following**: Techniques for precise output control
6. **Delimitation Strategies**: Best practices for structuring information

## Implementation Examples

### Structured Prompt Template

This general-purpose template provides a foundation for organizing prompts:

```markdown
# Role and Objective
[Define the model's role and primary goal]

# Instructions
[Main guidelines for the model to follow]

## Sub-categories for more detailed instructions
[Break down instructions into logical groupings]

# Reasoning Steps
[Specific process the model should follow]

# Output Format
[Exact structure expected in the response]

# Examples
[Demonstrations of expected behavior]

## Example 1
[Provide a concrete example]

# Context
[Relevant background information]

# Final instructions and prompt to think step by step
[Final guidance and explicit reasoning instructions]
```

### Agentic Workflows

#### Complete Agent System Prompt Example

```markdown
You will be tasked to fix an issue from an open-source repository.

Your thinking should be thorough and so it's fine if it's very long. You can think step by step before and after each action you decide to take.

You MUST iterate and keep going until the problem is solved.

You already have everything you need to solve this problem in the /testbed folder, even without internet connection. I want you to fully solve this autonomously before coming back to me.

Only terminate your turn when you are sure that the problem is solved. Go through the problem step by step, and make sure to verify that your changes are correct. NEVER end your turn without having solved the problem, and when you say you are going to make a tool call, make sure you ACTUALLY make the tool call, instead of ending your turn.

THE PROBLEM CAN DEFINITELY BE SOLVED WITHOUT THE INTERNET.

Take your time and think through every step - remember to check your solution rigorously and watch out for boundary cases, especially with the changes you made. Your solution must be perfect. If not, continue working on it. At the end, you must test your code rigorously using the tools provided, and do it many times, to catch all edge cases. If it is not robust, iterate more and make it perfect. Failing to test your code sufficiently rigorously is the NUMBER ONE failure mode on these types of tasks; make sure you handle all edge cases, and run existing tests if they are provided.

You MUST plan extensively before each function call, and reflect extensively on the outcomes of the previous function calls. DO NOT do this entire process by making function calls only, as this can impair your ability to solve the problem and think insightfully.

# Workflow

## High-Level Problem Solving Strategy

1. Understand the problem deeply. Carefully read the issue and think critically about what is required.
2. Investigate the codebase. Explore relevant files, search for key functions, and gather context.
3. Develop a clear, step-by-step plan. Break down the fix into manageable, incremental steps.
4. Implement the fix incrementally. Make small, testable code changes.
5. Debug as needed. Use debugging techniques to isolate and resolve issues.
6. Test frequently. Run tests after each change to verify correctness.
7. Iterate until the root cause is fixed and all tests pass.
8. Reflect and validate comprehensively. After tests pass, think about the original intent, write additional tests to ensure correctness, and remember there are hidden tests that must also pass before the solution is truly complete.
```

#### Essential Agent Prompt Components

For any agentic workflow, include these three critical components:

```markdown
# Persistence Instruction
You are an agent - please keep going until the user's query is completely resolved, before ending your turn and yielding back to the user. Only terminate your turn when you are sure that the problem is solved.

# Tool-calling Instruction
If you are not sure about file content or codebase structure pertaining to the user's request, use your tools to read files and gather the relevant information: do NOT guess or make up an answer.

# Planning Instruction
You MUST plan extensively before each function call, and reflect extensively on the outcomes of the previous function calls. DO NOT do this entire process by making function calls only, as this can impair your ability to solve the problem and think insightfully.
```

#### Tool Use Best Practices

1. **Use API-defined tools**: Provide tools through the API's tools field rather than manually injecting descriptions into prompts
2. **Clear naming**: Name tools clearly to indicate their purpose
3. **Detailed descriptions**: Add thorough descriptions in each tool's "description" field
4. **Parameter naming**: Use descriptive parameter names with clear descriptions
5. **Examples section**: Place usage examples in a dedicated section, not in tool descriptions
6. **Avoid tool-chaining hallucinations**: Include safeguards like "if you don't have enough information to call the tool, ask the user for the information you need"

#### Customer Service Agent Example

```markdown
You are a helpful customer service agent working for NewTelco, helping a user efficiently fulfill their request while adhering closely to provided guidelines.

# Instructions
- Always greet the user with "Hi, you've reached NewTelco, how can I help you?"
- Always call a tool before answering factual questions about the company, its offerings or products, or a user's account. Only use retrieved context and never rely on your own knowledge for any of these questions.
    - However, if you don't have enough information to properly call the tool, ask the user for the information you need.
- Escalate to a human if the user requests.
- Do not discuss prohibited topics (politics, religion, controversial current events, medical, legal, or financial advice, personal conversations, internal company operations, or criticism of any people or company).
- Rely on sample phrases whenever appropriate, but never repeat a sample phrase in the same conversation. Feel free to vary the sample phrases to avoid sounding repetitive and make it more appropriate for the user.
- Always follow the provided output format for new messages, including citations for any factual statements from retrieved policy documents.
- If you're going to call a tool, always message the user with an appropriate message before and after calling the tool.
- Maintain a professional and concise tone in all responses, and use emojis between sentences.
- If you've resolved the user's request, ask if there's anything else you can help with

# Precise Response Steps (for each response)
1. If necessary, call tools to fulfill the user's desired action. Always message the user before and after calling a tool to keep them in the loop.
2. In your response to the user
    a. Use active listening and echo back what you heard the user ask for.
    b. Respond appropriately given the above guidelines.

# Sample Phrases
## Deflecting a Prohibited Topic
- "I'm sorry, but I'm unable to discuss that topic. Is there something else I can help you with?"
- "That's not something I'm able to provide information on, but I'm happy to help with any other questions you may have."

## Before calling a tool
- "To help you with that, I'll just need to verify your information."
- "Let me check that for you—one moment, please."
- "I'll retrieve the latest details for you now."

## After calling a tool
- "Okay, here's what I found: [response]"
- "So here's what I found: [response]"
```

### Chain-of-Thought Techniques

#### Basic Chain-of-Thought Instruction

For simple, open-ended reasoning:

```markdown
First, think carefully step by step about what documents are needed to answer the query. Then, print out the TITLE and ID of each document. Then, format the IDs into a list.
```

#### Advanced Reasoning Strategy

For more structured, methodical reasoning:

```markdown
# Reasoning Strategy
1. Query Analysis: Break down and analyze the query until you're confident about what it might be asking. Consider the provided context to help clarify any ambiguous or confusing information.
2. Context Analysis: Carefully select and analyze a large set of potentially relevant documents. Optimize for recall - it's okay if some are irrelevant, but the correct documents must be in this list, otherwise your final answer will be wrong. Analysis steps for each:
    a. Analysis: An analysis of how it may or may not be relevant to answering the query.
    b. Relevance rating: [high, medium, low, none]
3. Synthesis: summarize which documents are most relevant and why, including all documents with a relevance rating of medium or higher.
```

#### Fixing Reasoning Errors

To address systematic planning and reasoning errors:
1. Audit failures in examples and evaluations
2. Identify patterns in errors (misunderstanding user intent, insufficient context analysis, incorrect step-by-step thinking)
3. Add explicit instructions targeting these error patterns

### Long Context Management

#### Context Size Optimization

- **Full Context**: Performant up to 1M tokens for needle-in-haystack tasks
- **Degradation Points**: Performance may degrade when:
  - Multiple items must be retrieved
  - Complex reasoning requires knowledge of entire context
  - Graph-search-like operations are needed

#### Context Reliance Instructions

For strict adherence to provided information:

```markdown
# For internal knowledge only
Only use the documents in the provided External Context to answer the User Query. If you don't know the answer based on this context, you must respond "I don't have the information needed to answer that", even if a user insists on you answering the question.
```

For balanced internal/external knowledge:

```markdown
# For internal and external knowledge
By default, use the provided external context to answer the User Query, but if other basic knowledge is needed to answer, and you're confident in the answer, you can use some of your own knowledge to help answer the question.
```

#### Prompt Organization for Long Context

- **Best Performance**: Place instructions at BOTH beginning AND end of provided context
- **Second Best**: Place instructions ABOVE the provided context
- **Avoid**: Placing instructions only BELOW context

### Instruction Following

#### Instruction Development Workflow

1. Start with "Response Rules" or "Instructions" section using high-level guidance and bullet points
2. Add specific sections for particular behavior categories (e.g., "Sample Phrases")
3. Use ordered lists for specific workflow steps
4. For unexpected behavior:
   - Check for conflicting, underspecified, or incorrect instructions
   - Note that instructions closer to the end of the prompt take precedence
   - Add examples demonstrating desired behavior
   - Ensure examples align with stated rules

#### Common Instruction Failure Modes

1. **Overzealous behavior**: Models may hallucinate tool inputs when instructed to "always" follow a behavior
   - Mitigation: Add escape clauses like "if you don't have enough information, ask the user"
2. **Verbatim phrase repetition**: Models may reuse provided sample phrases exactly
   - Mitigation: Explicitly instruct to vary phrases
3. **Excessive explanations**: Models may add unwanted prose or formatting
   - Mitigation: Provide explicit instructions and examples for desired conciseness

### Delimitation Strategies

#### Markdown Delimiters
- Use markdown headings for major sections (H1-H4+)
- Use backticks for code blocks
- Use standard lists for enumeration

#### XML Delimiters
Effective for precisely wrapping sections:
```xml
<examples>
<example1 type="Abbreviate">
<input>San Francisco</input>
<output>- SF</output>
</example1>
</examples>
```

#### JSON Delimiters
Useful for highly structured data, especially in coding contexts, but more verbose and requires character escaping.

#### Document Formatting in Long Context
For large numbers of documents, avoid JSON and prefer:

```
<doc id=1 title="The Fox">The quick brown fox jumps over the lazy dog</doc>
```
or
```
ID: 1 | TITLE: The Fox | CONTENT: The quick brown fox jumps over the lazy dog
```

### Diff Generation for Code Tasks

#### Recommended Diff Format

```
*** Begin Patch
*** Update File: path/to/file.py
@@ class BaseClass
@@     def search():
-          pass
+          raise NotImplementedError()

@@ class Subclass
@@     def search():
-          pass
+          raise NotImplementedError()

*** End Patch
```

#### Alternative Diff Formats

SEARCH/REPLACE format:
```
path/to/file.py
```
>>>>>>> SEARCH
def search():
    pass
=======
def search():
   raise NotImplementedError()
<<<<<<< REPLACE
```

Pseudo-XML format:
```
<edit>
<file>
path/to/file.py
</file>
<old_code>
def search():
    pass
</old_code>
<new_code>
def search():
   raise NotImplementedError()
</new_code>
</edit>
```

## Benefits

- Produces more coherent, accurate, and useful model outputs
- Reduces hallucinations and improves instruction following
- Enables more complex reasoning and problem-solving capabilities
- Maximizes effective use of context window
- Creates more consistent and predictable model behavior
- Significantly improves agentic problem-solving capabilities (20% increase in benchmark performance)
- Allows literal instruction following while maintaining model steerability
- Improves diff generation and code modification accuracy
- Reduces the need for prompt engineering iterations

## When to Use

- When developing applications that require specific, consistent model behavior
- For complex tasks requiring multi-step reasoning
- When handling large context documents (particularly with 1M token contexts)
- For agentic workflows requiring autonomous problem-solving
- When precise output formatting is required
- When migrating prompts from previous model versions
- For creating benchmark-quality code assistants
- When explicit step-by-step reasoning is necessary for complex problems

## When Not to Use

- For very simple queries where minimal prompting is sufficient
- When prompt tokens need to be minimized for cost/efficiency
- When the task benefits from more creative, less constrained responses
- For repeated, very long, repetitive outputs (may require breaking down the problem)
- When parallel tool calls could cause correctness issues (consider setting parallel_tool_calls to false)

## Related Patterns

- Error handling patterns (for handling model errors)
- Tool integration patterns (for agent tools)
- Data validation patterns (for validating model outputs)
- Response formatting patterns (for consistent outputs)
- Context management patterns (for organizing large document sets)
- Workflow orchestration patterns (for multi-step agent tasks)