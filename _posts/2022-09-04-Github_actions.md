---
layout: post
title: Continous Deployment of software using Github Actions
---

A couple of months ago, I wanted to seamlessly update the code of my small web app running on my server. 
I was tired of using SFTP or copying code over SSH to the terminal editor every time I update something as small as the shade of a color. 

So I started looking for something more DevOps like. Some software for CI (Continuous Integration), CD(Continuous Delivery) and CD (Continuous Deployment). Mostly for Continuous Delivery because I don't need integration considering I was doing the project alone. I looked into many solutions, such as CircleCI, GitLab CI/CD, Travis CI. They are all awesome tools, but they are not worth the hassle for small apps. They need to many configurations for something as simple as automatically downloading the new version from the git repository. Then I remembered the CI/CD platform Github made, Github Actions 

### Github Actions

Github Actions is platform for CI/CD and has many more options. It's integrated to the GitHub "ecosystem" and we can easily connect it to our repositories, integrate it with GitHub Pilot to compile and make releases for us and much more. It's very simple to use, you create so-called Workflows for everything you need and attach them to your desired repositories. The Workflows are some type of YAML config files, you can make them yourself or use one already made. There are many available on the so-called Marketplace for  nearly every possible job you need, and you don't have to bother much.


The way I've done this is very simple, Github Actions are called every time there is git push to the repository. After that, Github Actions run the workflow, in this case a simple SSH login and running git pull to the master/main branch. This is the simplest way, it's the easiest, but it is not secure, nor very smart to do this. But for my case, a simple web app, still in development, without any need of security or stability whatsoever. 


Now let's get started on how to do this.


First, we make a Github repository with our code, if we don't already have. After that, we go to the ***Actions*** tab in our repository, and we press on ***New Workflow***. Then we click on ***set up a workflow ourself***. Next we get a page with an editor, where we write our Workflow file. The Workflow file is written in YAML. For those who don't know, YAML is a data-serialization language like XML and JSON, and mostly used for configuration files. Now let's get started with configuring our Workflow file. This is the code we need:

```
name: remote ssh command
on: [push]
jobs:

  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
    - name: executing remote ssh commands using password
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.KEY }}
        port: ${{ secrets.PORT }}
        script: |
          cd folder/
          git pull origin main
```
Here we have a simple workflow code. In the code, we connect to a host, and run git pull. The values for the host, username and other sensitive information is stored in the repository secrets. Repository secrets is a neat feature. It allows us to store sensitive information such as usernames, host addresses and passwords, without having them in our files. When the script is executed, they are automatically inserted. Now we need set up the Environment secrets. We go to the repository ***Settings***, ***Secrets***, ***Actions*** and we click on the ***New repository secret***. In here, we need four secret variables, host, username, key (the SSH private key for the server) and the SSH port. Now in the script section of the file we put our simple script, in this place I change the folder to the repo folder and then run git pull in there. But this can be used for anything we want, any command can be run from here, so we are not limited. Now all we have to do is press ***Start commit***. After this step, the Workflow file is commited to the repository and it will instantly try to Run. To check on the Workflow run log, we go to the ***Actions*** tab. If the run is sucessful there will be a ✅ and if the run failed there will be ❌. While the Workflow is still runing, 🟠 will appear.
