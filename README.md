# Mastering code reviews

Over the past 10+ years, in total I've spent majority of my time doing code reviews. At least that's what GitHub statistics show :). Here are some things I look for in the diffs when checking pull requests and how I approach their resolution. I focus on some areas and patterns based on the frequency and severity of issues that I usually find there, this is not meant to be an exhaustive list, but rather "quality minimum". Note that I spent lot of time at [Mews](https://github.com/MewsSystems/developers) so the stuff here will definitely be biased and opinionated, gravitating towards backend development in C# using functional programming principles.

## Code style

It doesn't matter what the exact code style rules are (e.g. the infamous spaces vs. tabs discussion), what really matters is how consistently they're followed. With consistent code-style, it's easy to spot errors just based on visual "look" of the code. Or by simply noticing that something feels odd. **Consistency leads to error prevention.** Some things you are not able to express in the type system, however consistent follow through on conventions can partially make up for that. That's the main driver, but readability and understandability are important as well.

## Naming

## Performance

## Security
