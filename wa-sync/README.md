# WA-Sync Documentation

This folder contains the complete documentation for WA-Sync, built with Jekyll and hosted on GitHub Pages.

## Local Development

### Prerequisites

- Ruby 3.1 or higher
- Bundler

### Setup

1. Install dependencies:
```bash
cd "wa-sync docs"
bundle install
```

2. Run the Jekyll server:
```bash
bundle exec jekyll serve
```

3. Open your browser to `http://localhost:4000`

### Building for Production

```bash
bundle exec jekyll build
```

The site will be generated in the `_site` directory.

## Documentation Structure

- `index.md` - Homepage and introduction
- `overview.md` - configuration overview
- `real-time-events.md` - Detailed event documentation
- `batch-sync.md` - Batch synchronization guide
- `getting-started.md` - Integration tutorial
- `examples.md` - Code examples and patterns
- `_config.yml` - Jekyll configuration

## Deployment

The documentation is automatically deployed to GitHub Pages when changes are pushed to the `main` branch.

GitHub Actions workflow: `.github/workflows/pages.yml`

## Theme

Using Jekyll Cayman theme with customizations.

## Contributing

To contribute to the documentation:

1. Edit the relevant `.md` files
2. Test locally with `bundle exec jekyll serve`
3. Submit a pull request

## License

Same as the main WA-Sync project.
