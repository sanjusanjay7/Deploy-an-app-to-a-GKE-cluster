# Deploy-an-app-to-a-GKE-cluster
In this quickstart, you deploy a simple web server containerized application to a Google Kubernetes Engine (GKE) cluster. You will learn how to create a cluster, and how to deploy the application to the cluster so that it can be accessed by users.

This quickstart assumes a basic understanding of Kubernetes.

##Before you begin
Take the following steps to enable the Kubernetes Engine API:
In the Google Cloud console, on the project selector page, select or create a Google Cloud project.

Note: If you don't plan to keep the resources that you create in this procedure, create a project instead of selecting an existing project. After you finish these steps, you can delete the project, removing all resources associated with the project.
Go to project selector

Make sure that billing is enabled for your Google Cloud project. Learn how to check if billing is enabled on a project.

Enable the Artifact Registry and Google Kubernetes Engine APIs.

Enable the APIs

##Launch Cloud Shell
In this tutorial you will use Cloud Shell, which is a shell environment for managing resources hosted on Google Cloud.

Cloud Shell comes preinstalled with the Google Cloud CLI and kubectl command-line tool. The gcloud CLI provides the primary command-line interface for Google Cloud, and kubectl provides the primary command-line interface for running commands against Kubernetes clusters.

##Launch Cloud Shell:

Go to the Google Cloud console.

Google Cloud console

From the upper-right corner of the console, click the Activate Cloud Shell button: 

A Cloud Shell session opens inside a frame lower on the console. You use this shell to run gcloud and kubectl commands. Before you run commands, set your default project in the Google Cloud CLI using the following command:


***gcloud config set project PROJECT_ID***
1.Replace PROJECT_ID with your project ID.

2.Create a GKE cluster
A cluster consists of at least one cluster control plane machine and multiple worker machines called nodes. Nodes are Compute Engine virtual machine (VM) instances that run the Kubernetes processes necessary to make them part of the cluster. You deploy applications to clusters, and the applications run on the nodes.

3.Create an Autopilot cluster named hello-cluster:


***gcloud container clusters create-auto hello-cluster \
    --location=us-central1***
Note: It might take several minutes to finish creating the cluster.
4.Get authentication credentials for the cluster
After creating your cluster, you need to get authentication credentials to interact with the cluster:


***gcloud container clusters get-credentials hello-cluster \
    --location us-central1***
This command configures kubectl to use the cluster you created.

5.Deploy an application to the cluster
Now that you have created a cluster, you can deploy a containerized application to it. For this quickstart, you can deploy our example web application, hello-app.

GKE uses Kubernetes objects to create and manage your cluster's resources. Kubernetes provides the Deployment object for deploying stateless applications like web servers. Service objects define rules and load balancing for accessing your application from the internet.

7.Create the Deployment
To run hello-app in your cluster, you need to deploy the application by running the following command:


***kubectl create deployment hello-server \
    --image=us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0***
This Kubernetes command, kubectl create deployment, creates a Deployment named hello-server. The Deployment's Pod runs the hello-app container image.

In this command:

--image specifies a container image to deploy. In this case, the command pulls the example image from an Artifact Registry repository, us-docker.pkg.dev/google-samples/containers/gke/hello-app. :1.0 indicates the specific image version to pull. If you don't specify a version, the image with the default tag latest is used.
Expose the Deployment
After deploying the application, you need to expose it to the internet so that users can access it. You can expose your application by creating a Service, a Kubernetes resource that exposes your application to external traffic.

8.To expose your application, run the following kubectl expose command:


***kubectl expose deployment hello-server \
    --type LoadBalancer \
    --port 80 \
    --target-port 8080***
Passing in the --type LoadBalancer flag creates a Compute Engine load balancer for your container. The --port flag initializes public port 80 to the internet and the --target-port flag routes the traffic to port 8080 of the application.

Load balancers are billed per Compute Engine's load balancer pricing.

Inspect and view the application
9.Inspect the running Pods by using kubectl get pods:


***kubectl get pods***
You should see one hello-server Pod running on your cluster.

10.Inspect the hello-server Service by using kubectl get service:


***kubectl get service hello-server***
From this command's output, copy the Service's external IP address from the EXTERNAL-IP column.

Note: You might need to wait several minutes before the Service's external IP address populates. If the application's external IP is <pending>, run kubectl get again.
11.View the application from your web browser by using the external IP address with the exposed port:


***http://EXTERNAL_IP***
You have just deployed a containerized web application to GKE.

##Clean up
To avoid incurring charges to your Google Cloud account for the resources used on this page, follow these steps.

##Delete the application's Service by running kubectl delete:


##kubectl delete service hello-server
This command deletes the Compute Engine load balancer that you created when you exposed the Deployment.

Delete your cluster by running gcloud container clusters delete:


***gcloud container clusters delete hello-cluster \
    --location us-central1***
##Optional: hello-app code review
hello-app is a simple web server application that consists of two files: main.go and a Dockerfile.

hello-app is packaged as a Docker container image. Container images are stored in any Docker image registry, such as Artifact Registry. We host hello-app in a Artifact Registry repository at us-docker.pkg.dev/google-samples/containers/gke/hello-app.

main.go
Dockerfile
main.go is a web server implementation written in the Go programming language. The server responds to any HTTP request with a "Hello, world!" message.

hello-app/main.goView on GitHub

package main

import (
        "fmt"
        "log"
        "net/http"
        "os"
)

func main() {
        // register hello function to handle all requests
        mux := http.NewServeMux()
        mux.HandleFunc("/", hello)

        // use PORT environment variable, or default to 8080
        port := os.Getenv("PORT")
        if port == "" {
                port = "8080"
        }

        // start the web server on port and accept requests
        log.Printf("Server listening on port %s", port)
        log.Fatal(http.ListenAndServe(":"+port, mux))
}

// hello responds to the request with a plain-text "Hello, world" message.
func hello(w http.ResponseWriter, r *http.Request) {
        log.Printf("Serving request: %s", r.URL.Path)
        host, _ := os.Hostname()
        fmt.Fprintf(w, "Hello, world!\n")
        fmt.Fprintf(w, "Version: 1.0.0\n")
        fmt.Fprintf(w, "Hostname: %s\n", host)
}
