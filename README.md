# Git Colab

[Installation](#installation) · [Quick Start](#quick-start) · [Usage](#usage) · [Output](#example-output) · [Tips](#tips) · [Boundaries and Ownership](#ownership-boundaries) · [Bus Factor](#bus-factor)

---

A lightweight Git analytics toolkit that estimates **project ownership**, **engagement**, and **bus factor** for multi-project repositories.

The scripts scan a repository and calculate contributor statistics per project based on:

- Commit activity
- Code churn (lines added/removed)
- Engagement score
- Bus factor

This tool intentionally favors **ownership and engagement** over raw line counts.

More code does not necessarily mean more value.

The objective is to help answer questions such as:

- Who should review changes to this project?
- Who can help onboard a new engineer?
- Where is knowledge concentrated?
- Which projects have a bus-factor risk?

The goal is help you answer:

- `Who understands this project well enough to maintain it?`
- `How concentrated is the knowledge of this project?`
- `What happens if our most knowledgeable contributor disappears tomorrow?`

---

## Installation

```bash
git clone https://github.com/stark-antonio-almeida/project-colab-status.git
cd project-colab-status
./install.sh
```

Verify the installation:

```bash
git-colab-stats-path --help
```

Uninstall:

```bash
./install.sh --uninstall
```

### Requirements

- Git
- Bash
- A Git repository

---

## Quick Start

```bash
git-colab-stats-path my-project-path README.md | git-colab-summary --pretty
```

## Usage

The Git Colab toolchain consists of two layers:

- `git-colab-stats-*` extracts contributor ownership metrics from repositories.
- `git-colab-summary` aggregates multiple project reports into a portfolio-level view.

This separation allows data extraction and reporting to evolve independently.

### git-colab-stats-path

This is the base extraction script and it is the base for all others.
It calls git log to extract the metrics for each path and committer.

### git-colab-stats-*

These are meant to extract the git information for specific types of project.
Their naming schema is `git-colab-stats-` them followed by the project type.

### git-colab-summary

This is the script that can summarize the output from one or more git-colab-stats-*.

### Analyze the current repository

```bash
git-colab-stats-* --format tsv | git-colab-summary
```

### Stats options

```bash
git-colab-stats-path
```

Enable debug output:

```bash
git-colab-stats-path --debug
```

Generate Markdown:

```bash
git-colab-stats-path --format md
```

Generate TSV:

```bash
git-colab-stats-path --format tsv
```

Generate Pretty output:

```bash
git-colab-stats-path --format pretty
```

Save a report:

```bash
git-colab-stats-path --format md > report.md
```

### Example Output

**Output Formats**

| Format | Purpose |
| ---------- | ---------- |
| pretty | Human-readable terminal report |
| md | Documentation, GitHub, Azure DevOps Wiki |
| tsv | Excel, Power BI, data analysis |

#### Pretty

```text
Boundary: src/Integration/BackInBlackBus

Most Active Contributor: Freddie Mercury (42.0%)
Bus Factor: 2
Contributors: 6

Contributor                       Commit%     Churn%    Engage%
-----------                       -------     ------    -------
Freddie Mercury                      30.0       54.0       42.0
Dave Grohl                           24.0       20.0       22.4
James Hetfield                       18.0       11.0       15.2
Slash                                12.0        8.0       10.4
Bruce Dickinson                      10.0        5.0        7.0
Chester Bennington                    6.0        2.0        3.0
```

#### TSV

```text
Boundary Contributor CommitPct ChurnPct EngagementPct
...
```

#### Markdown

```markdown
## src/Speed94

- Most Active Contributor: Contributor A (66.2%)
- Bus Factor: 1
- Contributors: 4

| Contributor | Commit % | Churn % | Engagement % |
|------------- |----------|----------|--------------|
| Keanu Reeves | 42.8 | 89.6 | 66.2 |
| Sandra Bullock | 42.8 | 9.7 | 26.2 |
| ... | ... | ... | ... |
```

### Tips

#### Render the report

Add some [glow](https://github.com/charmbracelet/glow) to the mix.
Use a md renderer to instantly view the report.

```bash
git-colab-stats-dotnet-csharp | git-colab-summary --format md --mode all | glow -p
```

#### Aggregating Multiple Boundaries

The individual `git-colab-stats-*` scripts are designed to produce machine-readable output that can be combined into higher-level reports.

Example workflow:

```bash
git-colab-stats-go --format tsv \
&& git-colab-stats-rust --format tsv \
&& git-colab-stats-path README.md --format tsv \
| git-colab-summary
```

This allows repository-level metrics to be aggregated into a single summary report highlighting:

- Most active contributors
- Knowledge concentration
- Ownership distribution
- Bus-factor risk across projects

#### Use the git .mailmap

List all users with

```bash
git shortlog -sne --all
```

and check for duplicates.
Add or update the .mailmap file to remove duplicate names.

**Example**

If the command above returns:

```
Link <link@games.com>
zelda-link <link@games.com>
sonic <sonic@fast.speed>
```

Add `Link <link@games.com> zelda-link <link@games.com>` to the .mailmap.
This way only `Link` will be gathered when metric collection happens (git-colab-stats-*).

---

## Ownership Boundaries

Git Colab analyzes ownership boundaries.

An ownership boundary can be:

- A .NET project (`.csproj`)
- A Rust crate (`Cargo.toml`)
- A Go module (`go.mod`)
- A directory provided explicitly to `git-colab-stats-path`

All extractors produce the same output format, allowing results to be summarized consistently.

---

## Bus Factor

The script estimates bus factor using contributor engagement.

Contributors are sorted by engagement score and accumulated until they exceed a threshold.

Example:

```text
Contributor      Engage%
-----------      -------
Neo                51.0
Trinity            29.0
Morpheus           20.0
```

The first contributor already exceeds 50% ownership:

```text
Bus Factor = 1
```

A low bus factor indicates knowledge concentration and potential delivery risk.

---

## Disclaimer

Git history is an approximation of ownership.

Generated files, large refactorings, squashed commits, rebases, and inconsistent author identities can all influence the results.

The metrics produced by this tool should be used as indicators rather than absolute measures of expertise or ownership.
