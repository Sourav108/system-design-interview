# Contributing to System Design Interview Curriculum

Thank you for your interest in contributing to this comprehensive, implementation-first System Design interview curriculum.

## Repository Principles

Every concept and module must connect:
```
Concept → Building Block → Implementation → Case Study → Interview Question
```

Every module must answer:
1. What is it?
2. Why does it exist?
3. How does it work?
4. What problem does it solve?
5. What are its trade-offs?
6. What can fail?
7. How does it scale?
8. How would I implement it?
9. Where is it reused?
10. How would I explain it in an interview?

## Contribution Guidelines

1. **Originality**: All content must be 100% original. Do not copy proprietary course material, proprietary diagrams, or copyrighted text.
2. **Templates**: Strictly adhere to the standardized templates located in `/templates/`:
   - Building Blocks: `templates/building-block-template.md` (29-section architecture)
   - Case Studies: `templates/case-study-template.md` (29-section architecture)
   - System Failures: `templates/system-failure-template.md`
   - Implementations: `templates/implementation-template.md`
3. **Single Source of Truth**: When adding or updating a module, update `CURRICULUM.md` and `roadmap/coverage-matrix.md`.
4. **Mermaid Diagrams**: All diagrams must be validated Mermaid syntax explaining actual architecture, request flow, or failure modes.
