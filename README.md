# coli-ana

[![Test](https://github.com/gbv/coli-ana/actions/workflows/test.yml/badge.svg)](https://github.com/gbv/coli-ana/actions/workflows/test.yml)

> API to analyze DDC numbers

This repository contains an implementation of an API to analyze synthesized DDC numbers. The algorithm to split DDC numbers is not included. See <https://coli-conc.gbv.de/coli-ana/> for more information.

## Table of Contents <!-- omit in toc -->
- [Install](#install)
- [Usage](#usage)
  - [vc_day_srv Backend](#vc_day_srv-backend)
  - [Database Backend](#database-backend)
  - [Data dumps and statistics](#data-dumps-and-statistics)
  - [Database migrations](#database-migrations)
  - [Development](#development)
  - [Production](#production)
  - [K10plus enrichment](#k10plus-enrichment)
- [Configuration](#configuration)
- [API](#api)
  - [GET /](#get-)
  - [GET /analyze?notation=notations](#get-analyzenotationnotations)
  - [GET /analyze?member=members](#get-analyzemembermembers)
- [Maintainers](#maintainers)
- [Publish](#publish)
- [Contribute](#contribute)
- [License](#license)

## Install

~~~bash
git clone https://github.com/gbv/coli-ana.git
cd coli-ana
npm install
~~~

## Usage

The server provides a HTTP API at port 11033 by default. (Fun fact: 11033 = Octal 025431 (025.431=Dewey Decimal Classification))

It provides two possible backends to retrieve analysis results:

### vc_day_srv Backend

vc_day_srv is a server component developed as part of project colibri (currently closed source) that can analyse DDC numbers and return the analyses to a client. It can be configured via `.env`:

```env
BACKEND_HOST=my-server.com
BACKEND_PORT=7070
```

Note that requests are performed via nc/netcat, so no protocol should be given for host. If there is no backend configured or the backend does not return a result, the database will be queried as a fallback.

### Database Backend

coli-ana can use a PostgreSQL database. Instead of doing a live analysis, you can import pre-analyzed numbers into the database which will then be retrieved in case the vc_day_srv backend fails.

For instance create a database `coli-ana`:

```sql
CREATE DATABASE "coli-ana";
CREATE USER "username" PASSWORD 'password';
GRANT ALL ON DATABASE "coli-ana" TO "coli-ana";
```

Provide the connection string inside a `.env` file, e.g.

```env
DATABASE_URL="postgresql://username:password@localhost:5432/coli-ana"
```

We also need to create the necessary tables. For this, we are using [Prisma](https://www.prisma.io):

```bash
npx prisma db push --preview-feature
```

Now that the database and tables are prepared, you can import coli-ana results (in `slim` format), e.g.:

```bash
node ./bin/convert --import ~/path/to/ou_liu_t_de-slim-21-02-15-1121
```

Add `--reset` to delete old records from the database. At the moment, there are some basic error checks that exclude some analyses from being imported. To import those anyway, add the `--ignore-errors` flag.

### Data dumps and statistics

The script `./bin/stats.sh` creates a database dump (unless the file already exists, so it is not updated) and calculates some statistics into `public/stats.json`.

### Database migrations

Sometimes, a database migration might be necessary. It will be mentioned in the release notes. In that case, please run the following command to migrate your database:

```bash
npx prisma migrate dev --preview-feature
```

If the migration script shows "All data will be lost", you will need to reimport the data after migration.

### Development
```bash
npm run dev
```

### Production
```bash
# Bundle Vue files
npm run build
# Run server in production
npm run serve
```

### K10plus enrichment

Directory `k10plus` contains scripts to enrich K10plus library catalog with analyzed DDC notations. See [README.md](k10plus/README.md) in this directory.

The script `bin/labels.sh` created one text file for each DDC number in directory `labels` for full text indexing.

## Configuration

You can adjust a few configuration options in `.env`. Here are the available options and default values:

```bash
# Host and port for backend service vc_day_srv
BACKEND_HOST=
BACKEND_PORT=
# URL to access PostgreSQL database
DATABASE_URL=
# URL to Cocoda instance
COCODA=https://coli-conc.gbv.de/cocoda/app/
# Port for Express server
PORT=11033
# Base for URL (e.g. when not running under root of domain)
BASE=/
```

## API

### GET /

Shows a landing page with general information and a list of examples.

### GET /analyze?notation=notations

Analyzes one or more DDC numbers in parameter `notation` (separated by `|`) and returns a [JSKOS concept](https://gbv.github.io/jskos/jskos.html#concept) by default. Parameter `format` can be used to chose another format:

* `picajson` returns a [PICA/JSON](https://format.gbv.de/pica/json) record
* `pp` returns a [PICA Plain](https://format.gbv.de/pica/plain) record; **note:** only one record can be requested at the time.

Optional parameter `complete` with a truthy value enables filtering for completely analyzed numbers.

The response header `coli-ana-backend` will contain the backend used for the analysis (either "vc_day_srv" or "database"). If multiple numbers are analyzed, the backend for the last analyzed number in the list is returned.

### GET /analyze?member=members

Returns JSKOS concepts which contain one or more of the specified `member` URIs or notations (seperated by `|`) in their `memberList`.

Optional parameter `complete` with a truthy value enables filtering for completely analyzed numbers.

## Maintainers
- [@stefandesu](https://github.com/stefandesu)
- [@nichtich](https://github.com/nichtich)

## Publish
Please work on the `dev` branch or separate feature branch during development.

When a new release is ready (i.e. the features are finished, merged into `dev`, and all tests succeed), run the included release script (replace "patch" with "minor" or "major" if necessary):

```bash
npm run release:patch
```

This will:
- Run tests and build to make sure everything works
- Switch to `dev`
- Make sure `dev` is up-to-date
- Run `npm version patch` (or "minor"/"major")
- Push changes to `dev`
- Switch to `main`
- Merge changes from `dev`
- Push `main` with tags
- Switch back to `dev`

After running this, GitHub Actions will automatically create a new GitHub Release draft. Please edit and publish the release manually.

## Contribute
PRs accepted.

Small note: If editing the README, please conform to the [standard-readme](https://github.com/RichardLitt/standard-readme) specification.

## License
MIT Copyright (c) 2021 Verbundzentrale des GBV (VZG)
