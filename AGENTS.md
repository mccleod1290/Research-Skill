# AGENTS.md

This repo finds niche blogs. It does not study.

Study is a separate pipeline: <https://github.com/mccleod1290/Study-OS>

There is no build and no install step for this repo. Do not scan the tree for a skill.

## Read first

Read `skills/research/SKILL.md` before a hunt. Follow it. It is the rules, the tools, the guidelines, the plugin, and the extra features. The golden rule, date pinning, first-order sources, and the Medium rule sit beside those. They do not replace them. Names live in `skills/research/references/trusted.md`. Do not invent a second workflow.

`research skill v3/SKILL.md` only points there. `archive/` is dead work, including `archive/v3-draft/`. Do not open it to borrow a method.

## Rules that bind even before the skill is open

1. Deep research only for a specialized niche in offensive security, AI security, or vulnerability research. Azure architecture, mentorship, and a broad survey are not this repo. Say so. Point study to Study-OS.
2. One topic at a time. Four lanes of that topic are allowed. They share only `seen.md` and `rejected.md`. If the user asks for more topics, one parallel subagent per topic, separate folder, no shared files.
3. Set a goal before the first search. Default: niche blogs with unique knowledge common training data does not already hold. We are the good guys. Cite the blog. Do not write attack steps.

## Tools

`fetch`, then `curl`, then `curl` with a browser user agent, then headless PinchTab text. Not Playwright. Not screenshots.

Do not start with `web_search`. That is an API. If you need one, say why in the round log first.

On a 403, decide block or gone. Do not drop the URL. Do not quit to save tokens. A dead page goes to the Wayback Machine. The skill has the steps.

## Plugin

Check for the `ralph` tool every run. If it is in the session, call it with `maxRounds` 20. If it is not, it is not running. Say so, and spawn 4 parallel subagents, 5 turns each. Do not repeat a URL or a rejected claim. The skill has the file rule.

## Write

Only under `research/runs/{topic-slug}-{YYYY-MM-DD}/`. One topic, one folder.

A run is done when `research_links.md` has the goal and a 1-2 sentence brief on every blog, `rejected.md` holds the non-patterns, and no shipped URL was repeated.
