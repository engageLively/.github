# PowerShell

Follow the PowerShell community's conventions, as described in the [PowerShell Practice and Style guide](https://poshcode.gitbook.io/powershell-practice-and-style/) and enforced by PSScriptAnalyzer. The [shared principles](code.md) say what to document; this page says where.

Scripts that run on Windows machines without PowerShell 7 installed must run under **Windows PowerShell 5.1**. In those scripts, don't use PowerShell 7-only syntax (`??`, `?.`, ternaries, `ForEach-Object -Parallel`).

## Naming

| Thing | Convention | Example |
| --- | --- | --- |
| Functions, cmdlets | `Verb-Noun`, approved verb (`Get-Verb`), singular noun | `Get-ServiceHealth`, `Install-MetricsExporter` |
| Parameters | `PascalCase` | `-ConfigPath`, `-WhatIf` |
| Local variables | `camelCase` | `$nextRun` |
| Script files | `Verb-Noun.ps1` for commands; module files `PascalCase.psm1` | `Move-ExportToQueue.ps1`, `ServiceHealth.psm1` |

The hyphens are there so commands can be discovered: given the approved verbs and a noun, a reader can guess a command's name, and `Get-Command -Noun ServiceHealth` finds all of them. Write aliases only for interactive use, never in scripts.

## Comment-based help

Use comment-based help for operator-facing scripts, public functions, and operations with meaningful side effects. Put it at the top of a script or as the first thing in a function body so `Get-Help` and editors can read it. Use `.SYNOPSIS` for the summary, `.OUTPUTS` for results, and a `Side effects:` paragraph in `.NOTES`. An obvious private helper can use a short comment where it adds context.

```powershell
function Restart-ExportCopyTask {
    <#
    .SYNOPSIS
    Restarts the scheduled task that copies instrument exports to the upload queue.

    .DESCRIPTION
    Called by the installer after it changes the copy task's settings.
    Not a recovery action: diagnose a failing copy rather than bouncing it.

    .PARAMETER TaskName
    Name of the scheduled copy task.

    .OUTPUTS
    System.Boolean. $true if the task is running afterward.

    .NOTES
    Side effects: stops and starts the scheduled task, interrupting any copy
    in progress. Supports -WhatIf and -Confirm.
    #>
    [CmdletBinding(SupportsShouldProcess)]
    param(
        [Parameter(Mandatory)]
        [string] $TaskName
    )
    ...
}
```

Anything with side effects declares `SupportsShouldProcess` and wraps each change in `if ($PSCmdlet.ShouldProcess(...))`. Then `-WhatIf` gives a dry run for free, and the side effects are visible in the code itself.

Indent with 4 spaces. Put the opening brace at the end of the line (One True Brace Style, which is the community default).

## Tools

- **PSScriptAnalyzer** with the default rules, plus `PSUseApprovedVerbs` and `PSProvideCommentHelp`. Each suppression gets a `SuppressMessageAttribute` with a justification.
- **Pester 5** for tests. Use discoverable `*.Tests.ps1` files grouped by script, module, or workflow. Use `Describe` to group related behaviors and `It` for individual cases. Use `Mock` for external commands. Remember that variables set in an `It` block are visible to mocks through dynamic scope; say so in a comment where a test depends on it.
- CI parses every script that must run under Windows PowerShell 5.1 with Windows PowerShell 5.1 itself, not just `pwsh`.
