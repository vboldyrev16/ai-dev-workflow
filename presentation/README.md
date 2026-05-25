# Presentation — AI Development Workflow

The 20-slide talk that introduces the playbook. Single-file HTML + an audio/video outro.

## View

- **Live**: https://ai-dev-workflow-lake.vercel.app
- **Locally**: open `index.html` directly in a browser, or:
  ```bash
  python3 -m http.server 8000
  # then open http://localhost:8000
  ```

## Navigate

- `←` / `→` — previous / next slide
- `↑` first / `↓` last
- `1`–`9` — jump to slide
- `#N` in the URL — open at slide N (e.g. `#13`)
- Click left/right third of the screen (desktop) or swipe (mobile)

## Structure

20 slides total:

1. Title
2. Intro — why this talk
3. What we built — the dashboard (CSS mockup)
4. Timeline — 38 days from data to production
5. Cost story — 5–10× less core-team capacity
6. 11-stage pipeline overview
7. Backlog → Research & Spec
8. Design (CSS chat mockup of handoff)
9. Pre-Dev: QA Check
10. Pre-Dev: Tech Review
11. Development
12. Post-Dev: Design Review
13. Post-Dev: Code Review (the key process change)
14. QA Testing → Deploy → Done
15. Advantages
16. Where the 136 SP went (resource breakdown)
17. Where the bottlenecks are
18. What each role needs to do
19. Want to try it? (CTA + contacts)
20. Thank you (animated outro)

## Stack

- One `index.html` file (~70 KB)
- DM Serif Display + Inter (Google Fonts)
- No build step, no dependencies. Just open the file.

## Fork it

If you want to adapt the talk for your own context:

1. Copy `index.html` and edit the slide content
2. Update the title slide
3. Update slide 5 (cost story) with your numbers
4. Update slide 19 (CTA) with your links
5. Deploy anywhere static-hosting capable (Vercel, GitHub Pages, Cloudflare Pages, S3)

The structure is opinionated — 11 stages, 5–10× framing — but the design system (light minimalist, no AI slop, no purple gradients) is reusable.

## License

[MIT](../LICENSE) — same as the rest of the playbook. Adapt and ship.
