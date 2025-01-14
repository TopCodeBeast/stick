# stick

![](https://media.tenor.com/eK1dyB3TOLsAAAAC/anime-stick.gif)

KodaDot's [Squid](https://docs.subsquid.io) based data processor for [KodaDot](https://kodadot.xyz) NFT Marketplace.

## Prerequisites

* node 16.x
* docker
* npm -- note that `yarn` package manager is not supported

## Quickly running the sample

Example commands below use [sqd](https://docs.subsquid.io/squid-cli/).
Please [install](https://docs.subsquid.io/squid-cli/installation/) it before proceeding.

```bash
# 1. Update Squid SDK and install dependencies
npm run update
npm ci

# 2. Start target Postgres database and detach
sqd up

# 3. Build the project and start the processor
sqd process

# 4. The command above will block the terminal
#    being busy with fetching the chain data, 
#    transforming and storing it in the target database.
#
#    To start the graphql server open a separate terminal
#    and run
sqd serve
```

## Project structure

* `src/generated` - model/server definitions created by `codegen`. Do not alter the contents of this directory manually.
* `src/server-extension` - module with custom `type-graphql` based resolvers
* `src/types` - data type definitions for chain events and extrinsics created by `typegen`.
* `src/mappings` - mapping module.
* `lib` - compiled js files. The structure of this directory must reflect `src`.
* `.env` - hydra tools are heavily driven by environment variables defined here or supplied by a shell.

## Dev flow

### 1. Define database schema

Start development by defining the schema of the target database via `schema.graphql`.
Schema definition consists of regular graphql type declarations annotated with custom directives.
Full description of `schema.graphql` dialect is available [here](https://docs.subsquid.io/schema-file).

### 2. Generate TypeORM classes

Mapping developers use [TypeORM](https://typeorm.io) entities
to interact with the target database during data processing. All necessary entity classes are
generated by the squid framework from `schema.graphql`. This is done by running `npx squid-typeorm-codegen`
or (equivalently) `sqd codegen` command.

### 3. Generate database migration

All database changes are applied through migration files located at `db/migrations`.
`squid-typeorm-migration(1)` tool provides several commands to drive the process.
It is all [TypeORM](https://typeorm.io/#/migrations) under the hood.

```bash
# Connect to database, analyze its state and generate migration to match the target schema.
# The target schema is derived from entity classes generated earlier.
# Don't forget to compile your entity classes beforehand!
npx squid-typeorm-migration generate

# Create template file for custom database changes
npx squid-typeorm-migration create

# Apply database migrations from `db/migrations`
npx squid-typeorm-migration apply

# Revert the last performed migration
npx squid-typeorm-migration revert         
```
Available `sqd` shortcuts:
```bash
# Build the project, remove any old migrations, then run `npx squid-typeorm-migration generate`
sqd migration:generate

# Run npx squid-typeorm-migration apply
sqd migration:apply
```

### Dev hacks 

1. fast generate event handlers 

```
pbpaste | cut -d '=' -f 1 | tr -d ' '  | xargs -I_ echo "processor.addEventHandler(Event._, dummy);"
```

2. enable debug logs (in .env)

```
SQD_DEBUG=squid:log
```

3. generate metagetters from getters 

```
pbpaste | grep 'export'  | xargs -I_ echo "_  return proc.  }"
```