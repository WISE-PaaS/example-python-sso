# Example-python-SSO(Single Sign On)

This is WIES-PaaS iothub example-code include the sso and rabbitmq service。

[cf-introduce Training Video](https://advantech.wistia.com/medias/ll0ov3ce9e)

[SSO Training Video](https://advantech.wistia.com/medias/vay5uug5q6)

## Quick Start

cf-cli

[https://docs.cloudfoundry.org/cf-cli/install-go-cli.html](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html?source=post_page---------------------------)

python3

[https://www.python.org/downloads/](https://www.python.org/downloads/?source=post_page---------------------------)

![](https://cdn-images-1.medium.com/max/2000/1*iJwh3dROjmveF8x1rC6zag.png)

python3 package(those library you can try application in local):

    #python-backend
    pip3 install Flask

## Download this file

    git clone this repository

## Login to WISE-PaaS

First we need to login to the WISE-PaaS use cf login，and we need to chech out the domain name ex:wise-paas.io，and you need to have WISE-PaaS/EnSaaS account。

![Imgur](https://i.imgur.com/JNJmxFy.png)

    #cf login -skip-ssl-validation -a {api.domain_name}  -u "account" -p "password"

    cf login –skip-ssl-validation -a api.wise-paas.io -u xxxxx@advtech.com.tw -p xxxxxx

    #check the cf status
    cf target

## Application Introduce

#### index.py

This is a simple backend application use flask，you can run it use `python3 index.py` and listen on [localhost:3000](localhost:3000)，and the port can get the `3000` or port on WISE-PaaS。

If we run this code on localhost use `python3 index.py` go to `localhost:3000` you will get `hello world! iam in the local`，when we push application to the WISE-PaaS，we will send the `/templates/index.html` include the sso

```py
from flask import Flask
import os

app = Flask(__name__)

#port from cloud environment variable or localhost:3000
port = int(os.getenv("PORT", 3000))

@app.route('/')
def hello_world():
    if(port==3000):
        return 'hello world! iam in the local'
    elif(port==int(os.getenv("PORT"))):
        return render_template('index.html')


if __name__ == '__main__':
    # Run the app, listening on all IPs with our chosen port number
    app.run(host='0.0.0.0', port=port)

```

#### requirements.txt

Thie file help buildpack download the package for our application in WISE-PaaS。

```
Flask

```

#### mainfest config

This file can define our application config

- name:application name
- memory:how much memory we give to application
- disk_quota:how much disk we give to application
- buildpack:help us compile when we push our application to WISE-PaaS，you can list it use `cf buildpacks`
- command:command to start our application

open **`manifest.yml`** and editor the **application name** to yours，because the appication can't duplicate。

```yml
---
applications:
  #application name
  - name: python-demo-{your name}
    #memory you want to give to appliaction
    memory: 256MB
    #disk you want to give to appliaction
    disk_quota: 256MB
    #help use compile the file when you push to cloud
    buildpack: python_buildpack
    #let the backend application begin。
    command: python index.py
```

## SSO(Single Sign On)

This is the [sso](https://advantech.wistia.com/medias/vay5uug5q6) applicaition，open **`templates/index.html`** and editor the `ssoUrl` to your application name。

    #change this **`python-demo-try`** to your **application name**
    var ssoUrl = myUrl.replace('python-demo-try', 'portal-sso');

We define the three button to `signIn` `signOut` and also design a button jump to the `management portal`，if we already login the `ajax` will keep our credential token in cookie one hours。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>SSO Tutorial</title>
  </head>

  <body>
    <h1>hello</h1>
    <div id="demo"></div>

    <button class="btn btn-primary" id="signInBtn" style="display: none;">
      Sign in
    </button>
    <button class="btn btn-primary" id="signOutBtn" style="display: none;">
      Sign out
    </button>
    <button class="btn btn-primary" id="management" style="display: none;">
      Management Portal
    </button>
    <h1 id="helloMsg"></h1>

    <script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
    <script src=""></script>
  </body>
  <script>
    $(document).ready(function() {
      var myUrl = window.location.protocol + "//" + window.location.hostname;
      var ssoUrl = myUrl.replace("python-demo-try", "portal-sso");
      var manageUrl = "https://portal-management.wise-paas.io/organizations";
      document.getElementById("demo").innerHTML = myUrl;

      $("#signInBtn").click(function() {
        window.location.href = ssoUrl + "/web/signIn.html?redirectUri=" + myUrl;
      });

      $("#signOutBtn").click(function() {
        window.location.href =
          ssoUrl + "/web/signOut.html?redirectUri=" + myUrl;
      });
      $("#management").click(function() {
        window.location.href = manageUrl;
      });

      $.ajax({
        url: ssoUrl + "/v2.0/users/me",
        method: "GET",
        xhrFields: {
          withCredentials: true
        }
      })
        .done(function(user) {
          $("#signOutBtn").show();
          $("#management").show();
          $("#helloMsg").text(
            "Hello, " + user.firstName + " " + user.lastName + "!"
          );
        })
        .fail(function(jqXHR, textStatus, errorThrown) {
          $("#signInBtn").show();

          $("#helloMsg").text("Hi, please sign in first.");
        });
    });
  </script>
</html>
```

#### push application to your space

    #cf push {application name}
    cf push python-demo-try

![Imgur](https://i.imgur.com/ZjrjuTW.png)

#### Check our application in WISE-PaaS Application List

Choose your organization and space which you login with `cf login`，and find you application

![Imgur](https://i.imgur.com/jt4bf5U.png)

**No Login**

![Imgur](https://i.imgur.com/9GDgy8z.png)

**Login**

![Imgur](https://i.imgur.com/oN5C35x.png)
