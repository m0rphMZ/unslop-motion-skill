# Unslop Motion

A vendor-neutral skill that teaches AI models to design UI motion that feels intentional. 20 rules, one 40-second pass, plain Markdown.

## Structure

```text
.
├── README.md
├── LICENSE
└── unslop-motion/
    ├── SKILL.md
    ├── checklist.md
    ├── rules.md
    └── examples.md
```

`SKILL.md` is the entry point. The other three are references it points to: a fast checklist, the expanded rules, and before/after examples.

## Use it

Copy the `unslop-motion` folder into your skills directory:

```text
~/.claude/skills/unslop-motion/
~/.config/opencode/skills/unslop-motion/
.opencode/skills/unslop-motion/
```

For models without skill support, paste `unslop-motion/SKILL.md` into the system prompt or the start of a conversation.

## The 20 rules

1. Ease-out for entrances
2. Dropdowns at 150-250ms
3. Stagger list rows by 30ms
4. Scale to 0.97 on press
5. Stretch moving knobs 12%
6. Stagger as details settle
7. Arcs for object movement
8. Pulse to confirm success
9. Ease-in-out for moves
10. Start popovers at 0.95
11. Enter from the trigger
12. Exit to the trigger
13. No bounce on everyday UI
14. Don't tween while held
15. Make exits 20% shorter
16. Overlap old & new states
17. Masks for reveals & colour
18. Crossfade icons + blur
19. Rubber band past bounds
20. Reuse the same layoutId

## License

MIT
