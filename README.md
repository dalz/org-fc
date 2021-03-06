Org Flashcards
==============

Spaced-repetition system for use with Emacs org-mode.

![Review Demo](images/review.png)

Org-fc has been tested with org-mode version 9.3.6 and gawk version
5.1.0. You can check your versions with `M-x org-version` and `gawk -V`.

Introduction
------------

In the most abstract sense, this package deals with

1.  Attaching timestamped review information to headlines
2.  Querying all headings where reviews are due
3.  Reviewing due **positions** of headings

As mentioned in step 3, a heading can have multiple **positions**, e.g.
to implement cloze-deletions where multiple items are reviewed
independently from each other.

In the reviewing step, display functions can be registered by card type.
This allows easy addition of user-defined card types without having to
think about storing and updating review data.

Review functions are called with point on the headline of the card that
should be reviewed and get passed a single argument, the position to be
reviewed.

They are expected to return either `'quit` to end the review or one of
`'again`, `'hard`, `'good`, `'ease`, to rate the card.

See also:

-   [Alternative Applications](doc/alternative_applications.org)
-   [Incremental Reading](doc/incremental_reading.org) (todo)

Design Goals / Choices
----------------------

1.  Use of `awk` for quickly finding cards due for review
2.  Support for multiple **positions** in a card / heading
3.  All relevant data kept in org files for easy version control
4.  Review directly on the source org file for easy editing of cards
    during review

See [Design Choices](doc/design_choices.org) for more information.

Prior Art
---------

There are a few other packages for implementing a SRS based on org-mode.

Below, I\'ve listed a the ones I\'ve found so far that are actively
maintained and implement a lot of useful functionality.

-   [phillord/org-drill](https://gitlab.com/phillord/org-drill/)
-   [abo-abo/pamparam](https://github.com/abo-abo/pamparam)

Other (open source) flashcard / spaced repetition software:

-   [Anki](https://apps.ankiweb.net/)
-   [Mnemosyne Project](https://mnemosyne-proj.org/)

Thanks to the maintainers and all contributors for their work on these
packages!

Performance
-----------

All user-facing commands (especially during review) should be as fast as
possible (\<300ms).

Using the `awk` indexer, searching 2500 org files (\~200k lines in
total) for due flashcards takes around \~500ms on my laptop (Thinkpad
L470, SSD).

Using the lisp indexer based on `org-map-entries`, searching a single
6500 line file with 333 flashcards takes \~1000ms, indexing the same
file with `awk` takes around \~50ms.

Getting Started
---------------

Before using this package, `org-fc-directories` should be set to the
directory to search for org files containing flashcards.

The file used to store the review history can be customized with
`org-fc-review-history-file` and defaults to
`/path/to/config/org-fc-reviews.tsv`.

This package is not (yet) available on MELPA / ELPA, to install it,
clone the repository (e.g. to `src/org-fc/`) and follow the setup
instructions.

### Dependencies

-   The `gawk` extension of `awk` for extracting review data from `.org`
    files
-   `find` for finding all org files in the `org-fc-directories`
-   `xargs` for processing files in parallel

1.  Linux / UNIX

    `find` and `xargs` should be included in most distributions, `gawk`
    can be installed from source or using your package manager of
    choice.

    Examples:

    -   `pacman -S gawk` (Arch Linux)
    -   `apt-get install gawk` (Ubuntu, Debian, ...)
    -   `yum install gawk` (CentOS)

    For more information, see [gawk manual -
    Installation](https://www.gnu.org/software/gawk/manual/html_node/Installation.html).

2.  MacOS

    On MacOS all dependencies can be installed using
    [macports](https://www.macports.org/) or
    [homebrew](https://brew.sh/). `find` and `xargs` are part of the
    `findutils` package.

    -   `port install gawk findutils`
    -   `brew install gawk findutils`

    You might have to adjust your `$PATH` to make sure Emacs can find
    all relevant binaries.

    ``` {.bash}
    export PATH="/opt/local/libexec/gnubin:/opt/local/bin:$PATH"
    ```

    For more information, refer to the documentation of macports /
    homebrew.

### Setup with [use-package](https://github.com/jwiegley/use-package/)

``` {.commonlisp org-language="emacs-lisp"}
(use-package hydra)
(use-package org-fc
  :load-path "~/src/org-fc"
  :custom (org-fc-directories '("~/org/"))
  :config
  (require 'org-fc-hydra))
```

Or, using [straight.el](https://github.com/raxod502/straight.el/):

``` {.commonlisp org-language="emacs-lisp"}
(use-package hydra)
(use-package org-fc
  :straight
  (org-fc
   :type git :host github :repo "l3kn/org-fc"
   :files (:defaults "awk" "demo.org"))
  :custom
  (org-fc-directories '("~/org/"))
  :config
  (require 'org-fc-hydra))
```

Note that in this case, you don\'t have to clone the repository.

### Setup with [straight.el](https://github.com/raxod502/straight.el/)

``` {.commonlisp org-language="emacs-lisp"}
(straight-use-package 'hydra)
(straight-use-package
 '(org-fc
   :type git :host github :repo "l3kn/org-fc"
   :files (:defaults "awk" "demo.org")
   :custom (org-fc-directories '("~/org/"))
   :config
   (require 'org-fc-hydra)))
```

### Setup with [spacemacs](https://github.com/syl20bnr/spacemacs/)

You don\'t need to manually clone the repository, just put this in your
`.spacemacs`:

``` {.commonlisp org-language="emacs-lisp"}
;; ...
dotspacemacs-additional-packages
'((org-fc
   :location (recipe :fetcher github
                     :repo "l3kn/org-fc"
                     :files (:defaults "awk" "demo.org"))))
;; ...
(defun dotspacemacs/user-config ()
  ;; ...
  ;; Org-fc
  (use-package hydra)
  (require 'org-fc-hydra)
  (setq org-fc-directories '("~/org/"))
  ;; ...
  )
```

### Minimal Setup

Assuming [abo-abo/hydra](https://github.com/abo-abo/hydra) is already
loaded.

``` {.commonlisp org-language="emacs-lisp"}
(add-to-list 'load-path "~/src/org-fc/")
(setq org-fc-directories '("~/org/"))
```

### Demo Mode

A file demonstrating all card types is included. `M-x org-fc-demo`
starts a review of this file.

Card Types
----------

This package comes with a few predefined card types. They are documented
in [Card Types](doc/card_types.org).

[demo.org](demo.org) includes examples for each of these types.

Marking Headlines as Cards
--------------------------

A **card** is an org-mode headline with a `:fc:` tag attached to it.
Each card can have multiple **positions** reviewed independently from
each other, e.g. one for each hole of a cloze card.

Review data (ease, interval in days, box, due date) is stored in a table
in a drawer inside the card.

``` {.org}
:REVIEW_DATA:
| position | ease | box | interval | due                    |
|----------+------+-----+----------+------------------------|
|        2 | 2.65 |   6 |   107.13 |    2020-04-07T01:01:00 |
|        1 | 2.65 |   6 |   128.19 |    2020-04-29T06:44:00 |
|        0 | 2.95 |   6 |   131.57 |    2020-04-30T18:03:00 |
:END:
```

Review results are appended to a csv file to avoid cluttering the org
files.

Each card needs at least two properties, an **unique** `:ID:` and a
`:FC_TYPE:`. In addition to that, the date a card was created (i.e. the
headline was marked as a flashcard) is stored to allow making statistics
for how many cards were created in the last day / week / month.

``` {.org}
:PROPERTIES:
:ID:       4ffe66a7-7b5c-4811-bd3e-02b5c0862f55
:FC_TYPE:  normal
:FC_CREATED: 2019-10-11T14:08:32
:END:
```

Card types (should) implement a `org-fc-type-...-init` command that
initializes these properties and sets up the review data drawer

All timestamps created and used by org-flashcards use ISO8601 format
with second precision and without a timezone (timezone UTC0).

This prevents flashcard due dates from showing up in the org-agenda and
allows filtering for due cards by string-comparing a timestamp with one
of the current time.

(Un)suspending Cards
--------------------

Cards can be suspended (excluded from review) by adding a `suspended`
tag, either by hand or using the `org-fc-suspend-card` command.

The `suspended` tag is inherited, so all cards in a subtree can be
suspended by adding the tag to the parent heading, and all cards in a
file can be suspended by adding `#+FILETAGS: suspended` at the start.

Cards can be unsuspended using the `org-fc-unsuspend-card` command or by
manually removing the `suspended` tag.

It might be preferable to suspend multiple cards by adding the
`suspended` tag to each one, so they remain suspended when moved to
another headline or file.

In this case, you can use the following commands:

-   `org-fc-suspend-tree`, `org-fc-unsuspend-tree` for suspending all
    cards in a subtree
-   `org-fc-suspend-buffer`, `org-fc-unsuspend-buffer` for suspending
    all cards in the current buffer

Note that these commands don\'t affect filetags or tags of parent
headlines.

Spacing Algorithm
-----------------

This package uses a modified version of the SM2 spacing algorithm, based
on the one used by Anki.

See also:

-   [Custom Spacing Algorithms](doc/custom_spacing_algorithms.org)
    (todo)
-   [Anki Manual -
    Algorithm](https://apps.ankiweb.net/docs/manual.html#what-algorithm)
-   [SuperMemo - SM2
    Algorithm](https://www.supermemo.com/en/archives1990-2015/english/ol/sm2)

Review
------

A review session can be started with `M-x org-fc-review`. Due cards are
reviewed in random order.

If a card was rated \"again\", it will be reviewed again at the end of
the current review session. This can be disabled by setting
`org-fc-append-failed-cards` to `nil`.

### Review Process

1.  Open file of card
2.  Narrow to heading
3.  Set up card for review
4.  Activate `org-fc-flip-mode`
5.  Flip the card (user)
6.  Switch to `org-fc-rate-mode`
7.  Rate the card (user)
8.  Repeat process with next due card

### Flip Mode

  Key   Binding
  ----- --------------------------
  RET   flip card
  n     flip card
  s     suspend card
  p     pause review for editing
  q     quit review

### Rate Mode

  Key   Binding
  ----- --------------------------
  a     rate again
  h     rate hard
  g     rate good
  e     rate easy
  s     suspend card
  p     pause review for editing
  q     quit review

### See Also

-   [Review Internals](doc/review_internals.org)
-   [Review History](doc/review_history.org)

Use with `evil-mode`
--------------------

The key bindings used by the review modes of org-fc conflict with some
of the bindings used by evil mode.

As a workaround, you can add minor mode keymaps for each of the
evil-mode states you\'re using org-fc with.

``` {.commonlisp org-language="emacs-lisp"}
(evil-define-minor-mode-key '(normal insert emacs) 'org-fc-review-flip-mode
  (kbd "RET") 'org-fc-review-flip
  (kbd "n") 'org-fc-review-flip
  (kbd "s") 'org-fc-review-suspend-card
  (kbd "q") 'org-fc-review-quit)

(evil-define-minor-mode-key '(normal insert emacs) 'org-fc-review-rate-mode
  (kbd "a") 'org-fc-review-rate-again
  (kbd "h") 'org-fc-review-rate-hard
  (kbd "g") 'org-fc-review-rate-good
  (kbd "e") 'org-fc-review-rate-easy
  (kbd "s") 'org-fc-review-suspend-card
  (kbd "q") 'org-fc-review-quit)
```

Review Contexts
---------------

By default, two contexts are defined:

all
:   all cards in `org-fc-directories`

buffer
:   all cards in the current buffer

New contexts can be defined by adding them to the alist
`org-fc-custom-contexts`.

Contexts have the form `(:paths paths :filter filter)`.

-   `:paths` (optional) either a list of paths, a single path or
    `'buffer` for the current buffer. Paths don\'t have to be included
    in the `org-fc-directories`. Defaults to `org-fc-directories`.
-   `:filter` (optional), a card filter defaulting to a filter that
    matches all cards.

Filters can be combinations of the following expressions:

-   `(and ex1 ex2 ...)`
-   `(or ex1 ex2 ...)`
-   `(not ex)`
-   `(tag "tag")`
-   `(type card-type) or (type "card-type")`

### Examples

All double cards with tag \"math\":

``` {.commonlisp org-language="emacs-lisp"}
(add-to-list 'org-fc-custom-contexts
  '(double-math-cards . (:filter (and (type double) (tag "math")))))
```

All cards in that don\'t have one of the tags \"foo\" and \"bar\":

``` {.commonlisp org-language="emacs-lisp"}
(add-to-list 'org-fc-custom-contexts
  '(no-foo-bar-cards . (:filter (not (or (tag "foo") (tag "bar"))))))
```

All cards in `~/combinatorics/` or `~/number_theory.org`:

``` {.commonlisp org-language="emacs-lisp"}
(add-to-list 'org-fc-custom-contexts
  '(math-cards . (:paths ("~/combinatorics/" "~/number_theory.org"))))
```

All cards in `~/combinatorics/` with tag \"theorem\":

``` {.commonlisp org-language="emacs-lisp"}
(add-to-list 'org-fc-custom-contexts
  '(combinatorics-theorems .
    (:paths "~/combinatorics/" :filter (tag "theorem"))))
```

All double cards in the current buffer:

``` {.commonlisp org-language="emacs-lisp"}
(add-to-list 'org-fc-custom-contexts
  '(current-double .
    (:paths buffer :filter (type double))))
```

### Note

Because parsing of tags is done in AWK, tag filters don\'t work for tags
defined in the `#+FILETAGS:` of a `#+SETUP_FILE:`.

Dashboard / Statistics
----------------------

`org-fc-dashboard` shows a buffer with statistics for review performance
and cards / card types.

![](images/stats.png)

Only cards with a box \>= `org-fc-stats-review-min-box` (default: 0) are
included in the review statistics.

Setting this to a higher value (e.g. 2) excludes the first few
\"learning\" reviews of a card.

See also:

-   [Review History](doc/review_history.org)
-   [Advanced Statistics](doc/advanced_statistics.org) (todo)

Hooks
-----

-   `org-fc-before-setup-hook` Runs before a card is set up for review
-   `org-fc-after-setup-hook` Runs after a card is set up for review
-   `org-fc-after-review-hook` Runs when the review ends / is quit

Extensions
----------

Org-fc comes with a number of extensions that are not enabled by
default.

### `org-fc-audio`

Can be enabled with `(require 'org-fc-audio)`.

Adds audio attachments for cards that are played during review, either
before or after a card is set up. (This distinction is relevant for
text-input cards).

Commands:

-   `org-fc-audio-set-before`
-   `org-fc-audio-set-after`

### `org-fc-keymap-hint`

Can be enabled with `(require 'org-fc-keymap-hint)`.

Shows a list of available key bindings during the review, to recreate
the look & feel of the previous hydra-based implementation.

-   `[RET] flip [q] quit [s] suspend-card`
-   `[a] rate-again [h] rate-hard [g] rate-good [e] rate-easy [s] suspend-card [q] quit`

### `org-fc-hydra`

[file:org-fc-hydra.el](org-fc-hydra.el) defines a hydra for accessing
commonly used org-fc commands and for marking headlines as flashcards.

It can be loaded and bound to a hotkey like this:

``` {.commonlisp org-language="emacs-lisp"}
(require 'org-fc-hydra)
(global-set-key (kbd "C-c f") 'org-fc-hydra/body)
```

[Changelog](Changelog.org)
--------------------------

Other Documentation
-------------------

-   [Internals](doc/internals.org)
-   [Sharing Decks](doc/sharing_decks.org) (todo)

License
-------

Copyright © Leon Rische and contributors. Distributed under the GNU
General Public License, Version 3
