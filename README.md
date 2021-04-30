# Pix Engineering

[Blog site](https://engineering.pix.fr) of the IT teams of Pix.

## Installation

### Requirements

- Git 2.25.0
- Ruby 2.7.1
- Bundler 2.1.4

### Instructions

```
git clone git@github.com:1024pix/pix-engineering.git
cd pix-engineering
bundle install
bundle exec jekyll serve

# To test posts whose publication is set to a future date
bundle exec jekyll serve -- future
```

You can access application at http://localhost:4000


## Usage

### Articles

### Authors

#### Adding an author

**1/** Create a new file called `<user_id>.yml` (with `user_id` is the concatenation of `<firstname>_<lastname>`) in folder `/_data/authors`:

```yaml
# /_data/authors/jeremy_buget.yml
name: Jérémy Buget
description: CTO
links:
  github: jbuget
  twitter: jbuget
  website: https://jbuget.fr
```

**2/** Add the user's picture in `/assets/images/authors/<user_id>.png`.

> ℹ️ **Picture must be a PNG file.** It should have square dimensions. 

#### Set blog post's author(s)

In the post's front matter, add a property `authors`, that could be a string (`user_id`), an array of strings or nothing:

```
# No author
authors:

# Single author
authors: jeremy_buget

# Multiple authors
authors:
  - jeremy_buget
  - melanie_boudard
```

## License

Copyright (c) 2020 GIP PIX.

This program is free software: you can redistribute it and/or modify it under the terms of the GNU Affero General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License along with this program. If not, see [gnu.org/licenses](https://www.gnu.org/licenses/).
