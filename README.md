# nicolgo.github.io
The blog based on [Hugo](https://gohugo.io/) deployed on [GitHub Pages](https://docs.github.com/en/pages).
## 1 Docker
To create a Hugo site, the first thing you should install Hugo. To avoid installing Hugo on the PC, Docker is a good choice which allows users to package the application with all of its dependencies into inside container. 
### 1.1 Build container with Docker Compose(recommend)
#### 1.1.1 How it works
The configure file of `docker-compose.yml` as the below shown:
```
version: "3"
services:
  server:
    image: klakegg/hugo:0.93.2-ext-ubuntu-onbuild
    container_name: hugo-server
    # uncomment this when first run
    # command: new site /src -f yml
    command: server
    volumes: 
      - "./hugo:/src"
    ports: 
      - "1313:1313"
```
Let me explain the details of the configurations:
1. image: The server create container based an image from a public repository in [Docker Hub](https://hub.docker.com/r/klakegg/hugo/), which is recommended by Hugo. As for the tags, you can use other as you like, but `-ext` is recommended since some theme may use additional tools.
2. command: The default entrypoint of the container is `hugo`, when we set `command: [option]`, it will run `hugo [option]` inside container. However, the `hugo server` command will fail with an error `hugo-server | Error: Unable to locate config file or config directory. Perhaps you need to create a new site.` when we build a website firstly since we have nothing in current workspace. To solve the problem, we can uncomment `command: new site /src -f yml` and comment `command: server`, which will create a create a new Hugo site in current workspace.
3. volumes: The key mounts the project directory(`./hugo`) on the host to the path('/src') inside the container. Using `./hugo` instead of current directory is because an error `hugo-server | Error: /src already exists and is not empty. See --force.` will occur if the initial directory is not empty. Another way to solve the problem is use `--force` flag.
4. ports: The key binds the container and the host machine to the same port `1313`.

#### 1.1.2 How to do it
1. creating a `docker-compose.yml` file with the configurations, and create a new empty `hugo` directory
2. uncomment the line `command: new site /src -f yml` and comment the line `command: server` in the `docker-compose.yml` file
3. run `docker-compose up`
4. comment the line `command: new site /src -f yml` and uncomment the line `command: server` in the `docker-compose.yml` file
5. run `docker-compose up` again, then you can see an empty site in `localhost:1313` address by browser

### 1.2 Build with command line
Since there is no `docker-compose.yml`, we can create a website in current directory.
1. If you have no hugo website, run ` docker run --rm -it -v ${pwd}:/src klakegg/hugo:0.93.2-ext-ubuntu-onbuild new site .`to create a new Hugo site.
2. run server: `docker run --rm -it -v ${pwd}:/src -p 1313:1313 klakegg/hugo:0.93.2-ext-ubuntu-onbuild server`, then you can see an empty site in `locahost:1313`
### 1.3 Themes configure
There are so many beautiful themes for Hugo, we can select one in the [Hugo Themes](https://themes.gohugo.io/).  Here I use [PaperMod](https://themes.gohugo.io/themes/hugo-papermod/) theme to show how to apply a theme for our site. 

since changing theme opeation just related to git operation and file change, so we can change theme by two approaches, inside or outside container.
#### 1.3.1 setting theme outside container
If we want to upload the `docker-compose.yml` to your github repository, the method is recommended.
1. using `PaperMod` as submodule with
```
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git hugo/themes/PaperMod
git submodule update --init --recursive # needed when you reclone your repo (submodules may not get cloned automatically)
```
2. Updating the theme:
```
git submodule update --remote --merge
```

3. Setting in `config.yml`:
```
theme: "PaperMod"
```
#### 1.3.2 setting theme inside container
1. enter continer with:
```
docker exec -it hugo-server /bin/bash
```
2. Using theme as submodule with
```
git init
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git submodule update --init --recursive # needed when you reclone your repo (submodules may not get cloned automatically)
```
3. Updating the theme:
```
git submodule update --remote --merge
```
4. Setting in `config.yml` with:
```
echo theme = \"PaperMod\" >> config.yml
```
5. start container by docker-compose or docker command.

## 2 GitHub Actions
There are two ways to publish your website.



