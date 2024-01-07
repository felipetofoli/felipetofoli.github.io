---
layout: post
title:  "Code formatting for C# projects: dotnet format + GitHub Actions"
date:   2021-08-16 08:00:00 -0300
tags: dotnet csharp github formatting
---
Originally posted here: [dev.to](https://dev.to/felipetofoli/code-formatting-for-c-projects-dotnet-format-github-actions-54p8) account.

![Code formatting for C# projects: dotnet format + GitHub Actions - Banner]({{ site.baseurl }}/assets/images/code-formatting-for-csharp-projects-dotnet-format-and-github-actions/code-formatting-for-csharp-projects-dotnet-format-and-github-actions-banner.webp "Code formatting for C# projects: dotnet format + GitHub Actions - Banner")

## TL;DR

Getting people to use their code review time to pinpoint code formatting issues **is not productive or sustainable**.

So, in this blog post, I show you:
- **A better approach to this code formatting verification step** by using the **`dotnet format`** tool, and;
- How to **build a GitHub Action** to make this verification in an **automated and continuous** way.


## Why should I improve the code formatting verification in my projects?

Ensuring the consistent formatting of a project (especially those with long years of life and many people participating) is a challenge.

When the project does not use any tools to assist in this formatting process, the checks end up depending a lot on people's attention and will.

Furthermore, getting people to use their code review time to pinpoint code formatting issues (turning people into code linters) is not productive or sustainable.


## How can we do it in a better way?

To apply formatting styles consistently across C# projects you can use the global dotnet tool: [dotnet-format](https://github.com/dotnet/format).

The `dotnet-format` tool relies on the `.editorconfig` configuration file to apply formatting to the project.

The `.editorconfig` is a file format for defining and maintaining consistent coding styles for multiple developers working on the same project in different editors or IDEs.

A good option is to bring the team together and clarify the settings that will be adopted, avoiding confusion.

Having a `.editorconfig` is useful because, with the configuration file, it is possible to register, keep the history and track the standards adopted by the team.

If you don't know how to start, you can use [this .editorconfig file](https://docs.microsoft.com/en-us/dotnet/fundamentals/code-analysis/code-style-rule-options?view=vs-2019#example-editorconfig-file) available in Microsoft documentation.


## How to use dotnet format tool to apply code formatting

The `dotnet format` command can be run locally to check and fix code that does not adhere to the defined standards.

More than that, **code formatting verification can be added to the CI**, to automatically and continuously ensure that the desired formatting is being applied to the project.

### Running in a local environment

To **install** `dotnet-format` just run the following command:
```bash
 dotnet tool install -g dotnet-format
```

If you just want to **check** that your project's formatting is according to the `.editorconfig` set, you can run:
```bash
 dotnet format '.\' --folder --check --verbosity diagnostic
```

In the above command:
- `--folder` option indicates that the path should be treated as a folder;
- `--check` option says this is only a verification - the project code will not be changed / formatted if deviations from the `.editorconfig` settings are found;
- `--verbosity diagnostic` option is used to get the execution logs as verbose as possible.

To **apply** formatting to the code, simply remove the `--check` option from the previous command:
```bash
 dotnet format '.\' --folder --verbosity diagnostic
```

If you want to know more options for running `dotnet format`, you can refer to the [official documentation](https://github.com/dotnet/format#how-to-use).

### To automate checking the code formatting, we can use GitHub Actions

The creation of a GitHub Action can be used to check the code formatting compliance with each change applied to the repository, as shown in the `dotnet-format.yml` file below:

{% gist c62bf52aff120cd2fba6bd18d8836966 dotnet-format.yml %}

&nbsp;

#### Understanding the dotnet-format.yml GitHub Action

Below, step-by-step explanation of the `dotnet-format.yml` file:

1. Define the name of the GitHub Action.
    ```yml
     name: dotnet format
    ```

2. Set that the actions should be triggered by changes on the `.editorconfig` or C# sourcecode files (`**.cs`).
    ```yml
     on:
       push:
         paths:
           - "**.cs"
           - ".editorconfig"
    ```

3. As, in this case, we are dealing with a .Net Full Framework project - which is developed, compiled and published on Windows - we will keep this environment for checking the code formatting (considering the particular `crlf` end-of-line settings as well).
    ```yml
     jobs:
       check-format:
         runs-on: windows-latest
    ```

4. Define the section that will group all the steps performed, detailed in the next items.
    ```yml
         steps:      
    ```

5. Setup the dotnet core for later use of the dotnet CLI. Although our project is .Net Full Framework, we were able to use some global dotnet CLI tools for it.
    ```yml
           - name: Setup .NET Core
             uses: actions/setup-dotnet@v1
             with:
               dotnet-version: '5.0.x'
    ```

6. Install the `dotnet-format` tool for later use.
    ```yml
           - name: Install dotnet-format tool
             run: dotnet tool install -g dotnet-format
    ```

7. Checks out the code.
    ```yml
           - name: Check out code
             uses: actions/checkout@v2
    ```

8. Run the `dotnet format` command, for the specified path, only checking code formatting (not changing any files), and setting the maximum verbosity to get a detailed execution report.
    ```yml
           - name: Run dotnet format
             run: dotnet format '.\' --folder --check --verbosity diagnostic
    ```

#### Results

After implementing our code formatting verification step, just open a Pull Request by changing the `.editorconfig` file or any C# class (`**.cs`) to see the results of GitHub Action execution.

##### Failure :x:

To simulate a failure in a GitHub Action run, I added poorly formatted code to the project repository:
![Poorly formatted code]({{ site.baseurl }}/assets/images/code-formatting-for-csharp-projects-dotnet-format-and-github-actions/poorly-formatted-code.png "Poorly formatted code")

The check failed as the code formatting was not adherent to the `.editorconfig` settings, so we got some error messages about the incorrect code spacing:
> ...
> 
> D:\a\dotnet-full-framework-ci-sandbox\dotnet-full-framework-ci-sandbox\src\Sandbox.WebAPI\Controllers\ValuesController.cs(10,45): **error WHITESPACE: Fix whitespace formatting. Insert '\s'.** [D:\a\dotnet-full-framework-ci-sandbox\dotnet-full-framework-ci-sandbox\]
> 
> D:\a\dotnet-full-framework-ci-sandbox\dotnet-full-framework-ci-sandbox\src\Sandbox.WebAPI\Controllers\ValuesController.cs(10,46): **error WHITESPACE: Fix whitespace formatting. Insert '\s'.** [D:\a\dotnet-full-framework-ci-sandbox\dotnet-full-framework-ci-sandbox\]
>  
> ...

The check failure was displayed on Pull Request:
![Failure]({{ site.baseurl }}/assets/images/code-formatting-for-csharp-projects-dotnet-format-and-github-actions/failure.png "Failure")

You can see the complete logging of the failed execution [here](https://github.com/felipetofoli/dotnet-full-framework-ci-sandbox/runs/3237686242?check_suite_focus=true).

##### Success :heavy_check_mark:

We fixed the code formatting, according to what was reported by the `dotnet format` tool.

![Fix code formatting]({{ site.baseurl }}/assets/images/code-formatting-for-csharp-projects-dotnet-format-and-github-actions/fix-code-formatting.png "Fix code formatting")

After the code was fixed, the check passed successfully, as we can see directly on the PR screen.

![Success]({{ site.baseurl }}/assets/images/code-formatting-for-csharp-projects-dotnet-format-and-github-actions/success.png "Success")

You can see the complete logging of the succeded execution [here](https://github.com/felipetofoli/dotnet-full-framework-ci-sandbox/runs/3237725396?check_suite_focus=true)


## The project

Are you curious about the project? This is the repository where all the source code is available:
- [felipetofoli/dotnet-full-framework-ci-sandbox](https://github.com/felipetofoli/dotnet-full-framework-ci-sandbox)

Please, consider leaving a :star: in the repository it it helps you! :heart:

## Conclusion :dart:
Maintaining a large codebase, ensuring its quality and that it meets the defined formatting standards, is a big challenge.

Building a flow that automatically performs these steps is very important to keep the maintenance process easier and more sustainable. 

Using GitHub Actions with the `dotnet format` command helps achieve these goals in a simple and automated way.