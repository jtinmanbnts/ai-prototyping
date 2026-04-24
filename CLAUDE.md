# CLAUDE.md

## Project
Verizon Business Portal prototype.
Figma file key: TaFgKARVKCKhx2PEImuJ2A
Node IDs tracked in figma.config.js

## Stack
Single-file React components rendered via Babel standalone in index.html.
No build step. No Tailwind. No external dependencies beyond React CDN.
Inline styles only. Never hardcode hex values — use the token constants at the top of each component file.

## File locations
- Screens: src/screens/<Name>/component.js
- Main entry: index.html (imports screens via script tags)

## When implementing a Figma frame
1. Check figma.config.js for the node ID
2. Write/update the component at src/screens/<Name>/component.js
3. Update index.html to reference it if it's a new screen
4. Run: npx vercel --prod
5. Post the Vercel URL as a comment on the Figma frame

## When reading Vercel feedback
1. Check for pinned comments on the deployed URL
2. Implement approved changes in the component file
3. Update the corresponding Figma frame to match
4. Run: npx vercel --prod
