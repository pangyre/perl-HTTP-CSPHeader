# NAME

HTTP::CSPHeader - manage dynamic content security policy headers

# VERSION

version v0.1.0

# SYNOPSIS

```perl
use HTTP::CSPheader;

my $csp = HTTP::CSPheader->new(
  policy => {
     "default-src" => q['self'],
     "script-src"  => q['self' cdn.example.com],
  },
  nonces_for => [qw/ script-src /],
);

...

use HTTP::Headers;

my $h = HTTP::Headers->new;

$csp->reset;

$h->amend(
  "+script-src" => "https://captcha.example.com",
  "+style-src"  => "https://captcha.example.com",
);

my $nonce = $csp->nonce;
$h->header( 'Content-Security-Policy' => $csp->header );

my $body = ...

$body .= "<script nonce="${nonce}"> ... </script>";
```

# DESCRIPTION

This module allows you to manage Content-Security-Policy (CSP) headers.

It supports dynamic changes to headers, for example, adding a source
for a specific page, or managing a random nonce for inline scripts or
styles.

It also supports caching, so that the header will only be regenerated
if there is a change.

# ATTRIBUTES

## policy

This is a hash reference of policies.  The keys a directives, and the
values are sources.

There is no validation of these values.

## nonces\_for

This is an array reference of the directives to add a random ["nonce"](#nonce)
to when the ["policy"](#policy) is regenerated.

Note that the same nonce will be added to all of the directives, since
using separate nonces does not improve security.

It is emply by default.

A single value will be coerced to an array.

This does not validate the values.

Note that if a directive allows `'unsafe-inline'` then a nonce may
cancel out that value.

## nonce

This is the random nonce that is added to directives in ["nonces\_for"](#nonces_for).

The nonce is a hex string based on a random 32-bit number, which is generated
from [Math::Random::ISAAC](https://metacpan.org/pod/Math%3A%3ARandom%3A%3AISAAC).  The RNG is seeded by `/dev/urandom`.

If you do not have `/dev/urandom` or you want to change how it is generated,
you can override the `_build_nonce` method in a subclass.

## header

This is the value of the header.

# METHODS

## reset

This resets any changes to the ["policy"](#policy) and clears the ["nonce"](#nonce).
It should be run at the start of each HTTP request.

## amend

```perl
$csp->amend( $directive1 => $value1, $directive2 => $value2, ... );
```

This amends the ["policy"](#policy).

If the `$directive` starts with a `+` then the value will be
appended to it.  Otherwise the change will overwrite the value.

If the value if `undef`, then the directive will be deleted.

# SEE ALSO

[https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)

[Mojolicious::Plugin::CSPHeader](https://metacpan.org/pod/Mojolicious%3A%3APlugin%3A%3ACSPHeader)

# SOURCE

The development version is on github at [https://github.com/robrwo/perl-HTTP-CSPHeader](https://github.com/robrwo/perl-HTTP-CSPHeader)
and may be cloned from [git://github.com/robrwo/perl-HTTP-CSPHeader.git](git://github.com/robrwo/perl-HTTP-CSPHeader.git)

# BUGS

Please report any bugs or feature requests on the bugtracker website
[https://github.com/robrwo/perl-HTTP-CSPHeader/issues](https://github.com/robrwo/perl-HTTP-CSPHeader/issues)

When submitting a bug or request, please include a test-file or a
patch to an existing test-file that illustrates the bug or desired
feature.

# AUTHOR

Robert Rothenberg <rrwo@cpan.org>

# COPYRIGHT AND LICENSE

This software is Copyright (c) 2022 by Robert Rothenberg.

This is free software, licensed under:

```
The Artistic License 2.0 (GPL Compatible)
```