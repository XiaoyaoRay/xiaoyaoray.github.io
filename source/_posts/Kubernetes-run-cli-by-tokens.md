---
title: 使用服务账号token配置kubernetes的运行命令
date: 2018-10-22 11:40:29
categories:
- Kubernetes
tags:
- Kubernetes
---

## 转载：[Configuring the Kubernetes CLI by using service account tokens](https://www.ibm.com/developerworks/community/blogs/fe25b4ef-ea6a-4d86-a629-6f87ccf4649e/entry/Configuring_the_Kubernetes_CLI_by_using_service_account_tokens1?lang=en)

#### Service account tokens

You can obtain a user token from the IBM Cloud Private management console. This  user token can be used by kubectl to authenticate against the Kubernetes API server. Once you are authenticated, you can then access your cluster from the command line (CLI). However, this token has an expiration time of 12 hours, which is not suitable for long running services.

Processes that are running inside a container, have a different mechanism for communicating with the Kubernetes API server. To facilitate this communication, authentication is done through a token known as a service account.

For more information about service accounts in Kubernetes, see <https://kubernetes.io/docs/user-guide/service-accounts/>.

 

For long running services we can use service account to access the Kubernetes API server, which allows access to the CLI for extended periods of time. 

 

While working with services in IBM Cloud Private, two methods are available for obtaining service account tokens: 

- If a long running service is spawned as a pod in a IBM Cloud Private cluster, the service account token is mounted onto the pod. You can use this service account token that is available in the pod to access the API server.
- If a long running  service is not available inside your IBM Cloud Private cluster, you can get the service account by using kubectl and the user token that is available from the management console.

For more information about installing kubectl and obtaining the user token from the IBM Cloud Private management console, see [Accessing your IBM® Cloud Private cluster by using the kubectl CLI](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0/manage_cluster/cfc_cli.html).

 

### Option 1: Working with service account tokens obtained from a running IBM Cloud Privatepod

------

 

#### 1. Locate the service account tokens

When a long running service is launched in the IBM Cloud Private cluster, the service account is mounted automatically. The directory that is used for mounting the service account is  /var/run/secrets/kubernetes.io/serviceaccount. The following three files are stored in this mounted directory:

- ca.crt - the certificate file needed for https access. 
- namespace - the namespace scope of the associated token. In IBM Cloud Private, non-admin users are only authorized to access resources in their own namespace.
- token - the service account token used for authentication.

The API service endpoint is kubernetes.default. The API service endpoint can also be obtained from the KUBERNETES_SERVICE_HOST environment variable.

 

#### 2. Connect to the Kubernetes API server

**Method 1**

To connect to the Kubernetes API server using the service account , issue the following command:

```
curl --cacert ca.crt -H "Authorization: Bearer {token}" [https://kubernetes.default/api/v1/pod/namespaces/{namespace}]
```

**Method 2**
If  kubectl is installed in the pod, you can also set kubectl to connect to the API server as follows:

 ```
kubectl config set-cluster cfc --server=https://kubernetes.default --certificate-authority=ca.crt
kubectl config set-context cfc --cluster=cfc
kubectl config set-credentials user --token={token}
kubectl config set-context cfc --user=user
kubectl config use-context cfc
 ```

#### 3. Access your cluster by using the CLI (kubectl)

You are now able to use the Kubenertes CLI (kubectl) to access your cluster without a time limit. To get you started working with kubectl, see <https://kubernetes.io/docs/user-guide/kubectl-overview/>.



## Option 2: Working with service account tokens obtained from kubectl

------

You can also obtain service account tokens by using kubectl. To obtain service account tokens by using kubectl:

1. Configure the CLI from the IBM Cloud Private management console. For instructions see [Accessing your IBM® Cloud Private cluster by using the kubectl CLI](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0/manage_cluster/cfc_cli.html).

2. Obtain details on your Kubernetes secret object. Secrets are used to store access credentials. To obtain secret details, issue the following command: 

   ```
    kubectl get secret --namespace={namespace}
   ```

 **\*Sample output:***

```
NAME                  TYPE                                  DATA      AGE


admin.registrykey     kubernetes.io/dockercfg               1         1h
default-token-2mfqv   kubernetes.io/service-account-token   3         1h
```

3. Get the details of the service account token by issuing the following command:

   ```
   kubectl get secret default-token-2mfqv --namespace={namespace} -o yaml
   ```

**\*Sample output:***

```
apiVersion: v1


data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURVRENDQWppZ0F3SUJBZ0lKQUxSSHNCazBtblM4TUEwR0NTcUdTSWIzRFFFQkN3VUFNQjh4SFRBYkJnTlYKQkFNTUZERXlOeTR3TGpBdU1VQXhORGczTWprNU1EZ3lNQjRYRFRFM01ESXhOekF5TXpnd01sb1hEVEkzTURJeApOVEF5TXpnd01sb3dIekVkTUJzR0ExVUVBd3dVTVRJM0xqQXVNQzR4UURFME9EY3lPVGt3T0RJd2dnRWlNQTBHCkNTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFDMTUwZWxjRXZXUDBMVFZZK09jNTl4ZG9PUCtXb08Kd3BGNGRxaGpDSDdyZGtUcGVKSE1zeW0raU4wMWxBSjNsc2UvYjB0V2h5L1A5MVZpZmpjazFpaDBldDg0eUZLawpuQWFaNVF6clJxQjk2WGZ3VVVyUElZc0RjRlpzbnAwZUlZU0xJdEhSSHQ3dlY0R3hqbG1TLzlpMzBIcW5rTWJTCmtCbU0xWEp2ZXdjVkROdE55NUE3K1RhNmJWcmt5TlpPZFFjZTkzMk0yTGZ2bUFORzI2UTRtd0x1MlAxNnZGV3EKbkdDd055OVl3Y0k2YVhpQTFSVTNLdWR5d00zZzN2aU03UVMyMXRGbkh4RzJrcU5NNHVKdWZDYnNNZ1gwd1hNQgpuZWZzZ053K0p1b2VnZzFVcHd5RmQydjVyMEpQVkxBN0N1T1d6RzVtK0RrNWNlWExOaGVwMDhxUkFnTUJBQUdqCmdZNHdnWXN3SFFZRFZSME9CQllFRkxlV3ZDOThkZFJxQ2t0eGVla2t5bnY1aCtDSU1FOEdBMVVkSXdSSU1FYUEKRkxlV3ZDOThkZFJxQ2t0eGVla2t5bnY1aCtDSW9TT2tJVEFmTVIwd0d3WURWUVFEREJReE1qY3VNQzR3TGpGQQpNVFE0TnpJNU9UQTRNb0lKQUxSSHNCazBtblM4TUF3R0ExVWRFd1FGTUFNQkFmOHdDd1lEVlIwUEJBUURBZ0VHCk1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQ2Z1ZTdQcVFFR2NMUitjQ2tJMXdISHR2ei9tZWJmNndqUHBqN0oKamV5TG5aeWVZMUVZeEJDWEJEYk9BaU5BRTh6aWQrcm1Fd0w5NndtOGFweUVnbEN6aDhmU1ZoZ1dtYmZKSUNQQQpTTGdFZ1ZjOFJDQk5OdjUwWTQ4L0NXWXFZL2pjZkxYQ1VOdVU5RXhQd1BKRE9jNHhFOFg1NHZDekxzZUF3ZnQ0CmlBS0R0QzZmS0FMNXZQL3RRbHBya2FuVC9zcEVackNZV2IyZXlkRjV4U1NMKzNUbVJTeXgvUkczd1FTWEtCT3cKVGdjaWxJdFQ1WlAwQ0V2WHI1OFBMRXZKMVE1TGZ2Q0w0bkliTEEzMmVucUQ4UlZkM01VbkgxSnFpLzU4VktLQgo4SFpBb1V2bkl2SG5SNGVVbnAwMXFWVFpsS21Xc0JtbjV3MkxaS1FWMEIvVzlnSFAKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
  namespace: ZGVmYXVsdA==
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJblI1Y0NJNklrcFhWQ0o5LmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUprWldaaGRXeDBJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpXTnlaWFF1Ym1GdFpTSTZJbVJsWm1GMWJIUXRkRzlyWlc0dE1tMW1jWFlpTENKcmRXSmxjbTVsZEdWekxtbHZMM05sY25acFkyVmhZMk52ZFc1MEwzTmxjblpwWTJVdFlXTmpiM1Z1ZEM1dVlXMWxJam9pWkdWbVlYVnNkQ0lzSW10MVltVnlibVYwWlhNdWFXOHZjMlZ5ZG1salpXRmpZMjkxYm5RdmMyVnlkbWxqWlMxaFkyTnZkVzUwTG5WcFpDSTZJbVJtTkRReFl6WTVMV1kwWW1FdE1URmxOaTA0TVRVM0xUVXlOVFF3TURJeU5XSTFNeUlzSW5OMVlpSTZJbk41YzNSbGJUcHpaWEoyYVdObFlXTmpiM1Z1ZERwa1pXWmhkV3gwT21SbFptRjFiSFFpZlEuWlN4MTNtY3JPcEwteVFmQWtTV05Ja0VELUIxeTNnckJTREg0Z1lwMnNkb2FZNXBSaWMxc3hXWjRDb0M0YVlnN3pzc09oWHk0NDc5VTh6RTVmVmZ3eFdCSXVWUDVoTEJwWTFHOWhlMzZzSkw1dEpjY2dqSVZhaTFZcHUtQld0dERkRFhnUVZXSHZtQmt0STVPaG1GMTFoWFNqd05VUDhYb2NNY1lKMzZUcFZxbkZCLUZaZ1RnN2h5eWdoclN4MnZTTThHNWhPMWlEdXFFbGlrNTUzQy1razVMTGFnc01DRVpkblBKM2tFb0dzX3hoTVVsaDc3OEkweTMwV3FwYW9uOHBLS1I1NjIzMjd6eTdXNGY0UnJhc3VPSGZwUGE3SVE5cU1ub21fcWxBcWxDQ3lXVEkyV3dxQ09xdnNHUmdNUHJjemc3WnYzLWlXRktBaVc3ZU5VYnVR
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: default
    kubernetes.io/service-account.uid: df441c69-f4ba-11e6-8157-525400225b53
  creationTimestamp: 2017-02-17T02:43:33Z
  name: default-token-2mfqv
  namespace: default
  resourceVersion: "37"
  selfLink: /api/v1/namespaces/default/secrets/default-token-2mfqv
  uid: df5f1109-f4ba-11e6-8157-525400225b53
type: kubernetes.io/service-account-token
```

4. Decode and set the base64 encoded token.

   ```
   kubectl config set-credentials sa-user --token=$(kubectl get secret <secret_name> -o jsonpath={.data.token} | base64 -d)
   
   kubectl config set-context sa-context --user=sa-user
   ```

​      Where <secret_name> is the name of your service account secret.

5. Use curl to call the apiservice. For example,

   ```
   curl -k -H "Authorization:Bearer {token}" [https://](https:).......
   ```

6. Access your cluster by using the CLI (kubelet). 

   You are now able to use the Kubenertes CLI (kubectl) to access your cluster without a time limit. To get you started working with kubectl, see <https://kubernetes.io/docs/user-guide/kubectl-overview/>.