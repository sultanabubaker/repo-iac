# Secure network boundaries using NSP

## Scenario Information

This scenario is deploy a simple network security policy for kubernetes resources to create security boundaries.

* To get started with this scenario ensure you must be using a networking solution which supports `NetworkPolicy`

## Scenario Solution

* The below scenario is from [https://github.com/ahmetb/kubernetes-network-policy-recipes](https://github.com/ahmetb/kubernetes-network-policy-recipes)

If you want to control traffic flow at the IP address or port level (OSI layer 3 or 4), then you might consider using Kubernetes NetworkPolicies for particular applications in your cluster. NetworkPolicies are an application-centric construct which allow you to specify how a pod is allowed to communicate with various network "entities" (we use the word "entity" here to avoid overloading the more common terms such as "endpoints" and "services", which have specific Kubernetes connotations) over the network.

The entities that a Pod can communicate with are identified through a combination of the following 3 identifiers

1. Other pods that are allowed (exception: a pod cannot block access to itself)
Namespaces that are allowed
2. IP blocks (exception: traffic to and from the node where a Pod is running is always allowed, regardless of the IP address of the Pod or the node)
3. When defining a pod- or namespace- based NetworkPolicy, you use a selector to specify what traffic is allowed to and from the Pod(s) that match the selector.

Meanwhile, when IP based NetworkPolicies are created, we define policies based on IP blocks (CIDR ranges).

* We will be creating DENY all traffic to an application

> This NetworkPolicy will drop all traffic to pods of an application, selected using Pod Selectors.

Use Cases:

* It’s very common: To start whitelisting the traffic using Network Policies, first you need to blacklist the traffic using this policy.
* You want to run a Pod and want to prevent any other Pods communicating with it.
* You temporarily want to isolate traffic to a Service from other Pods.

![Scenario 20 NSP](images/sc-20-1.gif)

### Example

* Run a nginx Pod with labels `app=web` and expose it at port 80

```bash
kubectl run --generator=run-pod/v1 web --image=nginx --labels app=web --expose --port 80
```

* Run a temporary Pod and make a request to `web` Service

```bash
kubectl run --generator=run-pod/v1 --rm -i -t --image=alpine test-$RANDOM -- sh
```

```bash
# You will see the below output
# wget -qO- http://web
#    <!DOCTYPE html>
#    <html>
#    <head>
#    ...
```

* It works, now save the following manifest to `web-deny-all.yaml`, then apply to the cluster

```YAML
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: web-deny-all
spec:
  podSelector:
    matchLabels:
      app: web
  ingress: []
```

```bash
kubectl apply -f web-deny-all.yaml
```

### Try it out

* Run a test container again, and try to query `web`

```bash
kubectl run --generator=run-pod/v1 --rm -i -t --image=alpine test-$RANDOM -- sh
```

```bash
# You will see below error now
# wget -qO- --timeout=2 http://web
# wget: download timed out
```

* Traffic dropped

### Remarks

* In the manifest above, we target Pods with app=web label to policy the network. This manifest file is missing the spec.ingress field. Therefore it is not allowing any traffic into the Pod.

* If you create another NetworkPolicy that gives some Pods access to this application directly or indirectly, this NetworkPolicy will be obsolete.

* If there is at least one NetworkPolicy with a rule allowing the traffic, it means the traffic will be routed to the pod regardless of the policies blocking the traffic.

### Cleanup

```bash
kubectl delete pod web
kubectl delete service web
kubectl delete networkpolicy web-deny-all
```

* More referenecs and resources can be found at https://github.com/ahmetb/kubernetes-network-policy-recipes

## Cilium Editor - Network Policy Editor

A tool/framework to teach you how to create a network policy using the Editor. It explains basic network policy concepts and guides you through the steps needed to achieve the desired least-privilege security and zero-trust concepts.

* Navigate to the Cilium Editor [https://editor.cilium.io/](https://editor.cilium.io/)

![Scenario 20 NSP Cilium](images/sc-20-2.png)

## Miscellaneous

* [https://kubernetes.io/docs/concepts/services-networking/network-policies/](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
* [https://github.com/ahmetb/kubernetes-network-policy-recipes](https://github.com/ahmetb/kubernetes-network-policy-recipes)
* [https://editor.cilium.io/](https://editor.cilium.io/)
