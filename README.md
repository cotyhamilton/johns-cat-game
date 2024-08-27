# create-svelte

Everything you need to build a Svelte project, powered by [`create-svelte`](https://github.com/sveltejs/kit/tree/main/packages/create-svelte).

## Creating a project

If you're seeing this, you've probably already done this step. Congrats!

```bash
# create a new project in the current directory
deno run -A npm:create-svelte@latest

# create a new project in my-app
deno run -A npm:create-svelte@latest my-app

# add deno types
deno types > src/deno.d.ts
```

## Developing

```bash
DENO_FUTURE=1 deno install
```

Once you've created a project and installed dependencies with `deno install`, start a development server:

```bash
DENO_FUTURE=1 deno task dev

# or start the server and open the app in a new browser tab
DENO_FUTURE=1 deno task dev --open
```

## Building

To create a production version of your app:

```bash
DENO_FUTURE=1 deno task build
```

You can preview the production build with `DENO_FUTURE=1 deno task preview`.

> To deploy your app, you may need to install an [adapter](https://kit.svelte.dev/docs/adapters) for your target environment.

## deploy

```bash
DENO_FUTURE=1 deno task build
cd build
deployctl deploy --project=johns-cat-game --entrypoint=https://jsr.io/@std/http/1.0.0-rc.5/file_server.ts --prod
```
