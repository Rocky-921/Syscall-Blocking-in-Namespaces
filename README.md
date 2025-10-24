# Approaches We Tried for Intercepting Syscalls & Namespace Checking
## 1. Hooks (Implementing hooks as loadable module)
- **KProbe_syscall_block_v1.0** --> uses kernel panic to block syscalls, when they happens from a namespace in which they are supposed to be blocked
- **KProbe_syscall_block_v2.0** --> nulls out the registers values for the syscall, when they happens from a namespace in which they are supposed to be blocked, thus leading to killing of process if atleast one argument to the syscall contains a pointer.

## 2. LSM (Linux Security Module)

## 3. eBPF(Extended Berkeley Packet Filter)

It uses **BCC (BPF Compiler Collection)** to compile and attach the eBPF program.

---

### Step 1 — Install Build Dependencies for BCC Tools

#### For Focal (20.04.1 LTS)
```bash
sudo apt install -y zip bison build-essential cmake flex git libedit-dev \
  libllvm12 llvm-12-dev libclang-12-dev python zlib1g-dev libelf-dev libfl-dev python3-setuptools \
  liblzma-dev arping netperf iperf
```

#### For Hirsute (21.04) or Impish (21.10)
```bash
sudo apt install -y zip bison build-essential cmake flex git libedit-dev \
  libllvm12 llvm-12-dev libclang-12-dev python3 zlib1g-dev libelf-dev libfl-dev python3-setuptools \
  liblzma-dev arping netperf iperf
```

#### For Jammy (22.04)
```bash
sudo apt install -y zip bison build-essential cmake flex git libedit-dev \
  libllvm14 llvm-14-dev libclang-14-dev python3 zlib1g-dev libelf-dev libfl-dev python3-setuptools \
  liblzma-dev libdebuginfod-dev arping netperf iperf
```

#### For Lunar Lobster (23.04)
```bash
sudo apt install -y zip bison build-essential cmake flex git libedit-dev \
  libllvm15 llvm-15-dev libclang-15-dev python3 zlib1g-dev libelf-dev libfl-dev python3-setuptools \
  liblzma-dev libdebuginfod-dev arping netperf iperf libpolly-15-dev
```

#### For Mantic Minotaur (23.10)
```bash
sudo apt install -y zip bison build-essential cmake flex git libedit-dev \
  libllvm16 llvm-16-dev libclang-16-dev python3 zlib1g-dev libelf-dev libfl-dev python3-setuptools \
  liblzma-dev libdebuginfod-dev arping netperf iperf libpolly-16-dev
```

#### For other versions
```bash
sudo apt-get -y install zip bison build-essential cmake flex git libedit-dev \
  libllvm3.7 llvm-3.7-dev libclang-3.7-dev python zlib1g-dev libelf-dev python3-setuptools \
  liblzma-dev arping netperf iperf
```

#### For Lua support
```bash
sudo apt-get -y install luajit luajit-5.1-dev
```

### Step 2 — Install and Compile BCC

```bash
git clone https://github.com/iovisor/bcc.git
mkdir bcc/build; cd bcc/build
cmake ..
make
sudo make install
cmake -DPYTHON_CMD=python3 .. # build python3 binding
pushd src/python/
make
sudo make install
popd
```

### Step 3 — Run the Application
```bash
sudo python3 app.py
```

By default, the mkdir syscall will be blocked only in the current mount namespace.
If you want to block it in other mount namespaces, edit the configuration as described below.

 - Open app.py file
 - Look for these lines:- devinfo = os.stat("/proc/self/ns/mnt")
						  restricted_ns = [devinfo.st_ino]
 - Change the list to the list of inodes of mnt_namespace files you want to block.

### Step 4 — Testing

Now, try running mkdir in another terminal:
```bash
mkdir test_dir
```

It should fail — meaning the syscall is successfully blocked.

#### Note:
My system uses mkdirat syscall when mkdir is called, if yours don't call this, it won't be blocked. But still mkdirat syscall will be blocked.

### Step 6 — Checker Program

A helper program checker.c is included to test namespace-specific blocking.

It tries to create:

 - test_dir_curr_ns in the current mount namespace

 - test_dir_new_ns in a different mount namespace

When blocked, it prints appropriate error messages.

Compile and Run
```bash
gcc checker.c -o checker.o
sudo ./checker.o
```
