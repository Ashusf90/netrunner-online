# Netrunner Online

Browser-based tabletop for Android: Netrunner. See `spec/NORTHPOLE.md` for product vision and `spec/TECHNICAL_VISION.md`
for technical direction.

## CSS — LiteWind

This project uses [LiteWind](https://litewindcss.com/) for utility-first CSS. Class names are Tailwind-compatible.

LLM class reference (fetch when writing or editing HTML with utility classes):
https://raw.githubusercontent.com/html-first-labs/static-tailwind/main/src/classes.txt

## Holographic Card Effect

Cards use a cursor-reactive holographic sheen inspired by https://poke-holo.simey.me/. Reimplement the effect in plain
CSS (custom properties + gradients + mix-blend-mode) with minimal JS for tracking cursor position. No dependencies.
