# Tailwind CSS

- Rationale: utility-first CSS framework for rapid UI development with excellent TypeScript integration. Version 4 features zero configuration and CSS-first customization.
- Installation: Install `@tailwindcss/vite` plugin for Vite projects. No separate PostCSS configuration needed.
- Typical files: CSS entry file with `@import "tailwindcss"` directive. Configuration is done in CSS using `@theme` and custom properties, not a JavaScript config file.
- Agent checks: `pnpm run build` should process Tailwind classes correctly. Verify `@tailwindcss/vite` plugin is configured in `vite.config.ts`. Content detection is automatic - no manual content paths needed.
- Notes: Tailwind v4 uses CSS-first configuration (no `tailwind.config.ts` required). The Vite plugin provides tight integration. Automatic content detection eliminates the need for manual file path configuration. JIT mode is always enabled by default.
- Tailwind will be integrated with [Nuxt](./nuxt.md) app in `app/assets/css/main.css`, with `main.css` configured with an entry of `css: ['~/assets/css/main.css']` in the `nuxt.config.ts`.
