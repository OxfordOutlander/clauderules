# Assisted Search Pattern

## Problem

Information retrieval systems face challenges when dealing with complex information needs:

1. Traditional search engines return ranked lists of results rather than comprehensive answers
2. Complex queries often require multiple searches to gather complete information
3. Unstructured search results require manual extraction of relevant entities and facts
4. Results often lack focus on the specific entity types of interest

## Solution

The Assisted Search Pattern provides a structured approach to information retrieval using AI models in two complementary phases:

### Part 1: Enhanced Search with AI Models

The first phase uses LLM-integrated search capabilities to obtain comprehensive information:
- Leverage models with integrated web search (like GPT-4o with search)
- Formulate search-optimized prompts for comprehensive information gathering
- Obtain narrative responses that synthesize information from multiple sources

### Part 2: Structured Extraction and Recursive Refinement

The second phase processes raw search results to extract structured data and refine searches:
- Extract structured entities from raw search results using schema-based parsing
- Evaluate result completeness to identify information gaps
- Generate focused follow-up queries to fill those gaps
- Execute searches concurrently to improve efficiency
- Recursively explore the information space with controlled depth

## Implementation: Part 1 - Enhanced Search

```python
import os
import time
from typing import Dict, Any
from dotenv import load_dotenv
from openai import OpenAI

# Load environment variables
load_dotenv()

# Initialize client
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

def enhanced_search(query: str, search_params: Dict = None) -> Dict[str, Any]:
    """
    Perform an enhanced search using an LLM with integrated search capabilities
    
    Args:
        query: The search query
        search_params: Additional search parameters
    
    Returns:
        Dictionary containing search results and metadata
    """
    search_params = search_params or {}
    
    # Configure standard options
    max_tokens = search_params.get("max_tokens", 1500)
    
    print(f"🔍 Searching: {query}")
    start_time = time.time()
    
    try:
        # Execute the search using OpenAI's web search tools
        # Following OpenAI's documentation: https://platform.openai.com/docs/guides/tools-web-search
        response = client.chat.completions.create(
            model="gpt-4.1-mini",  # gpt-4.1-mini supports web search
            max_tokens=max_tokens,
            tools=[{
                "type": "web_search"
            }],
            tool_choice="auto",  # Let the model decide when to use web search
            messages=[
                {
                    "role": "system",
                    "content": """You are a helpful assistant with web search capabilities.
                    Provide comprehensive, accurate, and neutral information in response to the query.
                    Include key facts, entities, and relationships. Focus on finding specific 
                    information relevant to the query with citations where appropriate.
                    Be thorough in your search for information."""
                },
                {
                    "role": "user",
                    "content": query
                }
            ]
        )
        
        # Extract response content
        message = response.choices[0].message
        content = message.content
        
        # Get tool calls if available
        tool_calls = message.tool_calls if hasattr(message, 'tool_calls') else []
        
        # Calculate response time
        elapsed_time = time.time() - start_time
        
        # Log completion
        print(f"✅ Search completed in {elapsed_time:.2f}s")
        
        # Return structured result
        return {
            "content": content,
            "query": query,
            "elapsed_time": elapsed_time,
            "tool_calls": tool_calls,
            "raw_response": response
        }
        
    except Exception as e:
        print(f"❌ Web search failed: {str(e)}")
        # Try fallback to simulated search if web search fails
        return simulated_search(query, search_params)

def simulated_search(query: str, search_params: Dict = None) -> Dict[str, Any]:
    """
    Fall back to a simulated search when web search is unavailable
    
    Args:
        query: The search query
        search_params: Additional search parameters
    
    Returns:
        Dictionary containing simulated search results
    """
    search_params = search_params or {}
    max_tokens = search_params.get("max_tokens", 1500)
    
    print(f"🔍 Simulated searching: {query}")
    start_time = time.time()
    
    try:
        # Execute simulated search using regular model
        response = client.chat.completions.create(
            model="gpt-4.1-mini",
            max_tokens=max_tokens,
            messages=[
                {
                    "role": "system",
                    "content": """You are an expert at finding accurate information. For this task, 
                    simulate having access to web search results. Provide comprehensive, accurate, and 
                    detailed information in response to the query as if you had just searched the web.
                    
                    Include specific facts, data points, metrics, and figures whenever possible.
                    Organize your response for readability. If information might be contradictory or 
                    uncertain, acknowledge this in your response."""
                },
                {
                    "role": "user",
                    "content": query
                }
            ]
        )
        
        # Extract response content
        content = response.choices[0].message.content
        
        # Calculate response time
        elapsed_time = time.time() - start_time
        
        # Log completion
        print(f"✅ Simulated search completed in {elapsed_time:.2f}s")
        
        # Return structured result with search_type indicating simulation
        return {
            "content": content,
            "query": query,
            "elapsed_time": elapsed_time,
            "search_type": "simulated"
        }
        
    except Exception as e:
        print(f"❌ Simulated search failed: {str(e)}")
        raise

# Example usage
def search_example():
    query = "List the major AI safety research organizations and their key areas of focus"
    
    result = enhanced_search(query)
    
    # Print preview of result
    preview = result["content"][:200] + "..." if len(result["content"]) > 200 else result["content"]
    print(f"\nSearch result preview:\n{preview}")
    print(f"Search type: {result.get('search_type', 'web')}")
    
    return result
```

### Best Practices for Enhanced Search

1. **Modern OpenAI Web Search Implementation**
   - Use the tools API with `"type": "web_search"` to enable web search capability
   - Use GPT-4.1-mini as the recommended model for web search (best price-performance ratio)
   - Use `tool_choice="auto"` for natural search behavior or `tool_choice={"type": "web_search"}` to force search
   - Temperature and standard parameters work normally with web search tools
   - Always implement a fallback to simulated search for robustness

2. **Optimize Search Prompts**
   - Include explicit instructions for comprehensive coverage
   - Request specific entity types and attributes 
   - Specify desired level of detail and organization
   - Include instructions to note information gaps or uncertainties
   - For token count searches, explicitly ask for specific numbers

3. **Implement Robust Fallback Strategy**
   - Create a separate simulated_search function for when web search fails
   - Design fallbacks to gracefully handle API limitations
   - Label results with search_type to distinguish web vs. simulated results
   - Structure web searches to be idempotent for retry safety
   - Consider using a cache for repeated queries

4. **Handle Edge Cases**
   - Implement timeout handling for long-running searches
   - Add retry logic for intermittent failures
   - Validate responses for minimum expected content
   - Capture and process tool_calls response data
   - Return structured results with metadata

## Implementation: Part 2 - Structured Extraction and Recursive Refinement

```python
import instructor
from pydantic import BaseModel, Field
from typing import List, Dict, Any, Optional
import concurrent.futures
from concurrent.futures import ThreadPoolExecutor, as_completed

# Initialize instructor client for structured data extraction
structured_client = instructor.patch(OpenAI(api_key=os.getenv("OPENAI_API_KEY")))

# Data models for structured extraction
class EntityData(BaseModel):
    """Generic entity data model - customize for your specific entity types"""
    name: str = Field(description="The name or identifier of the entity")
    attributes: Dict[str, Any] = Field(description="Key attributes of the entity")
    source: Optional[str] = Field(description="Source of information about this entity", default=None)

class EntityCollection(BaseModel):
    """Collection of extracted entities"""
    entities: List[EntityData] = Field(description="List of extracted entities")
    entity_type: str = Field(description="The primary type of entity extracted")
    completeness_score: float = Field(description="Estimated completeness of entity coverage (0-1)")

class SearchEvaluation(BaseModel):
    """Evaluation of search result completeness"""
    is_comprehensive: bool = Field(description="Whether the search result provides comprehensive coverage")
    missing_information: List[str] = Field(description="Categories of information missing from the results")
    follow_up_queries: List[str] = Field(description="Suggested follow-up queries to fill information gaps")

def extract_entities(search_result: str, entity_type: str) -> EntityCollection:
    """
    Extract structured entities from search results
    
    Args:
        search_result: Raw text from search results
        entity_type: Type of entity to extract
        
    Returns:
        Collection of structured entity data
    """
    try:
        # Use instructor for structured extraction
        return structured_client.chat.completions.create(
            model="gpt-4o",
            response_model=EntityCollection,
            messages=[
                {
                    "role": "system",
                    "content": f"""Extract all {entity_type} entities from the provided text.
                    For each entity, extract its name and key attributes.
                    Only include entities that are explicitly mentioned in the text.
                    Estimate the completeness of the entity coverage on a scale of 0-1."""
                },
                {
                    "role": "user",
                    "content": f"Extract {entity_type} entities from this text:\n\n{search_result}"
                }
            ]
        )
    except Exception as e:
        print(f"Entity extraction failed: {str(e)}")
        # Return empty collection as fallback
        return EntityCollection(
            entities=[],
            entity_type=entity_type,
            completeness_score=0.0
        )

def evaluate_search_completeness(query: str, search_result: str) -> SearchEvaluation:
    """
    Evaluate whether search results are comprehensive and suggest follow-up queries
    
    Args:
        query: Original search query
        search_result: Raw search result text
        
    Returns:
        Evaluation with follow-up query suggestions
    """
    try:
        return structured_client.chat.completions.create(
            model="gpt-4o",
            response_model=SearchEvaluation,
            messages=[
                {
                    "role": "system",
                    "content": """Analyze search results to determine if they comprehensively answer the query.
                    Identify specific categories of missing information.
                    Suggest follow-up queries that would fill information gaps."""
                },
                {
                    "role": "user",
                    "content": f"""
                    Original query: {query}
                    
                    Search result:
                    {search_result}
                    
                    Is this result comprehensive? What information is missing?
                    What follow-up queries would provide missing information?
                    """
                }
            ]
        )
    except Exception as e:
        print(f"Evaluation failed: {str(e)}")
        # Return basic evaluation as fallback
        return SearchEvaluation(
            is_comprehensive=False,
            missing_information=["Evaluation failed"],
            follow_up_queries=[f"{query} more detailed"]
        )

def process_search_with_extraction(query: str, entity_type: str = None) -> Dict[str, Any]:
    """
    Perform a search and extract structured entities from the results
    
    Args:
        query: Search query
        entity_type: Type of entity to extract
        
    Returns:
        Dictionary with raw results, structured entities, and evaluation
    """
    # Step 1: Perform initial search
    search_result = enhanced_search(query)
    content = search_result["content"]
    
    # Step 2: Extract structured entities if entity type is specified
    entities = None
    if entity_type:
        entities = extract_entities(content, entity_type)
    
    # Step 3: Evaluate search completeness
    evaluation = evaluate_search_completeness(query, content)
    
    # Return combined results
    return {
        "raw_search": search_result,
        "entities": entities.dict() if entities and hasattr(entities, "dict") else entities,
        "evaluation": evaluation.dict() if hasattr(evaluation, "dict") else evaluation
    }

def recursive_search(
    initial_query: str,
    entity_type: str = None,
    max_depth: int = 2,
    max_queries_per_level: int = 3,
    parallel: bool = True
) -> Dict[str, Any]:
    """
    Perform recursive search with follow-up queries to ensure comprehensive coverage
    
    Args:
        initial_query: The initial search query
        entity_type: Type of entity to extract from results
        max_depth: Maximum recursion depth
        max_queries_per_level: Maximum number of follow-up queries per level
        parallel: Whether to execute follow-up queries in parallel
        
    Returns:
        Dictionary containing all search results and extracted entities
    """
    # Process initial search
    result = process_search_with_extraction(initial_query, entity_type)
    
    # Check if follow-up queries are needed
    evaluation = result["evaluation"]
    if not evaluation["is_comprehensive"] and max_depth > 0 and evaluation["follow_up_queries"]:
        # Limit number of follow-up queries
        follow_up_queries = evaluation["follow_up_queries"][:max_queries_per_level]
        result["follow_up_results"] = {}
        
        if parallel:
            # Process follow-up queries in parallel
            with ThreadPoolExecutor() as executor:
                # Create mapping of futures to queries
                future_to_query = {
                    executor.submit(
                        recursive_search, 
                        query, 
                        entity_type,
                        max_depth - 1,
                        max_queries_per_level,
                        parallel
                    ): query for query in follow_up_queries
                }
                
                # Process results as they complete
                for future in as_completed(future_to_query):
                    query = future_to_query[future]
                    try:
                        result["follow_up_results"][query] = future.result()
                    except Exception as e:
                        print(f"Error processing follow-up query '{query}': {str(e)}")
                        result["follow_up_results"][query] = {"error": str(e)}
        else:
            # Process follow-up queries sequentially
            for query in follow_up_queries:
                result["follow_up_results"][query] = recursive_search(
                    query,
                    entity_type,
                    max_depth - 1,
                    max_queries_per_level,
                    parallel
                )
    
    return result

def merge_entities(results: Dict) -> List[EntityData]:
    """
    Merge entities from recursive search results
    
    Args:
        results: Results from recursive search
        
    Returns:
        List of unique entities with merged attributes
    """
    entity_map = {}
    
    # Helper to process each result node
    def process_result_node(node):
        if "entities" in node and node["entities"] and "entities" in node["entities"]:
            for entity in node["entities"]["entities"]:
                # Use name as key for deduplication
                key = entity["name"].lower()
                
                if key not in entity_map:
                    entity_map[key] = entity
                else:
                    # Merge attributes from both instances
                    for attr_key, attr_value in entity["attributes"].items():
                        if attr_key not in entity_map[key]["attributes"]:
                            entity_map[key]["attributes"][attr_key] = attr_value
    
    # Process main result
    process_result_node(results)
    
    # Process follow-up results recursively
    if "follow_up_results" in results:
        for query, sub_result in results["follow_up_results"].items():
            process_result_node(sub_result)
            
            # Recursive processing
            if "follow_up_results" in sub_result:
                for _, sub_sub_result in sub_result["follow_up_results"].items():
                    if isinstance(sub_sub_result, dict):  # Safety check
                        process_result_node(sub_sub_result)
    
    # Return values as list
    return list(entity_map.values())

# Complete example
def comprehensive_search_example():
    query = "List the major AI safety research organizations and their key areas of focus"
    entity_type = "Organization"
    
    # Perform recursive search
    results = recursive_search(query, entity_type, max_depth=2, parallel=True)
    
    # Merge entities from all searches
    all_entities = merge_entities(results)
    
    print(f"\nFound {len(all_entities)} unique organizations:")
    for i, entity in enumerate(all_entities[:5], 1):  # Print first 5 for brevity
        print(f"{i}. {entity['name']}: {', '.join(key for key in entity['attributes'])}")
    
    if len(all_entities) > 5:
        print(f"... and {len(all_entities) - 5} more")
    
    return results
```

### Advanced Techniques

#### 1. Two-Stage Processing with Instructor

The pattern uses a two-stage approach that separates responsibilities:

1. **Raw Information Gathering**: Use models with search capabilities to gather comprehensive information
2. **Structured Data Extraction**: Use Instructor-enhanced models to convert raw text to structured data

Benefits:
- Leverages specialized models for what they do best
- Produces application-ready data structures
- Enables sophisticated validation of entity data
- Allows independent optimization of search vs. extraction stages

```python
# First stage - Raw search
search_result = enhanced_search("List of Fortune 500 CEOs")

# Second stage - Structured extraction with Instructor
from instructor import OpenAISchema
from pydantic import BaseModel, Field

class CEO(BaseModel):
    name: str = Field(description="The CEO's full name")
    company: str = Field(description="The company the CEO leads")
    appointed: str = Field(description="When the CEO was appointed")
    
class CEOList(BaseModel):
    ceos: List[CEO] = Field(description="List of CEOs extracted from the text")

# Extract structured data
ceo_data = structured_client.chat.completions.create(
    model="gpt-4o",
    response_model=CEOList,
    messages=[
        {"role": "system", "content": "Extract CEO information from the provided text"},
        {"role": "user", "content": search_result["content"]}
    ]
)
```

#### 2. Concurrent Execution Strategies

Parallel processing is essential for efficient recursive searches:

```python
# Process multiple queries concurrently
def process_queries_concurrently(queries, process_fn, max_workers=5):
    results = {}
    
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        future_to_query = {executor.submit(process_fn, query): query for query in queries}
        
        for future in as_completed(future_to_query):
            query = future_to_query[future]
            try:
                results[query] = future.result()
            except Exception as e:
                results[query] = {"error": str(e)}
    
    return results
```

Key benefits:
- Significantly reduces total execution time
- Efficiently handles I/O-bound operations
- Improves user experience through faster responses
- Enables exploration of more information paths

Considerations:
- Implement rate limiting to avoid API throttling
- Use appropriate error handling for parallel operations
- Monitor and adjust concurrency based on API limits

#### 3. Managing Token Budgets

LLM-based search can consume significant token budgets:

```python
def estimate_token_usage(query, depth, branches_per_level):
    # Approximate token costs
    avg_query_tokens = len(query.split()) + 20
    avg_response_tokens = 1000
    avg_processing_tokens = 2000
    
    # Calculate total branches
    total_branches = 0
    for i in range(depth + 1):
        total_branches += branches_per_level ** i
    
    # Estimate total tokens
    total_tokens = total_branches * (avg_query_tokens + avg_response_tokens + avg_processing_tokens)
    
    return {
        "estimated_total_tokens": total_tokens,
        "branches": total_branches,
        "depth": depth
    }
```

Cost management strategies:
- Dynamically adjust search depth based on query complexity
- Implement early termination when sufficient information is found
- Cache results for common queries
- Use smaller models for evaluation stages

#### 4. Custom Entity Extractors

Create specialized extractors for different entity types:

```python
# Organization-specific extractor
class Organization(BaseModel):
    name: str = Field(description="Organization name")
    focus_areas: List[str] = Field(description="Key research or focus areas")
    founding_year: Optional[int] = Field(description="Year founded", default=None)
    location: Optional[str] = Field(description="Headquarters location", default=None)
    
class OrganizationList(BaseModel):
    organizations: List[Organization] = Field(description="List of organizations")

def extract_organizations(text):
    return structured_client.chat.completions.create(
        model="gpt-4o",
        response_model=OrganizationList,
        messages=[
            {"role": "system", "content": "Extract detailed organization information from the text"},
            {"role": "user", "content": text}
        ]
    )
```

## Benefits

1. **Comprehensive Coverage**: Explores information space systematically and thoroughly
2. **Structured Data**: Produces application-ready data models rather than raw text
3. **Efficient Processing**: Uses parallel execution to improve performance
4. **Flexibility**: Works with various search providers and entity types
5. **Query Refinement**: Automatically generates and executes follow-up queries
6. **Entity Focus**: Maintains concentration on primary entity types
7. **Result Aggregation**: Merges related entities across multiple search paths

## When to Use

- When building research or information gathering applications
- When comprehensive entity coverage is critical
- When converting unstructured search results to structured data
- When automating research workflows
- When information synthesis from multiple sources is needed
- When building agents that need to explore topics systematically

## When Not to Use

- For simple queries with straightforward answers
- When API cost or rate limiting is a major concern
- When specialized search indices are more appropriate
- When human judgment between search steps is required
- When low latency is more important than completeness

## OpenAI Web Search Implementation Notes

This pattern has been updated to reflect the current OpenAI API implementation (as of April 2025) for web search. Important changes from previous versions:

1. **Tools API for Web Search**
   - The current API uses the tools framework with `"type": "web_search"`
   - Web search is implemented as a tool rather than a separate API feature
   - Standard tools parameters and patterns apply

2. **Model Recommendations**
   - Web search is available with GPT-4.1-mini
   - Also works with other GPT-4 variants
   - Lower cost models like gpt-3.5-turbo may not support web search functionality

3. **Parameter Implementation**
   - Use `tool_choice="auto"` to let the model decide when to use web search
   - Using `tool_choice={"type": "web_search"}` forces web search usage
   - Temperature and top_p parameters are compatible and can be adjusted

4. **Response Structure**
   - Web search responses may include `tool_calls` data
   - The response will have the same structure as other tool responses
   - The content will directly contain the synthesized search results

5. **Error Handling Requirements**
   - Always implement fallback to simulated search
   - Check for API compatibility errors
   - Structure code to gracefully degrade when web search is unavailable

This implementation provides backward compatibility with previous approaches while ensuring forward compatibility with the current API.

## Related Patterns

- **Instructor Pattern**: For structured data extraction
- **Chain-of-Thought Prompting**: For enhancing search quality
- **Parallel Processing Pattern**: For concurrent operations
- **Provider Abstraction Pattern**: For vendor independence
- **Adaptive Recursion Pattern**: For depth control in recursive operations
- **Graceful Degradation Pattern**: For handling service unavailability