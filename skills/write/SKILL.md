---
name: write
description: "Strips AI writing patterns and rewrites prose to sound natural in Korean, Chinese, or English. Only activates on explicit writing or editing requests. Not for code comments, commit messages, or inline docs."
when_to_use: "글 써줘, 원고 고쳐줘, 다듬어줘, AI 느낌 빼줘, 한 문단 써줘, draft, edit text, proofread, sound natural, polish, rewrite"
metadata:
  version: "3.18.0"
---

# Write: Cut the AI Taste

Prefix your first line with 🥷 inline, not as its own paragraph.

Strip AI patterns from prose and rewrite it to sound human. Do not improve vocabulary; remove the performance of improvement.

## Pre-flight

1. **Text present?** If the user gave only an instruction with no actual prose to edit, ask for the text in one sentence. Do not proceed.
2. **Audience locked?** If the intended audience is unclear and cannot be inferred from the text (blog reader vs RFC vs email), ask before editing. Junior engineer and senior architect prose should read completely different.
3. **Language detected from the text being edited**, not the user's command:
   - Contains Korean characters → load `references/write-ko.md`
   - Contains Chinese characters → load `references/write-zh.md`
   - Otherwise → load `references/write-en.md`

Read the loaded reference file. Then edit. No summary, no commentary, no explanation of changes unless explicitly asked.

## Hard Rules

- **Meaning first, style second.** If removing an AI pattern would change the author's intended meaning, keep the original.
- **No silent restructuring.** Do not reorganize headings, reorder paragraphs, or merge sections unless structural changes are explicitly requested. Edit in place.
- **Stop after output.** Deliver the rewritten text. Do not append a list of changes, a justification, or a closer.

## Bilingual Review Mode

Activate when: mixed Korean/English, "Korean copywriting", "bilingual consistency", "release notes"

**Korean rules**:
- Keep spacing natural around Korean and English terms.
- Do not mix punctuation styles without a reason.
- Keep terminology consistent across all instances.

**English in Korean documents**: Flag unexplained English, suggest translation or add context.

**Bilingual pairs**: Confirm EN and KO versions convey the same meaning; mark translation loss.

## Release Note Template Mode

Activate when: "release", "changelog", "version", "release notes"

Generate from commit messages:
- **Breaking Changes**
- **New Features**
- **Fixes & Improvements**
- **Deprecations**

Format: tw93/Mole style (numbered list, bold label, one sentence on user effect, bilingual).

## Output

Return only the edited prose. If the text was truncated or if multiple versions were possible, note that in one sentence after the body. Otherwise, no wrapper, no preamble, no postscript.
