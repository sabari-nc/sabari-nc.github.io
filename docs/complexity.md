# Portfolio complexity

The redesign adds no client-side JavaScript. Navigation uses standard links and the video uses native browser controls.

[Cyclomatic complexity reference](https://en.wikipedia.org/wiki/Cyclomatic_complexity)

| File | Control flow | Complexity |
| --- | --- | --- |
| `_layouts/portfolio.html` | HTML and Liquid value interpolation only | No executable functions; cyclomatic complexity not applicable |
| `_layouts/post.html` | HTML and Liquid value interpolation only | No executable functions; cyclomatic complexity not applicable |
| `_pages/about.md` | Static HTML | Not applicable |
| `_pages/blog.md` | One Liquid loop over existing posts; no conditional branches | Template control-flow estimate: 2 (base path + one loop) |
| `_pages/publications.md` | Static Markdown and HTML | Not applicable |
| `_pages/404.md` | Static HTML | Not applicable |
| `_pages/research.md` and `_pages/portfolio.html` | Static fallback links and redirect metadata | Not applicable |
| `assets/css/portfolio.css` | CSS, including responsive and reduced-motion media queries | Not applicable; CSS media queries are not program control-flow branches |
| `_config.yml` and `.gitignore` | Configuration | Not applicable |
| GitHub Pages workflow | One branch condition prevents preview branches from deploying | Declarative workflow; one conditional gate |

An existing broken image was removed from one blog article. Its prose and existing research code were not rewritten. Existing code samples in other articles are outside the redesign scope.

Temporary authoring, rendering, and validation helpers are not part of the deployed site. No JavaScript libraries or runtime dependencies were added to the website.
