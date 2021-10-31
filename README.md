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

Do not go overboard with your backups.
Or your logs or monitoring.
Make sure you have what you might need, but do not go overboard.
If you have your configs in git, and that git repo is backed up, ehhhh...

I plan to carry a VIP across this cluster, so networking is solved.
I am not planning for built in persistent storage.
I want to be able to throw away the cluster and recreate it without losing anything anyway.

## Requirements

You will need a git repository for the desired state of the cluster.
You will probably put configs for deployments in there as well.

You will need at least 3 servers - they can be vms or bare metal, each with an IP.
You should have a VIP, or an additional IP that can e assigned to any server in the cluster.
You also probably want a DNS record for each server in the cluster, plus one for the VIP.

## Local Utilities

You can put these in a container and hide the complexity.
Or you can understand what you are using.
I choose the latter and install these locally.

```bash
brew install kubectl # interact with a k8s cluster
brew install kubeseal # encrypt secrets to be used in a cluster
brew install fluxcd/tap/flux # automatically keep cluster up to date with git repo
```

You will also either need ansible, or need to do the stuff yourself by hand.

Before you go and rework this and put this stuff in containers, ask yourself:

- Are you making it reusable?
- Are you just hiding the complexity?
- If you hide the complexity, will it just grow?

## Bootstrap the Cluster

### Background

Before you kick a node, you might want to read up on the OS I am running.
[`k3os`][1] is a lightweight linux distro dedicated to this.
Think linux kernel with busybox running [`k3s`][2].
`k3s` is basically everything to make up `k8s`, but lighter.
[I will then use an overly complicated ansible playbook to install k3os][3] which:

- Grabs the current rpw, IP, nameservers, etc from the current linux distro on that server
- Writes a config reusing the above for `k3os`
- Downloads a bash script to install `k3os`.
- Points to a version of the latest k3os version.
- Runs the bash script with the config to install k3os.
- The script overwrites the boot disk to `k3os` and reboots and updated.

I might rewrite with Terraform, or contribute upstream to the bash script in the future, but this works for now.
And, honestly, this assumes whatever server you're going to rekick is already running linux.

### Kick first node

## External References

Just in case you want to do your reading and research ahead of time:

- [k3os][1]
- [k3s][2]
- [bootstrap k3os][3]
- [fluxcd][4]
- [multiple clusters with flux][5]
- [kube-vip][6]
- [kubeseal][7]

[1]: <https://github.com/rancher/k3os#sample-configyaml> "k3os"
[2]: <https://github.com/k3s-io/k3s/blob/master/README.md> "k3s"
[3]: <https://github.com/jakdept/bootstrap_k3os> "bootstrap k3os"
[4]: <https://fluxcd.io/docs/get-started/> "fluxcd"
[5]: <https://github.com/fluxcd/flux2-multi-tenancy> "multiple clusters in flux"
[6]: <https://kube-vip.io> "kube-vip"
[7]: <https://fluxcd.io/docs/guides/sealed-secrets/> "kubeseal"
