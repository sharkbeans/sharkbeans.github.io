I play Minecraft on three machines — a work PC, a home desktop, and a Thinkpad — and kept
hitting the same failure: build something on one, forget to copy the save folder, overwrite it
later from a two-week-old copy on another. Dropbox-style sync is the obvious fix and the wrong
one; it happily corrupts a live world mid-write, region files are huge, and there's no history
once it goes wrong.

## What it's built on

[MineCommit](https://github.com/HairlessVillager/minecommit) already does the hard part: it
turns a Minecraft save into something Git can diff and compress properly instead of a 60MB blob
per backup, using [`simdnbt`](https://github.com/azalea-rs/simdnbt) for NBT parsing and
[`gitoxide`](https://github.com/GitoxideLabs/gitoxide) for the Git I/O. I forked it to add the
part I actually needed — a remote — and then kept going: a Tauri + React GUI on top of the Rust
core, aimed at someone who just wants to play Minecraft, not operate a CLI.

Fetch before you play, push after. If both sides changed since the last sync, it stops and says
so rather than trying to merge two different afternoons of building — there's no sane automatic
merge of two divergent worlds, and that was the whole point.

## Auth, three tries in

The GUI's sign-in went through three designs before landing anywhere I'd hand to someone else:

1. **Paste a personal access token.** Works, but "generate a token and paste it into a desktop
   app" is a very developer thing to ask a non-technical user to do.
2. **OAuth device code flow.** No pasting, but OAuth apps only get offered one scope — full
   read/write on every repository on the account — which is a wildly bigger blast radius than a
   tool that touches one folder of backups needs.
3. **A GitHub App.** You pick exactly which repositories it can see when you install it, and it
   only ever gets access to those. The tradeoff is the app can no longer create the repository
   for you — that needs account-wide permission, the thing this change exists to avoid — so
   connecting a world now offers a prefilled link to create an empty repo and a link to grant the
   app access to it, then lists exactly what you granted.

## The safety pass

This is a backup tool; the one unforgivable bug is losing somebody's save. Auditing every path
that deletes or renames something on disk turned up a world that could be misidentified as one
of MineCommit's own throwaway copies on a name match (eligible for deletion), a symlink to a real
world that could be treated the same way, and a cleanup routine that had been searching the wrong
folder for two directory moves running — 9.3GB of orphaned copies on my own machine that the app
had never once mentioned.

## Current state

Public releases are up to 0.13.0: upload, download, a safety audit, real world info on the
dashboard (with launcher auto-detection across Prism, MultiMC, CurseForge, Modrinth, and others),
and an auth flow I'm comfortable handing to someone else. Still missing a history browser and any
real size analytics.

<p class="note-box">This is a fork of an upstream open-source project, kept in sync via a
tracked <code>upstream</code> remote. Releases and issues live on the fork.</p>

## Public links

- Repository: <a href="https://github.com/sharkbeans/minecommit" target="_blank" rel="noreferrer">github.com/sharkbeans/minecommit</a>
- Releases: <a href="https://github.com/sharkbeans/minecommit/releases" target="_blank" rel="noreferrer">github.com/sharkbeans/minecommit/releases</a>
- Upstream: <a href="https://github.com/HairlessVillager/minecommit" target="_blank" rel="noreferrer">HairlessVillager/minecommit</a>
