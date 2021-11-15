# hostbaitor

Flux repository for test k3os cluster - yes my dirty laundry belongs on the internet.

## Scope

Generally, a deployed service can be thought of as a combination of a few "things":

- The bin or code that is your service
- Configs to run your service
- Data produced by or used by your service
- Environemtnt the service needs to run (e.g. the kernel, glibc, libssl, etc.)
- The current running state of your service (mostly defined by configs and data)
- Logs from your service
- Monitoring of your service
- Backups of your data/configs/logs/monitoring/environment

Importance is top to bottom.
Do not go overboard with things.
The idea here is to be able to recreate from this repo.

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

### What does the Ansible Role Do?

The Ansible role is intentionally limited in scope.
It should only be used for creating new nodes.

- It writes a file similar to `bootstrap/example.k3os_conf.yaml`.
- It has a recent `k3os` image from [`k3os` releases][2].
- It runs [a command](https://github.com/jakdept/bootstrap_k3os/blob/main/tasks/main.yml#L82) similar to the following:

```bash
/usr/bin/install.sh \
 --takeover \
 --tty ttyS0 \
 --config {{ config_file }} \
 --no-format \
 {{ boot_part }} \
 {{ image_url }}
```

### Bootstrap Nodes

The parts I used are in the `bootstrap/` folder.

```bash
cd bootstrap/
ansible-galaxy role install -r requirements.yml --force
# kick the first node
ansible-playbook -i hosts.yaml init-node.yml --limit k3os-1.hostbaitor.com
# add additional nodes
ansible-playbook -i hosts.yaml -l k3os-2.hostbaitor.com add-node.yml -e 'k3os_server=k3os-1.hostbaitor.com'
```

You should now have a two node cluster.
Add additional nodes by running the `add-node` playbook.

### Apply Flux to the cluster

This is tied to my Github.
If you are doing this yourself, see <https://fluxcd.io/docs/installation/>.
You will have to change stuff.

Generate a token at <https://github.com/settings/tokens>.
Set that token locally:

```bash
export GITHUB_TOKEN=(pbpaste)
```

```bash
flux bootstrap github \
  --personal \
  --token-auth \
  --branch=main \
  --path=./ \
  --owner=jakdept \
  --repository=hostbaitor
```

### Keepers / Secrets to Save

#### `rancher` password

The rancher user has passwordless `sudo`.
The rancher password is what it was before the reimage.
However, to change that password, reimage the node.

#### `node-token`

Used to add additional nodes to the cluster.
Automatically retrieved from another node with the `add-node` playbook.
Someone could add a node and extract the `kubectl` config to gain access.

```bash
ssh rancher@host sudo cat /var/lib/rancher/k3s/server/node-token
```

#### `kubeconfig`

Root access to the cluster. Retrieve with:

```bash
ssh rancher@host kubectl config view --raw --flatten
```

Once you have that, adjust the connect URL as needed.
I also recommend [merging all `kubeconfigs` with contexts][9].

#### Github Token/Gitlab Token/Flux Deploy Key

If this is a personal cluster, use a personal token.
If this cluster is an organization, use a token for the organization.

Specifically, do not retain the token after Flux is installed.
It's a one time thing that just stays there.

## External References

It's a good idea to know the parts involved

- [`k3os`][1] is a lightweight linux distro dedicated to this.
- Think linux kernel with busybox running [`k3s`][3] - basically `k8s` but light.
- Using a [complicated ansible role][4] to generate my `k3os` config.
- [fluxcd][5] is used to put any configs on the cluster.
- [multiple clusters with flux][6] which going to do it with multiple clusters.
- [kube-vip][7] to put a VIP on the cluster.
- [kubeseal][8] to encrypt secrets for use in workloads.
- [kubectl][9] merging access for multiple clusters.

[1]: <https://github.com/rancher/k3os#sample-configyaml> "k3os"
[2]: <https://github.com/rancher/k3os/releases/tag/v0.20.11-k3s2r1> "k3os releases"
[3]: <https://github.com/k3s-io/k3s/blob/master/README.md> "k3s"
[4]: <https://github.com/jakdept/bootstrap_k3os> "bootstrap k3os"
[5]: <https://fluxcd.io/docs/installation/> "fluxcd"
[6]: <https://github.com/fluxcd/flux2-multi-tenancy> "multiple clusters in flux"
[7]: <https://kube-vip.io> "kube-vip"
[8]: <https://fluxcd.io/docs/guides/sealed-secrets/> "kubeseal"
[9]: <https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/> "kubectl contexts"
