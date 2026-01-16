## Repository Overview

This is a Zenn.dev article management repository for publishing technical content. The main content is published at https://zenn.dev/tonkotsuboy_com.

## Content Structure and Standards

### Article Organization
- `articles/` - Japanese technical articles (main content)
- `articles_en/` - English translations or English-specific content  
- `books/` - Zenn book publications
- `images/` - Article images organized by article slug
- `.kiro/` - AI-assisted content specification and design documents

### Article Format Standards

All articles follow Zenn's frontmatter format:

```yaml
---
title: "Article Title"
emoji: "🚀" 
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "css", "typescript"]
published: true
publication_name: ubie_dev # optional organization
---
```

### Writing Style Guidelines
Based on analysis of existing articles, maintain these characteristics:
- **親しみやすい一人称**: Use「私」「筆者」for personal voice
- **体験談ベース**: Include actual experiences and genuine reactions
- **結論先出し**: Lead with conclusions and benefits, then provide details
- **技術的正確性**: Maintain technical accuracy while keeping explanations accessible
- **実用性重視**: Provide actionable information readers can immediately use

### Markdown Heading Rules
- **大見出し**: Use `#`（h1）for main section headings in articles
- **小見出し**: Use `##`（h2）for subsections

## Development Commands

### Content Preview
```bash
npm run preview
# Starts Zenn preview server at http://localhost:8000 for local article review
```


## Content Creation Workflow

### When Adding New Articles
1. Create markdown file in `articles/` directory
2. Use descriptive filename that matches article slug
3. Follow the established frontmatter format
4. Create corresponding image directory in `images/` if needed
5. Ensure Japanese content standards are met

### Image Management
- Store images in `images/[article-slug]/` directories
- Use relative paths from article root: `/images/article-slug/image.png`
- Common image formats: `.png`, `.jpg`, `.webp`

## Important Notes

- All content should be primarily in Japanese unless specifically creating English versions
- Maintain consistency with existing article quality and style
- Technical accuracy is paramount - verify all technical claims and code examples
- Focus on practical, actionable content that provides real value to developers
