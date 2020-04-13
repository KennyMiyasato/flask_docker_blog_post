# Build And Deploy A Simple Flask App Using Docker

The tutorial assumes you have a basic understanding of the following topics:
1. Flask
2. BASH or Powershell
3. Docker

This tutorial is heavily influenced by: [Michael Okoh - digitalocean.com](https://www.digitalocean.com/community/tutorials/how-to-build-and-deploy-a-flask-application-using-docker-on-ubuntu-18-04). If you'd like to follow along please visit my [github page](https://github.com/KennyMiyasato/flask_docker_blog_post)

# The Project Folder Structure In Tree View
```
+---flask_docker_blog_post
|   |   .gitignore
|   |   README.md
|   |   
|   +---var
|   |   \---www
|   |       \---HelloWorldApp
|   |           |   Dockerfile
|   |           |   main.py
|   |           |   requirements.txt
|   |           |   start.ps1
|   |           |   start.sh
|   |           |   uwsgi.ini
|   |           |   
|   |           +---app
|   |           |   |   views.py
|   |           |   |   __init__.py
|   |           |   |   
|   |           |   +---static
|   |           |   +---templates
|   |           |   |       home.html
|   |           |   |       
|   |           |           
|   |                   
|           
```

## Running the Flask App Locally
- As we can see from the folder structure, the Flask app will be stored in `var/www`, the name of the app is called `HelloWorldApp`.
- To test to see if the flask app is working locally, we can run the app. To do this we first will need to set the `FLASK_APP` environment variable to be `main.py`
- In Unix/Linux we can do the following: `$ export FLASK_APP=main.py` then `$ flask run`
- In Windows we will use cmd: `C:\path\to\app>set FLASK_APP=main.py` then `C:\path\to\app>flask run`
- Once flask is running, we can navigate to `127.0.0.1:5000` on our browser of choice and see a simple string message `hello world!`
- If we navigate to `127.0.0.1:5000/template` we could also see that our routes are working. Our routes are in `../HelloWorldApp/app/views.py`. We can see that it will render the template called `home.html` which lives in `../HelloWorldApp/app/templates/home.html`

## The `../HelloWorldApp/uswsgi.ini` file
> uWSGI is a software application that "aims at developing a full stack for building hosting services" It is named after the Web Server Gateway Interface (WSGI), which was the first plugin supported by the project
> 
> uWSGI is often used for serving Python web applications in conjunction with web servers such as Cherokee and Nginx, which offer direct support for uWSGI's native uwsgi protocol
> 
> [-wikipedia](https://en.wikipedia.org/wiki/UWSGI)

The reason we need the `uswsgi.ini` is because we will be using [Nginx](https://en.wikipedia.org/wiki/Nginx) as our web server. The file contains the following:
```
[uwsgi]
module = main
callable = app
```
`module = main` means that it will be using the `main.py` file.

`callable = app` means that it will be using the name `app` given from our `main.py` file.

## The  `../HelloWorldApp/requirements.txt` file
This file tells Docker which Flask version we want installed.
```
Flask==1.1.1
```

## The  `../HelloWorldApp/Dockerfile` file
```
FROM tiangolo/uwsgi-nginx-flask:python3.6-alpine3.7
RUN apk --update add bash nano
ENV STATIC_URL /static
ENV STATIC_PATH /var/www/app/static
COPY ./requirements.txt /var/www/requirements.txt
RUN pip install -r /var/www/requirements.txt
```

`FROM tiangolo/uwsgi-nginx-flask:python3.6-alpine3.7` 
- We are telling docker to go to [dockerhub](https://hub.docker.com/r/tiangolo/uwsgi-nginx-flask) and use [this specific image](https://hub.docker.com/layers/tiangolo/uwsgi-nginx-flask/python3.6-alpine3.7/images/sha256-6d335e7ef2899c9a468b698020090e266ca24808ffcbf999a92796e2506970b1?context=explore). We can also look at what the Dockerfile contains by going to [the github repo](https://github.com/tiangolo/uwsgi-nginx-flask-docker/blob/master/python3.6-alpine3.7/Dockerfile)

`RUN apk --update add bash nano` 
- We are telling docker to install/update `bash nano`

`ENV STATIC_URL /static` and `ENV STATIC_PATH /var/www/app/static` 
- This is to map the environment variable `STATIC_URL` and `STATIC_PATH` to the location of our static folder.

`COPY ./requirements.txt /var/www/requirements.txt` 
- As the name describes, it will copy the `./requirements.txt` file to `var/www/requirements.txt`

`RUN pip install -r /var/www/requirements.txt` 
- It will run pip install followed by the dependencies, which would be `Flask==1.1.1`

## The `../HelloWorldApp/start.sh` or `../HelloWorldApp/start.ps1` files
These are the files that we will use to start our docker container locally. As described in the files names, `.sh` is for Unix/Linux and `.ps1` is for Windows using Powershell.

For this example, we will use the powershell file.
```
$app = 'docker.test'
$location = $PWD.ToString() + ':\app'
docker build -t $app .
docker run -d -p 56733:80 --name=$app -v $location $app
```

`$app = 'docker.test'`
- Creates a variable called `$app` and assigns the string `docker.test`
  
`$location = $PWD.ToString() + ':\app'`
- Created a variable called `$location` and assigns the current directory plus `:app`.
- Example: `$location` is equal to `C:\Users\YourName\flask_docker_blog_post\var\www\HelloWorldApp\:app`

`docker build -t $app .`
- Will build the docker container

`docker run -d -p 56733:80 --name=$app -v $location $app`
- Will run the docker image
- `-d` flag to run as daemon mode
- `-p` flag to host on the port 56733:80 (http://localhost:56733)
- `--name` name of the container `docker.test`
- `-v` volume, or the location where the flask app is

To run the powershell file, open up a powershell window and navigate to the folder where it lives. `C:\..\HelloWorldApp\` then type `. .\start.ps1`

The output should look something like this:
```
PS C:\Users\YourName\flask_docker_blog_post\var\www\HelloWorldApp> . .\start.ps1
Sending build context to Docker daemon  14.85kB
Step 1/6 : FROM tiangolo/uwsgi-nginx-flask:python3.6-alpine3.7
python3.6-alpine3.7: Pulling from tiangolo/uwsgi-nginx-flask
48ecbb6b270e: Pull complete                                                                                                                                                                                       692f29ee68fa: Pull complete                                                                                                                                                                                       f75fc7ac1098: Pull complete                                                                                                                                                                                       c30e40bb471c: Pull complete                                                                                                                                                                                       51a8cc25b36b: Pull complete                                                                                                                                                                                       5511896959f4: Pull complete                                                                                                                                                                                       d098e5464cae: Pull complete                                                                                                                                                                                       53a171e4bb73: Pull complete                                                                                                                                                                                       a6f4f7a827fa: Pull complete                                                                                                                                                                                       7155f8ba9e32: Pull complete                                                                                                                                                                                       86334b930d78: Pull complete                                                                                                                                                                                       31fa892d32c9: Pull complete                                                                                                                                                                                       a3eee99e8cf1: Pull complete                                                                                                                                                                                       adb3afca65f2: Pull complete                                                                                                                                                                                       0a4278caa37b: Pull complete                                                                                                                                                                                       fb3c0b7eba8c: Pull complete                                                                                                                                                                                       9c5ace36ebd4: Pull complete                                                                                                                                                                                       6d80169bc246: Pull complete                                                                                                                                                                                       aaedb05ae009: Pull complete                                                                                                                                                                                       4892d5572688: Pull complete                                                                                                                                                                                       Digest: sha256:6d335e7ef2899c9a468b698020090e266ca24808ffcbf999a92796e2506970b1
Status: Downloaded newer image for tiangolo/uwsgi-nginx-flask:python3.6-alpine3.7
 ---> 2437bfef1667
Step 2/6 : RUN apk --update add bash nano
 ---> Running in 3137521b188d
fetch http://dl-cdn.alpinelinux.org/alpine/v3.7/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.7/community/x86_64/APKINDEX.tar.gz
(1/4) Installing pkgconf (1.3.10-r0)
(2/4) Installing bash (4.4.19-r1)
Executing bash-4.4.19-r1.post-install
(3/4) Installing libmagic (5.32-r2)
(4/4) Installing nano (2.9.1-r0)
Executing busybox-1.27.2-r11.trigger
OK: 135 MiB in 62 packages
Removing intermediate container 3137521b188d
 ---> 0482d3c8a3ee
Step 3/6 : ENV STATIC_URL /static
 ---> Running in 175a15a1370c
Removing intermediate container 175a15a1370c
 ---> 676004597d60
Step 4/6 : ENV STATIC_PATH /var/www/app/static
 ---> Running in 2a3bd6d049e8
Removing intermediate container 2a3bd6d049e8
 ---> 3b1cc23dab71
Step 5/6 : COPY ./requirements.txt /var/www/requirements.txt
 ---> d626f19da4a8
Step 6/6 : RUN pip install -r /var/www/requirements.txt
 ---> Running in 6925639b6da0
Collecting Flask==1.1.1 (from -r /var/www/requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/9b/93/628509b8d5dc749656a9641f4caf13540e2cdec85276964ff8f43bbb1d3b/Flask-1.1.1-py2.py3-none-any.whl (94kB)
Requirement already satisfied: Jinja2>=2.10.1 in /usr/local/lib/python3.6/site-packages (from Flask==1.1.1->-r /var/www/requirements.txt (line 1)) (2.11.1)
Requirement already satisfied: click>=5.1 in /usr/local/lib/python3.6/site-packages (from Flask==1.1.1->-r /var/www/requirements.txt (line 1)) (7.1.1)
Requirement already satisfied: Werkzeug>=0.15 in /usr/local/lib/python3.6/site-packages (from Flask==1.1.1->-r /var/www/requirements.txt (line 1)) (1.0.1)
Requirement already satisfied: itsdangerous>=0.24 in /usr/local/lib/python3.6/site-packages (from Flask==1.1.1->-r /var/www/requirements.txt (line 1)) (1.1.0)
Requirement already satisfied: MarkupSafe>=0.23 in /usr/local/lib/python3.6/site-packages (from Jinja2>=2.10.1->Flask==1.1.1->-r /var/www/requirements.txt (line 1)) (1.1.1)
Installing collected packages: Flask
  Found existing installation: Flask 1.1.2
    Uninstalling Flask-1.1.2:
      Successfully uninstalled Flask-1.1.2
Successfully installed Flask-1.1.1
You are using pip version 19.0.1, however version 20.0.2 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
Removing intermediate container 6925639b6da0
 ---> 9a388c451977
Successfully built 9a388c451977
Successfully tagged docker.test:latest
SECURITY WARNING: You are building a Docker image from Windows against a non-Windows Docker host. All files and directories added to build context will have '-rwxr-xr-x' permissions. It is recommended to double check and reset permissions for sensitive files and directories.
4e8ebdb8ba31da6de872a7a16e38e7eeafd90282daf45adfe637aff207be1ef7
```

We can run the following commands to make sure our container is up and running:
`docker ps -a`

```
PS C:\Users\YourName\flask_docker_blog_post\var\www\HelloWorldApp> docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
f80ba8889271        docker.test         "/entrypoint.sh /sta…"   9 seconds ago       Up 8 seconds        443/tcp, 0.0.0.0:56733->80/tcp   docker.test
```

## Navigate To Our Newly Created Docker Container
Once all of that is finished we can go to our web brower and type the following: `http://localhost:56733`

## Congrats! We're Running Our First Docker Container!

### Resources:
- https://www.digitalocean.com/community/tutorials/how-to-build-and-deploy-a-flask-application-using-docker-on-ubuntu-18-04
- https://en.wikipedia.org/wiki/UWSGI
- https://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html
- https://en.wikipedia.org/wiki/Nginx
- https://flask.palletsprojects.com/en/1.1.x/quickstart/


