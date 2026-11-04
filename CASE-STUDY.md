# KeyNest: case study

KeyNest is a Windows app that keeps API keys, tokens and secrets in an encrypted vault on your own PC, and finds keys left lying around in plain text, especially by AI coding tools. It is an early preview. It isn't available to download yet, but you can [try it in your browser](https://keynest-demo.pages.dev/) with made-up data. The source code stays private.

## The brief

People who build software with AI coding tools collect API keys fast: one for the model, one for payments, one for the database, one for email. Most of them end up in `.env` files, notes and chat history. I wanted a tool for that person: offline, no account, and plain enough for a beginner.

I am not a developer. I led the project and made the product, design and money decisions.

## What the research said

Before deciding what to build beyond the basics, I read what developers say in public. A search of Hacker News from January 2024 onwards found 673 comments about secret managers. 52 of them were about AI coding tools reading or recording keys, 39 of those in the last 12 months, the fastest-growing theme. 72 were about subscription prices and 42 about wanting a vault that stays on their own machine.

This is one source, and a technical one, so the numbers show how often a topic comes up rather than how many people have the problem. It was enough to choose a direction, not enough to prove demand.

## What it does

- An encrypted vault for keys, with providers, tags, notes and expiry dates.
- Projects with stages, where a variable can point at a saved key, plus `.env` import and export.
- Quick Access: a shortcut opens a search box over any app, and Enter copies a key. The clipboard is cleared again afterwards.
- Health: expired, expiring, unchanged and duplicate keys.
- Two checks for loose keys: one in the folders AI coding tools use, one in a folder you choose.
- A guide to replacing a leaked key in the safe order.

## What happened

**The first real scan was mostly false alarms.** On my own PC the AI tool check listed hundreds of files, each told to "treat it as leaked". Almost all were old backup copies of KeyNest's own source code that a coding tool had kept, and the "keys" in them were made-up test values. The check matched text that looked like a key and assumed the worst.

The fix was to stop pretending to know more than it can. KeyNest can't tell whether a key works without sending it to the service, and it never sends anything. So it now asks two separate questions:

1. Does the value look real? This is judged from the value alone: sample words, runs like 123456, too little randomness, or far too short for the service. It is never judged from where the file sits, because beginners often paste a real key into a file called `test.py`.
2. Could someone else reach the copy? A key in web page code, under a name like `VITE_` that a build puts into the page, or in a file Git sends to an online copy goes to the top. A key that is only on this PC is described as not leaked, but worth moving into the vault.

Every key appears once, however many files hold it. All 98 key-shaped values in 210 old versions of KeyNest's own files now land in "probably not real". Random values shaped like each service's real keys are wrongly called made up at worst about 1 in 6,450.

Checking the published key formats also turned up a quiet miss: the scan insisted on a digit, so it never found Hugging Face tokens, which are letters only, and missed about 1 AWS key in 28.

**The full PC scan looked frozen.** It was not. It spent most of its time in folders of other people's code: 96% of the 99,863 files it would have opened in one user folder were installed tool documentation and packages. Those folders are now left out, progress counts files rather than folders, and the scan keeps running if you move to another screen.

**A test read my real folder.** A bug made the app forget the folder chosen for a scan when you left the screen, and an automated test then scanned my real user folder instead of its throwaway one. No key values were saved or shown. The bug is fixed, and test builds now refuse to scan anywhere outside their own folder.

## Decisions that mattered

**Nothing leaves the PC.** No accounts, analytics, crash reports or cloud copy. That rules out checking whether a key still works, and the app says so instead of guessing.

**Sort, never hide.** A key that looks made up is folded away, not deleted from the list, because a real key wrongly hidden is worse than a false alarm.

**Show as little as possible.** Scans store where a key was found and a keyed fingerprint, never the key. Show reveals the start and last 4 characters for 20 seconds.

**Replace in the safe order.** The guide only unlocks "delete the old key" after the new key is saved, in place and tested, because that is the only step that can break a live project.

**Written for easy reading.** The Health page uses larger text with more space between lines, short sentences and full-strength colour, so it is easier to read for people with dyslexia.

## Results

- 176 automated tests of the vault, crypto, scans, backups and projects pass (3 November 2026), and window tests that drive the real app pass on the production build.
- A source-only security review, quick profile, found no confirmed vulnerability. It raised two leads, and both were fixed with tests (3 November 2026). One lead: restoring a doctored backup could have weakened how the master password is protected. The other: a doctored path could have added instructions to File Explorer.
- Run on my own PC, where the sorted scan results were checked by hand and looked right (1 November 2026).

## What next

- An independent security review and code signing, before inviting strangers to rely on it.
- Show it to five people who build with AI coding tools, and keep going only if at least three want it.
- A command-line tool that starts a program with a project's keys without writing them to a `.env` file.
