# Production Readiness Reviewer using Gemini CLI and Agent Skills

This document provides an overview and tutorial of the Production Reviewer Skill developed for the Gemini CLI. It's designed to audit Lovable and GitHub projects for production readiness that includes code reviews, vulnerabilities, and infrastructure review.

## Problem Statement

Many developers and non-developers use rapid prototyping tools (like Lovable) or boilerplate GitHub repos to launch ideas quickly. However, these prototypes sometimes contain:

- Security Vulnerabilities: Hardcoded keys or permissive Row Level Security (RLS).

- Scalability Bottlenecks: Unoptimized database queries or lack of caching.

- Maintenance Debt: Outdated dependencies with known CVEs.

Therefore, the goal is to create an automated, multi-tool AI agent that audits these projects and provides an initial "Go/No-Go" consultation. This tool provides a starting point to discuss how your project can go from testing to production in a safe and secure way.

## System Design

The Skill uses a few Model Context Protocols (MCPs) to give the Gemini CLI access to local environments and external APIs.

The MCP Stack consists of:

- GitHub MCP: Monitors repository health, PR history, and Dependabot alerts.

- Snyk MCP: Executes static analysis (SAST) and dependency vulnerability scans.

- Filesystem MCP: Reads local source code for logic audits (Middleware, Auth).

- Supabase MCP: Platform-aware audit (Migrations, Edge Functions, RLS, Logs).

## Prompt Documentation

The core logic resides in a SKILL.md file. The prompt uses a Senior Security Architect persona to ensure professionalism, tone, and thoroughness.

## Tutorial & Building Process

1. Ensure you have the Gemini CLI installed. You can follow the [official quickstart](https://geminicli.com/docs/get-started/) or simply run:
```npm install -g @google/gemini-cli```
2. Once installed, create a dedicated directory for your skill within the user-level skills folder. A "skill" is defined as a directory containing a SKILL.md file. Create the directory hierarchy for the 'production-reviewer' (pr) skill by running ```mkdir -p ~/.gemini/skills/pr/SKILL.md``` and optionally creating Assets, References, and Scripts folders for more advanced configuration
```mkdir -p ~/.gemini/skills/pr/{assets,references,scripts}``` 
3. Global tools (like GitHub or Supabase) must be registered in your user [settings file](https://github.com/danblevins/production-readiness-agentic-ai/blob/main/skills/pr/settings.json).
- ```~/.gemini/settings.json (macOS/Linux)``` or ```%USERPROFILE%\.gemini\settings.json (Windows)```.
- Add your mcpServers configuration block here to ensure the production-reviewer can access external data.

4. To verify, run ```gemini skills``` list in your terminal. You should see pr (or production-reviewer) listed as an available skill.

## Real Usage and Benchmarkings

The output of the test examples are [here](https://github.com/danblevins/production-readiness-agentic-ai/tree/main/skills/pr/output).

In the ```danblevins-lovable.md``` file, it correctly identified this as a Vite/React stack and Snyk called out that ```react-router-dom``` needs to be upgraded. Outside of security, it also mentions the need for improved logging and to verify a backend database.

In the ```is-it-raining-in-seattle.md``` file, it correctly identified the simple HTML, CSS, Javascript stack and called out that the OpenWeatherMap API key management should be updated to something not hard-coded. Outside of security, it also highlights the need for a better UI experience.

## Findings and Reflection

Integrating the GitHub MCP proved helpful to understand the initial code, allowing the model to analyze repository health and cloning the repository. The Snyk MCP was the primary driver for security, uncovering critical vulnerabilities that standard Github alerts sometimes miss.

By telling the model to prioritize the top 5 high-impact fixes, the output shifted from a data dump into a focused output. Finally, by telling the model to audit its own limitations, it provided more transparent output of what could not be verified, allowing me to deep dive and optimize in further iterations.
