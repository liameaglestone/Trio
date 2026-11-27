---
description: Check upstream Trio repos for new commits, merge them into main and push. Never builds.
allowed-tools: Bash(git:*), Bash(python3:*)
---

Sync this fork's main with its upstreams. Do not trigger any build or workflow.

Upstreams (remote name, URL, branch):
- nightscout  https://github.com/nightscout/Trio.git  dev
- nightscout  https://github.com/nightscout/Trio.git  feat/tandem
- dnzxy       https://github.com/dnzxy/Trio.git       feat/glooko-service
- t1dude      https://github.com/t1dude/Trio.git      feat/settings-analysis

Steps:
1. Check out main and make it match origin/main. Add any missing remotes, then fetch each
   branch with --no-recurse-submodules. Ignore the .gitmodules warnings in old history.
2. For each upstream, list the commits in HEAD..upstream. Skip any that only bring in
   content main already has (for example a merge of dev into a feature branch).
3. Merge each upstream that has real changes with --no-ff, using the commit message
   "Merge <owner>/Trio <branch> (<APP_DEV_VERSION>) into main".
4. Conflicts:
   - Localizable.xcstrings: do a three-way JSON merge of "strings" (stages 1, 2 and 3).
     Keep upstream's key order and insert our extra keys after the key that comes before
     them in our version. Write empty objects as "{\n\n<indent>}" so the formatting
     matches Xcode's. If the same key changed on both sides, stop and ask me.
   - project.pbxproj and Package.resolved: accept upstream's package removals, but only
     after checking that nothing in Trio/, TandemKit or GlookoServiceKit still imports
     the package. Keep the fork's own packages (ASN1, BigInt, SwiftECC, Digest for
     TandemKit).
   - Anything else, or anything you're not sure about: abort the merge and report back.
     Don't guess.
5. After merging, check whether upstream removed any packages, files or symbols that
   fork-only code uses.
6. Run `git submodule update --init` so the submodule checkouts match the new pointers.
   Never commit a submodule pointer change that upstream didn't make.
7. Push to main only as a fast-forward. Confirm with `git ls-remote origin main`.
8. Report: what came in from each upstream, how each conflict was resolved, what was
   not verified, and a reminder that the Sunday scheduled build will pick it up.
