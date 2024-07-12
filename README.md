# Project Migrator

### Requirements

- You must have [Deno](https://deno.land/) installed. If you use Homebrew, run `brew install deno`.


### Considerations

- These scripts, `migrate.ts` and `source.ts`, are provided strictly as-is. LaunchDarkly Support cannot help run this.

### Known issues

- Importing LD API TypeScript types causes an import error, so they are commented out
  in various spots.
- Types in general are very loose, which Deno is not happy about. The scripts run as
  JavaScript overall instead of validating the TypeScript first.
- Due to the current API configuration, you cannot have more than 20 environments in a single project.
- Due to considerations around many API requests at once, monitor 400 errors for flag configurations that may not be up to date.

## Things you Should Consider when migrating flags?

- What can you scope down? Do all the flags need to moved over or can we use this as a way to clean up the environment?
- Do all my environments need to go? or maybe just a few?
- Am I able to stop edits in the destination project?  This script does not keep them in sync, so if changes need to be made they should be prior
- Who is going to run it and how? The calls can take a while, with rate limits, so should I run it on an EC2 or the like?
- If I have thousands or even hundreds of updates: what is critical, how will I verify the changes are correct?

## Instructions for use

1. Sourcing data

First, export your source data. The `source.ts` script writes the data to a newly created
`source/project/<source-project-key>` directory.

Here's how to export your source data:

```
deno run --allow-env --allow-read --allow-net --allow-write source.ts -p <SOURCE PROJECT KEY> -k <SOURCE LD API KEY>

```

2. Migrating data

Then, migrate the source data to the destination project. The `migrate.ts` script reads the source data out of the previously created `source/project/<source-project-key>` directory. Then it uses the
`DESTINATION PROJECT` as the project key, and updates the destination project using a series of `POST`s and `PATCH`s.

Here's how to migrate the source data to your destination project:

```
deno run --allow-env --allow-read --allow-net --allow-write migrate.ts -p <SOURCE PROJECT KEY> -k <DESTINATION LD API KEY> -d <DESTINATION PROJECT KEY>

```

**Important note** The script currently doesn't support merging two already existing projects - make sure the destination project doesn't exist before executing the `migrate.ts` script. If you have already created the destination project manually, delete the project before proceeding. 

### Resources migrated by the script
* Environments
* Flags
  * Flag variations
  * Flag prerequisites
  * Flag individual targets
  * Flag attribute-based targeting rules
* Standard User Segments (no Big Segments)

### Pointing to a different instance

Pass in the `-u` argument with the domain of the other instance. By default, these scripts apply to your projects on `app.launchdarkly.com`.
