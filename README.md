# Lab 11: Future-proofing research

💪 Under development

## Preparation

- Read/ annotatae: [Recipe \#11](https://qtalr.com/resources/recipes/recipe-11/). You can refer back to this document to help you at any point during this lab activity.

## Objectives

- To understand the importance of future-proofing research
- To learn how to future-proof your research using Docker, GitHub Actions, and {renv}.
- To apply the concepts learned in this lab to your own research project.

## Instructions

### Getting started

In this lab, you will be using an existing or new repository to use as your project repository. The repository you will select to use will need to have a well-formed research compendium structure. You can use one of the following repositories:

- [Lab 10 repository](https://github.com/qtalr/lab-10)
- [Minimal template](https://github.com/qtalr/project)
- [Web-based project template](https://github.com/qtalr/project_web)
- Other (consult your instructor)

If you are starting with a new repository, fork and clone the repository you selected to your local machine. Then orient yourself to the repository by opening the README file and reviewing the template configuration.

If you are using an existing reproducible research project repository, open that project on your local machine, and `pull` the latest changes from the remote repository to ensure that your local and remote repositories are in sync.

You will not be doing any analysis in this lab. Instead, you will be adding future-proofing elements to your project repository. In particular, we will develop a Dockerfile, add {renv} to your project, and set up GitHub Actions to automate the workflow to develop and deploy your project as a shared resource that can be easily reproduced by others.

### Future-proofing research

**Add {renv} to your project**

1. Open your project in RStudio.
2. Install the {renv} package if you haven't already.
3. Initialize {renv} in your project by running the following command in the R console:

```r
renv::init()
```

4. Add the `renv/` directory to your `.gitignore` file. Very important!
5. Save your changes and commit them to your repository.
6. Add a package to your project. For example, you can add the `dplyr` package by running the following command in the R console:

```r
renv::install("dplyr")
```

7. Use the `renv::status()` function to check the status of your project.
8. As you will be prompted, run `renv::snapshot()` to save the state of your project.
9. Save your changes and commit them to your repository.
10. Review the `renv.lock` file and peruse the information about R and the R packages that are now part of your pinned R/R package environment.

**Dockerfile**

1. Create a new file in the root of your project directory called `Dockerfile`.
2. Add the following content to the `Dockerfile`:

```Dockerfile
# Use the official R Rocker rocker/rstudio image
FROM rocker/rstudio:4.1.0

# Install Ubuntu system dependencies
RUN apt-get update && apt-get install -y build-essential

# Change to default user
USER rstudio

# Change working directory to the home directory
WORKDIR /home/rstudio

# Install key R packages ({pak} and {renv})
RUN R -e "install.packages(c('pak', 'renv'))"

# Copy the project files into the Docker image
# (see note below)
COPY . /home/rstudio/project_name

# Start the RStudio server
CMD ["/init"]

```

Note: You may or may not want to copy your entire project into the Docker image. If you have large data files or other files that are not necessary for the analysis, you may want to exclude them from the Docker image. You can do this by creating a `.dockerignore` file in the root of your project directory and adding the files you want to exclude from the Docker image. For example, you can add the following content to the `.dockerignore` file:

```plaintext
data/*
```

3. Save your changes and commit them to your repository.

**GitHub Actions**

> The goal here will be to set up a GitHub Action that will build a Docker image of your project and push it to Docker Hub. This will allow you to share your project with others and make it easy for them to reproduce your analysis. You will need to have a [Docker Hub account](https://app.docker.com/signup) and a [Docker Hub Acess Token](https://app.docker.com/settings/personal-access-tokens) to complete this step.

1. Create a new directory in the root of your project directory called `.github/workflows`.
2. Create a new file in the `.github/workflows` directory called `image-build.yml`.
3. Add the following content to the `image-build.yml` file:

```yaml
name: Docker Build and Push to Docker Hub
on:
  push:
    branches: [main]
jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      - name: Docker Login
        uses: docker/login-action@v3.0.0
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
      - name: Build and push Docker images
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKER_HUB_USERNAME }}/qtalr-r:latest
          platforms: linux/amd64,linux/arm64
```

4. Before you push your changes to your repository, you will need to add your Docker Hub username and access token as secrets to your repository. To do this, go to your repository on GitHub, click on the 'Settings' tab, and then click on the 'Secrets' link in the left sidebar. Click on the 'New repository secret' button and add the following secrets:

- `DOCKER_HUB_USERNAME`: Your Docker Hub username
- `DOCKER_HUB_ACCESS_TOKEN`: Your Docker Hub access token

5. From now on, when you stage and commit your changes to your repository, the GitHub Action you created will be triggered, building a Docker image of your project and pushing it to Docker Hub. You can then share the Docker image with others, allowing them to easily reproduce your analysis.

## Assessing your progress

1. In your repository on Github, open an issue to provide feedback on your experience with this lab (Click on the 'Issues' tab and then click the 'New issue' button). Title the issue "Lab 11 feedback" and provide your feedback in the body of the issue.

Some questions to consider:

- What did you learn?
- What did you find most/ least challenging?
- What resources did you consult?
  - Instructor? R packages, Docker, Github, etc.
- What more would you like to know about reproducible research?
  - Find potential resources you might consult to continue your learning. Provide links and a brief description of the resource.

## Submission for feedback

- Provide a link to your GitHub repository and the link to the image on Docker Hub in your issue/ feedback submission.

