# Misc

## Fix Keepass2 segfault on Ubuntu
```bash
sudo sed -i "s/cli/cli --verify-all/" $(which keepass2)
```

## Mount NFS during the boot safely

The system will not hang up if there is no connection
to the NFS server. Put the following line into `/etc/fstab`.

```bash
192.168.1.100:/nfsshare /mnt/nfsshare nfs auto,nofail,x-systemd.device-timeout=30s,noatime,nolock,intr,tcp,actimeo=1800,_netdev 0 0
```

## jq examples

```bash
curl https://jsonplaceholder.typicode.com/posts

curl https://jsonplaceholder.typicode.com/posts | jq
curl --silent https://jsonplaceholder.typicode.com/posts | jq

curl --silent https://jsonplaceholder.typicode.com/posts | jq '.[]'

curl --silent https://jsonplaceholder.typicode.com/posts | jq '.[0]'

curl --silent https://jsonplaceholder.typicode.com/posts | jq '.[].title'

curl --silent https://jsonplaceholder.typicode.com/posts | jq '.[] | {name: .title, post_id: .id}'
```

## yq examples

```bash
cat clusters.yaml | yq eval '.' -
cat clusters.yaml | yq eval '.[]' -

cat clusters.yaml | yq eval '."my-cluster"' -

cat clusters.yaml | yq eval '."my-cluster".labels.team' -

cat clusters.yaml | yq eval '.[].labels.team' -


operators
cat clusters.yaml | yq eval 'keys' -
cat clusters.yaml | yq eval '."my-cluster".labels.team | length' -

yq eval '(.[].labels.team | select(. == "infrastructure")) |= "stars"' clusters.yaml
```
