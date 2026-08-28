# Chapter 8 — Utility hookups: real addresses, and one that doesn't match

Chapter 7 built the outline of three houses. Before any of them can
actually go up, each one needs a utility hookup — a real address the
outside world can find, plus an internal wiring diagram that means
nothing outside the house's own walls. This chapter is about
[`modules/network.py`](https://github.com/phrankson/platform-core/blob/main/modules/network.py),
the file that draws both.

## Two addresses, and only one of them is real

Two neighboring houses can both wire "Circuit 3" to the kitchen without
the slightest conflict, because nobody outside either house's walls ever
needs to reference "Circuit 3" directly. But the street address each
house sits at has to be unique, or the utility company can't find either
one. `network.py` creates exactly these two categories of address, and
treats them very differently on purpose:

- **`vpcCidr`** — a real address on your machine's actual Docker network:
  `10.0.0.0/16` for `platform-sandbox`, `10.1.0.0/16` for `app-dev`,
  `10.2.0.0/16` for `app-prod`. These have to differ per house, the same
  way three houses need three different street addresses.
- **`podCidr` / `serviceCidr`** — the internal wiring diagram. Kubernetes
  invents these purely for its own bookkeeping (which pod talks to which
  service, inside one cluster). Nothing outside that one cluster ever
  routes to them, so every house reuses the identical values
  (`10.244.0.0/16` / `10.96.0.0/12`) with zero conflict.

`ensure_docker_network()` is the literal, one-line equivalent of calling
the utility company and requesting a hookup at a specific address:

```python
return local.Command(
    "docker:net",
    create=f"docker network create {cfg.dockerNetwork} --subnet {cfg.vpcCidr} || true",
    delete=f"docker network rm {cfg.dockerNetwork} || true",
)
```

That `|| true` is a normal idempotency trick — don't fail the whole
deploy just because the hookup already exists. It has a sharp edge worth
finding yourself before it gets explained, because the discovery is the
actual lesson.

<details>
<summary><strong>Predict before reading on:</strong> <code>Pulumi.platform-sandbox.yaml</code> declares <code>vpcCidr: 10.0.0.0/16</code>. If the hookup at that address had already existed under a different address the very first time this command ever ran, what does <code>|| true</code> do to that mismatch on every call since?</summary>

`docker network create` fails with an error if a network by that name
already exists — it does not silently update an existing network to
match new flags you pass it. `|| true` swallows that failure so Pulumi
doesn't treat "already exists" as a problem worth stopping over. But it
swallows every failure that way, including "exists, with the wrong
address" — and there's no way to tell those two cases apart from the
command's exit code alone.
</details>

## Checking it live, right now, rather than trusting the config file

The source material for this book found exactly that mismatch on
`platform-sandbox-net` — created, almost certainly, before `vpcCidr` was
a config value anyone was checking carefully, so it picked up Kind's own
default addressing instead of the declared one. Rather than repeat that
finding secondhand, here's the same check, run again just now against
this project's actual live Docker networks:

```console
$ docker network inspect platform-sandbox-net --format '{{json .IPAM.Config}}'
[{"Subnet":"fc00:19bf:c38:776c::/64","Gateway":"fc00:19bf:c38:776c::1"},{"Subnet":"172.18.0.0/16","Gateway":"172.18.0.1"}]

$ docker network inspect app-dev-net --format '{{json .IPAM.Config}}'
[{"Subnet":"10.1.0.0/16","Gateway":"10.1.0.1"}]

$ docker network inspect app-prod-net --format '{{json .IPAM.Config}}'
[{"Subnet":"10.2.0.0/16","Gateway":"10.2.0.1"}]
```

Compared against what `Pulumi.platform-sandbox.yaml` actually declares:

```yaml
app:network:
  dockerNetwork: platform-sandbox-net
  vpcCidr: 10.0.0.0/16
```

The mismatch is still there. `app-dev-net` and `app-prod-net` both match
their declared `vpcCidr` exactly. `platform-sandbox-net` is running on
`172.18.0.0/16` — Kind's own default bridge addressing — not the
`10.0.0.0/16` the config file says it should be. This isn't a story about
something that used to be broken and got fixed before publication; it's
still true on this project's real infrastructure as of right now, and
`pulumi preview` will never flag it, because as far as Pulumi's own
bookkeeping is concerned, the `docker:net` command already ran
successfully and nothing about its command line changed since.

Nothing breaks because of it — pods still get valid addresses, and
`platform-sandbox` runs exactly as well as the other two houses. But
reading the config file alone would tell you the wrong subnet, and
finding that out required checking by hand, the way you just watched
happen. The lesson outlasts Docker entirely: an idempotent "create if it
doesn't already exist" is a different guarantee than "make reality match
this config," and the gap between the two is invisible until somebody
checks.

## Writing the wiring diagram without letting the shell mangle it

`render_kind_config()` builds the actual document Kind reads to know
which internal addresses to use for one house — plain YAML, built in
memory as a Python dictionary before anything touches disk:

```console
$ cat .pulumi/kind/pe-sandbox.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
nodes:
- role: control-plane
- role: worker
```

Getting that document safely onto disk turned out to be its own small
problem, with a genuinely reusable fix. An earlier version of this code
inlined the YAML text directly into a shell command, and YAML is full of
exactly the characters that break shell parsing: colons, quotes, dashes.
A config value containing any of those could silently corrupt the file
being written — like handing a contractor a spec sheet where a stray
comma in the address turned "10.0.0.0/16, Suite 2" into two garbled
instructions instead of one.

The fix sidesteps the problem entirely, rather than trying to escape
every dangerous character correctly:

```python
b64 = base64.b64encode(yaml_content.encode("utf-8")).decode("ascii")
script = f"mkdir -p .pulumi/kind && echo {b64} | base64 -d > {path}"
```

Base64 output is guaranteed to be plain letters, digits, `+`, `/`, and
`=` — nothing a shell could ever misread as a command or a special
character. This is a genuinely reusable pattern any time structured text
has to pass through a shell: encode first, decode on the other side, and
the shell never gets a chance to "helpfully" reinterpret the content.

---

**Next:** [Chapter 9 — Pulumi's escape hatch](09-pulumis-escape-hatch.md)
