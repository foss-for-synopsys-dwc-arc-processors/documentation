# Building the Eclipse Plugin

## Preparing an Environment

!!! info

    All builds and mirrors for Eclipse IDE itself may be found on
    [release's page](https://www.eclipse.org/downloads/packages/release/2020-03/r/eclipse-ide-cc-developers-includes-incubating-components).

We will assume that our build directory is `/build/eclipse`. All necessary
tools, directories and prerequisite will be place there. Thus, go to this
directory first:

```shell
$ mkdir /build/eclipse
$ cd /build/eclipse
```

Also create a directory for storing downloaded Maven packages:

```shell
$ mkdir /build/eclipse/repository
```

Download and extract Maven:

```shell
$ wget https://archive.apache.org/dist/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
$ tar -xf apache-maven-3.5.4-bin.tar.gz
```

Download and extract
[JDK 11](https://www.oracle.com/eg/java/technologies/javase/jdk11-archive-downloads.html)
to `/build/eclipse/jdk`.

Download and extract Eclipse 2020.03 with CDT for Linux:

```shell
$ wget https://www.eclipse.org/downloads/download.php?file=/technology/epp/downloads/release/2020-03/R/eclipse-cpp-2020-03-R-incubation-linux-gtk-x86_64.tar.gz
$ tar -xf eclipse-cpp-2020-03-R-incubation-linux-gtk-x86_64.tar.gz
```

Download and extract Eclipse 2020.03 with CDT for Windows:

```shell
wget https://www.eclipse.org/downloads/download.php?file=/technology/epp/downloads/release/2020-03/R/eclipse-cpp-2020-03-R-incubation-win32-x86_64.zip
tar -xf eclipse-cpp-2020-03-R-incubation-win32-x86_64.zip
```

Clone a plugin's repository:

```shell
$ git clone https://github.com/foss-for-synopsys-dwc-arc-processors/arc_gnu_eclipse
```

## Building the Plugin

Use this commands:

```shell
$ export JAVA_HOME=/build/eclipse/jdk
$ export PATH=$JAVA_HOME/bin:$PATH
$ /build/eclipse/apache-maven-3.5.4/bin/mvn \
    -Dmaven.repo.local=/build/eclipse/repository clean install
```

After that an archive with the plugin will be saved here:

```text
arc_gnu_eclipse/repository/target/repository-2019.9.0-SNAPSHOT.zip
```

## Installing the Plugin

You can install the plugin to Eclipse for Linux this way:

```shell
$ unzip arc_gnu_eclipse/repository/target/repository-2019.9.0-SNAPSHOT.zip \
      -d eclipse-cpp-2020-03-R-incubation-linux-gtk-x86_64/dropins
$ rm -f eclipse-cpp-2020-03-R-incubation-linux-gtk-x86_64/dropins/artifacts.jar \
      eclipse-cpp-2020-03-R-incubation-linux-gtk-x86_64/dropins/content.jar
$ echo "-Dosgi.instance.area.default=@user.home/ARC_GNU_IDE_Workspace" >> eclipse-cpp-2020-03-R-incubation-linux-gtk-x86_64/eclipse.ini
```

You can install the plugin to Eclipse for Windows this way:

```shell
$ unzip arc_gnu_eclipse/repository/target/repository-2019.9.0-SNAPSHOT.zip \
      -d eclipse-cpp-2020-03-R-incubation-win32-x86_64/dropins
$ rm -f eclipse-cpp-2020-03-R-incubation-win32-x86_64/dropins/artifacts.jar \
      eclipse-cpp-2020-03-R-incubation-win32-x86_64/dropins/content.jar
$ echo "-Dosgi.instance.area.default=@user.home/ARC_GNU_IDE_Workspace" >> eclipse-cpp-2020-03-R-incubation-win32-x86_64/eclipse.ini
```
