# Pix Engineering

Pix IT team's [blog](https://engineering.pix.fr) 

## Setup

### Local

#### Requirements

- Git 2.25.0
- Ruby 3.3.6
- Bundler 4.0

#### Instructions

```
git clone git@github.com:1024pix/pix-engineering.git
cd pix-engineering
bundle install
bundle exec jekyll serve
```

### Local with Docker

#### Requirements
- Docker
- Docker-compose > 3

#### Instructions
```
git clone git@github.com:1024pix/pix-engineering.git
cd pix-engineering
docker-compose up --detach
```

## Usage
You can access application at http://localhost:4000

### Articles

### Authors

#### Adding an author

**1/** Create a new file called `<user_id>.yml` (with `user_id` is the concatenation of `<firstname>_<lastname>`) in folder `/_data/authors`:

```yaml
# /_data/authors/firstname_lastname.yml
name: FirstName LastName
description: Developer
links:
  github: github_username
  twitter: twitter_username
  website: https://yourwebsite.com
```

**2/** Add the user's picture in `/assets/images/authors/<user_id>.png`.

> ℹ️ **Picture must be a PNG file.** It should have square dimensions. 

#### Set blog post's author(s)

In the post's front matter, add a property `authors`, that could be a string (`user_id`), an array of strings or nothing:

```
# No author
authors:

# Single author
authors: firstname_lastname

# Multiple authors
authors:
  - firstname_lastname
  - firstname2_lastname2
```

## License

Copyright (c) 2020 GIP PIX.

This program is free software: you can redistribute it and/or modify it under the terms of the GNU Affero General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License along with this program. If not, see [gnu.org/licenses](https://www.gnu.org/licenses/).
