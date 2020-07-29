# kubernetes-operator-with-operator-sdk
### Why Operator
For stateless application normal kubernetes deployment is more than enough. But What about stateful applications like databases where we may need to add some business logic to restore them from where they were unavailable. Cluster scaling,
disaster recovery type jobs need human intervention as normal kubernetes deployment can't handle this. Operator is the perfect tool to automate these kind of jobs.

## What is an operator?
 
 An operator is nothing but a CRD asscociated with controller. 
 
## What is a CRD? 

CRD stands for Custom Resource Definition, It's basically a scheme. There's another resource named ```CR (Custom Resource)``` which is the json or yaml body format of our api. 

## Controller

Controller holds the business logic of pod/deployment/other kinds lifecycle management. Here developer can modify or enhance or incorporate their own logic of creating kubernetes objects.

## Type

This is the accepted struct of CR body. Developer can add attributes as their need and use those attributes while managing due kubernetes object.

## How many ways are there to create operator?

There's many ways of creating operator. Some of the standard ways are listed below,

```
With Go:
    - Using client go.
    - Using KubeBuilder
    - using Operator-sdk
With Java:
    - Using kubernetes Java client
    - using Fabric8
With python:
    - Using Kubernetes python client
Others options:
    - helm
    - ansible
```
We are using operator-sdk;operator SDK is a framework that uses the controller-runtime library to make writing Operators easier by providing:

 - High level APIs and abstractions to write the operational logic more   intuitively.
 - Tools for scaffolding and code generation to bootstrap a new project   fast.
 - Extensions to cover common Operator use cases.


### Install operator-sdk

#### Prerequisites

- [git](https://git-scm.com/downloads)
- [go](https://golang.org/dl/) version v1.13+.
- [mercurial](https://www.mercurial-scm.org/downloads) version 3.9+
- [bazaar](http://wiki.bazaar.canonical.com/Download) version 2.7.0+

- [docker](https://docs.docker.com/install/) version 17.03
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) version sourcev1.12.0+.
- Access to a Kubernetes v1.12.0+ cluster.
- Optional: [delve](https://github.com/go-delve/delve/tree/master/Documentation/installation) version 1.2.0+ (for up local --enable-delve).

#### Opetrator-sdk Linux installation:

Download the release binary 
```
$ RELEASE_VERSION=v0.14.0
$ curl -LO https://github.com/operator-framework/operator-sdk/releases/download/${RELEASE_VERSION}/operator-sdk-${RELEASE_VERSION}-x86_64-linux-gnu
```

Verify the downloaded release binary
```
$ RELEASE_VERSION=v0.14.0
$ curl -LO https://github.com/operator-framework/operator-sdk/releases/download/${RELEASE_VERSION}/operator-sdk-${RELEASE_VERSION}-x86_64-linux-gnu.asc
```
To verify a release binary using the provided asc files, place the binary and corresponding asc file into the same directory and use the corresponding command:
```
$ gpg --verify operator-sdk-${RELEASE_VERSION}-x86_64-linux-gnu.asc
```
If you do not have the maintainers public key on your machine, you will get an error message similar to this:

```
$ gpg --verify operator-sdk-${RELEASE_VERSION}-x86_64-apple-darwin.asc
$ gpg: assuming signed data in 'operator-sdk-${RELEASE_VERSION}-x86_64-apple-darwin'
$ gpg: Signature made Fri Apr  5 20:03:22 2019 CEST
$ gpg:                using RSA key <KEY_ID>
$ gpg: Can't check signature: No public key
```
To download the key, use the following command, replacing $KEY_ID with the RSA key string provided in the output of the previous command:

```
$ gpg --recv-key "$KEY_ID"
```
You'll need to specify a key server if one hasn't been configured. For example:

```
$ gpg --keyserver keyserver.ubuntu.com --recv-key "$KEY_ID"
```

Install the release binary in your PATH

```
$ chmod +x operator-sdk-${RELEASE_VERSION}-x86_64-linux-gnu && sudo mkdir -p /usr/local/bin/ && sudo cp operator-sdk-${RELEASE_VERSION}-x86_64-linux-gnu /usr/local/bin/operator-sdk && rm operator-sdk-${RELEASE_VERSION}-x86_64-linux-gnu
```

Compile and install from master

```
$ go get -d github.com/operator-framework/operator-sdk # This will download the git repository and not install it
$ cd $GOPATH/src/github.com/operator-framework/operator-sdk
$ git checkout master
$ make tidy
$ make install
```

## Steps to create an operator

```
$ operator-sdk new app-operator --repo github.com/example-inc/app-operator

$ cd app-operator
```
Operator-sdk will create an application with following Tree:
```
.
├── build
│   ├── bin
│   │   ├── entrypoint
│   │   └── user_setup
│   └── Dockerfile
├── cmd
│   └── manager
│       └── main.go
├── deploy
│   ├── operator.yaml
│   ├── role_binding.yaml
│   ├── role.yaml
│   └── service_account.yaml
├── go.mod
├── go.sum
├── pkg
│   ├── apis
│   │   └── apis.go
│   └── controller
│       └── controller.go
├── tools.go
└── version
    └── version.go

```

``` Operator-sdk new``` command created a Dockerfile and necessary scripts, a ```main.go```, ```deployment related menufests``` of an operator and others.


```
# Add a new API for the custom resource AppService
$ operator-sdk add api --api-version=app.example.com/v1alpha1 --kind=AppService
```
```
.
├── build
│   ├── bin
│   │   ├── entrypoint
│   │   └── user_setup
│   └── Dockerfile
├── cmd
│   └── manager
│       └── main.go
├── deploy
│   ├── crds
│   │   ├── app.example.com_appservices_crd.yaml
│   │   └── app.example.com_v1alpha1_appservice_cr.yaml
│   ├── operator.yaml
│   ├── role_binding.yaml
│   ├── role.yaml
│   └── service_account.yaml
├── go.mod
├── go.sum
├── pkg
│   ├── apis
│   │   ├── addtoscheme_app_v1alpha1.go
│   │   ├── apis.go
│   │   └── app
│   │       ├── group.go
│   │       └── v1alpha1
│   │           ├── appservice_types.go
│   │           ├── doc.go
│   │           ├── register.go
│   │           └── zz_generated.deepcopy.go
│   └── controller
│       └── controller.go
├── tools.go
└── version
    └── version.go

```
```Operator-sdk add api``` command generated CRD and CR menufests under ``` deploy/crd directory```, a go file named ``` appservice_types.go ``` under ``` pkg/apis/app/v1alpha1 ``` directroy. It also generated some other supportive go files.

Here ```appservice_types.go``` contains the struct of CR file ```app.example.com_v1alpha1_appservice_cr.yaml```.
```

# Add a new controller that watches for AppService
$ operator-sdk add controller --api-version=app.example.com/v1alpha1 --kind=AppService

```
```
.
├── build
│   ├── bin
│   │   ├── entrypoint
│   │   └── user_setup
│   └── Dockerfile
├── cmd
│   └── manager
│       └── main.go
├── deploy
│   ├── crds
│   │   ├── app.example.com_appservices_crd.yaml
│   │   └── app.example.com_v1alpha1_appservice_cr.yaml
│   ├── operator.yaml
│   ├── role_binding.yaml
│   ├── role.yaml
│   └── service_account.yaml
├── go.mod
├── go.sum
├── pkg
│   ├── apis
│   │   ├── addtoscheme_app_v1alpha1.go
│   │   ├── apis.go
│   │   └── app
│   │       ├── group.go
│   │       └── v1alpha1
│   │           ├── appservice_types.go
│   │           ├── doc.go
│   │           ├── register.go
│   │           └── zz_generated.deepcopy.go
│   └── controller
│       ├── add_appservice.go
│       ├── appservice
│       │   └── appservice_controller.go
│       └── controller.go
├── tools.go
└── version
    └── version.go



```

```operator-sdk add controller``` command generated a controller named ```appservice_controller.go``` under ```pkg/controller/appservice``` directory. This controller go file is going to contain the logic of our desire pod/deployment/statefulset or any other kubernetes object creation.

### Let's explore ```appservice_types.go```

Inside ```appservice_types.go``` file we will find structs like following,

```
type AppServiceSpec struct {

}

type AppServiceStatus struct {

}

type AppService struct {
	metav1.TypeMeta   `json:",inline"`
	metav1.ObjectMeta `json:"metadata,omitempty"`

	Spec   AppServiceSpec   `json:"spec,omitempty"`
	Status AppServiceStatus `json:"status,omitempty"`
}

```
We're going to add are needed atrributed in these struct. The ```CR``` file contains request body in ```AppService``` struct format.

Let's add a field ```Replicas``` in ```AppServiceStruct``` and ``` PodNames ``` in ```AppServiceStatus```

```
type AppServiceSpec struct {
  Replicas int32 `json:"replicas"`
}

type AppServiceStatus struct {
  PodNames []string `json:"podNames"`
}

```

### Let's explore ```AppService_Controller.go``` file

Inside this file we will get a function named ```Reconcile```. This is the function that gets triigered by kubernetes in every 10 seconds or some other threshold. So if we write logic, like if a deployment should run 3 replicas and if it's running 2 , we can run another replica.

```
func (r *ReconcileAppService) Reconcile(request reconcile.Request) (reconcile.Result, error) {
	reqLogger := log.WithValues("Request.Namespace", request.Namespace, "Request.Name", request.Name)
	reqLogger.Info("Reconciling PodSet")

	// Fetch the PodSet instance
	podSet := &appv1alpha1.PodSet{}
	err := r.client.Get(context.TODO(), request.NamespacedName, podSet)
	if err != nil {
		if errors.IsNotFound(err) {
			// Request object not found, could have been deleted after reconcile request.
			// Owned objects are automatically garbage collected. For additional cleanup logic use finalizers.
			// Return and don't requeue
			return reconcile.Result{}, nil
		}
		// Error reading the object - requeue the request.
		return reconcile.Result{}, err
	}

	// List all pods owned by this PodSet instance
	lbls := labels.Set{
		"app":     podSet.Name,
		"version": "v0.1",
	}
	existingPods := &corev1.PodList{}
	err = r.client.List(context.TODO(),
		existingPods,
		&client.ListOptions{
			Namespace:     request.Namespace,
			LabelSelector: labels.SelectorFromSet(lbls),
		})
	if err != nil {
		reqLogger.Error(err, "failed to list existing pods in the podSet")
		return reconcile.Result{}, err
	}
	existingPodNames := []string{}
	// Count the pods that are pending or running as available
	for _, pod := range existingPods.Items {
		if pod.GetObjectMeta().GetDeletionTimestamp() != nil {
			continue
		}
		if pod.Status.Phase == corev1.PodPending || pod.Status.Phase == corev1.PodRunning {
			existingPodNames = append(existingPodNames, pod.GetObjectMeta().GetName())
		}
	}

	reqLogger.Info("Checking podset", "expected replicas", podSet.Spec.Replicas, "Pod.Names", existingPodNames)
	// Update the status if necessary
	status := appv1alpha1.PodSetStatus{
		Replicas: int32(len(existingPodNames)),
		PodNames: existingPodNames,
	}
	if !reflect.DeepEqual(podSet.Status, status) {
		podSet.Status = status
		err := r.client.Status().Update(context.TODO(), podSet)
		if err != nil {
			reqLogger.Error(err, "failed to update the podSet")
			return reconcile.Result{}, err
		}
	}
	// Scale Down Pods
	if int32(len(existingPodNames)) > podSet.Spec.Replicas {
		// delete a pod. Just one at a time (this reconciler will be called again afterwards)
		reqLogger.Info("Deleting a pod in the podset", "expected replicas", podSet.Spec.Replicas, "Pod.Names", existingPodNames)
		pod := existingPods.Items[0]
		err = r.client.Delete(context.TODO(), &pod)
		if err != nil {
			reqLogger.Error(err, "failed to delete a pod")
			return reconcile.Result{}, err
		}
	}

	// Scale Up Pods
	if int32(len(existingPodNames)) < podSet.Spec.Replicas {
		// create a new pod. Just one at a time (this reconciler will be called again afterwards)
		reqLogger.Info("Adding a pod in the podset", "expected replicas", podSet.Spec.Replicas, "Pod.Names", existingPodNames)
		pod := newPodForCR(podSet)
		if err := controllerutil.SetControllerReference(podSet, pod, r.scheme); err != nil {
			reqLogger.Error(err, "unable to set owner reference on new pod")
			return reconcile.Result{}, err
		}
		err = r.client.Create(context.TODO(), pod)
		if err != nil {
			reqLogger.Error(err, "failed to create a pod")
			return reconcile.Result{}, err
		}
	}
	return reconcile.Result{Requeue: true}, nil
}

func newPodForCR(cr *appv1alpha1.AppService) *corev1.Pod {
	labels := map[string]string{
		"app": cr.Name,
	}
	return &corev1.Pod{
		ObjectMeta: metav1.ObjectMeta{
			Name:      cr.Name + "-pod",
			Namespace: cr.Namespace,
			Labels:    labels,
		},
		Spec: corev1.PodSpec{
			Containers: []corev1.Container{
				{
					Name:    "busybox",
					Image:   "busybox",
					Command: []string{"sleep", "3600"},
				},
			},
		},
	}
}
```

As ```Image``` ```busybox``` has been used in ```newPodForCR``` function. Here we can get the image name, args or env from ```CR``` by adding field in ```AppService``` struct.

Now, we're almost ready to build our operator. But before that as we've changed or added fields to our structs in ```appservice_types.go``` file, we need to run ```operator-sdk generate k8s``` command to update the ```pkg/apis/app/v1alpha1/zz_generated.deepcopy.go``` file accordingly.

### Build operator image from source code

```$ operator-sdk build zeromsi/app-operator```

This commad basically does two things,
- build target
- build image from target

So we can build image by our own docker file by building target then building image from that target using docker client.

Now run following command,

``` sed -i 's|REPLACE_IMAGE|klovercloud/app-operator|g' deploy/operator.yaml```

This will replace the image name of app-oprerator's deployment menufest which is named operator.yaml.

### Install app-operator or run operator inside you kubernetes/minikube cluster as deployment

```
# Setup Service Account
$ kubectl create -f deploy/service_account.yaml
# Setup RBAC
$ kubectl create -f deploy/role.yaml
$ kubectl create -f deploy/role_binding.yaml
# Setup the CRD
$ kubectl create -f deploy/crds/app.example.com_appservices_crd.yaml
# Deploy the app-operator
$ kubectl create -f deploy/operator.yaml

```

```| Note: If you registry is private add secret with service account ```

### Let's Explore the ```CR``` file (```app.example.com_v1alpha1_appservice_cr```) from deploy/crds directory.

```
apiVersion: app.example.com/v1alpha1
kind: AppService
metadata:
  name: example-appservice
spec:
  size: 3
```
We can see the generated ```CR``` spec contains a variable ```size``` but our  ```AppServiceSpec``` struct of ```AppService_types.go``` file contains a variable  ``` Replicas int32 `json:"replicas"```. So let's change the generated ```CR``` file. 


```
apiVersion: app.example.com/v1alpha1
kind: AppService
metadata:
  name: example-appservice
spec:
  replicas: 3
```
Create a Appservice,

```$ kubectl create -f deploy/crds/app.example.com_v1alpha1_appservice_cr.yaml```

```$ kubectl describe appservice example-appservice```

```

Name:         example-appservice
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  app.example.com/v1alpha1
Kind:         AppService
Metadata:
  Cluster Name:        
  Creation Timestamp:  2018-12-17T21:18:43Z
  Generation:          1
  Resource Version:    248412
  Self Link:           /apis/app.example.com/v1alpha1/namespaces/myproject/appservices/example-appservice
  UID:                 554f301f-0241-11e9-b551-080027c7d133
Spec:
  Size:  3

```
