Ruby Sample App on the Digital Garage
============================

This is a basic ruby application for the Digital Garage that you can use as a starting point to develop your own application and deploy it on [the Digital Garage](http://www.thedigitalgarage.io).

If you'd like to install it, follow [these directions](https://github.com/thedigitalgarage/ruby-ex/blob/master/README.md#installation).  

The steps in this document assume that you have access to an the Digital Garage deployment that you can deploy applications on.

### Installation:

1. Fork a copy of [ruby-ex](https://github.com/thedigitalgarage/ruby-ex)
2. Add a Ruby application from your new repository:

		$ oc new-app openshift/ruby-20-centos7~https://github.com/< yourusername >/ruby-ex

3. A build should start immediately.  To run another build, run:

		$ oc start-build ruby-ex

4. Once the build is running, watch your build progress:  

		$ oc logs build/ruby-ex-1

5. Wait for ruby-ex pods to start up (this can take a few minutes):  

		$ oc get pods -w


	Sample output:  

		NAME               READY     STATUS       RESTARTS   AGE
		ruby-ex-1-build    0/1       ExitCode:0   0          2m
		ruby-ex-1-deploy   1/1       Running      0          25s
		ruby-ex-1-hrek2    1/1       Running      0          17s


6. Check the IP and port the ruby-ex service is running on:  

		$ oc get svc


	Sample output:  

		NAME      CLUSTER_IP      EXTERNAL_IP   PORT(S)    SELECTOR                   AGE
		ruby-ex   172.30.97.209   <none>        8080/TCP   deploymentconfig=ruby-ex   2m


In this case, the IP for ruby-ex is 172.30.97.209 and it is on port 8080.  
*Note*: you can also get this information from the web console.

### Adding Webhooks and Making Code Changes
Since the Digital Garage does not provide a git repository out of the box, you can configure your github repository to make a webhook call whenever you push your code.

1. From the console navigate to your project.  
2. Click on Browse > Builds  
3. From the view for your Build click on the link to display your GitHub webhook and copy the url.  
4. Navigate to your repository on GitHub and click on repository settings > webhooks  
5. Paste your copied webhook url provided by the Digital Garage - Thats it!  
6. After you save your webhook, if you refresh your settings page you can see the status of the ping that Github sent to the Digital Garage to verify it can reach the server.

### Optimizing the Build Configuration to run inside the free resource pool for Builds and Deployments.
The Digital Garage has created a free resource pool for Builds and Deployments (compute-resources-time-bound) inside your project. This free resource pool allows you to run both Builds and Deployments in a large resource pool without effecting your project quota for the long-running pods and containers. Unless your build was initiated from a pre-built Digital Garage template, the Build Configuration for this application may not be configured out-of-the-box to run builds in this resource pool.

In order to configure your Build to run inside the time-bound resource pool you can do the following:

1. In the "Builds" sidebar menu item, choose the build configuration for the ruby application you just built.
2. In the additional actions menu on the top, right side the Builds >> Summary page, (three dots in a vertical column) choose "Edit YAML" from the dropdown menu.
3. Find the heading `spec:` in the YAML file. Directly under `spec:` add the following `completionDeadlineSeconds: 900` as in the following example.
**WARNING** This is YAML so be careful about tabs and spacing.

```YAML
apiVersion: v1
kind: BuildConfig
metadata:
  name: test-ruby
  namespace: jm-demo-e6fe9f27
  selfLink: /oapi/v1/namespaces/jm-demo-e6fe9f27/buildconfigs/test-ruby
  uid: 1b831ccd-31c1-11e7-9899-226411292668
  resourceVersion: '11481185'
  creationTimestamp: '2017-05-05T18:31:51Z'
  labels:
    app: test-ruby
  annotations:
    openshift.io/generated-by: OpenShiftWebConsole
spec:
  completionDeadlineSeconds: 900
  triggers:
    -
      type: Generic
      generic:
        secret: f7c7ae3872af9476
    -
      type: GitHub
      github:
        secret: 4a79aad73414e073
    -
      type: ImageChange
      imageChange:
        lastTriggeredImageID: 'centos/ruby-23-centos7@sha256:462084074bae6465c9be294ea2b3d09d0f73181f7c19bbd94078128e19fdd01c'
    -
      type: ConfigChange
  runPolicy: Serial
  source:
    type: Git
    git:
      uri: 'https://github.com/thedigitalgarage/ruby-ex.git'
      ref: master
  strategy:
    type: Source
    sourceStrategy:
      from:
        kind: ImageStreamTag
        namespace: openshift
        name: 'ruby:2.3'
  output:
    to:
      kind: ImageStreamTag
      name: 'test-ruby:latest'
  resources:
  postCommit:
status:
  lastVersion: 2
```
4. Save your edits.
5. The next time you "Start Build" the build will run in the free time-bound pool. 



### License
This code is dedicated to the public domain to the maximum extent permitted by applicable law, pursuant to [CC0](http://creativecommons.org/publicdomain/zero/1.0/).
