# os-autoinst-needles-aeon

The needles for Aeon openQA tests are stored in this repository.

## Running and developing Aeon openQA tests locally

Follow these instructions to run Aeon openQA tests on your local machine.

### Clone repositories

Clone the following git repositories into a local directory:

* https://github.com/os-autoinst/os-autoinst-distri-opensuse
* https://github.com/AeonDesktop/os-autoinst-needles-aeon

The first one contains test scripts, the second contains needles.
If you intend to contribute changes to the Aeon OpenQA tests, you
should create forks of these repositories and clone those
instead.

### Spawn the openQA container

Spawn a single instance of openQA with the `podman` container
engine. If you use Aeon as your host system, you can run `podman`
straight from the system shell:

```
podman run --name openqa \
  --device /dev/kvm \
  -p 1080:80 \
  -p 5991:5991 \
  -v /path/to/os-autoinst-distri-opensuse:/var/lib/openqa/share/tests/aeon:z \
  -v /path/to/os-autoinst-needles-aeon:/var/lib/openqa/share/tests/aeon/products/aeon/needles:z \
  --rm -it \
  registry.opensuse.org/devel/openqa/containers/openqa-single-instance
```

The `-p` options setup port forwarding to localhost:
* OpenQA web UI on port 1080
* VNC server on port 5991

The `-v` options make the tests and needles repositories
available to openQA inside the container. **Change the paths** to
point to the location where you cloned the repositories.

The ":z" suffix applies SELinux labels to the repository
directories to prevent permission errors.

You can now open the openQA web UI on [localhost:1080](http://localhost:1080).

Enter the container. If you use ptyxis, you can enter the
container from the dropdown list in the top-left corner of the
terminal window. Otherwise, run:

```
podman exec -ti openqa /bin/bash
```

Run the script that imports the aeon tests template:

```
/var/lib/openqa/share/tests/aeon/products/aeon/templates
```

### Run the tests

Download and extract the installer image. This can be done from
inside the openQA container:

```
curl -L https://download.opensuse.org/tumbleweed/appliances/Aeon-Installer.x86_64.raw.xz | \
  xz -d > /var/lib/openqa/share/factory/hdd/Aeon-Installer.x86_64.raw
```

Run the tests:

```
openqa-cli api -X POST isos \
  DISTRI=aeon \
  VERSION=Tumbleweed \
  FLAVOR=Installer \
  ARCH=x86_64 \
  HDD_1=Aeon-Installer.x86_64.raw
```

### Develop tests

The Aeon test scripts are in the `os-autoinst-distri-opensuse`
repository located under `/tests/aeon`. If you create new test
scripts, add them to the main script (`/products/aeon/main.pl`).

The needles are in the `os-autoinst-needles-aeon` repository.

You can use a VNC client (for example Remmina) on
`localhost:5991` to connect to the "live" system under test
(pause the test first in developer mode).

Read the [openQA docs](https://open.qa) for instructions on how
to use the openQA web UI and how to develop openQA tests.

More information about needles and how they are used can be found
in the [openQA Getting Started guide](https://github.com/os-autoinst/openQA/blob/master/docs/GettingStarted.asciidoc#needles).
