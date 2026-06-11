# Drops and Points Farmer

Mine Twitch drops and farm channel points in the background, without leaving a stream open. This is an opt-in add-on that lives outside the core app, so the app stays lean and you run this only if you want it.

It runs as its own program that StreamNook starts and talks to. It does its own networking and uses your Twitch login, handed over once you allow it, so it can watch and claim on your account.

## What it does

Enabling it starts nothing on its own. You choose what runs:

- **Drops mining.** Click a drop in the Drops center and it mines that drop for you, with no stream open and nothing to watch. The Drops center shows live progress the whole time, the same as the built-in view, and the title bar shows the current drop's percentage. Stop or switch by clicking another drop. Mine All works through your campaigns by the priority you set.
- **Background channel points.** Off by default. Turn it on to farm points across your other followed live channels. The channel you are actively watching always earns on its own, so this just covers the rest.

It claims your bonus chests and completed drops automatically while it runs.

## Settings

Settings are a rich panel on the Plugins page: a follower search picker for priority channels, add-and-remove lists for preferred and excluded games, a selection strategy, and recovery controls for when a stream stalls or goes offline.

## What it asks for, and why

The first time it needs your Twitch login it asks once, specifically for that. It needs the login to report watch time and claim on your behalf. It talks to Twitch directly:

- **Your Twitch login**, to watch and claim for your account.
- **Network access**, to reach Twitch.
- **Followed live channels and watch ticks**, so it knows which channels are live and when to act.

Everything it does is something you can already do yourself by watching. Moving it into a separate, opt-in add-on keeps the core app lean and keeps this behavior entirely under your control.

## Tier

This is an Advanced (Tier C) plugin: it runs background automation on your account, so it is distributed as a community add-on rather than part of the built-in first-party set.
