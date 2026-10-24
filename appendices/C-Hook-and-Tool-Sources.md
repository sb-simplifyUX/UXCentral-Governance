---
title: "Appendix C: hook and tool sources"
description: Complete generalised source code for the four plugin hooks, the shared personal-information pattern library, the installer, the shared Markdown library and checker, the new-project tool, and the specification of the retrofit tools.
doc_type: governance
project: central
process_step: n/a
brand: all
status: approved
owner: Central administrator
created: 2026-10-23
updated: 2026-10-24
governance_version: 1.4.0
contains_pii: false
---

# Appendix C: hook and tool sources

These are this blueprint's scripts, generalised. Copy them verbatim and change only the `CONFIG` values. All scripts are Node ES modules (`.mjs`) with no dependencies, so they run unchanged on macOS and Linux.

Invisible characters (Unicode line separators, the byte-order mark) are built with `String.fromCharCode` rather than typed as `\u` escapes, because an escape typed through the AI's file tools can arrive as the raw character (see [10 §3](../10-Privacy-Accessibility-Stack-Markdown.md#3-stack-and-dependencies)).

## Shared configuration values

Every script that needs paths defines them at the top. Keep them identical across scripts:

| Constant | Reference value | Meaning |
|---|---|---|
| `ROOT_DIR` | `ux-work` | Folder under the home folder that holds everything |
| `CENTRAL_DIR` | `000-central` | Central repository folder name |
| `PROJECTS_DIR` | `001-projects` | Projects folder name |
| `WORK_DIR` | `_governance-work` | Local reports and backups |
| `PLUGIN` | `ux-governance` | Plugin name, for messages |

## hooks/hooks.json

See [14](../14-Hooks-Permissions-Enforcement.md#hooks) for the file. `${CLAUDE_PLUGIN_ROOT}` is expanded by Claude Code to the plugin folder. The PII guard, shell guard and central guard commands end with `|| exit 2`, so a missing `node` or a crash blocks the call instead of allowing it; the version check ends with `|| true`.

## hooks/lib/pii.mjs

Create this file in Phase 4 of the [implementation guide](../16-Implementation-Guide.md), together with `tools/lib/md.mjs`, because `md.mjs` imports its patterns. The hooks that also use it are added in Phase 6.

```js
// Personal-information patterns shared by the PII guard hook and the central tools.
// Lives inside the plugin so the hook still works when the plugin is installed as a cached copy.
// Lookarounds stop matches inside hex keys and IDs.
// CONFIG: email domains that are not personal information (example domains and your organisation's own).
const ALLOWED_EMAIL_DOMAINS = ['example.com']; // add your organisation's domain, <ORG_EMAIL_DOMAIN>

// CONFIG: replace the phone and identifier patterns with your region's formats (see 17 Adapting).
// The examples below are neutral shapes that run as-is:
//   phone_international: a + and a country code of 1 to 3 digits, then 8 to 12 digits with optional spaces or hyphens
//   phone_national:      a leading 0, then 8 to 11 digits in all (the 0 included) in any grouping of spaces or hyphens;
//                        adjust to your region
//   account_ref:         an organisation identifier made of a fixed prefix (here "ACC-") and 8 digits
export const PII_PATTERNS = {
  email: /(?<![\w.+-])[\w.+-]+@([\w-]+\.)+[A-Za-z]{2,}(?![\w-])/g,
  phone_international: /(?<![\w+])\+\d{1,3}[\s-]?(?:\d[\s-]?){8,12}(?!\w)/g,
  phone_national: /(?<![\w+.])0(?:[\s-]?\d){7,10}(?![\w.])/g,
  account_ref: /(?<!\w)ACC-\d{8}(?!\w)/g,
};

export function findPii(text) {
  const hits = {};
  for (const [kind, re] of Object.entries(PII_PATTERNS)) {
    const found = (text.match(re) || []).filter((v) => kind !== 'email'
      || !ALLOWED_EMAIL_DOMAINS.some((d) => v.toLowerCase().endsWith('@' + d)));
    if (found.length) hits[kind] = found;
  }
  return hits;
}

// First two and last two characters kept; six characters or fewer are fully hidden.
export function mask(value) {
  return value.length <= 6 ? '****' : value.slice(0, 2) + '*'.repeat(value.length - 4) + value.slice(-2);
}

// Replaces every match in a text with its masked form.
export function maskAll(text) {
  let out = String(text);
  for (const [kind, re] of Object.entries(PII_PATTERNS)) {
    out = out.replace(re, (v) => (kind === 'email' && ALLOWED_EMAIL_DOMAINS.some((d) => v.toLowerCase().endsWith('@' + d)) ? v : mask(v)));
  }
  return out;
}
```

The phone and identifier patterns above are neutral examples. Replace them with your region's exact formats (tighter patterns mean fewer false positives) and test against fake values, dates, version numbers and long hexadecimal keys. The national pattern is deliberately broad: it also matches other zero-led numbers of 8 to 11 digits, such as some postal codes with an extension, so expect an occasional false positive until you tighten it. The international pattern needs no change for most regions: it matches a number written with a leading `+`, a country code and 8 to 12 further digits.

## hooks/pii-guard.mjs

```js
#!/usr/bin/env node
// PreToolUse guard: blocks outbound tool calls (web, MCP, published pages, subagent prompts) whose input contains
// possible personal information. Exit 2 blocks the call and shows stderr to the AI.
// Fails closed: if the patterns cannot load, the input cannot be read, or anything else goes wrong, the call is blocked.
// Its command in hooks.json ends with `|| exit 2`, so a missing node or a crash blocks too.
const block = (msg) => { process.stderr.write(`Blocked by UX governance PII guard: ${msg}\n`); process.exit(2); };
const BOM = String.fromCharCode(0xfeff);

let lib;
try {
  lib = await import('./lib/pii.mjs');
} catch (e) {
  block(`its pattern library failed to load (${e.message}). Tell the user; the guard fails closed.`);
}

let raw = '';
process.stdin.setEncoding('utf8');
process.stdin.on('data', (c) => { raw += c; });
process.stdin.on('end', () => {
  try {
    const input = JSON.parse(raw.startsWith(BOM) ? raw.slice(1) : raw);
    if (!input || typeof input !== 'object') throw new Error('input is not an object');
    // Scan every string value of the input, one per line, plus its URL-decoded form when that differs. Scanning the
    // decoded values (not the JSON text) means a value after a line break or tab is still seen, and %40 in a link is
    // still an @. Adding the decoded form only when it differs keeps the counts in the block message honest.
    const leaves = (v) => (typeof v === 'string' ? [v] : Array.isArray(v) ? v.flatMap(leaves) : v && typeof v === 'object' ? Object.values(v).flatMap(leaves) : []);
    const decode = (s) => { try { return decodeURIComponent(s); } catch { return s; } };
    const kinds = Object.entries(lib.findPii(leaves(input.tool_input ?? {}).flatMap((s) => (decode(s) === s ? [s] : [s, decode(s)])).join('\n')));
    if (!kinds.length) process.exit(0);
    const summary = kinds.map(([k, v]) => `${k} x${v.length} (e.g. ${lib.mask(v[0])})`).join(', ');
    block(`the ${input.tool_name} input contains possible personal information: ${summary}. `
      + 'Customer personal information must not leave its research file (governance/privacy.md). '
      + 'Remove or de-identify it (use participant IDs such as P01) and retry. If this is a false positive, ask the user.');
  } catch (e) {
    block(`the tool input could not be checked (${e.message}). Retry; if it persists, tell the user.`);
  }
});
```

## hooks/shell-guard.mjs

```js
#!/usr/bin/env node
// PreToolUse guard for shell commands. Files change only through Claude's Edit and Write tools: an Edit fails when
// its anchor does not match, each call changes one file and shows a diff, and /rewind can undo it. This guard denies
// shell commands that write, copy, move or bulk-delete files, run inline code, reach the network, download and run
// packages, publish, or discard git work, and its reason names what to use instead.
// A deny goes to the AI, not to the person, so it never interrupts: the AI switches tool or asks the user.
// It reads the command text only: it stops the forms an AI writes, not a script file that writes files itself.
// Fails closed: input it cannot read is denied. Its command in hooks.json ends with `|| exit 2`, so a missing node
// or a crash blocks the command too.
const BOM = String.fromCharCode(0xfeff);

const USE_EDIT = 'Change files with the Edit or Write tool instead, one file per call, and read the change back.';
const ASK_USER = 'If it is really needed, ask the user to run it.';

const OUTWARD = new Set(['curl', 'wget', 'scp', 'sftp', 'rsync', 'ssh', 'nc', 'ncat', 'netcat', 'socat', 'telnet', 'ftp', 'lftp', 'aria2c', 'http', 'https']);
const SHELLS = new Set(['sh', 'bash', 'zsh', 'dash', 'ksh', 'fish']);
// The short flags that make each interpreter run code given on the command line.
const INLINE = { python: 'c', node: 'ep', nodejs: 'ep', perl: 'eEi', ruby: 'ei', php: 'r', bun: 'e', osascript: 'e', deno: '' };
// Flags that, used alone, run no code (an interpreter given no script and no flags reads code from standard input).
const SOLO = ['--version', '-v', '-V', '--help', '-h', '--test'];
// Commands that run the command after them, each with its flags that take a value.
const WRAPPERS = {
  sudo: ['-u', '-g', '-h', '-p', '-C', '-D', '-r', '-t', '-U'], doas: ['-u', '-C'], env: ['-u', '-C', '-S'], command: [], builtin: [],
  exec: ['-a'], nohup: [], nice: ['-n'], ionice: ['-c', '-n', '-p'], stdbuf: [], timeout: ['-s', '-k'], time: ['-f', '-o'],
  xargs: ['-I', '-d', '-E', '-L', '-n', '-P', '-s', '-a'],
};
const KEYWORDS = new Set(['{', '}', '!', 'if', 'then', 'else', 'elif', 'do', 'while', 'until']);
// > and >> with an optional stream number, &> and >|, then the target. 2>&1 and >&2 copy a stream and write no file.
const REDIRECT = /(?:&>>?|\d*>>?\|?)(&?)[ \t]*([^\s;&|<>()]*)/g;
const QUIET = /^\/dev\/(null|stdout|stderr|tty|fd\/\d+)$/;
const ASSIGNMENT = /^[A-Za-z_][A-Za-z0-9_]*=/;

const base = (word) => word.replace(/^.*\//, '').toLowerCase();
const isFlag = (word) => /^-./.test(word);
// True when a short flag cluster such as -rf or -i.bak holds any of the letters.
const hasLetter = (flags, letters) => flags.some((f) => {
  const m = /^-([A-Za-z]+)/.exec(f);
  return Boolean(m) && [...m[1]].some((l) => letters.includes(l));
});
const hasLong = (flags, ...names) => flags.some((f) => names.some((n) => f === n || f.startsWith(n + '=')));

// Where the ( at `open` closes, counting nested parentheses and skipping single-quoted text.
function closing(s, open) {
  let depth = 0;
  for (let k = open; k < s.length; k++) {
    if (s[k] === '\\') k++;
    else if (s[k] === "'") { const e = s.indexOf("'", k + 1); if (e < 0) return s.length; k = e; }
    else if (s[k] === '(') depth++;
    else if (s[k] === ')' && --depth === 0) return k;
  }
  return s.length;
}

// The command's structure: quoted text becomes Q, heredoc bodies are dropped because they are data, and the code
// inside $( ) or backticks within double quotes is appended as commands of its own, because it still runs.
function skeleton(cmd) {
  let out = '';
  const inner = [];
  const heredocs = [];
  let i = 0;
  while (i < cmd.length) {
    const c = cmd[i];
    if (c === '\\') { out += ' '; i += 2; continue; }
    if (c === "'") { const e = cmd.indexOf("'", i + 1); out += 'Q'; i = e < 0 ? cmd.length : e + 1; continue; }
    if (c === '"') {
      let j = i + 1;
      while (j < cmd.length && cmd[j] !== '"') {
        if (cmd[j] === '\\') j += 2;
        else if (cmd[j] === '$' && cmd[j + 1] === '(') { const e = closing(cmd, j + 1); inner.push(cmd.slice(j + 2, e)); j = e + 1; }
        else if (cmd[j] === '`') { let e = cmd.indexOf('`', j + 1); if (e < 0) e = cmd.length; inner.push(cmd.slice(j + 1, e)); j = e + 1; }
        else j++;
      }
      out += 'Q'; i = j + 1; continue;
    }
    // A heredoc (<<), but not a here-string (<<<).
    if (c === '<' && cmd[i + 1] === '<' && cmd[i + 2] !== '<' && cmd[i - 1] !== '<') {
      const m = /^<<(-?)[ \t]*(?:'([^'\n]*)'|"([^"\n]*)"|\\?([^\s;&|<>()]+))/.exec(cmd.slice(i));
      if (m) { heredocs.push({ end: m[2] ?? m[3] ?? m[4], tabs: m[1] === '-' }); out += ' '; i += m[0].length; continue; }
    }
    if (c === '\n' && heredocs.length) {
      let j = i + 1;
      for (const { end, tabs } of heredocs.splice(0)) {
        for (;;) {
          const e = cmd.indexOf('\n', j);
          const line = cmd.slice(j, e < 0 ? cmd.length : e);
          j = e < 0 ? cmd.length : e + 1;
          if ((tabs ? line.replace(/^\t+/, '') : line) === end || e < 0) break;
        }
      }
      out += ';'; i = j; continue;
    }
    out += c; i++;
  }
  return [out, ...inner.map(skeleton)].join(' ; ');
}

// Git subcommands that discard work, rewrite what was pushed, change remotes or write files outside git.
function checkGit(args) {
  let k = 0;
  while (k < args.length && isFlag(args[k])) k += ['-C', '-c', '--git-dir', '--work-tree', '--namespace'].includes(args[k]) ? 2 : 1;
  const sub = args[k];
  const rest = args.slice(k + 1);
  const flags = rest.filter(isFlag);
  const positional = rest.filter((a) => !isFlag(a));
  if (sub === 'push' && (hasLetter(flags, 'fd') || hasLong(flags, '--force', '--force-with-lease', '--force-if-includes', '--mirror', '--delete', '--prune')
    || positional.some((p) => /^[+:]/.test(p)))) {
    return `force pushes, mirror pushes and remote deletes rewrite or remove what was pushed. ${ASK_USER}`;
  }
  if ((sub === 'reset' && hasLong(flags, '--hard')) || sub === 'clean'
    || (sub === 'checkout' && (rest.includes('--') || positional.includes('.') || hasLetter(flags, 'f') || hasLong(flags, '--force')))
    || (sub === 'restore' && (!(hasLetter(flags, 'S') || hasLong(flags, '--staged')) || hasLetter(flags, 'W') || hasLong(flags, '--worktree')))
    || (sub === 'switch' && (hasLetter(flags, 'f') || hasLong(flags, '--force', '--discard-changes')))
    || (sub === 'stash' && ['drop', 'clear'].includes(positional[0]))) {
    return `\`git ${sub}\` used this way discards work that git cannot bring back. To undo an edit, use the Edit tool. ${ASK_USER}`;
  }
  if (sub === 'remote' && ['add', 'set-url'].includes(positional[0])) {
    return `changing where the repository fetches from or pushes to is the user's decision. ${ASK_USER}`;
  }
  if (hasLong(flags, '--output', '--output-directory') || sub === 'format-patch' || (sub === 'archive' && hasLetter(flags, 'o'))) {
    return `this git command writes files. ${USE_EDIT}`;
  }
  return undefined;
}

// The reason to deny one simple command, or undefined to leave it to the permission rules.
function check(words, viaXargs) {
  if (!words.length) return undefined;
  const name = base(words[0]);
  const args = words.slice(1);
  const flags = args.filter(isFlag);
  const positional = args.filter((a) => !isFlag(a));
  const lead = positional.slice(0, 3);
  const interpreter = /^python[0-9.]*$/.test(name) ? 'python' : name;
  const stdinOnly = !positional.length && !(flags.length && flags.every((f) => SOLO.includes(f)));

  if (OUTWARD.has(name)) return `\`${name}\` reaches the network. Read web pages with the WebFetch tool, which the PII guard checks. ${ASK_USER}`;
  if (['npm', 'pnpm', 'yarn', 'bun'].includes(name) && lead.includes('publish')) return `publishing a package is never the assistant's step. ${ASK_USER}`;
  if (['npx', 'pnpx', 'bunx'].includes(name) || (['pnpm', 'yarn'].includes(name) && lead.includes('dlx'))
    || (name === 'npm' && (lead.includes('exec') || lead.includes('x'))) || (name === 'bun' && positional[0] === 'x')) {
    return 'this downloads and runs a package that is not pinned in the lockfile. Add it as an exact-pinned dependency (an install, which asks the user) and run it with `pnpm exec`.';
  }
  if (name === 'eval' || (SHELLS.has(name) && (hasLetter(flags, 'cs') || hasLong(flags, '--command') || stdinOnly))) {
    return `\`${name}\` running code from a string or from standard input hides that code from the permission rules and this guard. Run the command itself.`;
  }
  if (Object.hasOwn(INLINE, interpreter) && (hasLetter(flags, INLINE[interpreter]) || hasLong(flags, '--eval', '--print')
    || positional.includes('-') || (interpreter === 'deno' && positional[0] === 'eval') || stdinOnly)) {
    return 'inline code (such as `python3 -c`, `node -e` or a heredoc into an interpreter) can change files with no diff and no rewind. '
      + `To read, use the Read or Grep tool; to run a check, run a script file from the repository. ${USE_EDIT}`;
  }
  if (['sed', 'gsed'].includes(name) && (hasLetter(flags, 'i') || hasLong(flags, '--in-place'))) {
    return `\`sed -i\` edits files in place: a pattern that matches nothing changes nothing and still succeeds, and a glob edits every file at once. ${USE_EDIT}`;
  }
  if (['awk', 'gawk'].includes(name) && /(^| )(-i ?|--include[= ]?)inplace( |$)/.test(args.join(' '))) return `\`${name} -i inplace\` edits files in place. ${USE_EDIT}`;
  if (name === 'tee' && positional.some((p) => !QUIET.test(p))) return `\`tee\` writes files. ${USE_EDIT}`;
  if ((name === 'dd' && args.some((a) => a.startsWith('of='))) || ['truncate', 'shred'].includes(name)) return `\`${name}\` writes files. ${USE_EDIT}`;
  if (['cp', 'mv', 'install'].includes(name)) {
    return 'copying and moving files bypass the Edit and Write tools and cannot be rewound. To create a file, Write it; to copy, move or rename files, ask the user.';
  }
  if (name === 'rm' && (viaXargs || hasLetter(flags, 'rR') || hasLong(flags, '--recursive') || positional.some((p) => /[*?[{]/.test(p)))) {
    return 'recursive, wildcard and piped deletes remove many files at once and cannot be rewound. Delete one named file per `rm`, which the user approves, or ask the user to delete the folder.';
  }
  if (name === 'find' && args.some((a) => ['-delete', '-exec', '-execdir', '-ok', '-okdir', '-fprint', '-fprint0', '-fprintf', '-fls'].includes(a))) {
    return '`find` with -delete or -exec acts on every file it matches. Find and search files with the Glob or Grep tool, then change each one with Edit or Write, or ask the user.';
  }
  if (name === 'git') return checkGit(args);
  return undefined;
}

// Skips variable assignments, shell keywords and wrappers such as sudo, env and xargs, then checks the command they run.
function analyse(words) {
  let i = 0;
  let viaXargs = false;
  while (i < words.length) {
    const word = words[i];
    const name = base(word);
    if (KEYWORDS.has(word) || ASSIGNMENT.test(word)) { i++; continue; }
    if (!Object.hasOwn(WRAPPERS, name)) break;
    if (name === 'xargs') viaXargs = true;
    i++;
    while (i < words.length && (isFlag(words[i]) || ASSIGNMENT.test(words[i]))) i += WRAPPERS[name].includes(words[i]) ? 2 : 1;
    if (name === 'timeout') i++; // its duration
  }
  return check(words.slice(i), viaXargs);
}

// The reason to deny the whole command line, or undefined when every part of it is left to the permission rules.
function review(command) {
  const shape = skeleton(command);
  for (const [, copy, target] of shape.matchAll(REDIRECT)) {
    if (!target || QUIET.test(target) || (copy && /^(\d+|-)$/.test(target))) continue;
    return `a redirection (\`>\` or \`>>\`) writes a file. ${USE_EDIT}`;
  }
  const parts = shape.replace(REDIRECT, ' ').replace(/\$\(|[()`]/g, ';').split(/\|\||&&|\|&|[;|&\n]/);
  for (const part of parts) {
    const reason = analyse(part.trim().split(/[ \t\r]+/).filter(Boolean));
    if (reason) return reason;
  }
  return undefined;
}

let raw = '';
process.stdin.setEncoding('utf8');
process.stdin.on('data', (c) => { raw += c; });
process.stdin.on('end', () => {
  let reason;
  try {
    const input = JSON.parse(raw.startsWith(BOM) ? raw.slice(1) : raw);
    if (!input || typeof input !== 'object') throw new Error('input is not an object');
    reason = review(String(input.tool_input?.command ?? ''));
  } catch (e) {
    reason = `the command could not be checked (${e.message}). Retry; if it persists, tell the user.`;
  }
  if (reason) {
    process.stdout.write(JSON.stringify({ hookSpecificOutput: { hookEventName: 'PreToolUse', permissionDecision: 'deny',
      permissionDecisionReason: `Blocked by UX governance shell guard: ${reason}` } }));
  }
});
```

The guard writes its decision once and lets Node exit by itself, so the output is complete before the process ends. A command it has no rule for gets no output, and the permission rules decide: `rm` of one named file, a plain `git push` and installs still ask, and everything else follows the session's permission mode.

It refuses some read-only commands that look like writes, such as `find . -exec grep …` or a `>` written as a comparison. That is deliberate: its reason names the tool to use instead (Grep, Glob, Read), so the assistant moves on without a prompt.

## hooks/central-guard.mjs

```js
#!/usr/bin/env node
// PreToolUse guard for shell commands: Edit permission rules do not cover files that a program writes itself,
// so every shell command that names central governance (or runs inside it) becomes an approval prompt,
// unless it is read-only or ordinary work on a short list. The shell guard refuses shell writes before this runs.
// Fails towards asking: unreadable input or any error also asks. Its command in hooks.json ends with `|| exit 2`,
// so a missing node or a crash blocks the command instead of letting it through.
import path from 'node:path';
import os from 'node:os';

// CONFIG
const ROOT_DIR = 'ux-work';
const CENTRAL_DIR = '000-central';
const PLUGIN = 'ux-governance';
const BOM = String.fromCharCode(0xfeff);

const ask = (reason) => {
  process.stdout.write(JSON.stringify({ hookSpecificOutput: { hookEventName: 'PreToolUse', permissionDecision: 'ask', permissionDecisionReason: reason } }));
  process.exit(0);
};

// Read-only commands allowed without a prompt, with plain arguments only: no quotes, $, backticks, redirection,
// chaining, subexpressions or line breaks. Separators are spaces and tabs only: \s would also match a newline
// and let a second command through.
const READ_ONLY = /^[ \t]*(ls|pwd|cat|head|tail|wc|grep|git[ \t]+status)([ \t]+[^\s"'$`|;&<>()]+)*[ \t]*$/i;

// Central's own read-only tools, run by relative or absolute path, quoted or not. For example:
//   node tools/check-md.mjs governance --json
//   node "/Users/sam/ux-work/000-central/tools/retrofit-audit.mjs" --project <dir> --decider "Sam Example" ...
// Nothing may come between node and the script (no node flags). Each word is plain text, or a quoted part with
// no $, backticks or backslashes inside. The script must resolve to central's own tools folder: a copy elsewhere asks.
const READ_ONLY_TOOLS = ['check-md.mjs', 'retrofit-audit.mjs'];
const WORD = `(?:[^\\s"'$\`|;&<>()\\\\]|"[^"$\`\\\\\\r\\n]*"|'[^'\\r\\n]*')+`;
const TOOL_CALL = new RegExp(`^[ \\t]*node[ \\t]+(${WORD})(?:[ \\t]+${WORD})*[ \\t]*$`);
// Ordinary work in central that asks for no approval: git commands that write no file or record edits the
// administrator approved one by one (a quoted commit message is fine), the tool tests and the plugin validator.
// --output would make git write a file, so it still asks; so do push, pull and every other git command.
const GIT_WORK = new RegExp(`^[ \\t]*git[ \\t]+(status|diff|log|show|add|commit|fetch)(?:[ \\t]+${WORD})*[ \\t]*$`);
const CHECKS = /^[ \t]*(node[ \t]+--test[ \t]+(['"])tools\/test\/\*\.test\.mjs\2|claude[ \t]+plugin[ \t]+validate([ \t]+[^\s"'$`|;&<>()]+)?)[ \t]*$/;
const unquote = (word) => word.replace(/"([^"]*)"|'([^']*)'/g, (m, a, b) => a ?? b);
// The shell expands a leading ~/ only when it is not quoted.
const scriptPath = (word) => (word.startsWith('~/') ? path.join(os.homedir(), unquote(word.slice(2))) : unquote(word));

const escapeRe = (s) => s.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');

let raw = '';
process.stdin.setEncoding('utf8');
process.stdin.on('data', (c) => { raw += c; });
process.stdin.on('end', () => {
  try {
    const input = JSON.parse(raw.startsWith(BOM) ? raw.slice(1) : raw);
    if (!input || typeof input !== 'object') throw new Error('input is not an object');
    const command = String(input.tool_input?.command ?? '');
    const central = path.join(os.homedir(), ROOT_DIR, CENTRAL_DIR).toLowerCase();
    const cwd = path.resolve(typeof input.cwd === 'string' && input.cwd ? input.cwd : process.cwd()).toLowerCase();
    const touchesCentral = new RegExp(escapeRe(CENTRAL_DIR), 'i').test(command) || cwd === central || cwd.startsWith(central + path.sep);
    if (!touchesCentral || READ_ONLY.test(command) || CHECKS.test(command)) process.exit(0);
    if (GIT_WORK.test(command) && !/--output/.test(command)) process.exit(0);
    const call = command.match(TOOL_CALL);
    if (call) {
      const script = path.resolve(cwd, scriptPath(call[1])).toLowerCase();
      if (READ_ONLY_TOOLS.some((t) => script === path.join(central, 'tools', t))) process.exit(0);
    }
    ask(`This shell command names central UX governance or runs inside it, and is not on the guard's read-only list. Central changes need the administrator's approval and go through /${PLUGIN}:governance-change.`);
  } catch {
    ask('UX governance could not read this shell command to check it; approve only if it does not change central governance.');
  }
});
```

Paths are compared in lower case, so a path typed with different capitals still matches on a case-insensitive file system, as the default macOS one is; on a case-sensitive one the comparison is unchanged in effect.

The guard asks for every other command that touches central, including `git push`, `git pull` and the write tools `new-project.mjs` and `retrofit-apply.mjs`. That is by design: publishing central, taking in changes, creating a project and applying a retrofit are approvals. `retrofit-audit.mjs` is exempt only because its specification below makes it refuse to write inside central. `git add` and `git commit` run without a prompt because every edit they record already asked (`ask` on `Edit` under central), and the core rules still have the assistant ask the administrator before committing.

## hooks/version-check.mjs

```js
#!/usr/bin/env node
// SessionStart: when the session is inside a UX project, report the project's governance version against central,
// its process step and its Doing count, the decider, plus a reminder when the design-library map is due a check.
// Prints facts only; prints nothing outside the governed folders, and nothing at all on any error.
// What it prints goes into the AI's context, so every value read from PRD.md is checked before it is printed.
// Self-contained (no imports from outside the plugin), so it also works from a cached plugin copy.
import fs from 'node:fs';
import path from 'node:path';
import os from 'node:os';
import { fileURLToPath } from 'node:url';

// CONFIG
const ROOT_DIR = 'ux-work';
const CENTRAL_DIR = '000-central';
const PROJECTS_DIR = '001-projects';
const PLUGIN = 'ux-governance';
const MAP_FILE = ['governance', 'brand', 'design-library-structure.md'];

const central = path.join(os.homedir(), ROOT_DIR, CENTRAL_DIR);
const projects = path.join(os.homedir(), ROOT_DIR, PROJECTS_DIR);
const BOM = String.fromCharCode(0xfeff);
const BREAKS = `\\x00-\\x1f\\x7f${String.fromCharCode(0x85, 0x2028, 0x2029)}`;
const PERSON_BAD = new RegExp(`[${BREAKS}$|@\`]|\\{\\{`);

function centralVersion() {
  try { return fs.readFileSync(path.join(central, 'VERSION'), 'utf8').trim(); } catch { /* fall back to the plugin version */ }
  const manifest = path.join(path.dirname(fileURLToPath(import.meta.url)), '..', '.claude-plugin', 'plugin.json');
  try { return JSON.parse(fs.readFileSync(manifest, 'utf8')).version; } catch { return 'unknown'; }
}

// Double-quoted values are JSON strings; single-quoted ones use '' for a quote.
function unquote(v) {
  if (/^".*"$/.test(v)) { try { return JSON.parse(v); } catch { return v; } }
  if (/^'.*'$/.test(v)) return v.slice(1, -1).replaceAll("''", "'");
  return v;
}

function frontmatter(text) {
  const m = (text.startsWith(BOM) ? text.slice(1) : text).match(/^---\r?\n([\s\S]*?)\r?\n---[ \t]*(?:\r?\n|$)/);
  const data = {};
  if (m) for (const line of m[1].split(/\r?\n/)) { const kv = line.match(/^([\w-]+):\s*(.*)$/); if (kv) data[kv[1]] = unquote(kv[2].trim()); }
  return data;
}

function badPersonValue(v) {
  return typeof v !== 'string' || !v.trim() || v.startsWith('--') || PERSON_BAD.test(v);
}

// Reminder line when the design-library map is past its next-check date; null when not due or unreadable.
function mapReminder() {
  try {
    const data = frontmatter(fs.readFileSync(path.join(central, ...MAP_FILE), 'utf8'));
    const d = new Date();
    const today = `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`;
    if (!data.figma_next_check || today < data.figma_next_check) return null;
    return `Design-library map last verified ${data.figma_verified || 'unknown'}; its ${data.figma_check_every_days || 14}-day check was due ${data.figma_next_check}. Run /${PLUGIN}:figma-library-check.`;
  } catch { return null; }
}

let raw = '';
process.stdin.setEncoding('utf8');
process.stdin.on('data', (c) => { raw += c; });
process.stdin.on('end', () => {
  try {
    let cwd = process.cwd();
    try { const c = JSON.parse(raw.startsWith(BOM) ? raw.slice(1) : raw)?.cwd; if (typeof c === 'string' && c) cwd = c; } catch { /* use process cwd */ }
    const norm = (p) => path.resolve(p).toLowerCase();
    const version = centralVersion();
    const reminder = mapReminder();

    if (norm(cwd) === norm(central) || norm(cwd).startsWith(norm(central) + path.sep)) {
      console.log(`UX governance: this session is in central governance (version ${version}). Changes here go through /${PLUGIN}:governance-change.`);
      if (reminder) console.log(reminder);
      return;
    }
    if (!norm(cwd).startsWith(norm(projects) + path.sep)) return;

    const projectDir = path.join(projects, path.relative(projects, cwd).split(path.sep)[0]);
    const prdPath = path.join(projectDir, 'PRD.md');
    const name = path.basename(projectDir);
    if (!fs.existsSync(prdPath)) {
      console.log(`UX governance ${version}: project "${name}" has no PRD.md, so it has not been set up or retrofitted. Retrofit with /${PLUGIN}:retrofit-project.`);
      if (reminder) console.log(reminder);
      return;
    }
    const text = fs.readFileSync(prdPath, 'utf8');
    const data = frontmatter(text);
    // Only allowed values are printed; anything else is reported as invalid, never echoed.
    const STEPS = ['understand', 'define', 'sketch', 'decide', 'validate', 'deliver', 'n/a'];
    const projectVersion = /^\d+\.\d+\.\d+$/.test(data.governance_version ?? '') ? data.governance_version : 'unknown';
    const step = STEPS.includes(data.process_step) ? data.process_step : 'unknown (not an allowed value)';
    // The board's Doing line: a count of 0 to 3, then `none` or exactly that many card IDs (C-001, C-002).
    const doing = text.match(/■ \*\*Doing \(([0-3])\/3\):\*\*[ \t]*(none|C-\d{3,}(?:, C-\d{3,})*)[ \t]*$/m);
    const doingValid = doing && (doing[2] === 'none' ? 0 : doing[2].split(', ').length) === Number(doing[1]);
    const lines = [
      `UX governance: project "${name}" follows governance ${projectVersion}; central is ${version}.`,
      `Current process step (PRD.md): ${step}.`,
      doingValid ? `Doing: ${doing[1]}/3 (${doing[2]}).` : 'Kanban board: no valid Doing line found in PRD.md.',
    ];
    const decider = data.decider;
    const address = data.decider_address;
    if ((decider && badPersonValue(decider)) || (address && badPersonValue(address))) lines.push('PRD.md decider value is invalid; ask the user who the decider is and how to address them.');
    else if (decider && address) lines.push(`Decider: ${decider} (address as ${address}); they approve everything in this project.`);
    else if (decider) lines.push(`Decider: ${decider} (no decider_address yet: address them by name and they/them, and ask once how they would like to be addressed); they approve everything in this project.`);
    else lines.push('PRD.md has no decider: ask the user who the decider is and how to address them before any project work.');
    if (projectVersion !== version) lines.push(`The project's governance version is not central's; /${PLUGIN}:retrofit-project brings it to ${version}.`);
    if (reminder) lines.push(reminder);
    console.log(lines.join('\n'));
  } catch { /* silent on error: a session-start report must never get in the way of a session */ }
});
```

## Installer

`user-config/install.mjs`:

```js
#!/usr/bin/env node
// Installs central user-level config into ~/.claude, then registers the governance plugin.
// Backs up the current files first. Merges settings: central keys win, other keys you have are kept,
// permission lists are combined without duplicates. Never replaces your CLAUDE.md: the governance text sits
// between two marker lines, appended on the first run and replaced on later runs; the rest of the file is kept.
// Each file is written to a .tmp copy and then renamed into place. Exits non-zero if any step fails.
// Usage: node user-config/install.mjs [--name "<full name>"] [--address "<how to address you>"] [--dry-run]
import fs from 'node:fs';
import path from 'node:path';
import os from 'node:os';
import readline from 'node:readline/promises';
import { execFileSync } from 'node:child_process';
import { fileURLToPath } from 'node:url';

// CONFIG
const ROOT_DIR = 'ux-work';
const WORK_DIR = '_governance-work';
const MARKETPLACE = 'ux-central';
const PLUGIN = 'ux-governance';
const BEGIN = '<!-- ux-governance:begin -->';
const END = '<!-- ux-governance:end -->';

const dryRun = process.argv.includes('--dry-run');
const arg = (n) => { const i = process.argv.indexOf(`--${n}`); return i > -1 ? process.argv[i + 1] : undefined; };
const here = path.dirname(fileURLToPath(import.meta.url));
const central = path.resolve(here, '..');
const claudeDir = path.join(os.homedir(), '.claude');
// Milliseconds keep two runs in the same second apart; a backup is never overwritten (COPYFILE_EXCL below).
const stamp = new Date().toISOString().slice(0, 23).replace(/[:T.]/g, '-');
const backupDir = path.join(os.homedir(), ROOT_DIR, WORK_DIR, 'backups', `${stamp}-user-config`);
const run = (cmd, args) => execFileSync(cmd, args, { encoding: 'utf8', stdio: ['ignore', 'pipe', 'pipe'] }).trim();
const die = (msg) => { console.error(`Install failed: ${msg}`); process.exit(1); };
const BREAKS = `\\x00-\\x1f\\x7f${String.fromCharCode(0x85, 0x2028, 0x2029)}`;
const PERSON_BAD = new RegExp(`[${BREAKS}$|@\`]|\\{\\{`);

// The plugin hooks run `node`; if it is not on PATH they cannot run and the guards would not block anything.
try { console.log(`node ${run('node', ['--version'])} found on PATH`); } catch { die('`node` is not on PATH. Install Node LTS and restart your editor, then re-run.'); }
try { console.log(`claude ${run('claude', ['--version'])} found on PATH`); } catch { die('`claude` is not on PATH.'); }

const userSettingsPath = path.join(claudeDir, 'settings.json');
const userClaudeMdPath = path.join(claudeDir, 'CLAUDE.md');
let current = {};
try { if (fs.existsSync(userSettingsPath)) current = JSON.parse(fs.readFileSync(userSettingsPath, 'utf8')); } catch (e) { die(`${userSettingsPath} is not valid JSON (${e.message}). Fix it by hand, then re-run.`); }
if (!current || typeof current !== 'object' || Array.isArray(current)) die(`${userSettingsPath} does not hold a JSON object. Fix it by hand, then re-run.`);
// These settings switch the guards off (no hook runs, or nothing asks), so the install would protect nothing.
if (current.disableAllHooks === true || current.permissions?.defaultMode === 'bypassPermissions') {
  die(`${userSettingsPath} turns the governance guards off ("disableAllHooks": true or "defaultMode": "bypassPermissions"). Remove those settings, then re-run.`);
}

// Permission paths are written centrally as {{HOME_POSIX}} and {{CENTRAL_POSIX}} and resolved for this user:
// the absolute path without its leading slash, so /Users/sam becomes Users/sam and /home/sam becomes home/sam.
const homePosix = os.homedir().replace(/^\//, '');
const centralPosix = central.replace(/^\//, '');
for (const [what, p] of [['home folder', homePosix], ['central folder', centralPosix]]) {
  if (/["*?[\]()]/.test(p)) die(`the ${what} path contains a character that is not safe in a permission pattern: ${p}`);
}
const fillTokens = (s) => s.replaceAll('{{HOME_POSIX}}', () => homePosix).replaceAll('{{CENTRAL_POSIX}}', () => centralPosix);
const resolveTokens = (v) => (typeof v === 'string' ? fillTokens(v)
  : Array.isArray(v) ? v.map(resolveTokens)
    : v && typeof v === 'object' ? Object.fromEntries(Object.entries(v).map(([k, x]) => [k, resolveTokens(x)])) : v);
const ours = resolveTokens(JSON.parse(fs.readFileSync(path.join(here, 'settings.json'), 'utf8')));
// Refuse to install a placeholder left in central's settings.json, such as <OPUS_MODEL_ID> or an unknown {{TOKEN}}.
const leftover = [...new Set(JSON.stringify(ours).match(/<[A-Za-z][\w .-]*>|\{\{[A-Za-z_]+\}\}/g) || [])];
if (leftover.length) die(`user-config/settings.json still holds placeholders: ${leftover.join(', ')}. Fill them in, then re-run.`);

const personProblem = (v) => {
  if (typeof v !== 'string' || !v.trim()) return 'is required and cannot be blank';
  if (v.startsWith('--')) return 'is missing a value (the next flag was taken as the value)';
  if (PERSON_BAD.test(v)) return 'must be one line and cannot contain control characters, {{, $, |, @ or backticks';
  return null;
};

// The governance block in the user's CLAUDE.md: '' when absent, null when the markers are damaged.
const governanceBlock = (text) => {
  const begins = text.split(BEGIN).length - 1;
  const ends = text.split(END).length - 1;
  if (begins === 0 && ends === 0) return '';
  if (begins !== 1 || ends !== 1 || text.indexOf(END) < text.indexOf(BEGIN)) return null;
  return text.slice(text.indexOf(BEGIN), text.indexOf(END) + END.length);
};
// Returns the user's CLAUDE.md with the governance block appended (first run) or replaced (later runs).
const withGovernanceBlock = (text, body) => {
  const old = governanceBlock(text);
  const block = `${BEGIN}\n${body.trim()}\n${END}`;
  if (old) return text.replace(old, () => block);
  return text.trim() ? `${text.trimEnd()}\n\n${block}\n` : `${block}\n`;
};

const previous = fs.existsSync(userClaudeMdPath) ? fs.readFileSync(userClaudeMdPath, 'utf8') : '';
const previousBlock = governanceBlock(previous);
if (previousBlock === null) die(`${userClaudeMdPath} has damaged governance markers. Keep exactly one "${BEGIN}" line and one "${END}" line after it (or remove both), then re-run.`);
const fromPrevious = (label) => { const v = previousBlock.match(new RegExp(`^- ${label}: (.+)$`, 'm'))?.[1]?.trim(); return v && !personProblem(v) ? v : undefined; };
const fromFlag = (n) => {
  if (!process.argv.includes(`--${n}`)) return undefined;
  const problem = personProblem(arg(n));
  if (problem) die(`--${n} ${problem}`);
  return arg(n).trim();
};
let userName = fromFlag('name') || fromPrevious('Name');
let userAddress = fromFlag('address') || fromPrevious('Address as');
if (!userName || !userAddress) {
  if (!process.stdin.isTTY) die('pass --name "<full name>" and --address "<how to address you>" (no terminal to ask in).');
  const rl = readline.createInterface({ input: process.stdin, output: process.stdout });
  const ask = async (question, what) => {
    const v = await rl.question(question);
    const problem = personProblem(v);
    if (problem) { rl.close(); die(`${what} ${problem}`); }
    return v.trim();
  };
  userName ||= await ask('Your full name: ', 'the name you entered');
  userAddress ||= await ask('How should the assistant address you (name, and pronouns if you like)? ', 'the address you entered');
  rl.close();
}

const uniq = (a = [], b = []) => [...new Set([...a, ...b])];
const merged = {
  ...current,
  ...ours,
  modelSettings: { ...(current.modelSettings || {}), ...(ours.modelSettings || {}) },
  env: { ...(current.env || {}), ...(ours.env || {}) },
  permissions: {
    ...(current.permissions || {}),
    ...(ours.permissions || {}),
    allow: uniq(current.permissions?.allow, ours.permissions?.allow),
    ask: uniq(current.permissions?.ask, ours.permissions?.ask),
    deny: uniq(current.permissions?.deny, ours.permissions?.deny),
  },
};
const governanceText = fs.readFileSync(path.join(here, 'CLAUDE.md'), 'utf8')
  .replaceAll('{{USER_NAME}}', () => userName).replaceAll('{{USER_ADDRESS}}', () => userAddress);
if (governanceText.includes(BEGIN) || governanceText.includes(END)) die('user-config/CLAUDE.md or a person value contains a governance marker line.');
const claudeMd = withGovernanceBlock(previous, governanceText);

console.log(`User:      ${userName} (address as: ${userAddress})`);
console.log(`Backup:    ${backupDir}`);
if (dryRun) {
  console.log(`\n${userSettingsPath} would become:\n${JSON.stringify(merged, null, 2)}`);
  console.log(`\n${userClaudeMdPath} would become:\n${claudeMd}`);
  process.exit(0);
}

// Writes next to the real file (a linked settings file stays linked), then renames: a crash never leaves half a file.
const writeAtomic = (file, text) => {
  const target = fs.existsSync(file) ? fs.realpathSync(file) : file;
  fs.writeFileSync(`${target}.tmp`, text);
  fs.renameSync(`${target}.tmp`, target);
};
try {
  fs.mkdirSync(backupDir, { recursive: true });
  for (const f of [userSettingsPath, userClaudeMdPath]) {
    if (fs.existsSync(f)) fs.copyFileSync(f, path.join(backupDir, path.basename(f)), fs.constants.COPYFILE_EXCL);
  }
  fs.mkdirSync(claudeDir, { recursive: true });
  writeAtomic(userSettingsPath, JSON.stringify(merged, null, 2) + '\n');
  writeAtomic(userClaudeMdPath, claudeMd);
} catch (e) { die(`writing the user files: ${e.message}. Any backups made before the failure are in ${backupDir}.`); }
console.log('Settings and CLAUDE.md written.');

const marketplaceDir = path.join(central, 'marketplace');
let marketplaces = '';
try { marketplaces = run('claude', ['plugin', 'marketplace', 'list']); } catch { /* treated as not yet added */ }
if (!marketplaces.includes(MARKETPLACE)) {
  try { console.log(run('claude', ['plugin', 'marketplace', 'add', marketplaceDir])); } catch (e) { die(`marketplace add: ${e.stderr || e.message}`); }
}
let plugins = '';
try { plugins = run('claude', ['plugin', 'list']); } catch { /* treated as not installed */ }
if (!plugins.includes(`${PLUGIN}@${MARKETPLACE}`)) {
  try { console.log(run('claude', ['plugin', 'install', `${PLUGIN}@${MARKETPLACE}`])); } catch (e) { die(`plugin install: ${e.stderr || e.message}`); }
}
console.log('Done. Restart your editor (or run /reload-plugins) so new sessions pick up the plugin, PATH and settings.');
```

In central's `user-config/settings.json`, write the central Edit rule as `Edit(//{{CENTRAL_POSIX}}/**)`. The installer fills `{{CENTRAL_POSIX}}` with central's absolute path without its leading slash, and `{{HOME_POSIX}}` the same way with the home folder. Run it with `--dry-run` first: it prints both files as they would be written and changes nothing.

## tools/lib/md.mjs

`md.mjs` imports its personal-information patterns from the plugin's `hooks/lib/pii.mjs`, so create `pii.mjs` (above) in Phase 4 together with this file; the import fails without it.

```js
// Shared helpers for the governance scripts: frontmatter parsing, the md standard schema, PII patterns.
import fs from 'node:fs';
import path from 'node:path';
import { fileURLToPath } from 'node:url';

export const REQUIRED_FIELDS = [
  'title', 'description', 'doc_type', 'project', 'process_step', 'brand',
  'status', 'owner', 'created', 'updated', 'governance_version', 'contains_pii',
];

// CONFIG: `all`, then one slug per brand (one brand is fine; add more as needed).
export const BRANDS = ['all', 'brand-a'];

export const ALLOWED = {
  doc_type: ['readme', 'prd', 'research-note', 'meeting-notes', 'synthesis', 'define', 'sketch',
    'decision', 'test-plan', 'spec', 'governance', 'index', 'other'],
  process_step: ['understand', 'define', 'sketch', 'decide', 'validate', 'deliver', 'n/a'],
  brand: BRANDS,
  status: ['draft', 'in-review', 'approved', 'superseded'],
  contains_pii: ['true', 'false'],
};

// The AI tool defines the format of these files, so the md standard does not apply. Case-insensitive.
// Only the tool's own agents folders are exempt (.claude/agents, a plugin's agents folder).
export function isExempt(file) {
  const base = path.basename(file).toLowerCase();
  const parts = file.split(path.sep);
  return base === 'skill.md' || base === 'claude.md' || base === 'claude.local.md'
    || parts.some((p, i) => p === 'agents' && (parts[i - 1] === '.claude' || parts[i - 2] === 'plugins'))
    || parts.includes('node_modules') || parts.includes('.git');
}

const BOM = String.fromCharCode(0xfeff);
export function readText(file) {
  const text = fs.readFileSync(file, 'utf8');
  return text.startsWith(BOM) ? text.slice(1) : text;
}

function unquote(v) {
  if (/^".*"$/.test(v)) { try { return JSON.parse(v); } catch { return v; } }
  if (/^'.*'$/.test(v)) return v.slice(1, -1).replaceAll("''", "'");
  return v;
}

// Minimal parser for flat key: value frontmatter. The closing line is --- with optional trailing spaces or tabs.
export function parseFrontmatter(text) {
  const m = text.match(/^---\r?\n([\s\S]*?)\r?\n---[ \t]*(?:\r?\n|$)/);
  if (!m) return { data: null, body: text, close: -1 };
  const data = {};
  for (const line of m[1].split(/\r?\n/)) {
    const kv = line.match(/^([A-Za-z_][\w-]*):\s*(.*)$/);
    if (kv) data[kv[1]] = unquote(kv[2].trim());
  }
  return { data, body: text.slice(m[0].length), close: m[1].split(/\r?\n/).length + 1 };
}

// H1 lines outside fenced code blocks.
export function h1Lines(body) {
  let fenced = false;
  const out = [];
  for (const line of body.split(/\r?\n/)) {
    if (/^\s*(```|~~~)/.test(line)) { fenced = !fenced; continue; }
    if (!fenced && /^# /.test(line)) out.push(line.slice(2).trim());
  }
  return out;
}

// Control characters and Unicode line breaks, built from char codes (no raw separators in the source).
const BREAKS = `\\x00-\\x1f\\x7f${String.fromCharCode(0x85, 0x2028, 0x2029)}`;
const LINE_BREAKS = new RegExp(`[${BREAKS}]+`, 'g');
const PERSON_BAD = new RegExp(`[${BREAKS}$|@\`]|\\{\\{`);

export function oneLine(v) {
  return String(v).replace(LINE_BREAKS, ' ').trim();
}

// Quotes a value for flat YAML frontmatter when YAML would treat it specially. contains_pii is written bare.
export function yamlValue(v, key) {
  const s = oneLine(v);
  if (key === 'contains_pii' && (s === 'true' || s === 'false')) return s;
  const scalar = /^(true|false|yes|no|y|n|on|off|null|~|=|<<|[-+]?(\d[\d._]*(e[-+]?\d+)?|\.\d[\d_]*(e[-+]?\d+)?|\.inf|\.nan)|[-+]?0x[0-9a-f_]+|[-+]?0o[0-7_]+|[-+]?0b[01_]+)$/i;
  return /[:#\[\]{}&*!|>'"%@`]|^[-?,]/.test(s) || scalar.test(s) ? JSON.stringify(s) : s;
}

// Checks a one-line person value (decider, how to address them). Returns a problem message, or null.
export function personValueProblem(v) {
  if (typeof v !== 'string' || !v.trim()) return 'is required and cannot be blank';
  if (v.startsWith('--')) return 'is missing a value (the next flag was taken as the value)';
  if (PERSON_BAD.test(v)) return 'must be one line and cannot contain control characters, {{, $, |, @ or backticks';
  return null;
}

// Fills {{KEY}} placeholders: YAML-quoted inside frontmatter, plain in the body. Unknown keys are left in place.
export function fillTemplate(text, values) {
  const fill = (s) => s.replace(/\{\{([A-Z_]+)\}\}/g, (m, k) => (k in values ? String(values[k]) : m));
  const m = text.match(/^---\r?\n[\s\S]*?\r?\n---[ \t]*(?:\r?\n|$)/);
  if (!m) return fill(text);
  const fm = m[0].replace(/^([A-Za-z_][\w-]*):[ \t]*(.*\{\{[A-Z_]+\}\}.*)$/gm, (line, key, value) => {
    const filled = fill(value);
    return filled === value ? line : `${key}: ${yamlValue(filled, key)}`;
  });
  return fm + fill(text.slice(m[0].length));
}

// Never follows symbolic links: they can point outside the project or loop.
export function walk(dir, filter = () => true, out = []) {
  for (const entry of fs.readdirSync(dir, { withFileTypes: true })) {
    if (entry.name === 'node_modules' || entry.name === '.git' || entry.isSymbolicLink()) continue;
    const full = path.join(dir, entry.name);
    if (entry.isDirectory()) walk(full, filter, out);
    else if (filter(full)) out.push(full);
  }
  return out;
}

// One source for personal-information patterns: the plugin's hook library. CONFIG: plugin folder name.
export { PII_PATTERNS, findPii, mask, maskAll } from '../../marketplace/plugins/ux-governance/hooks/lib/pii.mjs';

export function centralRoot() {
  return path.resolve(path.dirname(fileURLToPath(import.meta.url)), '..', '..');
}

export function governanceVersion() {
  return fs.readFileSync(path.join(centralRoot(), 'VERSION'), 'utf8').trim();
}

export function today() {
  const d = new Date();
  return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`;
}
```

The retrofit tools need more helpers in this library: `encodingProblem` (UTF-16, NUL bytes, invalid UTF-8), `frontmatterProblem` (keys the tools cannot read, `---` dividers mistaken for frontmatter), `fieldValueProblem` (placeholders, allowed values, real calendar dates, version format), `isInside` (case-insensitive, as the default macOS file system is; resolves symbolic links), `toFrontmatter` and `withDefineCandidates`. They are not supplied here. Build them, with `retrofit-audit.mjs` and `retrofit-apply.mjs`, as their own card on central's board when a project first needs a retrofit (see [Retrofit tools: specification](#retrofit-tools-specification)).

## tools/check-md.mjs

```js
#!/usr/bin/env node
// Validates Markdown files against governance/md-standard.md.
// Usage: node tools/check-md.mjs <file-or-dir>... [--template] [--json]
//   --template  accept {{PLACEHOLDER}} values (for project-template/); without it, any {{PLACEHOLDER}} is a problem
// Exit codes: 0 all pass, 1 a file fails, 64 no file or folder given, 66 a file or folder given does not exist.
import fs from 'node:fs';
import path from 'node:path';
import { REQUIRED_FIELDS, ALLOWED, isExempt, readText, parseFrontmatter, walk, h1Lines } from './lib/md.mjs';

const args = process.argv.slice(2);
const template = args.includes('--template');
const asJson = args.includes('--json');
const targets = args.filter((a) => !a.startsWith('--'));
if (!targets.length) {
  console.error('Usage: node tools/check-md.mjs <file-or-dir>... [--template] [--json]');
  process.exit(64);
}

const isPlaceholder = (v) => template && /^\{\{[A-Z_]+\}\}$/.test(v);
const missing = targets.filter((t) => !fs.existsSync(t));
if (missing.length) { console.error(`Not found: ${missing.join(', ')}`); process.exit(66); }
const files = targets.flatMap((t) => (fs.statSync(t).isDirectory() ? walk(t, (f) => f.endsWith('.md')) : [t]))
  .filter((f) => !isExempt(f));

const results = files.map((file) => {
  const problems = [];
  const { data, body } = parseFrontmatter(readText(file));
  if (!data) {
    problems.push('no frontmatter');
  } else {
    for (const field of REQUIRED_FIELDS) {
      const v = data[field];
      if (v === undefined || v === '') { problems.push(`missing ${field}`); continue; }
      if (isPlaceholder(v)) continue;
      if (!template && /\{\{[A-Z_]+\}\}/.test(v)) { problems.push(`${field}: unfilled placeholder in "${v}"`); continue; }
      if (ALLOWED[field] && !ALLOWED[field].includes(v)) problems.push(`${field}: "${v}" not in [${ALLOWED[field].join(', ')}]`);
      if ((field === 'created' || field === 'updated') && !/^\d{4}-\d{2}-\d{2}$/.test(v)) problems.push(`${field}: "${v}" is not YYYY-MM-DD`);
    }
  }
  const h1s = h1Lines(body);
  if (h1s.length !== 1) problems.push(`expected exactly one H1, found ${h1s.length}`);
  else if (data?.title && !isPlaceholder(data.title) && h1s[0] !== data.title) {
    problems.push(`H1 "${h1s[0]}" does not match title "${data.title}"`);
  }
  return { file, ok: problems.length === 0, problems };
});

if (asJson) {
  console.log(JSON.stringify(results, null, 2));
} else {
  for (const r of results.filter((x) => !x.ok)) console.log(`FAIL ${path.relative(process.cwd(), r.file)}\n  - ${r.problems.join('\n  - ')}`);
  console.log(`${results.filter((r) => r.ok).length}/${results.length} files pass the md standard`);
}
process.exit(results.every((r) => r.ok) ? 0 : 1);
```

## tools/new-project.mjs

```js
#!/usr/bin/env node
// Creates a project folder from project-template/ and fills the mechanical placeholders.
// Content placeholders ({{OBJECTIVES}} etc.) are left for the new-project skill to draft from the brief.
// Usage: node tools/new-project.mjs --name "<Project name>" [--brand all|<slug>] --summary "<one line>"
//          --decider "<full name>" --decider-address "<how to address them>" [--projects-dir <dir>]
import fs from 'node:fs';
import path from 'node:path';
import os from 'node:os';
import { centralRoot, governanceVersion, today, walk, fillTemplate, personValueProblem, BRANDS } from './lib/md.mjs';

// CONFIG
const ROOT_DIR = 'ux-work';
const PROJECTS_DIR = '001-projects';
// One label per entry in BRANDS (md.mjs), using each brand's display name.
const BRAND_LABELS = { all: 'All brands', 'brand-a': 'Brand A only' };
// Under phases/: a research folder in each of the first five steps (local only, never committed), prototypes in the sixth.
const PHASE_FOLDERS = ['01-understand/research', '02-define/research', '03-sketch/research',
  '04-decide/research', '05-validate/research', '06-deliver/prototypes'];

const BREAKS = new RegExp(`[\\x00-\\x1f\\x7f${String.fromCharCode(0x85, 0x2028, 0x2029)}]`);
const arg = (name, fallback) => { const i = process.argv.indexOf(`--${name}`); return i > -1 ? process.argv[i + 1] : fallback; };
const name = arg('name');
const brand = arg('brand', 'all');
const summary = arg('summary', '{{ONE_LINE_SUMMARY}}');
const decider = arg('decider');
const deciderAddress = arg('decider-address');
const projectsDir = arg('projects-dir', path.join(os.homedir(), ROOT_DIR, PROJECTS_DIR));
const refuse = (msg) => { console.error(msg); process.exit(64); };

if (!name) refuse('--name is required');
if (name.startsWith('--')) refuse('--name is missing a value (the next flag was taken as the value)');
if (!BRANDS.includes(brand)) refuse(`--brand must be one of ${BRANDS.join(', ')}`);
// Names must be valid folder names on macOS and Linux and safe to pass through a shell.
if (/[<>:"/\\|?*]/.test(name) || BREAKS.test(name) || /[. ]$/.test(name)) refuse('Project name contains characters not allowed in folder names');
if (BREAKS.test(summary) || summary.startsWith('--')) refuse('--summary must be one line with no control characters and no leading --');
for (const [flag, v] of [['--decider', decider], ['--decider-address', deciderAddress]]) {
  const problem = personValueProblem(v);
  if (problem) refuse(`${flag} ${problem}`);
}

const dest = path.join(projectsDir, name);
if (fs.existsSync(dest)) { console.error(`Refusing: ${dest} already exists. Use retrofit-project for existing projects.`); process.exit(1); }

// Read everything that can fail (VERSION) before the first write, so a failure leaves no half-made project.
const values = {
  PROJECT_NAME: name, BRAND: brand, BRAND_SCOPE: BRAND_LABELS[brand],
  DATE: today(), GOVERNANCE_VERSION: governanceVersion(),
  DECIDER: decider, DECIDER_ADDRESS: deciderAddress,
};
if (summary !== '{{ONE_LINE_SUMMARY}}') values.ONE_LINE_SUMMARY = summary;

fs.cpSync(path.join(centralRoot(), 'project-template'), dest, { recursive: true });
// Git ignores the empty research folders inside central, so make sure they exist in the copy.
for (const d of PHASE_FOLDERS) fs.mkdirSync(path.join(dest, 'phases', d), { recursive: true });
for (const file of walk(dest, (f) => f.endsWith('.md'))) {
  fs.writeFileSync(file, fillTemplate(fs.readFileSync(file, 'utf8'), values));
}

// Everything still to fill: {{NAME}} content placeholders, and <NAME> placeholders left in central's template.
const remaining = walk(dest, (f) => /\.(md|json)$/.test(f))
  .flatMap((f) => [...fs.readFileSync(f, 'utf8').matchAll(/\{\{[A-Z_]+\}\}|<[A-Z][A-Z0-9_]*>/g)].map((m) => `${path.relative(dest, f)}: ${m[0]}`));
console.log(JSON.stringify({ created: dest, governance_version: values.GOVERNANCE_VERSION, placeholders_to_fill: remaining }, null, 2));
```

## Retrofit tools: specification

`retrofit-audit.mjs` and `retrofit-apply.mjs` are specified here, not supplied. Build them as their own card on central's board when a project first needs a retrofit: implemented by a Sonnet subagent, reviewed by `code-stack-reviewer`, tests first.

### retrofit-audit.mjs (read-only)

```text
node tools/retrofit-audit.mjs --project <dir> --out <WORK>/reports/YYYY-MM-DD-<Project name> --brand <slug>
                              --decider "<full name>" --decider-address "<address form>"
```

| Requirement | Detail |
|---|---|
| Never writes inside the project or central | Refuses an `--out` inside the project or inside central (case-insensitive, link-aware); refuses a non-empty `--out` unless it is this project's previous report. The central guard lets this tool run without a prompt only because of this refusal |
| Inventory | Every file: path, type, size, timestamps, frontmatter state, personal-information hit counts (from `pii.mjs`) |
| Markdown changes (`md_changes`) | For each non-exempt Markdown file: frontmatter to add or merge (only fields being added), `auto_fields` the auditor agent should refine; dates from file history unless the content dates itself |
| New files (`new_files`) | README, PRD, CLAUDE.md, `.claude/settings.json`, `.gitignore`, phase `_index.md` files from the template, with placeholders listed in plan section 1 |
| Renames (`proposed_renames`) | Phase folders to the template names; only a folder's own name; never a folder inside one being renamed |
| Body changes (`body_changes`) | With `path`, `change`, `insert_text`, `insert_before` (for example the Named decider section in an older PRD); "fix by hand" items for unsafe files, PII without `contains_pii: true`, research file names that may hold a personal name |
| Process step | Inferred, presented as a proposal; candidate Define sentences, never agreed |
| Privacy | Reports carry counts and masked samples only, never existing frontmatter values or research text |
| Links | Reported, not followed |
| Outputs | `plan.md` (sections: 1 placeholders, 2 summary, 3 frontmatter, 4 new files, 5 renames, 6 body changes, 7 personal information, 8 duplicates, 9 excluded), `inventory.json`, `changes.diff` |

### retrofit-apply.mjs

```text
node tools/retrofit-apply.mjs --inventory <report>/inventory.json --backup <WORK>/backups/YYYY-MM-DD-<Project name>[-n]
```

| Requirement | Detail |
|---|---|
| Only approved items | Reads `approved`: `md_changes`, `new_files` (true/false), exact `body_changes` and `renames` (must match proposals on every field), with the approval record |
| Phase 1: plan and check everything | Paths normalised and inside the project, no duplicates by any name; files are UTF-8 Markdown, not exempt, writable and not locked; frontmatter readable; values valid (no placeholders, allowed values, real dates, version format); new files have no placeholders and valid JSON; renames do not overlap, sources hold no links, targets' parents are folders; backup folder empty. Any failure: refuse, change nothing, create no backup |
| Phase 2: back up, then write | Copy every file to change (and every file an approved body change will touch) to `<backup>/files/<path>`; renamed items to `<backup>/renamed/` without `node_modules` and `.git`; never overwrite a backup |
| Frontmatter merge | Insert lines before the closing `---`; keep the BOM and every existing line ending; read back the new frontmatter |
| New files | Created with no-overwrite flags |
| Body changes | Not applied by the script; the AI applies them with Edit afterwards |
| Log | Every step logged; on a write failure stop and report what was applied; tell the user to run `pnpm install` for any moved `node_modules` |

### Tests

Run with `node --test 'tools/test/*.test.mjs'` (on current Node.js, the folder form `node --test tools/test` fails). Fixture projects in a temporary folder covering: dry run changes no file hash; `--out` inside the project or inside central refused; each apply refusal leaves the project unchanged and creates no backup; BOM and CRLF preserved on merge; backups never overwritten; exempt and non-UTF-8 files refused; altered approvals refused; overlapping renames refused; link sources refused; personal information never appears in report files.
