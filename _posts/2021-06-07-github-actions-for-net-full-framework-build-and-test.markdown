---
layout: post
title:  "GitHub Actions for .Net Full Framework: Build and Test"
date:   2021-06-07 08:00:00 -0300
tags: devops dotnet github ci
---
Originally posted here: [dev.to](https://dev.to/felipetofoli/github-actions-for-net-full-framework-build-and-test-299h) account.

![GitHub Actions for .Net Full Framework: Build and Test - Banner]({{ site.baseurl }}/assets/images/github-actions-for-net-full-framework/github-actions-for-net-full-framework-banner.webp "GitHub Actions for .Net Full Framework: Build and Test - Banner")

## Can we create GitHub Actions to Build and Test .Net Full Framework projects?
:white_check_mark: YES, we can! And I will show you how I did it.


## But first, let me give you some context :thought_balloon:

I recently had to deal with a legacy code base of a .Net Full Framework project.

The source code was versioned on GitHub and there was a pull request and code review flow for integrating source code into the main branch. However, there were no checks to verify the code that was being integrated, which would allow source code that did not compile, or that broke unit tests, to be included in our main branch without any alerts.

The absence of quick feedback on the health of the integrated source code created some sense of insecurity in the developers during the approval of the pull requests, since problems inserted through the addition of new code could take much longer than desired to be identified.

In order to improve our workflow, I thought about creating a CI workflow with [GitHub Actions](https://docs.github.com/en/actions) to speed up the flow of feedback during our source code integrations.

When checking the [documentation](https://docs.github.com/en/actions/guides/building-and-testing-net) of GitHub Actions, I realized that there was no template for creating a workflow for .Net Full Framework projects. And then I asked the same question at the beginning of this text:
> &nbsp;
> Can we create GitHub Actions to Build and Test .Net Full Framework projects?
> &nbsp;

Throughout my research journey, I discovered some resources, both official and from the community:
- [setup-msbuild](https://github.com/marketplace/actions/setup-msbuild);
- [setup-nuget](https://github.com/marketplace/actions/setup-nuget-exe-for-use-with-actions);
- [Setup-VSTest](https://github.com/marketplace/actions/setup-vstest-console-exe).

The use of the above actions allowed me to create the CI pipeline for my .Net Full Framework project. I configured the CI pipeline to be triggered by each new `push` or `pull request` in the main branch, as shown in the `ci.yml` file below:

{% gist f94ec0edfe074220c5448d9cc9671b09 ci.yml %}

<br/>

## Understanding the created action

Below, step-by-step explanation of the `ci.yml` file:

1. Define the name of the GitHub Action.
    ```yml
     name: Build and Tests
    ```

2. Set that the actions should be triggered by `push` and creation of `pull requests` for the branch `main` (default branch of this project).
    ```yml
     on:  
       push:
         branches: [ main ]
       pull_request:
         branches: [ main ]
    ```

3. The build and test will be performed on a Windows operating system, because we are dealing with .Net Full Framework (which does not support other operating systems).
    ```yml
     jobs:  
       build:    
         runs-on: windows-latest
    ```

4. Define the section that will group all the steps performed, detailed in the next items.
    ```yml
         steps:      
    ```

5. Checkout the code.
    ```yml
           - uses: actions/checkout@v2
    ```

6. MSBuild setup, for later use.
    ```yml
           - name: Setup MSBuild
             uses: microsoft/setup-msbuild@v1
    ```

7. Nuget setup, for later use.
    ```yml
           - name: Setup NuGet
             uses: NuGet/setup-nuget@v1.0.5
    ```

8. VSTest setup, for later use.
    ```yml
           - name: Setup VSTest
             uses: darenm/Setup-VSTest@v1
    ```

9. Navigate to the GitHub workspace to start the restore, build and test of the application.
    ```yml
           - name: Navigate to Workspace
             run: cd $GITHUB_WORKSPACE
    ```

10. Use `nuget` to restore packages used by the application.
    ```yml
           - name: Restore Packages
             run: nuget restore Sandbox.sln
    ```

11. Build the solution with `msbuild.exe`, release mode.
    ```yml
           - name: Build Solution
             run: |
               msbuild.exe Sandbox.sln /p:platform="Any CPU" /p:configuration="Release"
    ```

12. Run the tests of the `Sandbox.Tests.dll`, using `vstest.console.exe`.
    ```yml
           - name: Run Tests
             run: vstest.console.exe .\tests\Sandbox.Tests\bin\Release\Sandbox.Tests.dll
    ```

## Results
After we implanted our CI pipeline, just open a `pull request` too see the result of our implementation.

### Opened PR, actions pending execution
When opening a `pull request`, we can now see that some checks are pending execution.

![Actions pending execution]({{ site.baseurl }}/assets/images/github-actions-for-net-full-framework/actions-pending-execution.png "Actions pending execution")


### Opened PR, actions performed
After the execution of our checks, the result is already shown on the PR page. Now that the checks are ok, the `Merge pull request` button is highlighted.

![Actions performed, result ok]({{ site.baseurl }}/assets/images/github-actions-for-net-full-framework/actions-performed-result-ok.png "Actions performed, result ok")


### Details of actions performed
When analyzing the checks tab, we can verify the execution step-by-step.
If there was any failure, on this page it would be possible to check which step was broken and analyze the logs to identify the error.

![Details of actions performed]({{ site.baseurl }}/assets/images/github-actions-for-net-full-framework/details-of-actions-performed.png "Details of actions performed")

If you got curious to browse a page with details of the actions' executions, [here you can find a successful execution](https://github.com/felipetofoli/dotnet-full-framework-ci-sandbox/pull/6/checks), and [here you find a failed execution](https://github.com/felipetofoli/dotnet-full-framework-ci-sandbox/runs/2357988527?check_suite_focus=true).

## The project
Are you curious about the project? This is the repository where all the source code is available:
- [felipetofoli/dotnet-full-framework-ci-sandbox](https://github.com/felipetofoli/dotnet-full-framework-ci-sandbox)

Please, consider leaving a :star: in the repository it it helps you! :heart:

## Conclusion :dart:
Dealing with legacy code can be very complicated, so proposing workflow improvements can help us dramatically change the reality of the project, by encouraging quick feedback and mitigating points of insecurity that harm the development team.

GitHub Actions are very useful tools for automating CI processes. Although it does not contain an initial template in the official documentation, it is possible to create workflows with GitHub Actions for .Net Full Framework, thanks to the official actions and also those created and made available by the community.

I had some work to get my CI pipeline viable, if you have any problems in the implementation of yours and want to share in the comments of this post, I will be happy to help you.

Bye! :wink: :kissing_heart:

…

:warning: It is worth remembering that, before using the workflow, it is important to check if it and its dependencies are in accordance with the project policies where you want to apply them. :warning:
