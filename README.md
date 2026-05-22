The Arabic Content Intelligence Engine (Gemini Project)

An automated AI-driven workflow orchestration built on **n8n** that monitors, scrapes, filters, and summarizes Middle Eastern digital content from "A Digital Boom". The engine leverages Advanced Language Models via LangChain and Google Gemini to classify and store content intelligence automatically.

---

## 📋 Table of Contents
1. [Project Overview](#-project-overview)
2. [Core Architecture & Flow](#%EF%B8%8F-core-architecture--flow)
3. [Expected Business & Technical Value](#-expected-business--technical-value)
4. [Estimated Operational Cost](#-estimated-operational-cost)
5. [Database Schema (Supabase SQL)](#-database-schema-supabase-sql)
6. [Setup & Installation Guide](#-setup--installation-guide)

---

## 🔍 Project Overview

This engine serves as an automated intelligence gatherer. Instead of manually parsing news sites, the workflow triggers daily, extracts article references, utilizes LLMs to discard irrelevant global news, and performs deep text summarization on Middle East-centric tech/business content. The structured knowledge is then pushed into a relational database (**Supabase**) for downstream applications (dashboards, newsletters, or data feeds).

---

## ⚙️ Core Architecture & Flow

The system operates via a structured loop-and-filter design managed by n8n:

1. **Triggering & Data Collection:**
   * A **Schedule Trigger** fires daily at 10:00 AM.
   * **ScrapingBee** safely handles web scraping to bypass potential bot-detection on the target domain.

2. **Parsing & Intelligence Filtering:**
   * **HTML Node** extracts the titles and hyperlinks using CSS selectors (`.post-title a`).
   * A **JavaScript Node** maps and normalizes data objects.
   * **LangChain Text Classifier** backed by **Google Gemini** categorizes titles immediately to isolate articles related to the "Middle East".

3. **Deep Scraping & Processing Loop:**
   * **Split In Batches** loops through approved items.
   * **ScrapingBee** fetches the internal content of each relevant article (`h1`, `article p`, `time`).
   * A **Wait Node** stalls for 10 seconds between iterations to enforce strict API rate limiting and avoid blocking.
   * **LangChain Summarization Chain** (powered by **Google Gemini**) condenses the full body into a brief insight.

4. **Storage & Error Management:**
   * Structured data is appended directly to **Supabase**.
   * Comprehensive custom error catchers send precise event debugging notifications directly to **Telegram** if any point of failure occurs.

---

## 🚀 Expected Business & Technical Value

* **Zero-Ops Content Ingestion:** Saves hours of manual media monitoring daily by automating content curation.
* **Smart Noise Reduction:** Uses Gemini at the very front of the pipeline to filter out irrelevant posts before executing secondary deep scrapes, saving API credits.
* **Structured Data Asset:** Converts raw, unstructured HTML into a clean, searchable database asset optimized for analytical queries.
* **Resilient Infrastructure:** Includes native rate-limiting handling and granular Telegram error alerting, guaranteeing high uptime.

---

## 💰 Estimated Operational Cost

The operational model is highly cost-efficient due to the structural filtering. Assuming an execution rate of **once per day** (~30 times a month), parsing around **20-30 articles per run** where only **50%** are Middle East relevant:

| Service / Tool | Expected Monthly Usage | Cost Tier | Estimated Cost ($/mo) |
| :--- | :--- | :--- | :--- |
| **n8n Platform** | Self-Hosted (Docker / WSL2) | Community Edition | **$0.00** |
| **ScrapingBee API** | ~1,000 API Credits / month | Free Tier (1,000 credits) | **$0.00** |
| **Google Gemini API** | ~50K Input / 20K Output Tokens daily | Pay-As-You-Go (Gemini 1.5 Flash) | **~$0.10 - $0.30** |
| **Supabase Database** | < 5MB Data storage per month | Free Tier (Up to 500MB) | **$0.00** |
| **Telegram Bot API** | Alerting Webhooks | Always Free | **$0.00** |
| **Total Estimated Cost** | — | — | **<$0.50 / Month** |

---

Setup & Installation Guide
Prerequisites
n8n Instance running locally via Docker or hosted.

ScrapingBee API Key.

Google Gemini (AI Studio) API Key.

Supabase Project setup with the table snippet executed.

Telegram Bot token and Target Chat ID.

Deployment Steps
Copy the raw JSON content of the .json file provided in this repository.

Open your n8n workspace dashboard.

Use the shortcut Ctrl + V (or Cmd + V on Mac) to paste the entire workflow seamlessly into your editor canvas.

Open the Credentials settings inside n8n and bind your specific API accounts for:

ScrapingBeeApi

googlePalmApi

supabase

telegramApi

Toggle the workflow state from Inactive to Active.

Developed as an advanced workflow blueprint exploring autonomous content pipelines and agentic LLM routing.




## 🗄️ Database Schema (Supabase SQL)

Execute the following DDL query inside your Supabase SQL Editor to instantly provision the required relational layout. It includes strict constraints, automatic metadata auditing, and indexes for performant query rendering.

```sql
-- Create the articles storage table
CREATE TABLE public.articles (
    id BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    title TEXT NOT NULL,
    url TEXT UNIQUE NOT NULL,
    published_date TIMESTAMP WITH TIME ZONE,
    extracted_content TEXT,
    summary TEXT,
    category VARCHAR(50) DEFAULT 'MiddleEast',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT TIMEZONE('utc'::text, NOW()) NOT NULL
);

-- Add comments for documentation documentation
COMMENT ON TABLE public.articles IS 'Holds filtered and AI-summarized tech/business articles.';

-- Create index on URL and Creation Date for fast lookup performance
CREATE INDEX idx_articles_url ON public.articles(url);
CREATE INDEX idx_articles_created_at ON public.articles(created_at);


--
