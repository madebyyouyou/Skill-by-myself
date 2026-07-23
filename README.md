# Skill-by-myself

Skills for [Claude Code](https://claude.com/claude-code), written from things that actually broke.

Each skill is a folder under `skills/` containing a `SKILL.md` with YAML frontmatter (`name`,
`description`). Claude reads the `description` to decide when a skill applies, so it is phrased as a
list of symptoms rather than a topic.

The rule for this repo: **a skill earns its place by containing facts, not principles.** Advice like
"respect safe-area insets" is already obvious to anyone hitting the problem and helps nobody. Measured
numbers, the exact failure mode, and code you can paste — those are worth writing down.

## Skills

| Skill | What it covers |
|---|---|
| [`ios-pwa-fullscreen`](skills/ios-pwa-fullscreen/SKILL.md) | Making an iOS home-screen web app render edge-to-edge: why the status bar's 62pt is unreachable and what to do instead, the three CSS height-propagation traps that white-screen only on WebKit, on-device diagnostics for when Web Inspector is unavailable, and status-bar colour matching. |
| [`game-feel-ui`](skills/game-feel-ui/SKILL.md) | Building game frontends that read as living scenes rather than card-and-panel webpages: diegetic UI, ambient idle animation, squash-and-stretch feedback, particle/number-popup juice, spatial navigation. Targets Phaser 3 + TypeScript, adapts to DOM/CSS, React, or canvas. |

## Using these

Copy a skill folder into your project's `skills/` (or `~/.claude/skills/` for all projects):

```bash
git clone https://github.com/madebyyouyou/Skill-by-myself
cp -r Skill-by-myself/skills/ios-pwa-fullscreen your-project/skills/
```

## Notes

Where a skill states measurements, it names the device and OS version they came from. Platform
behaviour changes; if a number here disagrees with your device, trust your device — and please open an
issue with what you measured.

## License

MIT
