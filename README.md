# d-utils

`d-utils` is a set of simple utils to use docker images without docker.

-	`d-pull NAME[:TAG] [DIR]` will download a docker image and save it to `DIR`.
	It is saved as a single `rootfs` folder along with a `config.json`.
-	`d-run DIR [CMD]` will execute `CMD` in the container give by `DIR`.

# Config

This is the full list of values from docker's
[`config.json`](https://github.com/opencontainers/image-spec/blob/main/config.md)
that are actually used by `d-run`:

-	`Hostname`
-	`WorkingDir`
-	`Env`
-	`Volumes`
-	`Entrypoint`
-	`Cmd`

`d-run` uses the following additional values:

-	`net` (bool) - enable networking (default: false)
-	`fakeroot` (bool) - map UID and GID to 0 (default: false)

You are encouraged to modify this file, e.g. to add a volume or change the
default command.

You can also modify the rootfs, both from a running container and from the host
system. If you need a new container based on the same image you can just run
`d-pull` again. The layers are cached in `~/.cache/d-utils/` for 30 days.

# Motivation

>	That (Linux) Containers are a userspace fiction is a well-known dictum
>	nowadays. […] This is achieved by combining a multitude of Linux kernel
> features.
> -- [Christian Brauner](https://people.kernel.org/brauner/the-seccomp-notifier-new-frontiers-in-unprivileged-container-development)

I (think I) can remember when cgroups and namespaces were added to linux. Back
then they were announced as low-level features that were not supposed to be
used directly, but that could enable exciting new high-level tools.

And boy did that make waves. Nowadays there are many tools that use these
low-level features: systemd uses them to isolate system services, flatpak and
snap do something similar for desktop applications, and docker has popularized
the idea of "containers" that have now spread far beyond docker itself.

My trouble is: None of these tools feel like they have nailed the "high level"
aspect of this. `systemd-analyze security` for example lists 80 (!) different
settings. What I expect from a high-level tool is a good mix between
flexibility and simplicity, and these tools seem to give me neither.

Docker is the worst offender in my opinion. In an attempt to make it easier for
users there is a lot of implicit behavior: When I start a container the
necessary files are implicitly downloaded and stored somewhere on my machine.
And the container is implicitly started with root permissions. In my opinion
this implicit behavior does more harm than good.

And even with this behavior, docker is still far from simple to use. There is
an abundance of subcommands and options that are hard to understand without a
deep understanding of both docker and the underlying primitives.

So let's start from scratch.

As far as I understand, containerization has two goals: Bundle an application
with libraries and configuration so it can run anywhere, and then isolate the
whole thing so it cannot mess up the host system.

To run such a container you would basically just need a chroot. Namespaces can
then help to further isolate the container, which is good but not essential.
`bwrap` (also used in flatpak) provides all of that and actually has good UX,
so we are up to a promising start.

But then you also need the container itself. And this is where docker makes a
comeback: The ideas of images, containers, layers, volumes as well as
Dockerfiles and an online registry are seriously great and probably a big part
why docker blew up.

So with this project I tried to combine docker images with bwrap. The guiding
principles are:

-	Simple is better than complex
	-	Less than 1000 lines of code
	-	Completeness can be sacrificed in favor of simplicity
		([worse is better](https://www.jwz.org/doc/worse-is-better.html))
	-	Use established tools for the complicated bits
	-	The filesystem is stored in a single folder, no layerfs/aufs/overlayfs
	-	Linux only
-	Explicit is better than implicit
	-	Containers have a simple folder structure
	-	Users can identify containers by their (manually chosen) path
-	Everything is unprivileged

# Limitations

-	This approach will use more disk space because it does not share
	layers/images between containers.
-	It is currently not possible to do any network configuration (e.g. map ports)
	other than sharing network or not sharing network.
-	Some tools are known to cause issues when not running as root:
	-	dpkg ([workaround](https://github.com/opencontainers/runc/issues/2517#issuecomment-1030859646))

# Prior Art

-	https://github.com/containers/bubblewrap
-	https://github.com/NotGlop/docker-drag
-	https://github.com/twosigma/debootwrap
