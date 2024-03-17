<p align="center">
<img src="https://raw.githubusercontent.com/rustic-rs/assets/main/logos/readme_header_docs.png" height="400" />
</p>
<p align="center">
<b>user documentation</b>
</p>

<p align="center">
<a href="https://crates.io/crates/rustic-rs"><img src="https://img.shields.io/crates/v/rustic-rs.svg" /></a>
<a href="https://raw.githubusercontent.com/rustic-rs/docs/main/LICENSE"><img src="https://img.shields.io/badge/license-Apache2.0/MIT-blue.svg" /></a>
<a href="https://crates.io/crates/rustic-rs"><img src="https://img.shields.io/crates/d/rustic-rs.svg" /></a>
<p>

An open source user documentation book for
[rustic](https://github.com/rustic-rs/rustic) that you can read
[here](https://rustic.cli.rs/docs).

## Installation

This book is built with [mdbook](https://rust-lang.github.io/mdBook/). You can
install it by running `cargo install mdbook`.

### Additional dependencies

- `cargo install mdbook-last-changed` for date changes in the footer

- `cargo install mdbook-pandoc` for rendering the book to PDF

#### Texlive

```sh
# Source the .env file to get the PANDOC_VERSION
. ./.env

sudo apt-get update

sudo apt-get install -y texlive texlive-latex-extra texlive-luatex texlive-lang-cjk librsvg2-bin fonts-noto

curl -LsSf https://github.com/jgm/pandoc/releases/download/$PANDOC_VERSION/pandoc-$PANDOC_VERSION-linux-amd64.tar.gz | tar zxf -
```

## Building with mdbook

If you want to build it locally you can run one of these two commands in the
root directory of the repository:

- `mdbook build`

  Builds static html pages as output and places them in the `/book` directory by
  default.

- `mdbook serve`

  Serves the book at `http://localhost:3000` (port is changeable, take a look at
  the terminal output to be sure) and reloads the browser when a change occurs.

## License

The content of this repository is licensed under **MPL-2.0**; see
[LICENSE](./LICENSE).
