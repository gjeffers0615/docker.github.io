{% assign section = include.section %}

{% comment %}

Include a chunk of this file, using variables already set in the file
where you want to reuse the chunk.

Usage: {% include ee-linux-install-reuse.md section="ee-install-intro" %}

{% endcomment %}


{% if section == "ee-install-intro" %}

There are two ways to install and upgrade [Docker Enterprise](https://www.docker.com/enterprise-edition/){: target="_blank" class="_" }
on {{ linux-dist-long }}:

- [YUM repository](#repo-install-and-upgrade): Set up a Docker repository and install Docker Engine - Enterprise from it. This is the recommended approach because installation and upgrades are managed with YUM and easier to do.

- [RPM package](#package-install-and-upgrade): Download the {{ package-format }} package, install it manually, and manage upgrades manually. This is useful when installing Docker Engine - Enterprise on air-gapped systems with no access to the internet.

{% if linux-dist == "rhel" or linux-dist == "oraclelinux" %}
Docker Engine - Community is _not_ supported on {{ linux-dist-long }}.
{% endif %}
{% if linux-dist == "centos" %}
For Docker Community Edition on {{ linux-dist-cap }}, see [Get Docker Engine - Community for CentOS](/install/linux/docker-ce/centos.md).
{% endif %}

{% elsif section == "find-ee-repo-url" %}

To install Docker Enterprise, you will need the URL of the Docker Enterprise repository associated with your trial or subscription:

1.  Go to [https://hub.docker.com/my-content](https://hub.docker.com/my-content){: target="_blank" class="_" }. All of your subscriptions and trials are listed.
2.  Click the **Setup** button for **Docker Enterprise Edition for {{ linux-dist-long }}**.
3.  Copy the URL from **Copy and paste this URL to download your Edition** and save it for later use.

You will use this URL in a later step to create a variable called, `DOCKERURL`.


{% elsif section == "using-yum-repo" %}

The advantage of using a repository from which to install Docker Engine - Enterprise (or any software) is that it provides a certain level of automation. RPM-based distributions such as {{ linux-dist-long }}, use a tool called YUM that work with your repositories to manage dependencies and provide automatic updates.


{% elsif section == "set-up-yum-repo" %}
You only need to set up the repository once, after which you can install Docker Engine - Enterprise _from_ the repo and repeatedly upgrade as necessary.

{% if linux-dist == "rhel" %}

<ul class="nav nav-tabs">
  <li class="active"><a data-toggle="tab" data-target="#RHEL_7" data-group="7">RHEL 7</a></li>
  <li><a data-toggle="tab" data-target="#RHEL_8" data-group="8">RHEL 8</a></li>
</ul>
<div class="tab-content" id="myFirstTab">
<div id="RHEL_7" class="tab-pane fade in active" markdown="1">
1.  Remove existing Docker repositories from `/etc/yum.repos.d/`:

    ```bash
    $ sudo rm /etc/yum.repos.d/docker*.repo
    ```

2.  Temporarily store the URL (that you [copied above](#find-your-docker-ee-repo-url)) in an environment variable. Replace `<DOCKER-EE-URL>` with your URL in the following command. This variable assignment does not persist when the session ends:

    ```bash
    $ export DOCKERURL="<DOCKER-EE-URL>"
    ```

3.  Store the value of the variable, `DOCKERURL` (from the previous step), in a `yum` variable in `/etc/yum/vars/`:

    ```bash
    $ sudo -E sh -c 'echo "$DOCKERURL/{{ linux-dist-url-slug }}" > /etc/yum/vars/dockerurl'
    ```

    Also, store your OS version string in `/etc/yum/vars/dockerosversion`. Most users should use `7` or `8`, but you can also use the more specific minor version, starting from `7.2`.

    ```bash
    $ sudo sh -c 'echo "7" > /etc/yum/vars/dockerosversion'
    ```


4.  Install required packages: `yum-utils` provides the _yum-config-manager_ utility, and `device-mapper-persistent-data` and `lvm2` are required by the _devicemapper_ storage driver:

    ```bash
    $ sudo yum install -y yum-utils \
      device-mapper-persistent-data \
      lvm2
    ```

5.  Enable the `extras` RHEL repository. This ensures access to the `container-selinux` package required by `docker-ee`.

    The repository can differ per your architecture and cloud provider, so review the options in this step before running:

    **For all architectures _except_ IBM Power:**

    ```bash
    $ sudo yum-config-manager --enable rhel-7-server-extras-rpms
    ```

    **For IBM Power only (little endian):**

    ```bash
    $ sudo yum-config-manager --enable extras
    $ sudo subscription-manager repos --enable=rhel-7-for-power-le-extras-rpms
    $ sudo yum makecache fast
    $ sudo yum -y install container-selinux
    ```

    Depending on cloud provider, you may also need to enable another repository:

    **For AWS** (where `REGION` is a literal, and does _not_ represent the region your machine is running in):

    ```bash
    $ sudo yum-config-manager --enable rhui-REGION-rhel-server-extras
    ```

    **For Azure:**

    ```bash
    $ sudo yum-config-manager --enable rhui-rhel-7-server-rhui-extras-rpms
    ```
</div>
<div id="RHEL_8" class="tab-pane fade" markdown="1">
1.  Remove existing Docker repositories from `/etc/yum.repos.d/`:

    ```bash
    $ sudo rm /etc/yum.repos.d/docker*.repo
    ```

2.  Temporarily store the URL (that you [copied above](#find-your-docker-ee-repo-url)) in an environment variable. Replace `<DOCKER-EE-URL>` with your URL in the following command. This variable assignment does not persist when the session ends:

    ```bash
    $ export DOCKERURL="<DOCKER-EE-URL>"
    ```

3.  Store the value of the variable, `DOCKERURL` (from the previous step), in a `yum` variable in `/etc/yum/vars/`:

    ```bash
    $ sudo -E sh -c 'echo "$DOCKERURL/{{ linux-dist-url-slug }}" > /etc/yum/vars/dockerurl'
    ```

    Also, store your OS version string in `/etc/yum/vars/dockerosversion`. Most users should use `8`, but you can also use the more specific minor version.

    ```bash
    $ sudo sh -c 'echo "8" > /etc/yum/vars/dockerosversion'
    ```


4.  Install required packages: `yum-utils` provides the _yum-config-manager_ utility, and `device-mapper-persistent-data` and `lvm2` are required by the _devicemapper_ storage driver:

    ```bash
    $ sudo yum install -y yum-utils \
      device-mapper-persistent-data \
      lvm2
    ```
</div>
</div>
{% endif %}

{% if linux-dist != "rhel" %}

1.  Remove existing Docker repositories from `/etc/yum.repos.d/`:

    ```bash
    $ sudo rm /etc/yum.repos.d/docker*.repo
    ```

2.  Temporarily store the URL (that you [copied above](#find-your-docker-ee-repo-url)) in an environment variable. Replace `<DOCKER-EE-URL>` with your URL in the following command. This variable assignment does not persist when the session ends:

    ```bash
    $ export DOCKERURL="<DOCKER-EE-URL>"
    ```

3.  Store the value of the variable, `DOCKERURL` (from the previous step), in a `yum` variable in `/etc/yum/vars/`:

    ```bash
    $ sudo -E sh -c 'echo "$DOCKERURL/{{ linux-dist-url-slug }}" > /etc/yum/vars/dockerurl'
    ```

4.  Install required packages: `yum-utils` provides the _yum-config-manager_ utility, and `device-mapper-persistent-data` and `lvm2` are required by the _devicemapper_ storage driver:

    ```bash
    $ sudo yum install -y yum-utils \
      device-mapper-persistent-data \
      lvm2
    ```

{% if linux-dist == "oraclelinux" %}

5.  Enable the `ol7_addons` Oracle repository. This ensures access to the `container-selinux` package required by `docker-ee`.

    ```bash
    $ sudo yum-config-manager --enable ol7_addons
    ```

{% endif %}

6.  Add the Docker Engine - Enterprise **stable** repository:

    ```bash
    $ sudo -E yum-config-manager \
        --add-repo \
        "$DOCKERURL/{{ linux-dist-url-slug }}/docker-ee.repo"
    ```
{% endif %}

{% elsif section == "install-using-yum-repo" %}

> **Note**: If you need to run Docker Engine - Enterprise 2.0, please see the following instructions:
> * [18.03](https://docs.docker.com/v18.03/ee/supported-platforms/) - Older Docker Engine - Enterprise Engine only release
> * [17.06](https://docs.docker.com/v17.06/engine/installation/) - Docker Enterprise Edition 2.0 (Docker Engine,
> UCP, and DTR).

1.  Install the latest patch release, or go to the next step to install a specific version:

    ```bash
    $ sudo yum -y install docker-ee docker-ee-cli containerd.io
    ```

    If prompted to accept the GPG key, verify that the fingerprint matches `{{ gpg-fingerprint }}`, and if so, accept it.


2.  To install a _specific version_ of Docker Engine - Enterprise (recommended in production), list versions and install:

    a.  List and sort the versions available in your repo. This example sorts results by version number, highest to lowest, and is truncated:

    ```bash
    $ sudo yum list docker-ee  --showduplicates | sort -r

    docker-ee.x86_64      {{ site.docker_ee_version }}.ee.2-1.el7.{{ linux-dist }}      docker-ee-stable-18.09
    ```

    The list returned depends on which repositories you enabled, and is specific to your version of {{ linux-dist-long }} (indicated by `.el7` in this example).

    b.  Install a specific version by its fully qualified package name, which is the package name (`docker-ee`) plus the version string (2nd column) starting at the first colon (`:`), up to the first hyphen, separated by a hyphen (`-`). For example, `docker-ee-18.09.1`.

    ```bash
    $ sudo yum -y install docker-ee-<VERSION_STRING> docker-ee-cli-<VERSION_STRING> containerd.io
    ```

    For example, if you want to install the 18.09 version run the following:

    ```bash
    sudo yum-config-manager --enable docker-ee-stable-18.09
    ```

    Docker is installed but not started. The `docker` group is created, but no users are added to the group.

3.  Start Docker:

    > If using `devicemapper`, ensure it is properly configured before starting Docker, per the [storage guide](/storage/storagedriver/device-mapper-driver/){: target="_blank" class="_" }.

    ```bash
    $ sudo systemctl start docker
    ```

4.  Verify that Docker Engine - Enterprise is installed correctly by running the `hello-world`
    image. This command downloads a test image, runs it in a container, prints
    an informational message, and exits:

    ```bash
    $ sudo docker run hello-world
    ```

    Docker Engine - Enterprise is installed and running. Use `sudo` to run Docker commands. See
    [Linux postinstall](/install/linux/linux-postinstall.md){: target="_blank" class="_" } to allow
    non-privileged users to run Docker commands.


{% elsif section == "upgrade-using-yum-repo" %}

1.  [Add the new repository](#set-up-the-repository).

2.  Follow the [installation instructions](#install-from-the-repository) and install a new version.


{% elsif section == "package-installation" %}

To manually install Docker Enterprise, download the `.{{ package-format | downcase }}` file for your release. You need to download a new file each time you want to upgrade Docker Enterprise.

{% elsif section == "install-using-yum-package" %}

{% if linux-dist == "rhel" %}
<ul class="nav nav-tabs">
  <li class="active"><a data-toggle="tab" data-target="#RHEL-7" data-group="7">RHEL 7</a></li>
  <li><a data-toggle="tab" data-target="#RHEL-8" data-group="8">RHEL 8</a></li>
</ul>

<div class="tab-content" id="mySecondTab">

<div id="RHEL-7" class="tab-pane fade in active" markdown="1">

1.  Enable the `extras` RHEL repository. This ensures access to the `container-selinux` package which is required by `docker-ee`:

    ```bash
    $ sudo yum-config-manager --enable rhel-7-server-extras-rpms
    ```

    Alternately, obtain that package manually from Red Hat. There is no way to publicly browse this repository.

2.  Go to the Docker Engine - Enterprise repository URL associated with your
    trial or subscription in your browser. Go to
    `{{ linux-dist-url-slug }}/`. Choose your {{ linux-dist-long }} version,
    architecture, and Docker version. Download the
    `.{{ package-format | downcase }}` file from the `Packages` directory.

    > If you have trouble with `selinux` using the packages under the `7` directory,
    > try choosing the version-specific directory instead, such as `7.3`.

3.  Install Docker Enterprise, changing the path below to the path where you downloaded
    the Docker package.

    ```bash
    $ sudo yum install /path/to/package.rpm
    ```

    Docker is installed but not started. The `docker` group is created, but no
    users are added to the group.

4.  Start Docker:

    > If using `devicemapper`, ensure it is properly configured before starting Docker, per the [storage guide](/storage/storagedriver/device-mapper-driver/){: target="_blank" class="_" }.

    ```bash
    $ sudo systemctl start docker
    ```

5.  Verify that Docker Engine - Enterprise is installed correctly by running the `hello-world`
    image. This command downloads a test image, runs it in a container, prints
    an informational message, and exits:

    ```bash
    $ sudo docker run hello-world
    ```

    Docker Engine - Enterprise is installed and running. Use `sudo` to run Docker commands. See
    [Linux postinstall](/install/linux/linux-postinstall.md){: target="_blank" class="_" } to allow
    non-privileged users to run Docker commands.

</div>

<div id="RHEL-8" class="tab-pane fade" markdown="1">

1.  Go to the Docker Engine - Enterprise repository URL associated with your
    trial or subscription in your browser. Go to
    `{{ linux-dist-url-slug }}/`. Choose your {{ linux-dist-long }} version,
    architecture, and Docker version. Download the
    `.{{ package-format | downcase }}` file from the `Packages` directory.

    > If you have trouble with `selinux` using the packages under the `8` directory,
    > try choosing the version-specific directory instead.

2.  Install Docker Enterprise, changing the path below to the path where you downloaded
    the Docker package.

    ```bash
    $ sudo yum install /path/to/package.rpm
    ```

    Docker is installed but not started. The `docker` group is created, but no
    users are added to the group.

3.  Start Docker:

    > If using `devicemapper`, ensure it is properly configured before starting Docker, per the [storage guide](/storage/storagedriver/device-mapper-driver/){: target="_blank" class="_" }.

    ```bash
    $ sudo systemctl start docker
    ```

4.  Verify that Docker Engine - Enterprise is installed correctly by running the `hello-world`
    image. This command downloads a test image, runs it in a container, prints
    an informational message, and exits:

    ```bash
    $ sudo docker run hello-world
    ```

    Docker Engine - Enterprise is installed and running. Use `sudo` to run Docker commands. See
    [Linux postinstall](/install/linux/linux-postinstall.md){: target="_blank" class="_" } to allow
    non-privileged users to run Docker commands.

</div>
</div>

{% endif %}
{% if linux-dist != "rhel" %}
{% if linux-dist == "centos" %}
1.  Go to the Docker Engine - Enterprise repository URL associated with your trial or subscription
    in your browser. Go to `{{ linux-dist-url-slug }}/7/x86_64/stable-<VERSION>/Packages`
    and download the `.{{ package-format | downcase }}` file for the Docker version you want to install.
{% endif %}

{% if linux-dist == "oraclelinux" %}
1.  Go to the Docker Engine - Enterprise repository URL associated with your
    trial or subscription in your browser. Go to
    `{{ linux-dist-url-slug }}/`. Choose your {{ linux-dist-long }} version,
    architecture, and Docker version. Download the
    `.{{ package-format | downcase }}` file from the `Packages` directory.

{% endif %}

2.  Install Docker Enterprise, changing the path below to the path where you downloaded
    the Docker package.

    ```bash
    $ sudo yum install /path/to/package.rpm
    ```

    Docker is installed but not started. The `docker` group is created, but no
    users are added to the group.

3.  Start Docker:

    > If using `devicemapper`, ensure it is properly configured before starting Docker, per the [storage guide](/storage/storagedriver/device-mapper-driver/){: target="_blank" class="_" }.

    ```bash
    $ sudo systemctl start docker
    ```

4.  Verify that Docker Engine - Enterprise is installed correctly by running the `hello-world`
    image. This command downloads a test image, runs it in a container, prints
    an informational message, and exits:

    ```bash
    $ sudo docker run hello-world
    ```

    Docker Engine - Enterprise is installed and running. Use `sudo` to run Docker commands. See
    [Linux postinstall](/install/linux/linux-postinstall.md){: target="_blank" class="_" } to allow
    non-privileged users to run Docker commands.
{% endif %}

{% elsif section == "upgrade-using-yum-package" %}

1.  Download the newer package file.

2.  Repeat the [installation procedure](#install-with-a-package), using
    `yum -y upgrade` instead of `yum -y install`, and point to the new file.


{% elsif section == "yum-uninstall" %}

1.  Uninstall the Docker Engine - Enterprise package:

    ```bash
    $ sudo yum -y remove docker-ee
    ```

2.  Delete all images, containers, and volumes (because these are not automatically removed from your host):

    ```bash
    $ sudo rm -rf /var/lib/docker
    ```

3.  Delete other Docker related resources:
    ```bash
    $ sudo rm -rf /run/docker
    $ sudo rm -rf /var/run/docker
    $ sudo rm -rf /etc/docker
    ```

4.  If desired, remove the `devicemapper` thin pool and reformat the block
    devices that were part of it.

You must delete any edited configuration files manually.


{% elsif section == "linux-install-nextsteps" %}

- Continue to [Post-installation steps for Linux](/install/linux/linux-postinstall.md){: target="_blank" class="_" }

- Continue with user guides on [Universal Control Plane (UCP)](/ee/ucp/){: target="_blank" class="_" } and [Docker Trusted Registry (DTR)](/ee/dtr/){: target="_blank" class="_" }

{% endif %}
