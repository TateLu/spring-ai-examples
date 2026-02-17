# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a collection of Spring AI examples demonstrating various AI integration patterns and use cases. The repository is organized as a multi-module Maven project with examples covering:

- **Agentic Patterns**: Chain workflows, orchestrator-workers, parallelization, routing, and evaluator-optimizer patterns
- **Model Context Protocol (MCP)**: Extensive examples for MCP clients and servers including SQLite, filesystem, weather, web search, sampling, and annotations
- **Advisors**: Recursive advisors, tool argument augmenters, and evaluation patterns for ChatClient customization
- **Prompt Engineering**: Advanced prompt patterns and techniques
- **Kotlin Examples**: Hello world, function callbacks, and RAG implementations (currently disabled in root pom.xml)
- **Miscellaneous**: Function callbacks, streaming responses, and Claude Skills demos

## Build System and Commands

This project uses Maven with the Maven Wrapper (mvnw). Each module can be built and run independently.

### Common Development Commands

**Build entire project:**
```bash
./mvnw clean package
```

**Build and run individual module:**
```bash
cd <module-directory>
./mvnw clean package
./mvnw spring-boot:run
```

**Build from root (all modules):**
```bash
mvn clean package
```

**Run single test:**
```bash
cd <module-directory>
./mvnw test
```

**Run integration test for specific module:**
```bash
./integration-testing/scripts/run-integration-tests.sh <module-name>
```

### Version Management

**Update Spring AI version across all modules:**
```bash
./scripts/update-spring-ai-version.sh 1.0.1  # or 1.1.0-SNAPSHOT
```

The version management script handles two patterns:
- **17 modules** use `<spring-ai.version>` property
- **15 modules** use direct BOM version in dependencyManagement
- **Total: 32 pom.xml files** updated across the repository

**Check current versions:**
```bash
./scripts/check-spring-ai-version.sh
```

**Restore from backup:**
```bash
./scripts/restore-spring-ai-version.sh /path/to/backup
```

## Architecture

### Module Structure
- Each example is a self-contained Spring Boot application
- Modules follow standard Maven directory structure: `src/main/java` and `src/test/java`
- Common Spring AI version: `1.1.0-SNAPSHOT`
- Java version: 17
- Spring Boot parent: `3.4.5`

### Key Module Categories

**Agentic Patterns (`agentic-patterns/`):**
- Multi-module Maven project (chain-workflow, orchestrator-workers, parallelization-workflow, routing-workflow, evaluator-optimizer)
- Demonstrates AI agent workflow patterns based on Anthropic's research
- Each pattern is a separate module with its own Application class

**Model Context Protocol (`model-context-protocol/`):**
- MCP client and server implementations
- Covers various protocols: SQLite, filesystem access, weather APIs, web search, sampling, annotations
- Includes both manual and starter-based implementations
- Server types: WebMVC, WebFlux, and STDIO
- Dynamic tool update examples for runtime tool modifications

**Advisors (`advisors/`):**
- recursive-advisor-demo: Demonstrates recursive advisor patterns
- tool-argument-augmenter-demo: Tool argument augmentation examples
- evaluation-recursive-advisor-demo: Evaluation with recursive advisors (requires Ollama Docker)

**Prompt Engineering (`prompt-engineering/`):**
- Advanced prompt patterns and techniques for ChatClient

**Agents (`agents/`):**
- reflection: Reflection-based agent patterns

**Kotlin Examples (`kotlin/`):**
- kotlin-hello-world, kotlin-function-callback, rag-with-kotlin
- Currently disabled in root pom.xml (commented out)

### Configuration Patterns
- Application properties typically in `src/main/resources/application.properties` or `application.yaml`
- MCP configurations often use `mcp-servers-config.json`
- Docker Compose files for complex setups (e.g., `compose.yml`)

## Testing Framework

This repository uses a comprehensive integration testing framework with **AI-powered validation** for ensuring all examples work correctly across releases. The framework currently covers **31 modules with integration tests** with intelligent validation for interactive applications.

### Testing Approaches

| Example Type | Test Location | Command | Purpose |
|--------------|---------------|---------|---------|
| **Simple** | `src/test/java/` | `./mvnw test` | Unit tests, basic functionality |
| **Complex** | `integration-tests/` | `jbang integration-tests/Run*.java` | End-to-end integration with AI validation |

### Key Features

- **31 Modules Tested**: Integration tests for all major example categories
- **AI Validation**: Intelligent analysis of application logs for non-deterministic outputs
- **Interactive Application Support**: Testing of chatbots and Scanner-based applications
- **Centralized Architecture**: Single source of truth with 84% code reduction
- **Comprehensive Logging**: Full application output preserved for debugging

### Primary Integration Testing Tools

The framework provides **two essential tools** for integration testing:

**1. ⭐ Run all integration tests (primary test runner):**
```bash
./integration-testing/scripts/run-integration-tests.sh
```

**Run specific integration test:**
```bash
./integration-testing/scripts/run-integration-tests.sh module-name
```

**Run with clean logs:**
```bash
./integration-testing/scripts/run-integration-tests.sh --clean-logs
```

**2. ⭐ Create integration tests for new examples (scaffolding tool):**
```bash
# Essential for extending the framework - creates ExampleInfo.json + Run*.java files
python3 integration-testing/scripts/scaffold_integration_test.py <module-path> [--complexity simple|complex|mcp]
```

**Create integration test with AI validation:**
```bash
# Simple example (hybrid AI validation)
python3 integration-testing/scripts/scaffold_integration_test.py kotlin/kotlin-hello-world --complexity simple

# Complex workflow (primary AI validation)
python3 integration-testing/scripts/scaffold_integration_test.py agentic-patterns/chain-workflow --complexity complex

# Interactive application (fallback AI validation)
python3 integration-testing/scripts/scaffold_integration_test.py agents/reflection --complexity complex

# MCP example (hybrid validation)
python3 integration-testing/scripts/scaffold_integration_test.py model-context-protocol/weather/starter-webmvc-server --complexity complex
```

**Test specific example:**
```bash
cd <module-directory>
jbang integration-tests/Run*.java  # For complex examples
./mvnw test                         # For simple examples
```

### Integration Test Structure

Complex examples include an `integration-tests/` directory with AI validation:
```
module/
├── integration-tests/
│   ├── ExampleInfo.json      # Test configuration with AI validation support
│   └── RunModule.java        # JBang launcher (uses centralized utilities)
```

### JBang Script Pattern & AI Validation

All integration test scripts use centralized utilities with AI validation support:
- **Centralized Architecture**: Each script is only ~18 lines (84% code reduction)
- **AI Integration**: Automatic AI validation using Claude for intelligent analysis
- **Universal Support**: All test logic in `integration-testing/jbang-lib/IntegrationTestUtils.java`
- **Interactive Apps**: Special handling for Scanner-based applications

**Validation Modes:**
- **Primary**: AI-only validation for unpredictable AI outputs
- **Hybrid**: Regex patterns + AI validation for reliability
- **Fallback**: Regex primary with AI backup for interactive applications

See `integration-testing/docs/README.md` for complete guide and `integration-testing/docs/TROUBLESHOOTING.md` for troubleshooting.

### 🤖 AI Validation (NEW)

The integration testing framework now includes AI-powered validation using Claude to intelligently analyze test outputs. This goes beyond regex pattern matching to understand context, validate unpredictable AI outputs, and assess complex workflows.

**Key Benefits:**
- **Intelligent Assessment** - Understands if examples achieved their intended purpose
- **Handles AI Outputs** - Validates jokes, conversations, creative content  
- **Context-Aware** - Uses README documentation for validation context
- **Multi-Component Support** - Validates distributed systems holistically
- **Cost Efficient** - ~400 tokens per validation with high cache utilization

**When AI Validation Excels:**
- Chat examples (jokes, conversations, creative responses)
- Agentic patterns (multi-step reasoning, workflow completion)
- MCP examples (protocol validation, tool discovery)
- Function calling (realistic tool usage verification)
- Complex workflows with unpredictable outputs

**Configuration Example:**
```json
{
  "timeoutSec": 300,
  "successRegex": ["BUILD SUCCESS", "Started.*Application"],
  "requiredEnv": ["OPENAI_API_KEY"],
  "aiValidation": {
    "enabled": true,
    "validationMode": "hybrid",
    "expectedBehavior": "Application should demonstrate chat functionality with coherent AI responses",
    "promptTemplate": "chat_example_validation"
  }
}
```

**Validation Modes:**
- `primary` - AI validation only (best for unpredictable AI outputs)
- `hybrid` - Both regex and AI must pass (recommended for most cases)  
- `fallback` - AI validation if regex fails (gradual migration)

For complete details, see `integration-testing/docs/AI_VALIDATION.md`.

## Development Notes

- Use the integration testing framework for comprehensive example testing and validation
- Each module has its own Maven wrapper for isolated builds
- Examples demonstrate both OpenAI and other AI model integrations
- MCP examples often include database setup scripts (e.g., `create-database.sh`)
- Complex examples should include integration tests for CI/CD validation

### CI/CD Manual Triggers

Integration tests can be triggered manually via GitHub Actions:

```bash
# Run all tests
gh workflow run integration-tests.yml

# Run specific test group
gh workflow run integration-tests.yml -f test_group=agentic

# Run specific test by name
gh workflow run integration-tests.yml -f test_filter=chain-workflow
```

**Test groups**: `mcp-servers`, `agentic`, `openai-2`, `openai-3`, `anthropic-multi`, `docker-tests`

See `.github/workflows/integration-tests.yml` for complete test group definitions and requirements.

## Commit Message Guidelines

**IMPORTANT**: When committing code to this repository, commit messages should appear to be authored by humans:

- ❌ **DO NOT include**: Robot emoji (🤖), "Generated with Claude Code", "Co-Authored-By: Claude", or any AI attribution
- ✅ **DO use**: Professional commit messages following established conventions

### The Seven Rules of a Great Git Commit Message

Based on [Chris Beams' classic guide](https://cbea.ms/git-commit/), follow these rules:

1. **Separate subject from body with a blank line**
2. **Limit the subject line to 50 characters**
3. **Capitalize the subject line**
4. **Do not end the subject line with a period**
5. **Use the imperative mood in the subject line** ("Add feature" not "Added feature")
6. **Wrap the body at 72 characters**
7. **Use the body to explain what and why vs. how**

### Modern Conventions (Recommended)

Combine the classic rules with conventional commit format:

- `feat(scope): add new feature` - new functionality
- `fix(scope): resolve issue description` - bug fixes  
- `docs(scope): update documentation` - documentation changes
- `refactor(scope): restructure without changing behavior` - code refactoring
- `test(scope): add or update tests` - test changes
- `chore(scope): maintenance tasks` - build, dependencies, etc.

### Examples

**Good:**
```
feat(mcp): add weather server with OAuth2 support

Implements OAuth2 authentication for weather data access.
Resolves rate limiting issues and improves security.
```

**Good (simple):**
```
fix: resolve port conflicts in integration tests
```

**Bad:**
```
🤖 Generated with Claude Code: Fixed some stuff

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Writing Tip
Your commit message should complete: "If applied, this commit will _[subject line]_"