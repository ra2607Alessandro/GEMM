 # Cursor Rules for AI-Powered Search Assistant

## Project Overview
This is a Rails 8 application that builds an AI-powered search assistant similar to Perplexity. The system performs semantic search using vector embeddings with PostgreSQL + pgvector, providing natural language responses with proper citations grounded in real web content.

## Core Architecture Principles

### 1. Truth-Grounded Responses
- **NEVER** generate responses that aren't based on scraped web content
- All claims MUST be supported by citations from actual sources
- AI responses should synthesize and tailor found information, not hallucinate
- Include proper source attribution for every factual claim

### 2. Three-Input Search Model
The search interface has exactly 3 input fields:
- **Main Query** (required): The primary search question/topic
- **Goal Context** (optional): User's objective or desired outcome
- **Rules/Suggestions** (optional): Constraints or preferences for the search

### 3. Database & Vector Search
- Use PostgreSQL with pgvector extension
- Store embeddings using OpenAI's text-embedding-ada-002 (1536 dimensions)
- Implement semantic search using neighbor gem
- Maintain data integrity with proper foreign keys and validations

## Coding Standards & Practices

### Ruby/Rails Conventions
- Follow Rails 8 conventions and use modern Rails patterns
- Use service objects for complex business logic
- Implement background jobs with Solid Queue (Rails 8 default)
- Write descriptive method names and maintain clean, readable code
- Use strong parameters and proper validations
- Follow RESTful routing patterns

### Service Layer Organization
```
app/services/
├── ai/
│   ├── embedding_service.rb
│   └── response_generation_service.rb
├── search/
│   └── web_search_service.rb
└── scraping/
    └── content_scraper_service.rb
```

### Background Job Structure
- `SearchProcessingJob`: Main orchestrator
- `WebScrapingJob`: Individual URL scraping
- `EmbeddingGenerationJob`: Generate and store embeddings
- Always handle errors gracefully with proper logging

### Error Handling Requirements
- Implement comprehensive error handling for all external API calls
- Use timeouts for web scraping operations
- Gracefully handle API rate limits
- Provide user-friendly error messages
- Log errors appropriately for debugging

### Frontend & UX Guidelines
- Use Turbo Streams for real-time updates
- Implement Stimulus controllers for interactive elements
- Show search progress indicators
- Display results with proper citation formatting
- Ensure responsive design with Tailwind CSS

## API Integration Rules

### OpenAI Integration
- Use tiktoken for token counting and content chunking
- Implement proper rate limiting
- Handle API errors gracefully
- Store API keys securely in Rails credentials

### Web Search (Google Custom Search/SerpAPI)
- Limit results to 10 max per search
- Handle API quotas and errors
- Structure responses as `{title, url, snippet}` arrays
- Validate URLs before processing

### Web Scraping
- Use Mechanize + Nokogiri for robust scraping
- Implement ruby-readability for content extraction
- Set appropriate timeouts (30 seconds max)
- Handle JavaScript-heavy sites appropriately
- Sanitize all scraped content
- Respect robots.txt when possible

## Data Management

### Content Storage
- Deduplicate content by URL
- Store both raw and cleaned content
- Track scraping timestamps
- Implement content freshness checks

### Vector Embeddings
- Generate embeddings asynchronously
- Chunk large content appropriately
- Store embeddings with proper indexing
- Implement similarity search efficiently

### Performance Optimization
- Use database indexes effectively
- Implement caching for repeated queries
- Optimize vector similarity searches
- Batch process embeddings when possible

## Security & Privacy

### Data Protection
- Don't store sensitive user information
- Sanitize all user inputs
- Validate URLs before scraping
- Implement rate limiting for user requests

### API Security
- Store all API keys in Rails credentials
- Use environment variables for configuration
- Implement proper CORS policies
- Validate all external API responses

## Testing Requirements

### Test Coverage
- Write comprehensive tests for all service classes
- Test error handling scenarios
- Mock external API calls appropriately
- Test background job processing
- Include integration tests for the full search flow

### Test Structure
- Use RSpec for testing framework
- Implement FactoryBot for test data
- Use VCR for API call recording
- Write meaningful test descriptions

## Code Quality Standards

### Documentation
- Document all service class methods and functions
- Include inline comments for complex logic
- Maintain clear README instructions
- Document API integrations and configuration
- Always write and explain every single passage thinking the person who will review the code doesn't understand anything about code.  

### Performance Monitoring
- Log search performance metrics
- Monitor API response times
- Track embedding generation times
- Implement proper database query optimization

## Deployment Considerations

### Environment Setup
- Configure proper environment variables
- Set up database with pgvector extension
- Configure background job processing
- Set up proper logging and monitoring

### Scaling Preparation
- Design for horizontal scaling
- Implement proper caching strategies
- Consider CDN for static assets
- Plan for database optimization

## Key Implementation Priorities

1. **Core Services First**: Implement WebSearchService, ContentScraperService, and EmbeddingService
2. **Job Orchestration**: Build SearchProcessingJob to coordinate the entire flow
3. **UI/UX**: Create the three-input search interface with real-time updates
4. **AI Response Generation**: Implement natural language response generation with citations
5. **Error Handling & Polish**: Comprehensive error handling and user experience refinement

## Development Workflow

### Commit Standards
- Make atomic commits with clear messages
- Test changes before committing
- Follow conventional commit format
- Keep commits focused and logical

### Code Review Focus
- Verify all external API integrations
- Check error handling completeness
- Validate data model relationships
- Ensure proper citation implementation
- Test search result accuracy

Remember: This system must NEVER hallucinate or invent information. Every response must be grounded in actual web content with proper citations.