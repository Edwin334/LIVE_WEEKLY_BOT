# Project NEXUS - Student News 

Project NEXUS is an automated system for discovering, verifying, summarizing, and restyling news relevant to international students at various universities. It aims to provide a centralized and accessible source of important campus and community information.

## Features

-   **Configurable News Discovery**:
    -   Scans user-specified news category pages for the latest article links.
    -   Utilizes Google Programmable Search Engine (PSE) for targeted searches on user-configured university domains using relevant keywords.
-   **Content Extraction**: Fetches and parses the full text content from discovered article URLs.
-   **AI-Powered Verification (Gemini API)**:
    -   Determines publication dates (from text and URL parsing).
    -   Verifies article recency based on a configurable threshold.
    -   Assesses relevance to the general student body of the configured university/community.
    -   Identifies the article type (e.g., "News article", "Opinion/Blog") to filter for news.
-   **News Summarization (Gemini API)**:
    -   Generates detailed yet concise English summaries (configurable length) focusing on key information.
-   **Restyling (Gemini API)**:
    -   Generates a relevant, catchy, and summary-style Chinese title for the news.
    -   Translates the English summary into Simplified Chinese.
    -   Rewrites the translation in a serious, formal, and objective news reporting style.
    -   Incorporates publication date and source attribution.
    -   Formats English names appropriately for Chinese readers.
    -   Includes an additional AI-powered refinement step for the Chinese news report to improve conciseness and logical flow.
-   **Structured Output & Export**:
    -   Saves final news reports (English summary, Chinese title, initial Chinese report, refined Chinese report) to a timestamped JSON file.
    -   Exports the refined Chinese news reports to a Google Document:

## Project Structure

```
NEXUS/
├── news_bot/                   # Main application package
│   ├── __init__.py
│   ├── main_orchestrator.py    # Main script
│   ├── core/
│   │   ├── __init__.py
│   │   └── config.py           # Configuration, API keys, targets
│   ├── discovery/
│   │   ├── __init__.py
│   │   └── search_client.py    # News discovery (category scan, Google PSE)
│   ├── processing/
│   │   ├── __init__.py
│   │   └── article_handler.py  # Text extraction, Gemini verification
│   ├── generation/
│   │   ├── __init__.py
│   │   └── summarizer.py       # English summarization
│   ├── localization/
│   │   ├── __init__.py
│   │   └── translator.py       # Chinese translation, title, refinement
│   ├── reporting/
│   │   ├── __init__.py
│   │   └── google_docs_exporter.py # Google Docs export
│   └── utils/
│       ├── __init__.py
│       └── file_manager.py     # JSON saving utility
│
├── .env.example                # Example environment file
├── requirements.txt            # Dependencies
└── README.md                   # This file
```

## Setup

1.  **Clone Repository & Navigate**: 
    ```bash
    git clone <repository_url> NEXUS
    cd NEXUS
    ```
2.  **Virtual Environment (Recommended)**:
    ```bash
    python -m venv venv
    source venv/bin/activate  # Windows: venv\Scripts\activate
    ```
3.  **Install Dependencies**:
    ```bash
    pip install -r requirements.txt
    ```
4.  **API Keys & Configuration (`.env` file)**:
    *   Create a `.env` file in the `NEXUS` root (copy from `.env.example` if provided).
    *   Add your API keys:
        ```env
        GEMINI_API_KEY="YOUR_GEMINI_API_KEY"
        GOOGLE_API_KEY="YOUR_GOOGLE_API_KEY_FOR_PSE"
        CUSTOM_SEARCH_ENGINE_ID="YOUR_GOOGLE_PSE_CX_ID"
        
        # Optional: For Google Docs Export - ID of an existing Doc to update
        # TARGET_GOOGLE_DOC_ID="YOUR_GOOGLE_DOCUMENT_ID_HERE"
        
        # Optional: Override default model names or parameters from config.py
        # GEMINI_FLASH_MODEL='gemini-1.5-flash-latest' 
        # RECENCY_THRESHOLD_DAYS=7
        ```
    *   **Google Programmable Search Engine (PSE) Setup**:
        1.  Project in [Google Cloud Console](https://console.cloud.google.com/).
        2.  Enable "Custom Search API".
        3.  Create API Key (this is `GOOGLE_API_KEY`), restrict to "Custom Search API".
        4.  Create PSE at [Programmable Search Engine](https://programmablesearchengine.google.com/), configure sites to search, get Search Engine ID (`CUSTOM_SEARCH_ENGINE_ID`).
    *   **Google Docs API Setup (for Docs Export)**:
        1.  In the same Google Cloud Project, enable "Google Docs API".
        2.  Create OAuth 2.0 Client ID (Type: Desktop app) under "Credentials".
        3.  Download the client secret JSON file, rename it to `credentials.json` (or as specified in `config.OAUTH_CREDENTIALS_FILENAME`), and place it in the project root.
5.  **Target Configuration (`news_bot/core/config.py`)**: 
    *   Review/update `TARGET_NEWS_SOURCES_DOMAINS`, `CATEGORY_PAGES_TO_SCAN`, and `RELEVANCE_KEYWORDS` to suit your target university/community.

## Usage

Run the main script from the `NEXUS` root directory:
```bash
python -m news_bot.main_orchestrator
```
-   On the first run involving Google Docs export, a browser window will open for OAuth 2.0 authorization. You'll need to sign in and grant permissions.
-   Output JSON files will be saved in the directory specified by `DEFAULT_OUTPUT_DIR` in `config.py` (default: `news_reports`).
-   If Google Docs export is successful, the URL of the created/updated document will be printed in the console.

## Modules

-   **`config.py`**: Manages all configurations (API keys, model names, target URLs, keywords, paths, etc.). Many can be overridden by `.env` variables.
-   **`search_client.py`**: Discovers news articles by scanning category pages and querying Google PSE.
-   **`article_handler.py`**: Fetches article text from URLs; uses Gemini for verification (date, recency, relevance, article type).
-   **`summarizer.py`**: Generates detailed English summaries of verified articles with Gemini.
-   **`translator.py`**: Translates English summaries to formal Chinese news reports, generates Chinese titles, refines Chinese text, and formats names, using Gemini.
-   **`reporting/google_docs_exporter.py`**: Handles authentication and export of refined Chinese news reports to a Google Document (updates existing or creates new).
-   **`utils/file_manager.py`**: Saves structured data to JSON files.
-   **`main_orchestrator.py`**: Orchestrates the entire workflow.

## Customization for a New University

1.  **Google PSE**: Update your Google Programmable Search Engine settings to include the new university's specific news domains.
2.  **`.env` / `config.py`**: 
    *   Modify `TARGET_NEWS_SOURCES_DOMAINS` in `config.py`.
    *   Add relevant `CATEGORY_PAGES_TO_SCAN` for the new university in `config.py`.
    *   Adjust `RELEVANCE_KEYWORDS`.
    *   Optionally set `TARGET_GOOGLE_DOC_ID` in `.env` if you want to update a specific doc for this new university.
3.  **Prompts (Optional)**: For significantly different target audiences or news styles, review and tweak prompts in `article_handler.py`, `summarizer.py`, and `translator.py`.