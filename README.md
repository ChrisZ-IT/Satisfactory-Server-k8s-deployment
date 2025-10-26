# Satisfactory game server
This project formed while I study for the CKA. I hope to add to this as time allows. My goal was to not use Ai at all but only to use [k8s docs](https://kubernetes.io/docs/home/) and the [Satisfactory wiki](https://satisfactory.wiki.gg/wiki/Dedicated_servers) when writing all this code.

That said. This is how to deploy a base Satisfactory game server into a kubernetes cluster.
My current cluster is a 3 node k8s cluster that was deployed via kubeadm, but should work in any type of cluster as long as you have enough resources on your worker nodes.

## How I setup my cluster ##
  1. I deployed 1 control node and 3 worker nodes(all Ubuntu Server 24.04) as VMs in proxmox and my synology NAS. I put my control node and 1 working node on my synology for lighter workloads (2 vCPU, 4 GB ram) and the other 2 nodes in proxmox (4 vCPU 12 GB ram). I used my [ansible playbook](https://github.com/ChrisZ-IT/initial_k8s_node_prep) to prep each node. Then used kubeadm init per k8s docs to setup my cluster.

  2. Installed Calico as my K8s CNI

  3. Deployed MetalLB as my cluster loadbalancer([Example of how I set these up](https://github.com/ChrisZ-IT/base_k8s_deployment))

  4. Setup an NFS share on my synology and mounted it to each k8s node @ `/opt/persistent_volumes`
    I have volume snapshots and file level backups setup on my synology for backing up my persistent volume data


## Satisfactory deployment
  1. Add the label `rquiredCPU=high` to the worker node(s) you want allow this pod to go on `kubectl label nodes kubewrk01 kubewrk02 rquiredCPU=high`
  2. Create the satisfactory namespace `kubectl create ns satisfactory`
  3. Clone this project down and CD into that directory
  4. Make any necessary changes (examples: change loadbalancerIP, volume hostPaths..etc )
  5. Deploy resources to k8s `kubectl apply -f .`
  6. Validate your deployment `kubectl get all -n satisfactory`
  7. At this point you should be able to start playing your new server. Setup any port forwards on your router to your LoadBalancer's external IP to allow external people to be able to join your server.

## TODO
  1. Setup network policies to further lock down who and what can connect to this server.
  2. Setup vertical pod autoscaling. I haven't had any performance issues with this server yet but this gives me an excuse to setup metrics server and better monitoring of my pods.
  3. Replace local-storage class & nfs fstab mounts with the nfs storage class for my persistent storage.
  4. Setup chron job to restart the pod once a day to mirror the default behavior the server's service within the pod.
  5. Move server config file to a configMap/Secret volume. Maybe look at adding the server's password to this as well this is a manual step when you 'claim' your server when you add it to your server list in game.
