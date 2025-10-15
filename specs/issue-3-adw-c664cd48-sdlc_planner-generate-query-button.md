# Feature: Generate Query Button

## Feature Description
This feature adds an intelligent "Generate Query" button that automatically creates natural language queries based on the existing database tables and their structure. When clicked, the button uses an LLM to generate interesting, contextually relevant queries that demonstrate the data relationships and structure, then automatically populates the query input field with the generated query. The query is limited to two sentences maximum and always overwrites any existing content in the input field.

The button provides users with query inspiration, helps new users understand what types of questions they can ask, and serves as a learning tool to demonstrate the application's capabilities with their specific dataset.

## User Story
As a user of the Natural Language SQL Interface
I want to generate example queries based on my loaded tables
So that I can quickly explore my data without having to think of queries myself

## Problem Statement
Users often experience "blank page syndrome" when first encountering the query interface - they don't know what questions to ask or how to phrase them. This is especially true when they have just uploaded data and are unfamiliar with what insights they can derive. Currently, users must manually type queries, which can be intimidating for new users or those unfamiliar with their dataset's structure.

## Solution Statement
We will add a "Generate Query" button positioned separately from the primary action buttons (Query and Upload Data) that leverages the existing `llm_processor.py` module to analyze the current database schema and generate contextually appropriate natural language queries. The button will use the Upload Data button's visual style (secondary-button) and be placed with appropriate spacing using justify-apart layout. When clicked, it will generate a concise query (max two sentences) based on the available tables and their structures, then automatically overwrite the query input field with the generated query, ready for the user to execute manually.

## Relevant Files
Use these files to implement the feature:

- **app/server/core/llm_processor.py** - Contains existing LLM integration functions (`generate_sql_with_openai`, `generate_sql_with_anthropic`, `format_schema_for_prompt`). We'll add a new function `generate_natural_language_query()` here to create natural language queries based on database schema.

- **app/server/core/sql_processor.py** - Contains `get_database_schema()` function that retrieves current database schema. We'll use this to get table information for query generation.

- **app/server/core/data_models.py** - Contains Pydantic models for API requests/responses. We'll add `GenerateQueryRequest` and `GenerateQueryResponse` models.

- **app/server/server.py** - Main FastAPI server file. We'll add a new `/api/generate-query` endpoint here.

- **app/client/src/main.ts** - Main frontend TypeScript file. We'll add button initialization and click handler for the generate query button, including logic to overwrite the query input field.

- **app/client/src/api/client.ts** - API client functions. We'll add a new `generateQuery()` function to call the backend endpoint.

- **app/client/src/types.d.ts** - TypeScript type definitions. We'll add `GenerateQueryRequest` and `GenerateQueryResponse` interfaces to match backend models.

- **app/client/index.html** - Main HTML structure. We'll add the new "Generate Query" button in the query controls section with appropriate layout.

- **app/client/src/style.css** - Styling definitions. The button will reuse existing `.secondary-button` class for visual consistency with Upload Data button.

### New Files

- **.claude/commands/e2e/test_generate_query.md** - E2E test specification for the generate query button feature, following the pattern of existing E2E tests.

- **app/server/tests/core/test_generate_query.py** - Unit tests for the new `generate_natural_language_query()` function in llm_processor.py.

- **app/server/tests/test_generate_query_endpoint.py** - Integration tests for the new `/api/generate-query` endpoint.

## Implementation Plan

### Phase 1: Foundation
First, we'll extend the backend LLM processing module to support generating natural language queries (not SQL) based on database schema. This requires creating a new function that analyzes table structures, column names, data types, and row counts to craft interesting questions. We'll also create the necessary data models and API endpoint to expose this functionality.

### Phase 2: Core Implementation
Next, we'll implement the frontend button UI and wire it to call the backend API. The button will be styled consistently with the Upload Data button and positioned appropriately in the query controls section. The click handler will fetch a generated query from the backend and populate (overwrite) the query input field, providing immediate feedback to the user.

### Phase 3: Integration
Finally, we'll ensure the feature integrates seamlessly with the existing query execution flow. Users should be able to generate a query, review it in the input field, optionally modify it, and then execute it using the existing Query button. We'll add comprehensive tests including E2E validation to ensure the feature works end-to-end with zero regressions.

## Step by Step Tasks
IMPORTANT: Execute every step in order, top to bottom.

### 1. Backend: Add Data Models
- Create `GenerateQueryRequest` model in `app/server/core/data_models.py` with optional `table_name` field
- Create `GenerateQueryResponse` model with `query` (string), `tables_used` (list), and optional `error` fields
- Ensure models follow existing Pydantic patterns in the file

### 2. Backend: Implement Query Generation Function
- Add `generate_natural_language_query()` function to `app/server/core/llm_processor.py`
- Function should accept schema_info (Dict) and optional llm_provider parameter
- Create prompts for both OpenAI and Anthropic that instruct the LLM to generate interesting natural language questions (not SQL)
- Prompt should request queries that explore table relationships, aggregations, filters, and date ranges
- Enforce max two sentences in the prompt
- Follow existing patterns for `generate_sql_with_openai` and `generate_sql_with_anthropic`
- Add provider routing logic similar to `generate_sql()` function
- Handle errors gracefully with appropriate error messages

### 3. Backend: Add API Endpoint
- Create new POST endpoint `/api/generate-query` in `app/server/server.py`
- Use `GenerateQueryRequest` and `GenerateQueryResponse` models
- Get database schema using `get_database_schema()`
- Call `generate_natural_language_query()` with schema
- Return generated query with tables referenced
- Add appropriate error handling and logging
- Follow existing endpoint patterns in the file

### 4. Backend: Write Unit Tests
- Create `app/server/tests/core/test_generate_query.py`
- Test `generate_natural_language_query()` with mock LLM responses
- Test with empty database (no tables)
- Test with single table
- Test with multiple tables
- Test error handling
- Ensure test coverage matches existing test patterns

### 5. Backend: Write Endpoint Integration Tests
- Create `app/server/tests/test_generate_query_endpoint.py`
- Test `/api/generate-query` endpoint with various database states
- Test request/response validation
- Test error scenarios
- Follow patterns from existing endpoint tests

### 6. Frontend: Add TypeScript Types
- Add `GenerateQueryRequest` interface to `app/client/src/types.d.ts`
- Add `GenerateQueryResponse` interface with matching fields to backend model
- Ensure types match backend Pydantic models exactly

### 7. Frontend: Add API Client Function
- Add `generateQuery()` function to `app/client/src/api/client.ts`
- Follow existing API method patterns
- Return Promise<GenerateQueryResponse>
- Handle errors appropriately

### 8. Frontend: Add HTML Button
- Add "Generate Query" button to `app/client/index.html` in the `.query-controls` section
- Use `id="generate-query-button"` and `class="secondary-button"`
- Position it using CSS flexbox with justify-apart spacing from Query and Upload Data buttons
- Place it logically in the button group

### 9. Frontend: Implement Button Handler
- Add `initializeGenerateQueryButton()` function to `app/client/src/main.ts`
- Call it from the DOMContentLoaded event listener
- Implement click handler that:
  - Disables button and shows loading state
  - Calls `api.generateQuery()`
  - On success: overwrites `#query-input` textarea value with generated query
  - On error: displays error message using existing `displayError()` function
  - Re-enables button when complete
- Follow existing patterns from `initializeQueryInput()` function

### 10. Frontend: Style Adjustments (if needed)
- Review button layout in `.query-controls` section
- Ensure proper spacing using CSS flexbox with space-between or appropriate gap
- Verify button uses existing `.secondary-button` class for consistency
- Test responsive behavior

### 11. Create E2E Test Specification
- Read `.claude/commands/test_e2e.md` and `.claude/commands/e2e/test_basic_query.md` to understand the E2E test format
- Create `.claude/commands/e2e/test_generate_query.md` following the established pattern
- Include test steps for:
  - Uploading sample data
  - Clicking Generate Query button
  - Verifying query input field is populated with generated query
  - Verifying query is max two sentences
  - Taking screenshots at each step
- Define clear success criteria
- Include verification steps marked with **Verify**

### 12. Execute Validation Commands
- Run all validation commands listed below to ensure zero regressions
- Fix any issues that arise
- Re-run tests until all pass

## Testing Strategy

### Unit Tests
- **Test query generation with various schemas**: Verify that `generate_natural_language_query()` produces valid natural language queries (not SQL) for different database configurations
- **Test with no tables**: Should return appropriate message indicating no data available
- **Test with single table**: Should generate relevant query about that table
- **Test with multiple tables**: Should generate queries that potentially involve joins or multiple tables
- **Test query length validation**: Verify generated queries don't exceed two sentences
- **Test LLM provider routing**: Verify correct LLM (OpenAI/Anthropic) is used based on available API keys
- **Test API endpoint responses**: Verify `/api/generate-query` returns correct structure and handles errors

### Edge Cases
- No database tables loaded (empty schema)
- Only one table with minimal columns
- Multiple complex tables with many columns
- API keys not configured
- LLM API failures or timeouts
- Network errors between frontend and backend
- Button clicked multiple times rapidly (debouncing)
- Very long generated queries (should be truncated to two sentences)

## Acceptance Criteria
1. A "Generate Query" button appears in the query controls section, styled like the Upload Data button
2. Button is positioned with space-between layout, visually separated from Query and Upload Data buttons
3. When clicked, button shows loading state and is disabled during API call
4. Generated query overwrites (not appends to) the query input field
5. Generated query is contextually relevant to the loaded database tables
6. Generated query is maximum two sentences long
7. If no tables are loaded, appropriate error message is displayed
8. If API call fails, error is displayed using existing error handling
9. Backend uses existing `llm_processor.py` infrastructure with OpenAI/Anthropic support
10. All existing functionality continues to work without regression
11. All new code has appropriate unit tests and integration tests
12. E2E test validates the feature works end-to-end

## Validation Commands
Execute every command to validate the feature works correctly with zero regressions.

- Read `.claude/commands/test_e2e.md`, then read and execute the new E2E test `.claude/commands/e2e/test_generate_query.md` to validate this functionality works
- `cd app/server && uv run pytest` - Run server tests to validate the feature works with zero regressions
- `cd app/client && bun tsc --noEmit` - Run frontend type checking to validate the feature works with zero regressions
- `cd app/client && bun run build` - Run frontend build to validate the feature works with zero regressions

## Notes

### LLM Provider Configuration
- The feature uses the existing LLM provider infrastructure in `llm_processor.py`
- By default, OpenAI is used if `OPENAI_API_KEY` is available, otherwise Anthropic
- Ensure at least one API key is configured in the `.env` file

### Query Generation Prompt Strategy
The LLM prompt should encourage generation of queries that:
- Ask about specific columns or relationships visible in the schema
- Use aggregations (count, sum, average) where appropriate for numeric columns
- Reference date/time columns for temporal queries
- Explore interesting patterns or insights
- Are natural and conversational (not technical SQL language)
- Are concise (max two sentences as specified in requirements)

### Frontend UX Considerations
- The button should use the same visual style as Upload Data button (`.secondary-button` class)
- Loading state provides feedback during API call
- Overwriting (not appending) ensures clean user experience
- Users can still manually edit the generated query before executing

### Security Considerations
- The feature only generates natural language queries, not SQL directly
- No user input is passed to LLM (only database schema)
- All SQL execution still goes through existing security validation in `sql_security.py`
- No new security vulnerabilities introduced

### Performance Considerations
- LLM API calls typically take 1-3 seconds
- Button disabled state prevents multiple concurrent requests
- Consider adding request debouncing if users click rapidly

### Future Enhancements (Out of Scope)
- Allow users to regenerate queries with different focus areas
- Store history of generated queries for quick access
- Add query complexity levels (simple, medium, complex)
- Multi-language support for generated queries
