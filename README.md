### "You Can't Automate Curiosity"

## Overview
This application is an a little and unfinished draft of a Rails 8 AI-powered search assistant that answers questions using only information scraped from the web. It runs a multi-stage pipeline: web search → scraping → vectorization → truth-grounded AI synthesis with citations. Responses must never hallucinate and must always include proper citations.

- **Truth-grounded**: Every factual statement is supported by citations to scraped sources.
- **Three-input model**: A search has a required `query` and optional `goal` and `rules`.
- **Semantic search**: Vector embeddings (1536 dims) stored in PostgreSQL with pgvector; nearest-neighbor queries via `neighbor` gem.
- **Real-time UX**: Progress and results stream to the UI using Turbo Streams; background processing with Solid Queue.

## Core Principles
- **Personalized Search**: The search will be shaped around the goal and instruction provided to the assistant in order to receive the most personalized output possible
- **Truth only**: Synthesize from scraped content; never inject model “knowledge.”
- **Citations everywhere**: Inline citations like [1], [2] must appear next to claims.
- **Robustness**: Timeouts, retries, rate-limiting, fallbacks for scraping failures.
- **Safety**: Sanitize inputs and scraped HTML; validate URLs; store only non-sensitive data.

## High-Level Architecture

### Data Flow
1. **User submits a search** with `query`, optional `goal`, optional `rules`.
2. **SearchProcessingJob**
   - Generates a query embedding via `Ai::EmbeddingService` (OpenAI `text-embedding-3-small`, 1536 dims).
   - Chooses `Search::WebSearchService` (via [SerpAPI](https://serpapi.com)) or `Search::YoutubeSearchService` depending on the query.
   - Creates or reuses `Document` records and `SearchResult` associations.
   - Enqueues `WebScrapingJob` for each candidate URL.
3. **WebScrapingJob**
   - Uses `Scraping::ContentScraperService` (Mechanize + Nokogiri + ruby-readability) with timeouts and retries.
   - Normalizes/sanitizes content; stores `content`, `cleaned_content`, and `content_chunks`.
   - If scraping fails, falls back to `Scraping::FallbackContentService` using the search result snippet.
   - After each document, calls `Scraping::ScrapingCompletionService.check`.
4. **Scraping completion**
   - Monitors when enough sources have content to proceed.
   - Triggers `AiResponseGenerationJob` when content thresholds are met or fallback rules apply.
5. **AiResponseGenerationJob**
   - `Ai::ResponseGenerationService` prepares a truth-grounded context from scraped sources only.
   - Calls OpenAI (`gpt-4o-mini`) to synthesize an answer, enforcing citation rules.
   - Creates `Citation` records tied to the `SearchResult`s.
   - Asynchronously enqueues `EmbeddingGenerationJob` for each `Document` with content.
6. **Embeddings for sources**
   - `EmbeddingGenerationJob` computes and stores vector embeddings for `Document.cleaned_content`.
7. **Realtime updates**
   - `SearchesController` broadcasts Turbo Streams updates for status, results, and AI response.

### The future of Search

It's hard to predict the future of search.
It's a field that has been the foundation of how we travel on the web for decades.
Some say AI has completely solved the problem of search, but we all know that the problem of search has never been finding the answers: It has always been finding the relevant answers. We want to be able to read the answer and scream: "It just gets me".
And that's exactly what the draft of this project is meant to start. 

Thanks for everyone who is visiting this repository. 

If you believe in this project and the premise upon which is built, don't hesitate in leaving a star and contacting me. 


### License
This project is licensed under the MIT License. See the LICENSE.md file
