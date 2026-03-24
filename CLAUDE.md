# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **travel guide project** for a 14-day family trip (3 people) across France, Germany, Switzerland, and Italy, scheduled for **2026.3.31 - 4.13**. The goal is to produce a detailed, accurate, and illustrated travel guide suitable for the user's parents (elderly travelers).

## Key Files

- `介绍.md` — Project brief: goals, workflow, principles, and client requirements
- `法德瑞意14天行程_最终确认版v3.docx` — The finalized 14-day itinerary (Word document)
- `购票与交通完全手册.html` — Comprehensive ticketing and transportation manual with pricing, booking URLs, and timelines

## Work Principles

1. **All information must be backed by solid sources** — official websites, high-rated reviews on mainstream platforms, Google Maps for transport. No guessing or speculation.
2. **Information must be thorough** — the audience is elderly parents who need complete, clear guidance including viewpoints, highlights, timing, and photos.
3. **Use web research capabilities** — the user can provide Claude-in-Chrome access for verification. Always request capability support when needed.
4. **Continuously check work status** — validate findings, update as needed, and flag any capability gaps.

## Itinerary Structure (4 countries, 14 days)

- **France (Days 1-4):** Paris (Louvre, Eiffel Tower, Arc de Triomphe, Versailles)
- **Germany (Days 4-7):** Cologne (cathedral), Munich (Allianz Arena), Neuschwanstein Castle (rental car)
- **Switzerland (Days 7-9):** Grindelwald, Jungfraujoch
- **Italy (Days 9-14):** Venice, Rome (Colosseum, Vatican, AS Roma match), departure from FCO

## Pending Tasks (from client requirements)

1. Ticket purchasing list: categorize by urgency (must-buy-now vs. flexible)
2. Accommodation: prepare prompts for user to book via Airbnb/Google Maps with Claude-in-Chrome
3. Allianz Arena may be deprioritized if Roma match is confirmed — reassign that afternoon
4. Detailed per-attraction time planning with researched durations and transport buffers
5. Final deliverable: a comprehensive illustrated travel guide
6. Meta-goal: develop a reusable "travel planning skill" from this experience
