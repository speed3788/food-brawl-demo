# Getting this into GitHub

Written assuming you've never used it. Fifteen minutes.

## What GitHub is for here

A backup of your code with a full history, and the place Claude Code reads from and writes
to. You don't need to understand branches or pull requests yet.

## One-time setup

1. Install **GitHub Desktop** (desktop.github.com). It does the same thing as the command
   line with buttons, and you'll make fewer mistakes.
2. Sign in with the account you just made.
3. Install **Node.js** (the LTS version) and **Xcode** from the Mac App Store if you're
   building for iPhone. Xcode is a large download — start it now.

## Making the repository

1. In GitHub Desktop: File → New Repository.
2. Name: `food-brawl`. Local path: wherever you keep projects.
3. **Keep it private.** Your Google Places key will live near this code.
4. Tick "Initialize with a README".
5. Publish repository (top right). Leave "keep this code private" checked.

## Getting the design bundle in

1. Unzip the handoff bundle into the repository folder, so you have
   `food-brawl/design_handoff_food_brawl/`.
2. GitHub Desktop will show the files as changes. Write "add design handoff" in the summary
   box, click Commit, then Push origin.

That's the loop, forever: change files, commit with a short note, push.

## Pointing Claude Code at it

1. Install Claude Code (`npm install -g @anthropic-ai/claude-code`), then run `claude` from
   inside the `food-brawl` folder.
2. Paste the contents of `CLAUDE_CODE_FIRST_PROMPT.md`.
3. Let it work a step at a time. Commit after each step that leaves the app running — that
   way you can always get back to something that worked.

## Two habits worth having from day one

- **Never commit secrets.** Your Google Places key goes in a `.env` file, and `.env` goes in
  `.gitignore`. If you ever paste a key into a committed file, treat it as burned and rotate
  it.
- **Commit small and often**, with notes you'd understand in a month. "port frog jump loop"
  beats "updates".
