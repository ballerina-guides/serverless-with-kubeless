# Serverless with Kubeless 

[Kubeless](http://kubeless.io) is a Kubernetes-native serverless framework that lets you deploy small bits of code (functions) without having to worry about the underlying infrastructure. It is designed to be deployed on top of a Kubernetes cluster and take advantage of all the great Kubernetes primitives. If you are looking for an open source serverless solution that clones what you can find on AWS Lambda, Azure Functions, and Google Cloud Functions, Kubeless is for you!

In this guide, you learn to setup Kubeless and write FaaS using Ballerina.

This guide contains the following sections.

- [What you'll build](#what-youll-build)
- [Prerequisites](#prerequisites)
- [Implementation](#implementation)
- [Deployment](#deployment)
- [Testing](#testing)
- [Troubleshooting](#Troubleshooting)

## What you'll build 

In this guide, you will build a simple Ballerina function that tweets payload and you will deploy that function on Kubeless serverless framework. 


## Prerequisites 

- [Ballerina Distribution](https://ballerina.io/learn/getting-started/) 
- [Kubeless](http://kubeless.io) 
- A Text Editor or an IDE

## Implementation

### Create Ballerina Function with configuration file

Ballerina function deployed in kubeless supports passing configurations via kubeless.toml file.

Create a folder named twitter and create a file named tweet_func.bal and paste the following code. This function sends a tweet using credentials in the config file.

> Important things to note: The function signature should match the following. ``` public function <FunctionName>(kubeless:Event event, kubeless:Context context) returns (string|error) ```


```ballerina
import kubeless/kubeless;
import wso2/twitter;
import ballerina/config;

public function tweet(kubeless:Event event, kubeless:Context context) returns (string|error) {

   endpoint twitter:Client tw {
       clientId: config:getAsString("clientId"),
       clientSecret: config:getAsString("clientSecret"),
       accessToken: config:getAsString("accessToken"),
       accessTokenSecret: config:getAsString("accessTokenSecret"),
       clientConfig: {}
   };
   twitter:Status st = check tw->tweet(event.data);
   return "Tweeted: " + st.text + "\n";
}
```

Create file name kubeless.toml in the same folder and paste the following. Replace the credentials with actual values.
```ballerina
# twitter config
clientId = "xxx"
clientSecret = "xxx"
accessToken = "xxx"
accessTokenSecret = "xxx"
```

```bash
twitter/
├── tweet_func.bal
└── kubeless.toml
```

Create a zip file including the above two files. 

zip -r -j twitter.zip twitter/

It is important to use `-j` option to avoid the zipping the folder when creating the zip file.

## Deployment

Pass the zip file to kubeless command.

```bash
$ kubeless function deploy twitter --runtime ballerina0.970.1 --from-file ./twitter.zip --handler tweet_func.tweet

INFO[0000] Deploying function...
INFO[0000] Function twitter submitted for deployment
INFO[0000] Check the deployment status executing 'kubeless function ls twitter'
```

Verify the function is deployed:

```bash
$ kubeless function ls twitter

NAME   	NAMESPACE	HANDLER         	RUNTIME         	DEPENDENCIES	STATUS
twitter	default  	tweet_func.tweet	ballerina0.970.1	            	1/1 READY
```

## Testing

### Invoke the function

```bash
$ kubeless function call twitter --data 'Hello world! from @kubeless_sh and @ballerinaplat'

Tweeted: Hello world! from @kubeless_sh and @ballerinaplat
```

The tweet `Hello world! from @kubeless_sh and @ballerinaplat` should be posted on twitter account.

### Deleting functions

Deployed functions can be viewed and delete using following kubeless command.

```bash
$ kubeless function ls

NAME	NAMESPACE	HANDLER  	RUNTIME         	DEPENDENCIES	STATUS
echo	default  	echo.echo	ballerina0.970.1	            	1/1 READY

$ kubeless function delete echo
```

## Troubleshooting

Following commands can be used for troubleshooting. 

```bash
$ kubectl get pods
NAME                       READY     STATUS        RESTARTS   AGE
echo-7f7b79fbd5-7ptdz      1/1       Running       0          58m

$ kubectl logs -c compile echo-7f7b79fbd5-7ptdz
total 20
-rw-r--r--    1 1000     troupe         204 Jun  7 05:26 echo.bal
drwxr-sr-x    2 1000     troupe        4096 Jun  7 05:26 kubeless
drwxr-sr-x    2 1000     troupe        4096 Jun  7 05:26 func
drwxr-xr-x    1 root     root          4096 Jun  7 05:26 ..
drwxrwsrwx    4 root     troupe        4096 Jun  7 05:26 .

$ kubectl logs echo-7f7b79fbd5-7ptdz
ballerina: initiating service(s) in 'kubeless_run.balx'
ballerina: started HTTP/WS endpoint 0.0.0.0:8090
2018/06/07 05:26:37
2018/06/07 05:26:37 10.1.0.1:40900 "GET /healthz HTTP/1.1" 200 kube-probe/1.9
```

These commands will print any errors occurred while starting the function.
