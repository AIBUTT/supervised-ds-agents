---
applyTo: "**"
description: "Equations in every role's chat replies, rendered with KaTeX."
---
# Equations in chat

- All roles, including the Coordinator, write chat equations in KaTeX and define their terms: `$...$` inline and `$$...$$` for display blocks. Example: `$\delta = |d_{\mathrm{candidate}}| - |d_{\mathrm{reference}}|$`.
- Use only commands that KaTeX supports; do not rely on LaTeX packages, custom macros or environments that KaTeX does not render.
- Preserve LaTeX in Markdown report sources and rendered reports. This chat rule does not change scientific notation in source artifacts.
- Offer rendered offline reports for readers who do not use Markdown tools.
