# Setting Up a Tortoise Webcam Stream with OpenShift: Hairee's Home on the Web

## Introduction

Have you ever wanted to set up a live stream of your pet for the world to see? In this blog post, we'll walk through the process of creating a website that streams live video of Hairee, an 8-year-old Russian tortoise, using OpenShift. This project combines containerization, streaming technology, and web development to create a unique and educational experience.

## Project Overview

We'll be creating a system with the following components:

1. An NGINX RTMP server to handle the video stream
2. A web server to host our website
3. OpenShift configurations to deploy and manage our services
4. Let's Encrypt integration for HTTPS security

## Prerequisites

- An OpenShift cluster (we're using a home setup)
- Access to your router for port forwarding
- A domain name pointing to your home IP address
- Basic knowledge of Docker, YAML, and Kubernetes/OpenShift concepts

## Step-by-Step Guide

### 1. Set Up the NGINX RTMP Server

First, we need to create a Dockerfile for our NGINX RTMP server:

```dockerfile
FROM debian:buster-slim

# Install dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    libpcre3-dev \
    libssl-dev \
    zlib1g-dev \
    wget

# Download and compile NGINX with RTMP module
RUN wget http://nginx.org/download/nginx-1.21.0.tar.gz && \
    tar zxf nginx-1.21.0.tar.gz && \
    rm nginx-1.21.0.tar.gz && \
    wget https://github.com/arut/nginx-rtmp-module/archive/master.zip && \
    unzip master.zip && \
    rm master.zip && \
    cd nginx-1.21.0 && \
    ./configure --with-http_ssl_module --add-module=../nginx-rtmp-module-master && \
    make && \
    make install

EXPOSE 1935
EXPOSE 8080

CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]
```

### 2. Create the Website

Next, create an HTML file for our website:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hairee's Tortoise Cam - Watch Our Russian Friend Live!</title>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/video.js/7.10.2/video-js.min.css" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/video.js/7.10.2/video.min.js"></script>
    <!-- Add your styles here -->
</head>
<body>
    <header>
        <h1>Hairee's Tortoise Cam</h1>
        <p class="subtitle">Watch Our Russian Friend Live!</p>
    </header>
    <div class="container">
        <div id="video-container">
            <video-js id="hairee-video" class="vjs-default-skin" controls preload="auto" width="640" height="480">
                <source src="http://your-openshift-route/hls/stream.m3u8" type="application/x-mpegURL">
            </video-js>
        </div>
        <!-- Add more content about Hairee here -->
    </div>
    <script>
        var player = videojs('hairee-video');
    </script>
</body>
</html>
```

### 3. Configure OpenShift Deployment

Create a YAML file (`openshift-deployments.yaml`) with all necessary OpenShift resources:

```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    # Your NGINX configuration here

---
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: nginx-rtmp
spec:
  # NGINX RTMP deployment configuration

---
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: hairee-cam
spec:
  # Web server deployment configuration

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-rtmp
spec:
  # NGINX RTMP service configuration

---
apiVersion: v1
kind: Service
metadata:
  name: hairee-cam
spec:
  # Web server service configuration

---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: hairee-cam
spec:
  # Web server route configuration

---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: nginx-rtmp
spec:
  # NGINX RTMP route configuration

---
apiVersion: image.openshift.io/v1
kind: ImageStream
metadata:
  name: nginx-rtmp

---
apiVersion: image.openshift.io/v1
kind: ImageStream
metadata:
  name: hairee-cam

---
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: nginx-rtmp
spec:
  # NGINX RTMP build configuration

---
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: hairee-cam
spec:
  # Web server build configuration
```

### 4. Set Up Let's Encrypt for HTTPS

Install the cert-manager operator and create a ClusterIssuer:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-cluster-issuer
spec:
  acme:
    email: your-email@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-cluster-issuer-key
    solvers:
    - http01:
        ingress:
          class: openshift-default
```

Update your Routes to use Let's Encrypt:

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: hairee-cam
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-cluster-issuer"
spec:
  # ... other configurations ...
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
```

### 5. Deploy the Project

1. Apply the OpenShift configurations:
   ```
   oc apply -f openshift-deployments.yaml
   ```

2. Start the builds:
   ```
   oc start-build nginx-rtmp
   oc start-build hairee-cam
   ```

3. Configure your home router to forward ports 80 and 443 to your OpenShift cluster.

4. Update your DNS settings to point your domain to your home IP address.

## Conclusion

And there you have it! You've now set up a live streaming website for Hairee the tortoise using OpenShift. This project demonstrates the power of containerization and Kubernetes-based platforms in creating complex, scalable web applications.

Remember to monitor your application, keep your software updated, and most importantly, make sure Hairee is happy with his newfound internet fame!

Happy streaming!
