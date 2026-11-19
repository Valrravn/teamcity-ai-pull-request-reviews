# teamcity-ai-pull-request-reviews


# AI pull request reviews in TeamCity

A TeamCity project that reviews GitHub pull requests with [Codex](https://github.com/openai/codex) and posts the findings back to GitHub as a regular review, with inline comments on the changed lines. Reviews are advisory: the bot never approves or blocks a pull request.

How it works, step by step, is described in the TeamCity documentation: [AI in Builds](https://www.jetbrains.com/help/teamcity/ai-in-builds.html).

## Requirements

* TeamCity On-Premises with the [AI Agents](https://plugins.jetbrains.com/plugin/34639-ai-agents) plugin installed.
* A Linux build agent with the Codex CLI, `git`, `curl`, and `tar`.

The sample uses Kotlin DSL context parameters, and projects that use them cannot be edited in the TeamCity UI. That is why everything that holds credentials lives in the **parent** project of the sample, which you can edit as usual. Create the following there:

* A VCS root for the repository to review. Add credentials if the repository is private.
* An **AI Provider** connection that can serve OpenAI models: OpenAI, JetBrains Air, or LiteLLM.
* A **GitHub App** connection with **Enable build-scoped tokens** selected. Set the App's **Pull requests** permission to "Read and write".

## Use this sample

1. Fork this repository. Do not create a project from this repository directly: your project would keep syncing settings from it, and you could not change them.
2. In TeamCity, create a subproject of the project that holds the VCS root and connections, from the settings in your fork. See [Create a Project From Project Settings](https://www.jetbrains.com/help/teamcity/storing-project-settings-in-version-control.html#Create+a+Project+From+Project+Settings).
3. When TeamCity asks for context parameters, enter:

   | Parameter | Value |
   |-----------|-------|
   | `vcsRootId` | The external ID of the VCS root. |
   | `githubRepository` | The repository to review, as `owner/name`. Use the exact capitalization GitHub shows. |
   | `aiProviderConnectionId` | The ID of your AI Provider connection. |
   | `gitHubAppConnectionId` | The ID of your GitHub App connection. |
   | `reviewBotUserId` | The numeric user ID of the App's bot account. This is **not** the App ID. |

4. Open a pull request to `main` in the reviewed repository.

To get the bot's user ID, read it from a review the bot has already posted:

```shell
gh api repos/<owner>/<repository>/pulls/<number>/reviews --jq '.[].user | {login, id}'
```

## Customize

The model, reasoning effort, and tool versions are build configuration parameters (`review.model`, `review.effort`, `gh.version`, `jq.version`). Projects that use Kotlin DSL context parameters cannot be edited in the TeamCity UI, so change these values in `.teamcity/settings.kts` in your fork.
