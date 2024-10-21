# Pampa Labs Technical Blog

This repository contains the source for the Pampa Labs Technical Blog, built using MkDocs Material.

## Setup Instructions

1. Clone the repository:
   ```
   git clone <repository-url>
   cd <repository-name>
   ```

2. Create a virtual environment and activate it:
   ```
   python -m venv venv
   source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
   ```

3. Install dependencies:
   ```
   pip install -r requirements.txt
   ```

## Adding Content

1. Create a new Markdown file in the appropriate subdirectory under `docs/`:
   ```
   docs/category/your-new-page.md
   ```

2. The `nav` section in `mkdocs.yml` is now automatically generated based on the file structure in the `docs/` directory. There's no need to manually update it.

3. Ensure your new Markdown file includes the necessary front matter:
   ```yaml
   ---
   title: Your Page Title
   description: A brief description of your page
   date: YYYY-MM-DD
   authors:
     - Your Name
   tags:
     - tag1
     - tag2
   ---
   ```

To add a new author to the blog, open the `authors.yml` file and add the new author's information following this format:

```yaml
extra:
  authors:
    author_id:
      name: Full Name
      description: A brief bio or description
      avatar: path/to/avatar/image.jpg  # Optional
```

4. Commit your changes and push to the main branch.

The GitHub Actions workflow will automatically build and deploy your changes to the documentation site.

## Local Development

To preview your changes locally before pushing:

1. Run the MkDocs development server:
   ```
   mkdocs serve
   ```

2. Open your browser and go to `http://localhost:8000` to see the live preview of your documentation.

## Deployment

The site is automatically deployed via GitHub Actions when changes are pushed to the main branch. The workflow configuration can be found in `.github/workflows/ci.yml`.
