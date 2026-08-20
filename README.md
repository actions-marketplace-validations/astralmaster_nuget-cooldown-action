# NuGet Cooldown Action

A GitHub Action that fails your build when any NuGet package version is younger than N days — a
**dependency cooldown** (a.k.a. *minimum release age*) for .NET, run in CI.

Malicious package versions are usually detected and pulled within days of publication, so refusing
brand-new versions closes most of the supply-chain attack window. This action runs the
[**NuGetCooldown**](https://github.com/astralmaster/NuGetCooldown) tool against your restored
dependency graph — direct **and** transitive — and fails the job on a violation.

> Already run `dotnet build` in CI? You can skip this action entirely and add the
> [`NuGetCooldown.MSBuild`](https://www.nuget.org/packages/NuGetCooldown.MSBuild) package instead,
> which enforces the cooldown as part of every build. This action is for adding an explicit check
> step to a workflow.

## Usage

The project must be restored first (this action reads the restore output), so run it after your
restore or build step:

```yaml
- uses: actions/checkout@v4

- uses: actions/setup-dotnet@v4
  with:
    dotnet-version: '8.0.x'

- run: dotnet restore

- uses: astralmaster/nuget-cooldown-action@v1
  with:
    days: 7
```

That's it — the step fails if any resolved package is younger than 7 days.

### Examples

Sub-day window (like pnpm's 24h default), and allow-list your own packages:

```yaml
- uses: astralmaster/nuget-cooldown-action@v1
  with:
    hours: 24
    allow: 'MyCompany.*'
```

Report but don't fail the build (e.g. while you roll it out):

```yaml
- uses: astralmaster/nuget-cooldown-action@v1
  with:
    days: 7
    warn-only: true
```

Check only a specific solution, strict mode:

```yaml
- uses: astralmaster/nuget-cooldown-action@v1
  with:
    path: src/MyApp.sln
    days: 14
    on-feed-error: error
```

## Inputs

All inputs are optional.

| Input | Default | Description |
|-------|---------|-------------|
| `path` | `.` | Directory, solution, project, `project.assets.json`, or `packages.lock.json` to check |
| `days` | `7` | Cooldown window in days |
| `hours` | – | Extra hours added to the window; use alone for sub-day windows |
| `scope` | `all` | `all` (direct + transitive) or `direct` |
| `allow` | – | Allow-list patterns, `;`-separated (`MyCompany.*;Serilog@4.3.0`) |
| `sources` | nuget.org | NuGet V3 service index URLs, `;`-separated |
| `warn-only` | `false` | Report findings but always succeed |
| `on-unknown` | `warn` | `warn` \| `error` \| `ignore` for undeterminable publish dates |
| `on-unlisted` | `warn` | `warn` \| `error` \| `ignore` for unlisted versions |
| `on-feed-error` | `warn` | `warn` \| `error` \| `ignore` for feed failures |
| `on-not-restored` | `warn` | `warn` \| `error` \| `ignore` for unrestored projects |
| `config` | auto | Path to a `nuget-cooldown.json` config file |
| `format` | `text` | `text` or `json` |
| `timeout` | `30` | Per-request feed timeout, in seconds |
| `max-parallel` | `8` | Max concurrent feed lookups (1–32) |
| `tool-version` | latest | Version of the NuGetCooldown tool to install |

## Outputs

| Output | Description |
|--------|-------------|
| `exit-code` | `0` = passed (or `warn-only`), `1` = violations or policy errors |

## Requirements

- The .NET SDK must be available on the runner (e.g. via
  [`actions/setup-dotnet`](https://github.com/actions/setup-dotnet)); the action installs the
  NuGetCooldown dotnet tool.
- The project should be restored before this step, or have a committed `packages.lock.json`.

## Handling a security fix that's still in cooldown

When a CVE fix is newer than your window, allow-list that one reviewed version so you can take it
immediately while the cooldown keeps protecting everything else:

```yaml
- uses: astralmaster/nuget-cooldown-action@v1
  with:
    allow: 'Contoso.Http@8.4.2'
```

## License

[MIT](LICENSE) © 2026 George Andguladze

See the main project for full docs and diagnostics:
[github.com/astralmaster/NuGetCooldown](https://github.com/astralmaster/NuGetCooldown)
