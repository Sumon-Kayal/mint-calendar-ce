# Contributing

When contributing to the development of Mint Calendar CE, please first discuss the change you wish to
make via issue, email, or any other method with the maintainers before making a change.

## Code of Conduct

When interacting with the project, the [GNOME Code of Conduct](https://conduct.gnome.org/) applies.

## Use of Generative AI

Unlike upstream GNOME Calendar, this fork welcomes AI-assisted contributions — code, documentation,
or artwork produced or drafted with the help of LLMs, chatbots, or other generative AI tools —
provided the contribution is well documented and disclosed as such.

If you use AI tooling as part of a contribution, say so in the pull request description:

- Which tool(s) you used, and roughly how (e.g. "drafted the initial patch with an LLM, then
  hand-edited the recurrence-handling logic").
- What you personally reviewed, tested, or verified before submitting. AI-assisted contributions
  are held to the same bar as any other: they must build, must pass `meson test -C <builddir>`,
  and you should understand and be able to discuss them in review.
- Any part you're less confident about, so reviewers know where to look closely.

An AI-generated contribution submitted without disclosure, as if it were entirely hand-written, may
be rejected on that basis alone, independent of its technical merit.

Accepting or rejecting any contribution — AI-assisted or not — remains at the sole discretion of
the maintainer(s). Disclosure doesn't guarantee acceptance; it's what keeps review honest.

## Pull Request Process

1. Ensure your code compiles and doesn't break anything. Run `meson test -C <builddir>` before creating
   the pull request.
2. If you're adding new API, it must be properly documented.
3. The commit message is formatted as follows:

   ```plain
   component: <summary>

   A paragraph explaining the problem and its context.

   Another one explaining how you solved that.

   <link to the issue>
   ```

4. You may merge the pull request in once you have the sign-off of the maintainers, or if you
   do not have permission to do that, you may request the second reviewer to merge it for you.

## About the upstream codebase

Mint Calendar CE inherits its codebase from GNOME Calendar; learn more by reading our [HACKING.md](HACKING.md) file.

### Attribution

This Code of Conduct is adapted from the [Contributor Covenant][homepage], version 1.4,
available at [http://contributor-covenant.org/version/1/4][version]

[homepage]: http://contributor-covenant.org
[version]: http://contributor-covenant.org/version/1/4/
