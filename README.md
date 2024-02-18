# ricardoklein.nftables

[![Galaxy](https://img.shields.io/badge/galaxy-ricardoklein.nftables-blue.svg?style=flat)](https://galaxy.ansible.com/ui/standalone/roles/ricardoklein/nftables/documentation/)

Install and setup nftables firewall on opensuse Leap.
By default, we are also installing nftables-service, so the firewall
is up and running with configs from /etc/nftables.conf

Currently supports openSUSE and SLES

## Requirements

No extra requirements needed.

## Role Variables

This is an example of how to setup your variables, you can find more
variables in the template `nftables.conf.j2`.

Simple webserver host with a exclusive list of allowed SSH origins:

```YAML
nft_vars: |
  define external_if       = "eth0"
  define internal_if       = "virbr0"
  define allowed_ssh_hosts = { 192.168.0.1, 192.168.0.10 }

nft_filter_input: |
    tcp dport 22 saddr $allowed_ssh_hosts accept;
    tcp dport 80 accept;
    tcp dport 443 accept;

nft_filter_forward: |
    ip protocol icmp limit rate 2/second accept;
    ip6 nexthdr ipv6-icmp limit rate 2/second accept;
    iifname $internal_if oifname $internal_if counter accept;
    iifname $internal_if oifname $external_if counter accept;
    iifname $external_if oifname $internal_if counter accept;
    iifname $internal_if oifname $podman_if   counter accept;
    iifname $podman_if   oifname $external_if counter accept;
    iifname $podman_if   oifname $internal_if counter accept;

nft_nat_postrouting: |
    iifname $internal_if oifname $external_if counter masquerade random,persistent;
    iifname $podman_if   oifname $external_if counter masquerade random,persistent;
    iifname $podman_if   oifname $internal_if counter masquerade random,persistent;
```

Same webserver, but running workloads in podman containers:

```YAML
nft_vars: |
  define external_if       = "eth0"
  define podman_if         = "cni-podman0"
  define allowed_ssh_hosts = { 192.168.0.1, 192.168.0.10 }

nft_filter_input: |
    tcp dport 22 saddr $allowed_ssh_hosts accept;
    tcp dport 80 accept;
    tcp dport 443 accept;

nft_filter_forward: |
    ip protocol icmp limit rate 2/second accept;
    ip6 nexthdr ipv6-icmp limit rate 2/second accept;
    iifname $podman_if   oifname $external_if counter accept;

nft_nat_postrouting: |
    iifname $podman_if   oifname $external_if counter masquerade random,persistent;
```

Virtualization host with internal network for the VMs on virbr0
and serving DHCP+DNS to the VMs. SSH Allowed for any origin:

```YAML
nft_vars: |
  define external_if       = "eth0"
  define internal_if       = "virbr0"

nft_filter_input: |
    tcp dport 22 accept;
    iifname $internal_if udp dport 67 accept; 
    iifname $internal_if udp dport 53 accept; 
    

nft_filter_forward: |
    ip protocol icmp limit rate 2/second accept;
    ip6 nexthdr ipv6-icmp limit rate 2/second accept;
    iifname $internal_if oifname $internal_if counter accept;
    iifname $internal_if oifname $external_if counter accept;
    iifname $external_if oifname $internal_if counter accept;

nft_nat_postrouting: |
    iifname $internal_if oifname $external_if counter masquerade random,persistent;
```

## Dependencies

This role uses the following collections:

* community.general

## Example Playbook

```
    - hosts: servers
      roles:
         - { role: ricardoklein.nftables }
```

## License

GPL

## Author Information

If you want to suggest changes or request new features,
please feel free to create a issue or send a pull request.

If you find this useful, you can help donating Monero:

`86srKqPRnzpLCkihCHJ9ZB8sjg7B1QxBmT7xS6ebsL52JgCSHVsDsTqDCAqQveasfGh95AZei11Dc1fwjwrE42t3QhzHkdm`
