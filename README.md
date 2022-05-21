# nicolgo.github.io
The blog based on [Hugo](https://gohugo.io/) deployed on [GitHub Pages](https://docs.github.com/en/pages) with Github Actions.
## 1 Docker
To create a Hugo site, the first thing you should install Hugo. To avoid installing Hugo on the PC, Docker is a good choice which allows users to package the application with all of its dependencies into inside container. 
### 1.1 Build container with Docker Compose(recommend)

#### 1.1.1 How to do it
1. creating a `docker-compose.yml` file with the configurations, and create a new empty `hugo` directory
2. uncomment the line `command: new site /src` and comment the line `command: server` in the `docker-compose.yml` file
3. run `docker-compose up`
4. comment the line `command: new site /src` and uncomment the line `command: server` in the `docker-compose.yml` file
5. run `docker-compose up` again, then you can see an empty site in `localhost:1313` address by browser

#### 1.1.2 How it works
The configure file of `docker-compose.yml` as the below shown:
```
version: "3"
services:
  server:
    image: klakegg/hugo:0.93.2-ext-ubuntu-onbuild
    container_name: hugo-server
    # uncomment this when first run
    # command: new site /src
    command: server
    volumes: 
      - "./hugo:/src"
    ports: 
      - "1313:1313"
```
Let me explain the details of the configurations:
1. **image**: The server create container based an image from a public repository in [Docker Hub](https://hub.docker.com/r/klakegg/hugo/), which is recommended by Hugo. As for the tags, you can use other as you like, but `-ext` is recommended since some theme may use additional tools.
2. **command**: The default entrypoint of the container is `hugo`, when we set `command: [option]`, it will run `hugo [option]` inside container. However, the `hugo server` command will fail with an error `hugo-server | Error: Unable to locate config file or config directory. Perhaps you need to create a new site.` when we build a website firstly since we have nothing in current workspace. To solve the problem, we can uncomment `command: new site /src` and comment `command: server`, which will create a create a new Hugo site in current workspace.
3. **volumes**: The key mounts the project directory(`./hugo`) on the host to the path('/src') inside the container. Using `./hugo` instead of current directory is because an error `hugo-server | Error: /src already exists and is not empty. See --force.` will occur if the initial directory is not empty. Another way to solve the problem is using `--force` flag.
4. **ports**: The key binds the container and the host machine to the same port `1313`.

### 1.2 Build with command line
Since there is no `docker-compose.yml`, we can create a website in current directory.
1. If you have no hugo website, run ` docker run --rm -it -v ${pwd}:/src klakegg/hugo:0.93.2-ext-ubuntu-onbuild new site .`to create a new Hugo site.
2. run server: `docker run --rm -it -v ${pwd}:/src -p 1313:1313 klakegg/hugo:0.93.2-ext-ubuntu-onbuild server`, then you can see an empty site in `locahost:1313`

## 2 Themes configure
There are so many beautiful themes for Hugo, we can select one in the [Hugo Themes](https://themes.gohugo.io/).  Here I use [zozo](https://github.com/nicolgo/hugo-theme-zozo) theme(fork from [hugo-theme-zozo](https://github.com/varkai/hugo-theme-zozo)) to show how to apply a theme for our site. 

since changing theme opeation just related to git operation and file change, so we can change theme by two approaches, inside or outside container.
### 2.1 setting theme outside container
If we want to upload the `docker-compose.yml` to your github repository, the method is recommended.
1. using `zozo` as submodule with
```
git submodule add --depth=1 https://github.com/nicolgo/hugo-theme-zozo.git hugo/themes/zozo
git submodule update --init --recursive # needed when you reclone your repo (submodules may not get cloned automatically)
```
2. Updating the theme:
```
git submodule update --remote --merge
```

3. Setting in `config.toml`:
```
theme: "zozo"
```
### 2.2 setting theme inside container
1. enter continer with:
```
docker exec -it hugo-server /bin/bash
```
2. Using theme as submodule with
```
git init
git submodule add --depth=1 https://github.com/nicolgo/hugo-theme-zozo.git themes/zozo
git submodule update --init --recursive # needed when you reclone your repo (submodules may not get cloned automatically)
```
3. Updating the theme:
```
git submodule update --remote --merge
```
4. Setting in `config.toml` with:
```
echo theme = \"zozo\" >> config.toml
```
5. start container by docker-compose or docker command.

## 3 GitHub Actions
There are two ways to publish your website. 
 1. using two repositories. one is a private repository to store file. another is a public repository named `username.github.io`, which store the static pages generated in the `./public` directory by Hugo.
 2. only use one repository but two branches. one is the main branch with original files, another branch used to store static pages generated by Hugo. The repository can be public or private.
Here we use one public repository to store our blog files and deployed the pages to `username.github.io` with GitHub Actions automatically.
### 3.1 Getting ready
1. create a new public repository follow [the guide](https://pages.github.com/).
2. upload blog file to the repository.
### 3.2 How to do it
1. create a file in `.github/workflows/gh-pages.yml`
2. if you did not create a `Hugo` to store your file, you just follow the [guide](https://gohugo.io/hosting-and-deployment/hosting-on-github/) to write the `gh-pages.yml`. The details is:
```
name: github pages

on:
  push:
    branches:
      - main  # Set a branch to deploy
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          # extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: github.ref == 'refs/heads/main'
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```
if you use a isolated `Hugo` directory, you should change the file as below:
```
name: github pages

on:
  push:
    branches:
      - main  # Set a branch to deploy
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          # extended: true

      - name: Build
        working-directory: ./hugo
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: ${{ github.ref == 'refs/heads/main' }}
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./hugo/public
``` 
3. rename `baseURL` in `config.toml` with the value `https://<USERNAME>.github.io` and upload it.
4. In the repository profile, going Settings->Pages, change the source branch to `gh-pages`. Congratulation, your site created successfully!
### 3.3 How it works
see [GitHub Action](https://docs.github.com/en/actions)
## 4 Tips on Writing
### 4.1 create a new blog page
1. enter the docker container with: 
```
docker exec -it hugo-server /bin/bash
```
2. using `hugo` command to generate a content file base on templates in [`Archethypes`](https://gohugo.io/content-management/archetypes/) folder
```
hugo new posts/first-blog.md
```
or create a content by [`Leaf Bundle`](https://gohugo.io/content-management/page-bundles/) way:
```
hugo new posts/first-blog/index.md
```

### 4.2 structure your images
There are two ways to group your images: [Static Files](https://gohugo.io/content-management/static-files/#readout) or [Page Bundles](https://gohugo.io/content-management/page-bundles/).

#### 4.2.1 Static Files
1. create a `static/` directory in site directory
2. adding a file `static/image.png`, then include it in markdown file with `![Example image](/image.png)`
#### 4.2.2 Page Bundles
1. create a content with the command below, `index.md` is a must.
```
hugo new posts/first-blog/index.md
```
2. add an image to `posts/first-blog/image.png`
The file structure will like this:
- posts
  - first-blog
    - index.md
    - image.png
3. include the image in markdown file with `![Example image](./image.png)`

## 5 Useful links
[1] [docker for beginners](https://docker-curriculum.com/#docker-compose)

[2] [Hugo Documentation](https://gohugo.io/documentation/)

[3] [How to use Develop Jekyll (and GitHub Pages) locally using Docker containers](https://github.com/BillRaymond/my-jekyll-docker-website#readme)

[4] [Build a Personal Website With GitHub Pages and Hugo](https://levelup.gitconnected.com/build-a-personal-website-with-github-pages-and-hugo-6c68592204c7)

[5] [GitHub Pages Action](https://github.com/peaceiris/actions-gh-pages#readme)

[6] [Hugo Docker Guide](http://i.lckiss.com/?p=7252)

 
