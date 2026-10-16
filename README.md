# Git Colab

A lightweight Git analytics toolkit that estimates **project ownership**, **engagement**, and **bus factor** for multi-project repositories.

The scripts scan a repository, discover `.csproj` projects, and calculate contributor statistics per project based on:

- Commit activity
- Code churn (lines added/removed)
- Engagement score
- Bus factor

The goal is not to answer *"who wrote the most code?"* but rather:

> Who understands this project well enough to maintain it?

---

## Architecture

The Git Colab toolchain consists of two layers:

- `git-colab-stats-*` extracts contributor ownership metrics from repositories.
- `git-colab-summary` aggregates multiple project reports into a portfolio-level view.

This separation allows data extraction and reporting to evolve independently.

### git-colab-stats-*

These are meant to extract the git information for specific types of project.
Their naming schema is `git-colab-stats-` them followed by the project type.

### git-colab-summary

This is the script that can summarize the output from one or more git-colab-stats-*.

---

## Installation

```bash
git clone https://github.com/stark-antonio-almeida/project-colab-status.git &&
cd git-colab &&
./install.sh
```

Verify the installation:

```bash
git-colab-stats --help
```

Uninstall:

```bash
./install.sh --uninstall
```

---

## Example Output

### Pretty

```text
Project: src/Integration/BackInBlackBus

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

### TSV

```text
Project Contributor CommitPct ChurnPct EngagementPct
...
```

### Markdown

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

---

## Requirements

- Git
- Bash
- A Git repository

---

## Usage

Analyze the current repository:

```bash
git-colab-stats-* --format tsv | git-colab-summary
```

### Stats options

```bash
git-colab-stats
```

Enable debug output:

```bash
git-colab-stats --debug
```

Generate Markdown:

```bash
git-colab-stats --format md
```

Generate TSV:

```bash
git-colab-stats --format tsv
```

Generate Pretty output:

```bash
git-colab-stats --format pretty
```

Save a report:

```bash
git-colab-stats --format md > report.md
```

### Aggregating Multiple Projects

The individual `git-colab-stats-*` scripts are designed to produce machine-readable output that can be combined into higher-level reports.

Example workflow:

```bash
git-colab-stats-app --format tsv \
&& git-colab-stats-lib --format tsv \
&& git-colab-stats-api --format tsv \
| git-colab-summary
```

This allows repository-level metrics to be aggregated into a single summary report highlighting:

- Most active contributors
- Knowledge concentration
- Ownership distribution
- Bus-factor risk across projects

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

## Output Formats

| Format | Purpose |
| ---------- | ---------- |
| pretty | Human-readable terminal report |
| md | Documentation, GitHub, Azure DevOps Wiki |
| tsv | Excel, Power BI, data analysis |

---

## Philosophy

This tool intentionally favors **ownership and engagement** over raw line counts.

More code does not necessarily mean more value.

The objective is to help answer questions such as:

- Who should review changes to this project?
- Who can help onboard a new engineer?
- Where is knowledge concentrated?
- Which projects have a bus-factor risk?

---

## Future Work

Planned improvements include:

- Support for `.mailmap` to consolidate contributor identities.
- Additional ownership heuristics.
- Repository-level trend analysis.
- Historical comparison reports.
- Additional visualization and charting options.
- Native Markdown report aggregation.
- Mermaid-based ownership visualizations.

---

## Disclaimer

Git history is an approximation of ownership.

Generated files, large refactorings, squashed commits, rebases, and inconsistent author identities can all influence the results.

The metrics produced by this tool should be used as indicators rather than absolute measures of expertise or ownership.
