# Web Source Reliability for Research Scraping

Last verified: 2026-06. Status can change — re-verify when a previously-working source fails.

## Reliable Sources (webclaw -f text or -f llm)

| Source | Notes |
|--------|-------|
| Goodreads book pages | Best for reviews/ratings. Use `-f text` for full review content (45K+ chars for popular books). |
| Amazon product pages (.sg domain) | Returns aggregate ratings + per-star %. Individual reviews behind sign-in. |
| Wikipedia | Consistently extractable. Pages with no article return a "does not have an article" message. |
| NirandFar.com (author site) | Long-form articles work well with both -f text and -f llm. Some old URLs 404 (site restructured). |
| The Guardian | Excellent long-form content. Both -f text and -f llm work. Paywall-free for most articles. |
| FourMinuteBooks | Clean summaries, good for book overview content. |
| BBC | Search/index pages work. Individual article URLs sometimes 404 even for older content. |

## Unreliable / Blocked Sources

| Source | Failure Mode |
|--------|-------------|
| DuckDuckGo HTML search | Returns empty HTML (no result links) — confirmed broken as of 2026-06. |
| Medium (medium.com) | Cloudflare "Just a moment" challenge wall. |
| The Marginalian (formerly Brain Pickings) | Old URLs 404 after domain rename + site restructuring. |
| TechCrunch | Older articles return 404 even for historically-cited pieces. |
| HBR (hbr.org) | Old article URLs return "Page Not Found". |
| Psychology Today | URL structure changed; old blog post URLs 404. |
| StoryShots | Sucuri/BigSots WAF bot protection (CAPTCHA wall). |
| Readingraphics | BigSots WAF bot protection. |
| NYTimes | Requires JS; webclaw returns empty or sign-in redirect. |
| Wired | Old article URLs 404; returns generic "Oops" page. |
| NNGroup (nngroup.com) | Blocks webclaw (noted in deep-research skill). |
| Interaction Design Foundation | Anti-bot CAPTCHA. |
| James Clear book summaries | Pages removed/restructured. |
| Samuel Thomas Davies summaries | 404 on book summary URLs. |

## Workarounds

- **When DDG fails**: Use Brave Search (search.brave.com) via browser or `web_search` tool if available. Google blocks from this IP.
- **When a site 404s**: Try the Wayback Machine (web.archive.org/web/*/URL) — webclaw can fetch archived pages.
- **For paywalled content**: Try `-f llm` first (sometimes the LLM can extract from partial content). Fall back to browser with CloakBrowser for fingerprint-protected sites.
- **For JavaScript-heavy sites**: Use browser tools instead of webclaw.
