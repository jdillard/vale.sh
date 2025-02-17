---
title: Creating a plugin
lead: |
  Learn how to integrate Vale with other tools and services.
draft: false
images: []
menu:
  docs:
    parent: "integrations"
weight: 115
toc: true
---

Vale can provide JSON output that extensions can use.

Your extension should call the Vale CLI, which the user of your plugin
needs to have installed, setting the output to `JSON` mode, along with any other
arguments.

```shell
vale --output JSON {path/file}
```

Look at how the VS Code extension uses the [`buildCommand`](https://github.com/errata-ai/vale-vscode/blob/78cd80ff5bcc043f51aa22126997c4e86e5b13fd/src/features/vsUtils.ts#L290) method to create [a reusable command the extension can call](https://github.com/errata-ai/vale-vscode/blob/78cd80ff5bcc043f51aa22126997c4e86e5b13fd/src/features/vsProvider.ts#L97) with a variety of parameters.

## JSON output of checks

Whether using Vale CLI or Server, the JSON output of the checks configured to run is similar, consisting of a file name with an array of check objects.

```json
{
  "index.md": [
    {
      "Action": {
        "Name": "",
        "Params": null
      },
      "Check": "write-good.Passive",
      "Description": "",
      "Line": 6,
      "Link": "",
      "Message": "'was created' may be passive voice. Use active voice if you can.",
      "Severity": "warning",
      "Span": [
        59,
        69
      ],
      "Match": "was created"
    },
  ]
}
```

Each object contains the following information:

-   `Action`: An action or change to the text that Vale server can take with a rule, containing a `Name` for the action and `Params` passed to the action.
-   `Check`: The rule set and rule triggered.
-   `Description`: A more detailed explanation for a rule. You can use it with [custom output format](/manual/output) or an editor integration's UI.
-   `Line`: The line that contains the error.
-   `Link`: Link to explanation of style guide rule
-   `Message`: Help text output by the rule
-   `Severity`: The error level.
-   `Span`: The start and finish characters on the line.
-   `Match`: The text matched.

A plugin should loop through these checks, and parse the values, to output them to an appropriate part of the IDE or editor interface.

{{< alert context="info" icon="👉">}}
**Heads up**!

A plugin needs to implement actions itself, Vale CLI only provides the action passed from the style and the matched tokens from text checked. You can find [suggestions on how to implement the action in the styles guide](/docs/topics/styles#actions).
{{< /alert >}}

For example, the VS Code extension uses the shared [`handleJSON`](https://github.com/errata-ai/vale-vscode/blob/78cd80ff5bcc043f51aa22126997c4e86e5b13fd/src/features/vsProvider.ts#L110) method to take each check object and convert it to a [VS Code diagnostic](https://code.visualstudio.com/api/references/vscode-api#Diagnostic) that appears in the bottom status bar of the editor window. If a user uses Vale server, you can also implement some form of "fix" solution propagated from the rule to the JSON, to the IDE or editor with the `Action` property. For example, in the [Microsoft style ellipses check](https://github.com/errata-ai/Microsoft/blob/ec219cff4ef10c558945f25dcb47eb1fc6ebca24/Microsoft/Ellipses.yml), the action is to offer to remove the ellipses.

## General tips

### Configuration options

If the editor or IDE allows for user-configured settings for a plugin, then some of the following are good settings to include:

-   When to check a document with Vale, on every change, or on save?
-   Custom paths for the Vale CLI binary, configuration, or styles paths
