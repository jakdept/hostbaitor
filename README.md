# hostbaitor

Flux repository for test k3os cluster - yes my dirty laundry belongs on the internet.

## Scope

Generally, a deployed service can be thought of as a combination of a few "things":

- The bin or code that is your service
- Environemtnt the service needs to run (e.g. the kernel, glibc, libssl, etc.)
- The current running state of your service (mostly defined by configs and data)
- Configs to run your service
- Data produced by or used by your service
- Logs from your service
- Monitoring of your service
- Backups of your data/configs/logs/monitoring/environment

Do not go overboard with any part of this.
If you put your configs in git and have no data, you need no backups.
The idea is to be able to recreate from this repo.

## Requirements

- This git repository, or a fork of it for your own stuff.
- 3 servers at least, dedicated or vms, with IPs you can reach.
- A VIP, or an extra static IP you can bring up on any node.
- A DNS record for each node plus the VIP for convenience.

## Local Utilities

Just install these go bins locally, don't bother with docker containers.

```bash
brew install kubectl # interact with a k8s cluster
brew install kubeseal # encrypt secrets to be used in a cluster
brew install fluxcd/tap/flux # automatically keep cluster up to date with git repo
```

You will also either need ansible, or need to do the stuff yourself by hand.
There's automated ansible stuff for this later tho.

Before you go and rework this and put this stuff in containers, ask yourself:

- Are you making it reusable?
- Are you just hiding the complexity?
- If you hide the complexity, will it just grow?

## Setup the Cluster

### Bootstrap First Nodes

We're not looking to have a properly running HA cluster protected yet.
Just the first nodes that we'll harden shortly.

The parts I used are in the `bootstrap/` folder.

```bash
cd bootstrap/
ansible-galaxy role install -r requirements.yml --force
ansible-playbook -i hosts.yaml init-node.yml --limit k3os-1.hostbaitor.com
ansible-playbook -i hosts.yaml -l k3os-2.hostbaitor.com add-node.yml -e 'k3os_server=k3os-1.hostbaitor.com'
```

You should now have a two node cluster.
If you lose the second node, it will reconnect to the first node.
But that only goes one way.

Besides, the connection is currently DNS based.
And it's probably a good idea to eventually rekick those nodes to join by VIP.

## External References

It's a good idea to know the parts involved

- [`k3os`][1] is a lightweight linux distro dedicated to this.
- Think linux kernel with busybox running [`k3s`][2] - basically `k8s` but light.
- Using a [complicated ansible role][3] to generate my `k3os` config.
- [fluxcd][4] is used to put any configs on the cluster.
- [multiple clusters with flux][5] which going to do it with multiple clusters.
- [kube-vip][6] to put a VIP on the cluster.
- [kubeseal][7] to encrypt secrets for use in workloads.

[1]: <https://github.com/rancher/k3os#sample-configyaml> "k3os"
[2]: <https://github.com/k3s-io/k3s/blob/master/README.md> "k3s"
[3]: <https://github.com/jakdept/bootstrap_k3os> "bootstrap k3os"
[4]: <https://fluxcd.io/docs/get-started/> "fluxcd"
[5]: <https://github.com/fluxcd/flux2-multi-tenancy> "multiple clusters in flux"
[6]: <https://kube-vip.io> "kube-vip"
[7]: <https://fluxcd.io/docs/guides/sealed-secrets/> "kubeseal"
