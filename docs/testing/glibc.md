# Running Glibc Tests

Currently our toolchain Makefiles don't support automated testing,
but this may change in the near future. Look
[here](https://github.com/foss-for-synopsys-dwc-arc-processors/arc-gnu-toolchain#testing-gccc-binutils-and-newlib)
for possible support.

## Standalone testing

It might be useful to perform tests without having to build the entire toolchain.

To perform the tests, it is necessary to build glibc like:

```shell
$ export TOOLCHAIN_SRC=/absolute/path/to/parent                  # Path to parent of ARC sources (at least glibc, toolchain and arc-gnu-toolchain)
$ export GLIBC_PARENT=/new/empty/directory; cd $GLIBC_PARENT     # We might need to network share this directory, so create a new one
$ git clone https://github.com/foss-for-synopsys-dwc-arc-processors/glibc;
$ cd glibc; git checkout arc64                                   # Setup the glibc source we want to test
$ cd ../; mkdir glibc_build
$ ../glibc/configure --host=arc64-linux-gnu --prefix=/usr             \
--disable-werror --enable-shared --enable-obsolete-rpc                \
--with-headers=$TOOLCHAIN_SRC/arc-gnu-toolchain/linux-headers/include \
--disable-multilib --libdir=/usr/lib libc_cv_slibdir=/lib             \
libc_cv_rtlddir=/lib CFLAGS="-O2 -g3"
$ make -j <NMB_CORES>                                            # set NMB_CORES to whatever amount of cores you want 
```

There are two ways to run the GLIBC tests for ARC. We can use QEMU
[usermode](https://www.qemu.org/docs/master/user/main.html) or full system emulation.

## User mode testing

If you are running a linux host, usermode provides faster but possibly less accurate
testing, since it uses the host kernel instead of a fully simulated one.

### Setting up

There are a few ways to set this up but the most direct one is to setup [binfmt](https://www.kernel.org/doc/html/latest/admin-guide/binfmt-misc.html).
Fortunately, QEMU provides a script that sets up binfmt so the launched executables are interpreted by a shell script at `/usr/local/qemu-arc64`. An example of which is:

```shell
$ sudo $QEMU_SOURCE/scripts/qemu-binfmt-conf.sh
$ sudo cat > /usr/local/bin/qemu-arc64 << 'EOF'
#!/bin/bash
${QEMU_BINPATH}/qemu-arc64 -R 4G -L ${TOOLCHAIN_INST}/sysroot -cpu hs6x $@
EOF
```

Afterwards, launching an ARC executable will run this script and consequently, QEMU.

### Running the tests

Running the actual tests is simple.
```shell
$ cd $GLIBC_PARENT/glibc-build
$ make tests
```

The results are then stored in `*test-result` files.
See some debugging tips at the end of this wiki entry

## Full system

To run the more accurate but \`heavier\` full system tests, there are a few intermediary steps required.

### Linux image

We will need to generate local ssh keys (`ssh-keygen -t ed25519`)
Then, configure a linux image.
The configurations required are:

1. eth0 configured via dhcp
2. gdbserver, ssh and ftpd
3. User/Network namespace support
4. An overlay file system such that the generated public key is recognized (present inside '~/.ssh/authorized_keys')

### NFS

Second, we need to be able to share our build and source directories with the qemu instance that will be running the aforementioned Linux image.

There are many ways to do this, but the one contemplated here is NFS. Install and enable it for your respective platform and set up $GLIBC_PARENT as a shared directory.

### Running the instance

In a separate terminal, we run the QEMU instance that must stay alive for the entirety of the testing.

```shell
$ qemu-system-arc64 -M virt -cpu hs6x -nographic -no-reboot -global cpu.freq_hz=50000000 -netdev user,id=net0,hostfwd=tcp::2221-:21,hostfwd=tcp::2222-:22,hostfwd=tcp::2223-:23,hostfwd=tcp::6175-:6175 -device virtio-net-device,netdev=net0 -kernel $PATH_TO_LOADER/glicready_loader
Linux version 5.16.0-711640-g19e84a74062a (YOUR_USER@YOUR_MACHINE) (arc64-linux-gnu-gcc (ARCv3 ARC64 GNU/Linux f2bc3b4c762) 12.2.1 20220829, GNU ld (GNU Binutils) 2.40.50.20230314) #11 PREEMPT Tue May 9 10:32:00 WEST 2054
Memory @ 80000000 [1024M] 
Memory @ 100000000 [1024M] Not used
OF: fdt: Machine model: snps,zebu_hs
earlycon: uart8250 at MMIO32 0x00000000f0000000 (options '115200n8')
printk: bootconsole [uart8250] enabled

...

Welcome to Buildroot
buildroot login: root
# # Mount the network folders
# export SHARED_DIR=/same/absolute/path/as/for/GLIBC_PARENT
# mkdir -p $SHARED_DIR
# mount -t nfs -o nolock 10.0.2.2:$SHARED_DIR $SHARED_DIR
```

We can test the connection with:

```shell
$ ssh -p 2222 root@127.0.0.1
The authenticity of host '[127.0.0.1]:2222 ([127.0.0.1]:2222)' cant be established.
ED25519 key fingerprint is SHA256:jbqautMkwc2FSBo4c89RKpTDjaSDlmQ3dGk+TrWOCjo.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[127.0.0.1]:2222' (ED25519) to the list of known hosts.
# exit
Connection to 127.0.0.1 closed.
$
```

### Setting up the tests

For the tests to automatically run without entering the password for each one,
we must setup an ssh host with the keys shared with the linux image

```shell
$ cat ~/.ssh/config
...
HOST arcglibc
	HostName 127.0.0.1
	User root
	Port 2222
	IdentityFile ~/.ssh/id_ed25519
	HostKeyAlgorithms ssh-ed25519
...
$
```

### Running the tests

With the QEMU instance running, now we can run the actual tests.

```shell
$ cd $GLIBC_PARENT/glibc-build
$ make test-wrapper="/absolute/path/to/shared/glibc/scripts/cross-test-ssh.sh --timeoutfactor 30 arcglibc" tests
```

The results are then stored in `*test-result` files.

### Debugging in full system

In full system it is not a good idea to use QEMUs' gdbstub unless we want to debug the whole kernel.
As such we must use the gdbserver packaged in with the linux image:

```shell
# gdbserver 127.0.0.1:6175 $SHARED_DIR/glibc-build/math/test-float-tgamma
```

And then we simply connect GDB

```shell
$ arc64-elf-gdb $GLIBC_PARENT/glibc-build/math/test-float-tgamma
(gdb) target remote:6175
```

If there is any problem running the image, `Ctrl+A X` can be used to kill QEMU
After an instance has run, we might get the following error which will crash
the tests. It is best to run the `ssh-keygen` command below after stopping QEMU to prevent it.

```shell
$ ssh -p 2222 root@127.0.0.1
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@                        
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ED25519 key sent by the remote host is
SHA256:jbqautMkwc2FSBo4c89RKpTDjaSDlmQ3dGk+TrWOCjo.
Please contact your system administrator.
Add correct host key in /home/YOUR_USER/.ssh/known_hosts to get rid of this message.
Offending ED25519 key in /home/YOUR_USER/.ssh/known_hosts:16
Host key for [127.0.0.1]:2222 has changed and you have requested strict checking.
Host key verification failed.
$ ssh-keygen -f "/home/YOUR_USER/.ssh/known_hosts" -R "[127.0.0.1]:2222"
# Host [127.0.0.1]:2222 found: line 160
/home/YOUR_USER/.ssh/known_hosts updated.
Original contents retained as /home/YOUR_USER/.ssh/known_hosts.old
```

See more debugging tips below

## General debugging tips

### Single test

You can run a single test by replacing 'tests' with the test <subgroup>/<name>, i.e. 'test t=math/test-double-trunc'

### Single group of tests

You can run a single group of tests by adding 'subdirs=<subgroup>'' after 'tests', i.e.  'tests subdirs=math'
