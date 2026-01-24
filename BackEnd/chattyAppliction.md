# Server Setup
## `.editorconfig`
.editorconfig is a team-wide formatting agreement that your editor/IDE can auto-apply, so everyone saves files with the same basic style.

Why we use it (real necessity)

When multiple people work on the same codebase, they often use different editors/settings. That causes annoying diffs like:

tabs vs spaces

2 spaces vs 4 spaces

different line endings (LF vs CRLF)

missing final newline

trailing whitespace

Result: PRs show “changes” even though logic didn’t change.

.editorconfig prevents that by enforcing baseline rules at save time across editors.

root = true
“Stop searching.”
EditorConfig looks for .editorconfig up the folder tree. This tells it this is the top/root config, don’t inherit from any parent directory.
Global rules (applies to all files)
[*]

* means every file in this project.

charset = utf-8
Save files using UTF-8 encoding (avoids weird character issues).

indent_style = space
Use spaces, not tabs, for indentation.

indent_size = 2
Indent with 2 spaces per level.

end_of_line = lf
Use LF line endings (Linux/mac style).
Helps avoid Windows CRLF vs LF diffs in git.

insert_final_newline = true
Ensure the file ends with a newline (good POSIX practice; avoids tooling issues).

trim_trailing_whitespace = true
Remove extra spaces at the end of lines when saving (keeps diffs clean).
Rules only for JS/TS files
[*.{ts,js}]

Applies to files ending in .ts or .js.

quote_type = single
Prefer single quotes ' ' in these files.

⚠️ Note: quote_type is not part of the official EditorConfig core spec, so it works only in some editors/plugins. For guaranteed enforcement, use Prettier (singleQuote: true) or ESLint rules.
What this .eslintrc.json is doing (line by line)
1) "root": true

Same idea as EditorConfig: don’t look for ESLint config in parent folders. This file is the final authority.

2) "parser": "@typescript-eslint/parser"

ESLint by default understands JS, but TypeScript adds syntax (types, interfaces).
This parser lets ESLint read TypeScript files.

3) "plugins": ["@typescript-eslint"]

Plugins are “extra rule packs”.
This one adds TypeScript-specific rules like:

unused vars in TS context

no non-null assertions

better type-safe linting rules (depending on setup)

4) "extends": [...]

This is the most important part.

✅ "eslint:recommended"

Turns on a base set of rules that catch real bugs, like:

using undefined variables

unreachable code

accidental fallthrough in switch, etc.

✅ "plugin:@typescript-eslint/recommended"

Adds TS-focused recommended rules, like:

avoid any patterns (some)

handle TS-specific unsafe patterns

✅ "prettier"

This is to avoid fights between ESLint and Prettier.
Prettier formats code; ESLint should not try to enforce formatting rules that conflict.
So prettier disables those formatting-related ESLint rules.

5) "parserOptions"
"ecmaVersion": 2020

Allows modern JS syntax (optional chaining, etc. depending on version).

"sourceType": "module"

Allows import/export instead of require/module.exports.

Your "rules" section

ESLint rule severity levels:

0 = off

1 = warn

2 = error

"semi": [2, "always"]

Forces semicolons.
If missing → error.

"space-before-function-paren": [0, ...]

This rule is disabled (0), so ESLint won’t enforce spacing like:
function test () vs function test()

Also: this rule is mostly formatting → Prettier handles it anyway.

"camelcase": 0

Allows non-camelCase variables like:

user_id
Useful when dealing with DB fields / API responses.

"no-return-assign": 0

Allows patterns like:

return (x = 5)


This can be confusing, but you turned it off (so it won’t complain).

"quotes": ["error", "single"]

Forces 'single quotes'.
But note: Prettier also controls quotes if configured. If Prettier is running, it may already handle this.

TypeScript rules you turned off
@typescript-eslint/no-non-null-assertion: "off"

Allows value!.something.
Non-null assertion can hide runtime null bugs, but devs use it sometimes when they’re sure.

@typescript-eslint/no-namespace: "off"

Allows namespace X {}.
Usually avoided in modern TS (modules are preferred), but sometimes used in legacy patterns.

@typescript-eslint/explicit-module-boundary-types: "off"

Does NOT force you to write return types on exported functions.
Example: it won’t force:

export function foo(): string {}


This rule is often annoying, so many teams disable it.

Now the big question: Is ESLint still necessary in latest projects?
Yes — even with TypeScript + Prettier, ESLint is still valuable.

Because:

1) TypeScript ≠ ESLint

TypeScript mainly catches type-related issues.
ESLint catches logic + code-quality issues that TS won’t always catch, like:

unused variables/imports (TS catches some, but not always like ESLint does in mixed cases)

accidental shadowing of variables

bad promise handling patterns (with extra plugins)

inconsistent patterns across the codebase

dangerous JS patterns that are valid types

2) Prettier ≠ ESLint

Prettier only formats. It won’t tell you:

“this code is buggy”

“you forgot to handle error”

“this promise is not awaited”

“this variable is never used”

3) ESLint acts as a “team guardrail”

Even if you’re a good dev, ESLint:

keeps code consistent across devs

reduces PR review comments (“rename this”, “unused import”, “bad pattern”)

catches mistakes before they reach production

makes interviews/projects look professional (clean repo)

What you actually need ESLint for (practical list)

For a Node + TS backend, the most useful real-world benefits are:

✅ stop unused imports/vars

✅ stop accidental bugs (no-undef, unreachable code)

✅ enforce consistent code rules (quotes, semis, etc.)

✅ enforce safer TS patterns (as much as you want)

✅ keep PRs and codebase clean

One improvement suggestion for your setup (high impact)

Right now your config is fine, but many teams add these for production-level Node TS:

eslint-config-prettier (you already have via "prettier")

eslint-plugin-import (import order & errors)

eslint-plugin-promise (promise best practices)

eslint-plugin-security (basic security patterns)

and sometimes a stricter TS config: recommended-requiring-type-checking (optional)
.prettierrc.json is the config file for Prettier.

Mental model:
Prettier = auto-formatter. It rewrites your code to a consistent style (spacing, line breaks, quotes, commas) so humans don’t waste time arguing about formatting.

Why we use .prettierrc.json
1) One style for the whole team

Without it, everyone’s editor formats differently → PRs become full of “format-only” changes.

With it, formatting becomes predictable + same everywhere.

2) Saves time in reviews

Reviewers focus on logic, not:

“please fix indentation”

“use trailing commas”

“wrap this line”

3) Reduces merge conflicts

If everyone formats the same way, two branches are less likely to fight over whitespace.

4) Works across file types

Prettier formats not just JS/TS, but often:

JSON

Markdown

YAML

CSS/SCSS

HTML

What it controls (examples)

Prettier decides:

single vs double quotes

semicolons or not

trailing commas

max line width & wrapping

indentation size

bracket spacing

So your code looks consistent automatically.

Is it “necessary”?
Strictly speaking: No

Your project can run without Prettier.

Practically in real projects: Yes, it’s worth it

Because formatting is the most common, most annoying team friction, and Prettier removes it completely.

If you’re solo, you can skip it — but once you collaborate or open-source, it becomes super useful.

Prettier vs ESLint (quick clarity)

Prettier: formatting only ✅

ESLint: code correctness + best practices ✅

Best combo:

Prettier formats

ESLint catches bugs

ESLint config includes "prettier" to avoid rule conflicts
```json
{
  "trailingComma": "none",
  "tabWidth": 2,
  "semi": true,
  "singleQuote": true,
  "bracketSpacing": true,
  "printWidth": 140
}

```