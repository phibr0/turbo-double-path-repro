# Turbopack "Doubled Path" Issue in Monorepos

I stumbled upon this Issue when trying to use multiple Tailwind Configs in
nextjs while we are migrating from the pages to app directory: Usually you can
set the tailwind config using the `@config` directive in the imported css file,
but turbopack can't find the config file when running in a monorepo.

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@config "../app-tailwind.config.js";
```

```
 ⨯ ./apps/web/app/globals.css
Error evaluating Node.js code
CssSyntaxError: /Users/xxx/turbopack-tailwind-config-directive/apps/web/apps/web/app/globals.css:5:1: The config file at "../app-tailwind.config.js" does not exist. Make sure the path is correct and the file exists.
    [at /Users/xxx/turbopack-tailwind-config-directive/apps/web/apps/web/app/globals.css:5:1]
    at Input.error (node_modules/.pnpm/postcss@8.4.35/node_modules/postcss/lib/input.js:106:16) [/Users/xxx/turbopack-tailwind-config-directive/apps/web/.next/chunks/08b5e__pnpm_62fa9c._.js:4098:22]
    at AtRule.error (node_modules/.pnpm/postcss@8.4.35/node_modules/postcss/lib/node.js:115:32) [/Users/xxx/turbopack-tailwind-config-directive/apps/web/.next/chunks/08b5e__pnpm_62fa9c._.js:882:38]
    [at /Users/xxx/turbopack-tailwind-config-directive/node_modules/.pnpm/tailwindcss@3.4.1/node_modules/tailwindcss/lib/lib/findAtConfigPath.js:42:24]
```

After augmenting tailwinds `findAtConfigPath.js` with a few console.logs, I
found that turbopack seems to be passing a broken path to it:

```
   ▲ Next.js 14.1.1 (turbo)
   - Local:        http://localhost:3000

 ✓ Ready in 674ms
 ○ Compiling / ...
  configPath: '/Users/xxx/turbopack-tailwind-config-directive/apps/web/apps/web/app-tailwind.config.js',
  inputPath: '../app-tailwind.config.js',
  relativeTo: '/Users/xxx/turbopack-tailwind-config-directive/apps/web/apps/web/app/globals.css'
 ✓ Compiled / in 1015ms
 ⨯ ./apps/web/app/globals.css
Error evaluating Node.js code
CssSyntaxError: /Users/xxx/turbopack-tailwind-config-directive/apps/web/apps/web/app/globals.css:5:1: The config file at "../app-tailwind.config.js" does not exist. Make sure the path is correct and the file exists.
    [at /Users/xxx/turbopack-tailwind-config-directive/apps/web/apps/web/app/globals.css:5:1]
    at Input.error (node_modules/.pnpm/postcss@8.4.35/node_modules/postcss/lib/input.js:106:16) [/Users/xxx/turbopack-tailwind-config-directive/apps/web/.next/chunks/08b5e__pnpm_62fa9c._.js:4098:22]
    at AtRule.error (node_modules/.pnpm/postcss@8.4.35/node_modules/postcss/lib/node.js:115:32) [/Users/xxx/turbopack-tailwind-config-directive/apps/web/.next/chunks/08b5e__pnpm_62fa9c._.js:882:38]
    [at /Users/xxx/turbopack-tailwind-config-directive/node_modules/.pnpm/tailwindcss@3.4.1/node_modules/tailwindcss/lib/lib/findAtConfigPath.js:42:24]
```

The Paths passed to tailwind include the subdirectory of the web app in the
monorepo two times: `/apps/web*/apps/web*/app-tailwind.config.js` which seems to
be the issue here.

## Reproduction

1. `pnpm install`
2. `cd apps/web`
3. `pnpm dev --turbo`
