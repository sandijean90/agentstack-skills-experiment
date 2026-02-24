# Skill Authoring Learnings

## Voice & Structure
- Write instructions directly to the agent in second person and imperative ("do this"), not third person ("the agent should")
- Be declarative and prescriptive — rules and procedures, not narratives
- Use active voice; passive voice obscures who is responsible
- One skill = one domain; don't let scope creep in

## Content Design
- Lead with the single most important orienting principle (like "your agent code doesn't change")
- Put critical high-level guidance first; edge cases and specifics after
- Use consistent terminology throughout — don't synonym-swap
- Spell out edge cases explicitly; agents don't infer intent
- Separate conceptually different phases (e.g. Setup / Wrap / Deploy) even within a checklist
- Avoid redundancy across sections — pick one canonical place for each piece of information
- Include at least one complete, working code example as a concrete anchor

## Description Fields
- Description fields should read like a list of triggers, not an elevator pitch
- Front-load the most unambiguous trigger phrase
- Enumerate specific technical artifacts and actions users might actually type (`Server()`, `@server.agent()`, specific filenames)
- Keyword-dense and scannable beats well-written prose for routing reliability

## Gaps to Watch
- Specify behavioral defaults the agent might otherwise improvise — naming conventions, whether to ask before making choices, etc.
- Make explicit what the agent should *not* change, not just what it should do

## File Organization
- SKILL.md is the entry point only — if a section feels like reference material, it belongs in `references/`
- Name reference files by what they *are*, not what they *contain* (`DEPLOYMENT.md` not `STEPS_FOR_DEPLOYING.md`)
- Keep each reference file focused on one concern — agents load these on demand, smaller = less wasted context
- Use relative paths for all cross-file links so skills are portable across tools and mount points

## Progressive Disclosure
- Structure content in three tiers: description (~100 tokens, always loaded), SKILL.md body (loaded on activation), reference files (loaded on demand)
- Front-load the checklist/steps in SKILL.md — agents should be able to act without reading references
- Reserve reference files for "why" and edge cases; SKILL.md should cover "what" and "how" at a high level

## Code Examples
- Every example should be copy-paste runnable, not pseudocode — agents will use it literally
- Show the before *and* after when the skill involves transforming existing code
- Label what's "your code (unchanged)" vs what's wrapper — makes the boundary explicit
- Prefer one complete example over several partial ones

## Portability & Tooling
- Avoid tool-specific assumptions in the skill body (mount paths, env conventions) — those belong in compatibility metadata
- The `name` field must match the directory name exactly — enforce this in CI
- Don't put runtime paths (like `/mnt/skills/`) in the repo — those are deployment concerns, not skill concerns
- Test your skill against at least two different agent implementations (e.g. Claude and Cursor) before publishing

## Maintenance
- Treat the description field like an API contract — changing trigger phrases breaks routing silently
- Version your skills in metadata; breaking changes should bump the version
- When a skill grows past 500 lines, that's a signal it's become two skills, not just a refactor problem