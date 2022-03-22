# Creating a Helm Chart Repository

[FUENTE](https://tech.paulcz.net/blog/creating-a-helm-chart-monorepo-part-1/)

 In this part I will demonstrate creating a Helm chart repository using GitHub and GitHub Pages.

## create a new GitHub Repository

**genesiscastillo/my-helm-charts**

https://genesiscastillo.github.io/my-helm-charts/

You can now clone down this repository and get to work:

```bash
$ git clone https://github.com/genesiscastillo/my-helm-charts.git
Cloning into 'my-helm-charts'...
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (4/4), 4.52 KiB | 4.52 MiB/s, done.

$ cd my-helm-charts

$ tree
.
├── LICENSE
└── README.md

0 directories, 2 files
```

### creating charting

```bash
$ mkdir charts

$ [Helm](https://helm.sh) create charts/app1
Creating charts/app1

$ [Helm](https://helm.sh) create charts/app2
Creating charts/app2

$ tree
.
├── charts
│   ├── app1
│   │   ├── charts
│   │   ├── Chart.yaml
│   │   ├── templates
│   │   │   ├── deployment.yaml
│   │   │   ├── _helpers.tpl
│   │   │   ├── ingress.yaml
│   │   │   ├── NOTES.txt
│   │   │   ├── service.yaml
│   │   │   └── tests
│   │   │       └── test-connection.yaml
│   │   └── values.yaml
│   └── app2
│       ├── charts
│       ├── Chart.yaml
│       ├── templates
│       │   ├── deployment.yaml
│       │   ├── _helpers.tpl
│       │   ├── ingress.yaml
│       │   ├── NOTES.txt
│       │   ├── service.yaml
│       │   └── tests
│       │       └── test-connection.yaml
│       └── values.yaml
├── LICENSE
└── README.md
```

Push these changes to git:

```bash
echo ".deploy" >> .gitignore

$ git add .

$ git commit -m 'Initial Commit'
[master 2172141] Initial Commit
 18 files changed, 524 insertions(+)
...
...

$ git push origin master
Enumerating objects: 27, done.
Counting objects: 100% (27/27), done.
Delta compression using up to 4 threads
Compressing objects: 100% (24/24), done.
Writing objects: 100% (26/26), 4.72 KiB | 536.00 KiB/s, done.
Total 26 (delta 8), reused 0 (delta 0)
remote: Resolving deltas: 100% (8/8), done.
To github.com:paulczar/my-helm-charts.git
   abdcced..2172141  master -> master
```

## Publish your Helm Repository

### Prepare  GITHub Pages

```bash
$ git checkout --orphan gh-pages
Switched to a new branch 'gh-pages'

$ rm -rf charts

$ git add . --all

$ git commit -m 'initial gh-pages'
git commit -m 'initial gh-pages'
[gh-pages a9ce382] initial gh-pages
 18 files changed, 524 deletions(-)
...
...

$ git push origin gh-pages
Enumerating objects: 3, done.
...
...
To github.com:paulczar/my-helm-charts.git
 * [new branch]      gh-pages -> gh-pages
```

![figura00](./png/pic03.png)

After a few minutes you should have a default rendering on your README.md at the provided URL:

https://genesiscastillo.github.io/my-helm-charts/

![figura0](./png/pic04.png)

## Introducing chart-releaser

You could use a combination of helm package and helm repo commands to construct your Helm repository by hand, or you can simplify your life by using chart-releaser which will not only create your packages, but will upload them as binaries into an appropriately versioned GitHub Release.

https://github.com/helm/chart-releaser/releases


```bash
cd /tmp
$ curl -sSL https://github.com/helm/chart-releaser/releases/download/v0.2.1/chart-releaser_0.2.1_linux_amd64.tar.gz | tar xzf -

$ mv cr ~/bin/cr

$ cr help
Create [Helm](https://helm.sh) chart repositories on [GitHub](https://github.com) Pages by uploading Chart packages
and Chart metadata to [GitHub](https://github.com) Releases and creating a suitable index file

Usage:
  cr [command]

Available Commands:
  help        Help about any command
  index       Update Helm repo index.yaml for the given GitHub repo
  upload      Upload Helm chart packages to GitHub Releases
  version     Print version information

Flags:
      --config string   Config file (default is $HOME/.cr.yaml)
  -h, --help            help for cr

Use "cr [command] --help" for more information about a command.
```

There are two commands we care about **cr index** and **cr upload**, the first will create an appropriate __index.yaml__ and the second will upload the packages to __GitHub Releases__. 

In order to do the latter you’ll need __*to pass it in a GitHub Token so that it can use the GitHub APIs*.__

In your browser go to your github developer settings and create a new personal access token.

https://github.com/settings/tokens/new

![figura-01](./png/pic01.png)


your token
![figura-02](./png/pic02.png)

### set environment : << OPTIONAL >>>>

Create an environment variable (or a ~/.cr.yaml config file) containing the access token:
*.cr.yaml*
```yaml
CH_TOKEN:  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
```
or
```bash 
set CH_TOKEN=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

NOT WORKING
```

## Create and Upload Helm Packages

Your next step is to create and upload the packages:
```bash
$ git checkout main

$ my-helm-charts>helm package charts/app1 --destination .deploy
Successfully packaged chart and saved it to: .deploy\app1-0.1.0.tgz

$ my-helm-charts>helm package charts/app2 --destination .deploy
Successfully packaged chart and saved it to: .deploy\app2-0.1.0.tgz
```

Run cr upload to create releases and upload the packages, note if it runs correctly there’s no output.

```bash
$ cr upload -o genesiscastillo -r my-helm-charts -t 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 -p .deploy
```

Check your GitHub repository now has a releases page with two releases:

https://github.com/genesiscastillo/my-helm-charts/releases

![figura05](./png/pic05.png)

## Create and upload index file to GitHub Pages

```bash
$ git checkout gh-pages

$ cr index -i ./index.yaml -p .deploy --owner genesiscastillo --git-repo my-helm-charts --charts-repo https://genesiscastillo.github.io/my-helm-charts/

UpdateIndexFile new index at ./index.yaml
Found app1-0.1.0.tgz
Extracting chart metadata from .deploy\app1-0.1.0.tgz
Calculating Hash for .deploy\app1-0.1.0.tgz
Found app2-0.1.0.tgz
Extracting chart metadata from .deploy\app2-0.1.0.tgz
Calculating Hash for .deploy\app2-0.1.0.tgz
Updating index ./index.yaml
```

*RESULT my-helm-charts/index.yaml:*

```yaml
apiVersion: v1
entries:
  app1:
  - apiVersion: v2
    appVersion: 1.16.0
    created: "2022-03-21T18:29:10.0959739-03:00"
    description: A Helm chart for Kubernetes
    digest: dc4a728d7b37b77e2c0cf82d826388d1e4d8de57203a26be867413804e26b8ed
    name: app1
    type: application
    urls:
    - https://github.com/genesiscastillo/my-helm-charts/releases/download/app1-0.1.0/app1-0.1.0.tgz
    version: 0.1.0
  app2:
  - apiVersion: v2
    appVersion: 1.16.0
    created: "2022-03-21T18:29:10.3449054-03:00"
    description: A Helm chart for Kubernetes
    digest: a741d3f049a2e67200971450bdd9a558a37bb3199094db75b4653a6c37594abf
    name: app2
    type: application
    urls:
    - https://github.com/genesiscastillo/my-helm-charts/releases/download/app2-0.1.0/app2-0.1.0.tgz
    version: 0.1.0
generated: "2022-03-21T18:29:10.3458953-03:00"

```

Commit this to git and then wait a few minutes and check that it exists in your GitHub Pages url:

```bash
$ git add index.yaml

$ git commit -m 'release 0.1.0'
[gh-pages 696df18] release 0.1.0
 1 file changed, 23 insertions(+)
 create mode 100644 index.yaml

$ git push origin gh-pages
...
To github.com:paulczar/my-helm-charts.git
   75f1fe8..696df18  gh-pages -> gh-pages
```

Check it exists in GitHub Pages:

https://genesiscastillo.github.io/my-helm-charts/index.yaml

![figura6](./png/pic06.png)

# Test your new Helm Repostiory

```bash
$ helm repo add my-helm-charts https://genesiscastillo.github.io/my-helm-charts
"my-helm-charts" has been added to your repositories

$ kubectl create namespace test

$ helm install app1 my-helm-charts/app1 --namespace test
NAME: app1
LAST DEPLOYED: Tue Mar 22 08:51:15 2022
NAMESPACE: test
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace test -l "app.kubernetes.io/name=app1,app.kubernetes.io/instance=app1" -o jsonpath="{.items[0].metadata.name}")

    ---> app1-666cdfdd8d-6qfbp
  export CONTAINER_PORT=$(kubectl get pod --namespace test $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")

    ---> 80
  echo "Visit http://127.0.0.1:8181 to use your application"
  kubectl port-forward $POD_NAME 8181:$CONTAINER_PORT --namespace test 

    ---> kubectl port-forward app1-666cdfdd8d-6qfbp --namespace test 8181:80
```

![figura07](./png/pic07.png)

## checking that it deployed in cluster minikube

```bash
kubectl -n test get all
```

![figura08](./png/pic08.png)

## Clean up

```bash
$ helm uninstall app1 --namespace test 
release "app1" uninstalled
```

## Update the README.md with instructions on using

switch back to your master/main brach:

```bash
$ git checkout main
```
Edit your README.md to provide details on how to use charts from your repository:

```metadata
# My [Helm](https://helm.sh) Charts

This repository contains [Helm](https://helm.sh) charts for various projects

* [Application 1](charts/app1/)
* [Application 2](charts/app2/)

## Installing Charts from this Repository

Add the Repository to Helm:

    helm repo add my-helm-charts https://tech.paulcz.net/my-helm-charts

Install Application 1:

    helm install my-helm-charts/app1

Install Application 2:

    helm install my-helm-charts/app2
```

Commit the change up to GitHub:

```bash
$ git add README.md
$ git commit -m 'update readme with instructions'
$ git push origin main
```

![figura09](./png/pic09.png)