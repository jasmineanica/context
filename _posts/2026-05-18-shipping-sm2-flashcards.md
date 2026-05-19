---
layout: post
title: "Starting Fresh and Growing"
date: 2026-05-18
last_updated: 2026-05-19
---

Last week I crossed my first career milestone in my 4.5 years as a software engineer: a May 2026 layoff. A true badge of a SWE in silicon valley and something I had mentally prepared for. As I am figuring out my next play, I decided to keep the technical coding interview problems and solitions more fresh in my mind by building personal projects. All future personal projects will be under jasmineanica.com/projects/* . Each project has is its own GitHub repo with its own stack and live demo. Adding a new project is one YAML block plus one new repo.

The first one: Implement a Trie solution and understand it's uses.

## Things I learned
- **Empty-state matters in a demo.** The README claimed "Trie autocomplete on tags," but with an empty DB the autocomplete did nothing. I added a lifespan-hook seeder that drops six interview-prep cards on first boot, plus a search box on the home page that actually exercises the Trie via HTMX. Now the claim is visible, not just true.
- **`hx-ext="json-enc"` is silent if the extension script isn't loaded.** I had the attribute on my form but forgot the `<script src=".../json-enc.js">` tag, so HTMX quietly fell back to `application/x-www-form-urlencoded`, FastAPI couldn't parse the body as a JSON object, and every Save returned a 422 with `model_attributes_type`. Two-part fix: load the extension *and* add a Pydantic `field_validator` on `tags` so the same `/api/cards` endpoint accepts both `tags: ["a","b"]` (JSON clients) and `tags: "a, b"` (the form's single text input).

## What's next

The next project is probably a **rate limiter** (sliding window or token bucket, Redis-backed), and the loop is documented at [/docs/adding-a-project/](/docs/adding-a-project/).

Live demo: [sm2-flashcards.onrender.com](https://sm2-flashcards.onrender.com)
Code: [github.com/jasmineanica/sm2-flashcards](https://github.com/jasmineanica/sm2-flashcards)
