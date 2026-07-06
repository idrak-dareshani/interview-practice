# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A single-file Streamlit web app (`app.py`) that generates multiple-choice interview questions tailored to a candidate's role, skills, and experience using Anthropic's Claude API, then gives AI-generated feedback after the quiz is completed.

## Commands

```sh
pip install -r requirements.txt   # install dependencies
streamlit run app.py              # run the app locally
```

Requires an `ANTHROPIC_API_KEY` in a `.env` file at the project root (not tracked in git).

There is no test suite, linter, or build step currently configured in this repo.

## Architecture

Everything lives in `app.py` and follows a linear script structure typical of Streamlit apps (top-to-bottom execution on every interaction, state persisted via `st.session_state`):

- **Claude API calls** (`generate_mcqs`, `get_feedback`): build a prompt and call `client.messages.create(...)` using model `claude-4-sonnet-20250514`. `generate_mcqs` requires the model to follow a strict text format (`Q1. ... A) ... Answer: X`) which the UI code depends on for parsing — if you change the prompt's format instructions, the parsing logic below must be updated to match.
- **Question parsing** (UI section): the raw text response from `generate_mcqs` is parsed with regex (`re.split`, `re.match`) to split it into individual questions, options, and the correct answer letter. This is brittle by nature since it depends on the LLM reliably following the STRICT FORMAT prompt instructions.
- **Quiz flow**: `st.session_state.mcqs` holds the raw generated text; `st.session_state.answers` maps question index to `(choice, correct_answer, question_text)`, populated as the user selects radio options. Clicking "Finish Quiz" grades all answers and requires every question to be answered first.
- **Feedback flow**: after grading, `get_feedback` is called with a summary of the candidate's results to produce a short written evaluation (strengths/weaknesses/suggestions).
