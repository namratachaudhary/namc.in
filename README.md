### LaTeX guide

Set these parameters along with the other ones in the frontmatter for LaTeX formatting to work.

```
---
katex: true
markup: mmark
description: "When using Katex, this field is required for the listing page to work correctly."
---
```

Once that's done, you can render LaTeX using the following syntax.

This will render the formula in the same line:

```
Integrate $$\int x^3 dx$$
```

This will render the formula in its own line

```
$$\int_{a}^{b} x^2 dx$$
```

### Gotchas

If something doesn't render or renders funny, switching the markup parser to `mmark` should do the trick.

This can be done by adding `markup: mmark` to the frontmatter.
