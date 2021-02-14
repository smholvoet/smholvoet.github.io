---
published: true
title: "Using Dependabot with Azure DevOps"
excerpt: "Automated dependency scanning using Dependabot and Azure DevOps"
date: 2021-02-14T00:00:00-04:00
show_date: true
tags:
  - Azure DevOps
  - Dependabot
  - dependencies
  - security
  - .NET
---

![image-center](/assets/images/dependabot_AzDo_logo.png){: .align-center}

Keeping track of any outdated dependecies can be a real nightmare, especially if you have lots of them.
This is where **Dependabot**  enters the ring.

From the [Dependabot](https://dependabot.com/) website:

> 1. Dependabot pulls down your dependency files and looks for any outdated or insecure requirements.
> 2. If any of your dependencies are out-of-date, Dependabot opens individual pull requests to update each one.
> 3. You check that your tests pass, scan the included changelog and release notes, then hit merge with confidence.

The tool in question is actually completely free of charge (🤤), and natively integrates with [GitHub](https://github.com/) ever since they acquired it in [May 2019](https://dependabot.com/blog/hello-github/).

It currently supports the following languages:

- Ruby
- JavaScript
- Python
- PHP
- Elixir
- Rust
- Java (Maven + Gradle)
- **.NET**
- Go
- Elm
- Git Submodules
- Docker files
- Terraform files
- GitHub Actions

## Workflow

Normally, you'd configure Dependabot by checking in a `.dependabot/config.yml` ([reference](https://dependabot.com/docs/config-file/)) into your repo where you pass the type of package manager, how often to check for updates, location of your package manifest and so on. Have a look at the example below:

```yaml
version: 1
update_configs:
  - package_manager: "javascript"
    directory: "/app"
    update_schedule: "daily"
```

Unfortunately , there is no native integration with Azure DevOps 🤷‍♂️. However, using the [Dependabot Update Script](https://github.com/dependabot/dependabot-script) (which leverages the Dependabot Core logic), we *can* make 🤖 Dependabot play nice with Azure DevOps.

I've set up a pipeline which lets Dependabot work its magic in a .NET project containing multiple [`packages.config`](https://docs.microsoft.com/en-us/nuget/reference/packages-config) files.

- Clone the dependabot update script
- Pull `dependabot/dependabot-core` Docker image
- Install dependencies
- Scan the entire repository for any `packages.config` files, containing .NET dependencies
- Dependabot automagically opens a seperate PR for every out-of-date dependency
- Profit 🍾

![dependabot-flow](/assets/images/dependabot-flow.png){: .align-center}

## Pipeline (YAML)

This is what the pipeline looks like:

```yaml
pool:
  vmImage: 'ubuntu-20.04'

schedules:
- cron: "0 0 * * *"
  displayName: Daily Dependabot run

steps:

- script: |
    git clone https://github.com/dependabot/dependabot-script.git
  displayName: clone dependabot repo
  
- script: docker pull dependabot/dependabot-core
  displayName: pull dependabot-core Docker image

- script: |
    cd dependabot-script
    docker run -v "$(pwd):/home/dependabot/dependabot-script" \
               -w /home/dependabot/dependabot-script dependabot/dependabot-core bundle install \
               -j 3 \
               --path vendor
  displayName: install dependencies

- task: Bash@3
  inputs:
    filePath: 'dependabot-devops.sh'
  displayName: "run dependabot"
```

Let's break things down...

### Schedule

```yaml
schedules:
- cron: "0 0 * * *"
  displayName: Daily Dependabot run
```

As you can see from the `schedules` section, I'm using a scheduled trigger to run this pipeline on a regular interval (daily in this case). The format being used to define the schedule here is called [cron syntax](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/scheduled-triggers?view=azure-devops&tabs=yaml#cron-syntax). Have look at the the [Configure schedules for pipelines](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/scheduled-triggers?view=azure-devops&tabs=yaml) article over at MS Docs.

You could also use [crontab guru](https://crontab.guru/) which generates cron schedule expressions for you 🤯.

### Fetch dependabot-script

```yaml
- script: |
    git clone https://github.com/dependabot/dependabot-script.git
  displayName: clone dependabot repo
```

In the first step, we're simply cloning the [`dependabot-script`](https://github.com/dependabot/dependabot-script) repository, which we'll need later on to run our Docker image.

### Pull Docker image

```yaml  
- script: docker pull dependabot/dependabot-core
  displayName: pull dependabot-core Docker image
```

Pull the [dependabot/dependabot-core](https://hub.docker.com/r/dependabot/dependabot-core/tags?page=1&ordering=last_updated) image from 🐳 Docker Hub, which contains the core logic behind Dependabot's update PR creation.

### Install dependencies

```yaml
- script: |
    cd dependabot-script
    docker run -v "$(pwd):/home/dependabot/dependabot-script" \
               -w /home/dependabot/dependabot-script dependabot/dependabot-core bundle install \
               -j 3 \
               --path vendor
  displayName: install dependencies
```

Here we first navigate to the `dependabot-script` directory which we cloned [earlier](#fetch-dependabot-script). Then we launch a Docker container, based on the `dependabot-core` image we just [downloaded](#pull-docker-image). We're using the [docker run](https://docs.docker.com/engine/reference/run/) command, supplied with the following parameters:

- `-v`: mount the current working directory (`dependabot-script`) into the container.
- `-w`: set the working directory to `/home/dependabot/dependabot-script`.

We then run [`bundle install -j 3 --path vendor`](https://bundler.io/man/bundle-install.1.html) inside the container, which will install the necessary dependencies in order to get Dependabot up and running. We're using the following parameters:

- `-j`: the number of parallel download & install jobs to use, `3` in this case (instead of the default `1`).
- `--path`: location where our dependencies will be installed, `vendor` in this case.

### Run bash script

```yaml
- task: Bash@3
  inputs:
    filePath: 'dependabot-devops.sh'
  displayName: "run dependabot"
```

Finally, we'll run the [bash script](#bash-script), make sure the `filePath` is set correctly.
You could also copy and paste the contents of the script below and run it inline.

## Bash script

The `dependabot-devops.sh` Bash script is where the actual 🔎 scanning takes place. Here's the complete [GitHub Gist](https://gist.github.com/smholvoet/3526d7571d42b39779000aab5035d8e6) which you can download & commit to your repo:

{% gist 3526d7571d42b39779000aab5035d8e6 %}

### Setting up variables

```bash
SYSTEM_COLLECTIONURI_TRIM=`echo "${SYSTEM_COLLECTIONURI:22}"`
PROJECT_PATH="$SYSTEM_COLLECTIONURI_TRIM$SYSTEM_TEAMPROJECT/_git/$BUILD_REPOSITORY_NAME"
```

As  you can see, I make use of a couple of [predefined variables](https://docs.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops&tabs=yaml) within Azure DevOps in order to make the script as generic as possible.

- `SYSTEM_COLLECTIONURI`: The URI of the TFS collection or Azure DevOps organization. For example `https://dev.azure.com/fabrikamfiber/`. This variable gets trimmed in order to get rid of the `https://dev.azure.com/` portion.
- `SYSTEM_TEAMPROJECT`: The name of the team project that contains the pipeline.
- `BUILD_REPOSITORY_NAME`: The name of the triggering repository.

The variables aboved are appended and used to construct the `$PROJECT_PATH`. The variable contains the [remote](https://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes) which Dependabot will need in order to know *where* exactly it needs to create any pull requests. The `$PROJECT_PATH` will look like this: `<org>/<project>/_git/<repo>`.

### Scan for package manifests

```bash
FILECOUNT="$(find . -name packages.config | wc -l)"
echo "Found $FILECOUNT dependency file(s)."

find . -name packages.config | while read path; do
PARENTNAME="$(basename "$(dirname "$path")")"
DIRECTORY_PATH="/"$PARENTNAME
echo "directory: $DIRECTORY_PATH"
```

I then use the `find . -name packages.config` command to find any [`packages.config`](https://docs.microsoft.com/en-us/nuget/reference/packages-config) files contained within the repository. These are the dependency files or package manifests which Dependabot needs to detect your actual dependencies. Dependabot just needs the location where to look for any package manifests, which is  stored in the `$DIRECTORY_PATH` variable.

### Docker run 🚀

I then run the `dependabot-core` container for any package manifest (`packages.config` in case of my .NET project) that was found.

```bash
docker run  -v "$(pwd)/dependabot-script:/home/dependabot/dependabot-script" 
            -w '/home/dependabot/dependabot-script' 
            -e AZURE_ACCESS_TOKEN='<PAT>' 
            -e PACKAGE_MANAGER='nuget' 
            -e PROJECT_PATH=$PROJECT_PATH 
            -e DIRECTORY_PATH=$DIRECTORY_PATH 
            dependabot/dependabot-core bundle exec ruby ./generic-update-script.rb
```

As you can see there are a whole bunch of [environment variables](https://docs.docker.com/engine/reference/commandline/run/#set-environment-variables--e---env---env-file) (`-e <...>`) being passed to our Docker container:

- `AZURE_ACCESS_TOKEN`: Azure DevOps [PAT](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page) which allows to Dependabot to create PR's within Azure DevOps
- `PACKAGE_MANAGER`: What package manager to use (`nuget`in case of my .NET project)
- `PROJECT_PATH`: see my [earlier](#setting-up-variables) explanation about the `$PROJECT_PATH` variable
- `DIRECTORY_PATH`: see my [earlier](#setting-up-variables) explanation about the `$DIRECTORY_PATH` variable
- (optional) `GITHUB_ACCESS_TOKEN`: you might run into API rate limits depending on how often you run the pipeline. Create a [GitHub PAT](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) and add `-e GITHUB_ACCESS_TOKEN <PAT>` to increase these limits.

⚠️ Upon creating the Azure DevOps personal access token, make sure you check the **Code** scope so Dependabot has enough permissions to raise any PR's.
![PAT-scope-code](/assets/images/PAT-code-scope.png){: .align-center}
{: .notice--warning}

The `dependabot/dependabot-core bundle exec ruby ./generic-update-script.rb` command runs a ruby script inside the `dependabot/dependabot-core`Docker container which does the actual magic of scanning for any outdated dependencies, parsing the package manifest(s) and creation of the pull request(s).

## Results

This is what the end result looks like:

![dependabot-results-1](/assets/images/dependabot-results-1.png)

![dependabot-results-2](/assets/images/dependabot-results-2.png)

Dependabot created a seperate branch for any outdated package it found, and raised a pull request 🔥🔥🔥.

Combined with the necessary branch policies (e.g. [build validation](https://docs.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops#build-validation)), you could make sure any updated packages won't break your build. All in all a great tool which takes care of depedency scanning for you 💪.
