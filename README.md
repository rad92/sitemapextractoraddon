# Sitemap Extractor With SEO Data & GSC Connection

A Chrome extension that finds any website's sitemap, lists every URL in it, and reads each page's SEO metadata. Works on any site in any language — nothing is region-specific.
______________________________________________________________________________________________________________________________________________________________________________________
# How to Install?
1. Unzip the folder somewhere permanent. Chrome loads the extension from this folder every time it starts, so don't unzip it into Downloads and then clear it out. Something like Documents/chrome-extensions/sitemap-extractor is fine.
2. Open Chrome and go to chrome://extensions (type it into the address bar).
3. Turn on Developer mode using the toggle in the top-right corner.
4. Click Load unpacked (top-left) and select the sitemap-extractor folder — the one containing manifest.json, not the zip and not its parent folder.
5. The extension appears in the list. Click the puzzle-piece icon in Chrome's toolbar and pin Sitemap Extractor so it stays visible.

# Chrome will warn that the extension can "read your data on all websites." That is required: the extension fetches sitemaps and pages from whatever domain you point it at, and Chrome cannot express "any site the user types in" more narrowly. Everything runs locally in the extension; nothing is sent anywhere.
______________________________________________________________________________________________________________________________________________________________________________________
# How to Use It?
To use it: open any site, click the icon, click Extract. Or type a domain into the box — you don't need to be on the site.
After editing any file, return to chrome://extensions and click the refresh icon on the extension's card.
______________________________________________________________________________________________________________________________________________________________________________________
# What it Does?
1. Finds the sitemap
Reads /robots.txt and follows every Sitemap: line. This is the authoritative source, and it handles relative paths plus sitemaps hosted on a different domain or CDN.
If robots.txt declares none, probes 19 common paths covering WordPress, Yoast, RankMath, Shopify, Squarespace, Wix, Ghost, Drupal, Magento and Next.js conventions.
Follows <sitemapindex> files recursively, 5 at a time, up to 200 sitemaps.

2. robots.txt rules
Runs automatically. The tool already downloads robots.txt to find the sitemap, so the rules come free — it parses the Disallow/Allow directives and tests every sitemap URL against them.
The finding that matters: a URL in the sitemap that robots.txt blocks. That's two contradictory instructions — index this, don't crawl this — and nothing in the page's own markup hints at it. Worse, a blocked page's noindex tag can never be read, so blocking is not a way to deindex anything. Blocked URLs are grouped by the rule that blocked them, with examples, and a blocked URL that still earns impressions gets its own summary chip: Google is showing a page it was told not to crawl, so it has no title or description to display.
View contents shows the file exactly as served, with the lines that actually blocked something highlighted and labelled with how many sitemap URLs each one caught — so the rule and its consequence are visible together. Copy takes the raw text, and Open in tab loads the live URL. Those controls appear even when the file couldn't be read, since that's precisely when you need to check it yourself.
Matching follows Google's rules rather than a naive prefix test: * wildcards, $ anchoring, the longest matching pattern winning, ties going to Allow, case-sensitive paths, the query string being part of the match, and user-agent group selection (the most specific matching group, falling back to *). Evaluated as Googlebot. Verified against Google's own documented precedence examples, including the cases where Allow overrides Disallow.
3. lastmod per sitemap file
Expanding the sitemap count opens a sortable table with two views.
By sitemap file — each file with its URL count and the date range of its contents. By path segment — each section of the site (/motoring/, /brakes/, /brake-discs/) with how many URLs sit under it and their date range. On a large site this is usually the more useful overview, because it maps to sections rather than to however the CMS happened to split its files. Page slugs are excluded, so you get directories rather than a row per page.
Any column sorts, and Show URLs filters the main list to that file or section, with a clearable banner so the scope is never silently applied.
For each file the table also separates three notions of "when did this change", kept apart because they disagree in informative ways:
index says — the lastmod the parent index declares for this child file
server says — the file's own HTTP Last-Modified header
newest URL — the most recent lastmod among the URLs inside (hover for the full range)
Plus how many URLs in the file carry a lastmod at all, since partial coverage is common and weakens the signal.
The failure this exposes: an index entry whose lastmod is older than the newest URL inside the file. Google uses the index's date to decide whether re-fetching that child sitemap is worthwhile, so a stale claim means newly-published URLs can sit there unread indefinitely. It's flagged in red in the accordion and as an error in hygiene. An index that declares no lastmod at all for its children is flagged too — Google then has no way to tell which files changed and must re-fetch every one.
Each URL also records which sitemap file it came from, exported as sitemap_file in the CSV. The JSON export now carries the full sitemap file records alongside the URLs.
______________________________________________________________________________________________________________________________________________________________________________________
4. URL tree
A directory tree of the whole sitemap, drawn left to right. Two label modes: Compact starts at the hostname with bare segment names; Full path mirrors the address bar (/ → https/ → host/ → each directory).
What the table can't show is shape — whether a site is flat or deeply nested, where the bulk of URLs sit, and which branch holds the problems. Node colour is inherited from the worst descendant, so a red dot at the top tells you which section to open without expanding anything. Green is clean, amber is minor issues, red is severe.
Click any node to filter the list to that branch. Depth can be capped, parents with more than 25 children collapse into a "… N more" node, and Download SVG exports the diagram with its styles inlined so it stands alone in a report.
______________________________________________________________________________________________________________________________________________________________________________________
5. Structured data report
Validated during the meta pass, since every JSON-LD block is already being parsed.
Rules follow Google's rich-result requirements, not the whole of schema.org — a property schema.org merely allows isn't worth flagging, while one Google requires for eligibility is. Errors mean the page can't produce the rich result; warnings mean Google recommends the property and the result is weaker without it.
Covers Article/NewsArticle/BlogPosting, Product and Offer, LocalBusiness (including subtypes like Plumber and Restaurant), Organization, WebSite, BreadcrumbList, FAQPage, Event, Recipe, VideoObject, Review, AggregateRating, JobPosting, Course and more. Beyond missing properties it catches shape errors a presence check can't: breadcrumb items without position, FAQ entries whose answer text is empty, a ratingValue above its own bestRating, blocks with no @context, and malformed JSON — reported per block, so one bad script doesn't hide the valid ones.
A property present but empty ("", [], {}) counts as missing. That's a common CMS output and worse than an absent property: it looks populated in the source and counts for nothing.
The panel rolls findings up across the site — how many pages carry JSON-LD, which types and how often, and each distinct problem with a page count and examples. Pages using microdata or RDFa are noted separately rather than reported as having no structured data, since those syntaxes are valid but aren't validated here.
______________________________________________________________________________________________________________________________________________________________________________________
6. Sitemap hygiene
Checks the sitemap as a document rather than as a list of pages, all from data collected during the crawl:
Files over Google's 50,000-URL or 50 MB limits, and empty sitemap files
Invalid lastmod — Google ignores a date it can't parse, so a wrong format silently throws away the signal. Dates in the future are flagged separately
http:// URLs in an https:// sitemap
Duplicate URLs appearing in more than one sitemap file
# fragments, which Google drops, collapsing several entries onto one page
Tracking parameters (utm_*, gclid, fbclid and friends), which usually create duplicates of the clean URL
Cross-host URLs and mixed-case paths
7. Checks status (fast, opt-in)
______________________________________________________________________________________________________________________________________________________________________________________
Check status runs a HEAD request per URL — no page bodies are downloaded, so it's roughly 4–8× faster than the full meta pass and stays practical on sitemaps far too large to crawl properly. It answers the question most people open a sitemap tool to ask: does everything in here actually resolve?
It reports status code, response time, content type and size, plus:
Redirects, classified by comparing the requested URL with where you landed: http→https, www added/removed, slash added/removed, case change, different host, different path.
X-Robots-Tag headers, which block indexing exactly like the meta tag and are invisible if you only read HTML.
Link: headers carrying rel=canonical or rel=alternate hreflang — the only way to annotate PDFs and other non-HTML files.
Slow pages (over 2s), sortable with Slowest first.
It automatically falls back to GET for the servers and CDNs that refuse HEAD (405, 403, 501 and friends), cancelling the response body immediately so the page still isn't downloaded.
One honest limitation: the sweep reports the final status and destination, not each hop. Chrome returns an opaque-filtered response for a manual-redirect request — status 0, no readable Location — so extension fetch genuinely cannot see whether a redirect was a 301 or a 302. Reading per-hop codes would need the webRequest permission and a background service worker. That's a deliberate trade: a scarier install prompt for one extra field.
8. Validates hreflang
______________________________________________________________________________________________________________________________________________________________________________________
Runs automatically as soon as there's anything to check, and gets sharper after each pass.
Single tags are easy to eyeball. The failures that actually cost traffic are relational, and because the tool holds the whole sitemap plus every page's alternates, it has the complete graph:
Check	Why it matters
Missing self-reference	Google documents this as invalidating the entire set
Non-reciprocal	A points at B, B doesn't point back — Google silently ignores it
Incomplete set	one member forgot two languages, so the cluster is inconsistent
Conflicting target	the same tag resolving to different URLs across the set
Broken / redirecting / noindex target	alternates aimed at pages that can't rank
Canonical conflict	alternate points at a URL that canonicalises elsewhere
Invalid codes	see below
Duplicate tag, relative href, missing x-default	
Pages are grouped into clusters with union-find, so pages linked by any chain of alternates end up in one set even when no single page declares the whole thing.
Code validation is stricter than Intl. Chrome's Intl.DisplayNames resolves UK to "United Kingdom" through CLDR aliasing — but en-UK is invalid hreflang and is the single most common mistake in the wild. So an explicit blocklist sits on top, covering CLDR aliases (UK, EU, EZ, UN, QO), withdrawn codes (AN, CS, YU, SU, TP, ZR, …), placeholders (ZZ, XA) and numeric M49 regions, each with the correct replacement. Country codes typed where a language belongs (cn, jp, gb, dk, gr, cz, at) are rejected with the fix. Codes that are valid languages but commonly confused with countries (uk Ukrainian, se Northern Sami, br Breton, ch Chamorro, be Belarusian) get an advisory note instead of an error.
Redirecting URLs are excluded from the graph. If a URL redirects, anything parsed came from the destination's HTML — crediting those tags to the requested URL invents a reciprocity failure for every language on the destination page.
______________________________________________________________________________________________________________________________________________________________________________________
9. Reads each page (opt-in)
Click Fetch page meta to load every URL and extract:
Field	Sources, in order
SEO title	<title> → og:title → twitter:title
Meta description	meta[description] → og:description → twitter:description
Date published	JSON-LD datePublished → article:published_time → itemprop → DC.date.issued → <time pubdate>
Date modified	JSON-LD dateModified → article:modified_time → og:updated_time → itemprop
Language	<html lang> → og:locale → script detection
hreflang	link[rel=alternate][hreflang], plus sitemap alternates
Canonical, robots, H1 (+count), schema type, word count, charset	direct from the markup
robots also includes the X-Robots-Tag HTTP header, which is just as binding as the meta tag and invisible if you only read the HTML.
This pass is opt-in because it makes one request per URL and downloads each page. The buttons show counts and time estimates for both passes before you commit; Stop cancels either one and keeps what's already loaded.
______________________________________________________________________________________________________________________________________________________________________________________
10. Joins Search Console performance (optional)
See the Search Console setup section below. Once connected, every finding can be ordered by what it actually costs you.
______________________________________________________________________________________________________________________________________________________________________________________
11. Is a URL indexed by Google?
Check indexing inspects the URLs currently on screen, so the filter is the selection mechanism — filter to pages with traffic, or pages failing a check, then inspect those.
Per URL it reports whether Google has it indexed, Google's own wording for why ("Crawled – currently not indexed", "Discovered – currently not indexed", "Blocked by robots.txt"), the last crawl date, and the canonical Google actually chose. That last one is the most valuable field in the API: when Google's canonical differs from the page's own, Google has overruled your canonical tag, and nothing in the page's markup tells you that.
This is a sampling tool, not a sweep. Google allows 2,000 inspections per day and 600 per minute per property, and each takes several seconds. The tool inspects 100 per run by default, keeps a local tally of what you've used today, and stops cleanly when Google returns a quota error rather than hammering the API. A quota error doesn't say which limit was hit, so wait a minute for the per-minute cap or a day for the daily one. It also needs owner or full user permission on the property — restricted users get a 403.
______________________________________________________________________________________________________________________________________________________________________________________
12. Query-level data
Load queries pulls the page-and-query dimensions, which unlocks three things:
Top queries per page, shown in each row, so you can see what a page is actually being found for versus what you wrote it for.
Cannibalization — queries where two or more of your own pages compete. Google picks one and the others dilute it. Listed by total impressions, with each competing page's position, so you can see which to consolidate.
Title/query mismatch — pages whose title shares no words at all with their top query. Deliberately conservative: anything cleverer needs per-language stopword lists and would fire constantly on perfectly good titles.
13. Internal link graph
Free, in the sense that it costs no extra requests — the meta pass already downloads every page's HTML, so the links are extracted from a document that's already parsed.
Orphans — pages nothing internally links to. Cross-referenced with Search Console, an orphan that still earns impressions is the strongest version of this finding: Google found it despite your site structure.
Internal links to broken pages, with the anchor text so you can find them in the content. Use Check linked URLs for this: broken links almost always point at URLs a sitemap doesn't list — deleted pages, typos, stale hrefs — so the tool probes those targets directly rather than only checking links between sitemap members. A link to /dd-dwd returning 401 is found; it would be invisible to a sitemap-only check.
Internal links to redirects, which waste a hop on every crawl.
Inbound link counts and anchor text per page, sortable by fewest inbound links.
The internal links panel lists every linked URL that isn't in the sitemap, each with the status it returned and the page and anchor text it was linked from. They're listed whether or not they're broken, so you can always see exactly what was checked and what came back rather than a summary verdict.
Note that requests omit cookies, so links into a members' area or anything behind a login will report 401 or 403. Those are real responses for a logged-out visitor and for a crawler, but they aren't necessarily a mistake — check the status code before acting.
Two correctness details worth knowing. www.example.com/x and example.com/x are treated as the same page here — real sites mix both in their markup, and not unifying them produces a flood of false orphans. And orphan detection switches itself off when fewer than half the URLs were read, because a page can only look unlinked if you never fetched the page that links to it; the panel says so rather than reporting nonsense.
______________________________________________________________________________________________________________________________________________________________________________________
14. Per-page detail
Expanding a row opens a two-column table rather than a run of inline pairs — status, performance, index state, metadata, links and headings all read down one column.
Above it sits a SERP preview: an approximation of the Google result, built from what the page actually declares. Title and description are truncated at Google's usual desktop cut-off, measured in display width so CJK is handled correctly, and the preview tells you when either is being cut. Structured data feeds it directly — BreadcrumbList replaces the URL path with the breadcrumb trail, AggregateRating adds stars, FAQPage adds expandable rows, datePublished adds the date prefix. A footnote names which annotation produced each feature, so it doubles as a check that your markup is doing something.
Headings are captured at every level, not just H1: a per-level count, the total, and the full outline indented by depth. Two structural problems are flagged — skipped levels (an h2 followed by an h4 breaks the outline for anything reading structure rather than styling) and empty heading tags, which are usually a theme using headings for spacing. All six counts plus the total and any skipped levels export to CSV.
______________________________________________________________________________________________________________________________________________________________________________________
15. Flags problems
The summary chips are clickable filters. Detected:
Missing, duplicate, too-long or too-short titles and descriptions
noindex pages sitting inside the sitemap
Canonical tags pointing somewhere else
Redirects
Missing or multiple H1s
Invalid JSON-LD
Pages that failed to load
Stale lastmod — the sitemap's date disagreeing with the page's own dateModified, which is a common reason Google stops recrawling updated pages
Each row's dot is green (clean), amber (minor), or red (needs attention).
______________________________________________________________________________________________________________________________________________________________________________________
# Search Console setup
1. Go to the Google Cloud Console and create a project (any name).
2. APIs & Services → Library, search for Google Search Console API, click Enable. Skipping this is the most common failure; the tool will tell you if the API is off.
3. APIs & Services → OAuth consent screen. Choose External, fill in an app name and your email. Under Audience, add your own Google account as a Test user — otherwise Google blocks the sign-in.
4. APIs & Services → Credentials → Create credentials → OAuth client ID. Application type Chrome Extension. Paste the ID above into the Item ID field.
5. Copy the client ID it gives you (it ends in .apps.googleusercontent.com).
6. Open manifest.json in a text editor and add this block, replacing the placeholder. Mind the comma after the "icons" block:
7.    "oauth2": {
     "client_id": "PASTE_YOUR_CLIENT_ID_HERE.apps.googleusercontent.com",
     "scopes": ["https://www.googleapis.com/auth/webmasters.readonly"]
   }

8. Reload the extension at chrome://extensions (circular refresh icon on its card).
Extract a site, open the Search Console panel, click Connect, and approve the consent screen. Google will warn that the app isn't verified — that's expected for an extension only you use. Click Advanced → Go to (your app name).
# The scope is read-only. The extension cannot change anything in your Search Console.
______________________________________________________________________________________________________________________________________________________________________________________
# Client report (PDF)
PDF report opens a printable report in a new tab; Chrome's print dialog saves it as a PDF. There's a field for the client name and your own, which appear on the cover.
Extensions can't write a PDF directly, and bundling a PDF library would mean rebuilding layout, pagination and font handling. Printing a properly styled page is both simpler and better output: Chrome's own renderer handles pagination, text stays selectable and searchable, and the result is vector rather than a bitmap.
It contains everything the tool found. Every finding lists the URLs it applies to rather than just a count. Every URL appears in full, grouped under the sitemap file it came from, each with its search preview, metadata, heading outline, structured data, queries and any problems. Search Console is reproduced in full — totals, CTR benchmarks by position, every page with data, every query, cannibalisation — and robots.txt is included as served.
A Detail control in the toolbar switches between the full version and summary only, without regenerating. On a large site the full version runs to hundreds of printed pages, so the popup estimates the length before you open it.
The report leads with the findings a client should care about, in plain language — sitemap URLs that don't resolve, pages blocked by robots.txt, noindex pages in the sitemap, orphans earning impressions, pages ranking well but rarely clicked. Then the site-structure diagram, a pages to fix first table ordered by impressions so the work is ranked by what it costs rather than by page order, and a section per check.
# Open in a tab
The ⧉ button top-right reopens the extension as a full browser tab. Chrome destroys popups the moment they lose focus, which cancels a long run — a tab survives, uses the whole window, and is the right way to audit a large site.
