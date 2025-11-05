# Metal3 Documentation - AI Coding Assistant Instructions

## Project Overview

Central documentation repository for the Metal3 project using Hugo
static site generator. Contains architecture docs, user guides, API
references, and deployment workflows. Published at
<https://book.metal3.io>

## Structure

- `docs/` - Main documentation content (Markdown)
   - `user-guide/` - End-user documentation
   - `dev-env/` - Development environment setup
   - `api/` - API references
   - `architecture/` - Design documents
- `themes/` - Hugo theme customization
- `config.toml` - Hugo configuration
- `Makefile` - Build and serve commands

## Local Development

```bash
# Install Hugo
# Ubuntu: sudo snap install hugo
# macOS: brew install hugo

# Serve docs locally with live reload
make serve

# Build static site
make build

# Output in docs/_site/
```

## Adding Documentation

1. Create new Markdown file in appropriate `docs/` subdirectory

2. Add frontmatter:

   ```yaml
   ---
   title: "Your Page Title"
   weight: 10  # Ordering in navigation
   ---
   ```

3. Write content in Markdown

4. Test locally with `make serve`

5. Submit PR

## Documentation Guidelines

- Use clear, concise language
- Include code examples for technical content
- Add diagrams for architecture/workflows (use mermaid or images)
- Cross-link related pages
- Keep API docs in sync with actual APIs
- Test all commands/examples before publishing

## Common Pitfalls

1. **Broken Links** - Check internal links before committing
2. **Outdated Content** - Update docs when APIs/features change
3. **Missing Frontmatter** - Pages won't render properly without it
4. **Build Errors** - Hugo strict mode catches many issues, fix before merge

Documentation PRs should be reviewed for technical accuracy and clarity. When in doubt, ask project maintainers for guidance on organization and content.
