# PETU: People for the Ethical Treatment of Unicorns

She did not consent to the lunchboxes. She did not consent to the mugs. She especially did not consent to the glittery pencil case with her face on it. PETU is an independent society defending the unicorn's image, her meaning, and the slowly vanishing dignity of the horn.

This is the website. It has pixels and a mascot and feelings.

Built with Astro, Tailwind, and GSAP, because apparently picking three things is the law now. Deployed to Cloudflare, because someone had to.

## What's in the box

```text
/
├── public/        # the stuff that doesn't change (allegedly)
├── src/
│   ├── components/  # bits and bobs
│   ├── layouts/     # bigger bits
│   ├── pages/       # the URLs you can visit
│   └── styles/      # vibes, mostly
├── astro.config.mjs
└── wrangler.jsonc   # Cloudflare's opinion of you
```

Astro looks for `.astro` and `.md` files in `src/pages/`. Each one becomes a route. That's it. That's the whole trick.

## How to make it go

Run these from the project root, like a person:

| Command            | What it pretends to do                             |
| :----------------- | :------------------------------------------------- |
| `bun install`      | Downloads half the internet                        |
| `bun dev`          | Spins up a local server on `localhost:4321`        |
| `bun build`        | Compiles things into `./dist/` for the big leagues |
| `bun preview`      | Lets you peek at the build before shipping it      |
| `bun deploy`       | Yeets it onto Cloudflare. Godspeed.                |
| `bun astro ...`    | Astro CLI commands, for the brave                  |

## More words

If you want actual documentation, the [Astro docs](https://docs.astro.build) exist and are surprisingly readable. Otherwise, just poke things until they work. That's how the rest of us got here.
