# SmartDefaults

A Claude Code skill that makes a website start in the visitor's own theme and language. No white
flash before it paints, no forced redirect, no toggle to hunt for on the first visit.

## Install

Inside Claude Code, two lines:

```
/plugin marketplace add Mun1to/Vibeset
/plugin install smart-defaults@vibeset
```

If the install summary says `Run /reload-plugins to activate`, run it.

<details>
<summary>Prefer to install it by hand?</summary>

```bash
git clone https://github.com/Mun1to/SmartDefaults.git
ln -s "$(pwd)/SmartDefaults" ~/.claude/skills/smart-defaults
```

On Windows, with PowerShell:

```powershell
New-Item -ItemType Junction -Path "$env:USERPROFILE\.claude\skills\smart-defaults" -Target "C:\path\to\SmartDefaults"
```
</details>

## Use it

Ask for it when you build the boot of a page, when you add a theme or language switcher, or when
the page flashes light before turning dark. Claude also picks it up on its own from phrases like
"make the site detect the visitor's language".

## The two rules it enforces

**1. Theme: no toggle by default.** A sun and moon icon is something the visitor has to find and
press. The default here is pure CSS, zero JavaScript: a `:root` palette plus a
`@media (prefers-color-scheme: dark)` override. The browser re-evaluates that on its own, so the
page follows the operating system live, even while the tab is open, and a flash is not possible
because no script can arrive late.

A manual switch is the exception, not the starting point, and it only earns its place when the
product genuinely needs the visitor to pin a theme against their system setting. Then the choice
goes in `localStorage`, is read by an inline script placed before the first stylesheet, and wins
forever after.

**2. Language: auto-select, never redirect.** Redirecting by browser language breaks the back
button, breaks bookmarks and confuses crawlers, as Google Search Central states outright. So the
detection runs only on the first visit, when nothing is stored. Once the visitor picks a language,
that choice is respected forever and the site never guesses again. The manual selector always stays
visible.

## What is in the box

- `SKILL.md`, the whole procedure, the priority order for both decisions, and a checklist.
- `plantillas/vanilla/theme-css-only.html`, the default: live theme, no JS, no icon.
- `plantillas/vanilla/boot-prefs.html`, the exception: manual theme override plus full language
  detection, commented line by line.

## Don't trust it, check it

Open source only helps if somebody actually reads the code, and almost nobody does. So instead of
asking you to trust this project, here is the prompt to check it: point your own AI agent at this
repository and get a security report, in your language, in a few minutes, even if you do not know
how to program.

**[Open AI-AUDIT.md](AI-AUDIT.md)** and paste it into Claude Code, Codex, Cursor, Copilot or
whatever you use. It is the same prompt in every public repository here, so you can compare.

> **ES:** No hace falta que te fíes. Abre [AI-AUDIT.md](AI-AUDIT.md), pega ese texto en tu IA y
> te dirá en tu idioma qué hace este programa de verdad: qué envía por internet, qué toca en tu
> ordenador y qué ejecuta al instalarse.

There is nothing here that runs: Markdown instructions and two HTML reference files.

## License

MIT, see [LICENSE](LICENSE).
