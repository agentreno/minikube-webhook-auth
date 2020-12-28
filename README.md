# minikube-webhook-auth

## Description

It is possible to set minikube to use different apiserver authentication
methods, including [Webhook Token
Authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication).

Follow these steps:

1. Copy webhook-config-file.yaml file in this repo to
   `~/.minikube/files/etc/ca-certificates/`. Note that it must be at this path,
   to ensure it is mounted by the minikube apiserver container. See [this
   issue](https://github.com/kubernetes/minikube/issues/8762)

2. Edit this file to set cluster.server to the HTTPS webhook listener of your
   choosing. I used https://hookb.in to verify it received a TokenReview
   message but a custom web server responding with a completed TokenReview
   would finish the flow.

3. Create a new minikube cluster with the necessary apiserver flag: `minikube
   --extra-config="apiserver.authentication-token-webhook-config-file=/etc/ca-certificates/webhook-config-file.yaml"
   start`

4. Try to authenticate to apiserver and observe the TokenReview message passed
   to your listener. `http -v --verify=no
   https://192.168.49.2:8443/api/v1/namespaces/default/pods
   "Authorization:Bearer mytoken"`. Client cert auth is enabled alongside
   webhook auth, so I found that `kubectl --token=mytoken get po` would
   fallback to using the client cert put in the kubeconfig by minikube. Hence
   the use of httpie instead.
