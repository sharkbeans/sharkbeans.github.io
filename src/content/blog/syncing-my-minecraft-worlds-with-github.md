---
title: "Syncing minecraft saves/worlds using github"
description: "trying to sync a vanilla Minecraft world between my work/home PC and laptop"
pubDate: 2026-08-21
tags:
  - git
draft: true
---

I play Minecraft on three machines: a work PC, my home desktop, and a Thinkpad. Same world, three copies, and the classic failure mode — build something on one machine, forget to copy the folder, overwrite it later from a two-week-old save on another.

Cloud-synced folders are the obvious answer and also the wrong one. Dropbox-style sync happily corrupts a live world, region files are huge, and there's no history when it goes wrong anyway.

So I forked [MineCommit](https://github.com/HairlessVillager/minecommit), a Rust tool that already does the hard part — it turns a Minecraft save into something Git can actually diff and compress properly instead of a 60MB blob per backup — and built the part I actually needed: a remote. Fetch before you play, push after, and if both sides changed since the last sync it just stops and tells you, instead of trying to be clever about merging two different afternoons of building. There is no sane automatic merge of two divergent worlds. That was the whole idea, and it worked.

Then I published a version of this post and kept going. Four days later the version number had gone from 0.4 to 0.13 and I'd lost count of the releases somewhere in the twenties. Almost none of that time went into the sync logic itself. It went into everything around it that breaks the moment someone who isn't me tries to use it.

## auth 

First pass: sign in by pasting a GitHub access token. Works, but "generate a token and paste it into a desktop app" is a very developer thing to ask someone to do, and I was trying to build this for the users that just wants to play Minecraft, not debug OAuth scopes.

Second pass: device code flow instead — GitHub shows you a code, you type it on github.com, done, nothing pasted. Better, except it still asked for full read/write access to every repo on the account, because that's the only scope OAuth apps get offered. For a tool that touches one folder of Minecraft backups, that's a wildly bigger blast radius than it needs.

Third, a GitHub App instead of an OAuth app. You pick exactly which repos it can see when you install it, and it only gets access to those. The tradeoff is MineCommit can no longer create the repo for you — that needs account-wide permission, which is the whole thing I was trying to avoid — so now you click one prefilled link, make an empty repo, grant the app access to it. One more click, in exchange for the app never being able to touch anything I didn't explicitly hand it.

## the scary bug hunt

At some point I made myself sit down and think through every single place in the code that deletes or renames something on disk, and ask: what's the worst path that could end up there. This is a backup tool. The one unforgivable bug is "erased somebody's 7 year old world."

Found a few real ones. A world could get misidentified as one of MineCommit's own throwaway copies just because its name happened to look similar, which meant it was eligible for deletion. A symlink pointing at a real world could get treated the same way, which is worse. Neither had actually hurt anyone yet as far as I know, but I'd rather find that from staring at the code than from a very bad email.

While I was in there I also found that the "clean up old copies" feature had been searching the wrong folder for a while — I'd moved where those copies get stored, twice, and never updated where the cleanup tool looks. On my own machine that was 9.3GB of leftover world copies just sitting there, never once mentioned by the app. That one stung a little, ngl.

## small stuff that looked worse than it was

A pile of little things that don't sound like much individually but each one makes the app feel untrustworthy the moment you hit it: the account menu just didn't open when clicked, for a while. Windows popped open a black console window for every single git command it ran under the hood, which if you don't know better looks exactly like malware doing its thing. Signing in flashed "you're not signed in" for a split second on every launch before catching up with itself. None of these were dangerous, all of them make you not trust the thing, which for a tool asking to be responsible for your save file is basically the same problem.

## current progress

Version 0.13 now. Upload, download, the safety audit, real world info on the dashboard, launcher auto-detection, an auth flow I actually feel okay about. Still missing a history browser and any real size analytics, and the biggest gap left isn't a feature, it's time — four days of aggressively fixing things I find is not the same as four weeks of just quietly using it and seeing what breaks on its own schedule.

But the actual loop works: open the app, sync, play, back up, push. Three machines, one world, no more losing an evening's build because I forgot to copy a folder.
