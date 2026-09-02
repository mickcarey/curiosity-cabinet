# Tabs vs Spaces: The Tiny Indentation War ⚔️

## The Rivalry

Tabs say: indentation is structure, and each person can display that structure at their preferred width.

Spaces say: indentation is presentation, and everyone should see exactly the same layout.

This is one of programming's funniest arguments because both sides are partly right and most teams should solve it by letting a formatter end the meeting.

## The Real Difference

- **Tabs:** One character per indentation level. Display width can vary.
- **Spaces:** Fixed visual result. More characters, less ambiguity across tools.
- **Mixed indentation:** The swamp. Avoid the swamp.

Python's PEP 8 prefers spaces for indentation. Go chooses a different peace treaty: `gofmt` uses tabs for indentation and spaces for alignment.

## Why It Is Interesting

The argument looks petty until you notice what it is really about: should source code preserve semantic structure or visual layout?

That is a design question wearing a very small hat.

## Follow-Up Trails

- [PEP 8: Tabs or Spaces?](https://peps.python.org/pep-0008/#tabs-or-spaces)
- [Effective Go: Formatting](https://go.dev/doc/effective_go.html#formatting)
- [gofmt command documentation](https://go.dev/cmd/gofmt/)
