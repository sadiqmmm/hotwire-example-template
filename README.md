# Hotwire Example Template

A collection of demonstrations of how to implement features by sending [HTML
over the wire](https://hotwired.dev).

[![Deploy to Heroku](https://www.herokucdn.com/deploy/button.png)][heroku-deploy-app]

[heroku-deploy-app]: https://heroku.com/deploy?template=https://github.com/thoughtbot/hotwire-example-template/tree/hotwire-example-chat

## How to read this repository

Through the power of incremental Git diffs, each of this repository's
[branches][] provides a step-by-step demonstration of how to implement a feature
or behavior.

This repository's [main][] branch serves as the root all of the other branches,
and consists of a handful of commits generated by the Rails command line
interface.

When reading a branch's source code, read the changes commit-by-commit either on
the branch comparison page (for example,
[main...hotwire-example-live-preview][]), the branch's commits page (for
example, [hotwire-example-live-preview][]), or the branch's `README.md` file
(for example, [hotwire-example-live-preview][README]).

To experiment with a branch on your own, clone the repository, check out the
branch, execute its set up script, start the local server, then visit
<http://localhost:3000>:

```sh
bin/setup
bin/rails server
open http://localhost:3000
```

[branches]: https://github.com/thoughtbot/hotwire-example-template/branches/all
[main]: https://github.com/thoughtbot/hotwire-example-template/tree/main
[main...hotwire-example-live-preview]: https://github.com/thoughtbot/hotwire-example-template/compare/hotwire-example-live-preview
[hotwire-example-live-preview]: https://github.com/thoughtbot/hotwire-example-template/commits/hotwire-example-live-preview
[README]: https://github.com/thoughtbot/hotwire-example-template/blob/hotwire-example-live-preview/README.md
