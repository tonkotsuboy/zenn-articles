# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**重要: 必ず日本語で回答してください。**

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

## Development Commands

### Content Preview
```bash
npm run preview
# Starts Zenn preview server at http://localhost:8000 for local article review
```

### Creating New Content
```bash
npx zenn new:article
# Creates a new article with auto-generated slug in articles/

npx zenn new:book
# Creates a new book in books/

npx zenn list:articles
# Lists all articles with their publication status

npx zenn list:books
# Lists all books
```

### Dependencies and Environment
The project uses:
- **Node.js**: Version managed by Volta (currently 25.2.1)
  - Volta automatically switches to the correct Node version for this project
- **zenn-cli**: Zenn's official CLI for content management
- **prettier**: Code formatting (though manual formatting may be preferred for articles)

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

### AI-Assisted Content Development
The `.kiro/` directory contains specifications for AI-assisted content creation:
- `design.md` - Content design and architecture specifications
- `requirements.md` - Detailed requirements with acceptance criteria  
- `tasks.md` - Implementation task breakdowns

When working with AI-generated content, ensure alignment with these specifications while maintaining the repository's established writing standards.

## Important Notes

- All content should be primarily in Japanese unless specifically creating English versions
- Maintain consistency with existing article quality and style
- Technical accuracy is paramount - verify all technical claims and code examples
- Focus on practical, actionable content that provides real value to developers