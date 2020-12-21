# Notes for December 19, 2020

## Goals

- Try and get my zshrc configured via home-manager
- Build sheldon as an overlay

## Nix commands

- After installing flakes search
  ```sh
  nix search nixpkgs hello
  ```

## Nixpkgs - Overlays - Derivations

- SHA sums for derivation sources are usually recorded as
  [sha256][nix-sha256sum-doc] sums in base64 format. Seems like Nixpkgs is
  moving to base32 to convert back to base64 use `nix to-base32 ${sha254sum}`

- Use the following to [build an expression in function format][build-function]
  i.e. an overlay or [expression missing arguments][nix-build-args]

  ```sh
  nix-build -E 'with import <nixpkgs> {}; callPackage ./hello.nix {}'
  ```

- Building rust crates in NixOS, see the following sources: [1][rust-1]
  [2][https://nixos.org/manual/nixpkgs/stable/#rust]

```nix
{ stdenv, fetchFromGitHub, rustPlatform, pkgconfig, openssl, darwin }:

rustPlatform.buildRustPackage rec {
  pname = "cargo-outdated";
  version = "unstable-2019-04-13";

  src = fetchFromGitHub {
    owner = "kbknapp";
    repo = pname;
    rev = "ce4b6baddc94b77a155abbb5a4fa4d3b31a45598";
    sha256 = "0x00vn0ldnm2hvndfmq4g4q5w6axyg9vsri3i5zxhmir7423xabp";
  };

  cargoSha256 = "1xqii2z0asgkwn1ny9n19w7d4sjz12a6i55x2pf4cfrciapdpvdl";

  nativeBuildInputs = [ pkgconfig ];
  buildInputs = [ openssl ]
  ++ stdenv.lib.optionals stdenv.isDarwin [
    darwin.apple_sdk.frameworks.Security
  ];

  meta = with stdenv.lib; {
    description = "A cargo subcommand for displaying when Rust dependencies are out of date";
    homepage = https://github.com/kbknapp/cargo-outdated;
    license = with licenses; [ asl20 /* or */ mit ];
    platforms = platforms.all;
    maintainers = [ maintainers.sondr3 ];
  };
}

```

- Derivations expressed in a function format can be built with repl

  ```nix
  :l <nixpkgs>
  sheldon = super.callPackage sheldon {};
  ```

## References

[nix-sha256sum-doc]: https://logs.nix.samueldr.com/nixos/2020-10-31
[build-function]: https://releases.nixos.org/nix-dev/2017-July/024058.html
[nix-build-args]: https://nixos.org/manual/nix/stable/#sec-arguments
[rust-1]: https://github.com/sondr3/dotfiles/blob/a9415bf1f3253bfd282332dd754e637276e532a8/pkgs/cargo-outdated.nix
[rust-2]: https://nixos.org/manual/nixpkgs/stable/#rust
