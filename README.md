# Ananicy-cpp Installation & Troubleshooting Guide for Ubuntu/Debian

**Author:** Johnboscocjt  
**Project:** [ananicy-cpp](https://gitlab.com/ananicy-cpp/ananicy-cpp) (C++ Rewrite of Ananicy)

---

## 📖 Table of Contents
- [What is Ananicy-cpp?](#what-is-ananicy-cpp)
- [Prerequisites & Dependencies](#prerequisites--dependencies)
- [Building from Source](#building-from-source)
- [Installation & Service Setup](#installation--service-setup)
- [Adding the Ruleset](#adding-the-ruleset-crucial-step)
- [How to Use & Verify](#how-to-use--verify)
- [Troubleshooting](#troubleshooting)

---

## 🚀 What is Ananicy-cpp?

**Ananicy-cpp** (Another Auto Nice Daemon in C++) is a powerful, low-overhead system daemon designed to improve Linux desktop responsiveness.

### The Core Use Case

On a standard Linux setup, a heavy background task (like a video render or a system update) can "starve" your mouse movements, web browser, or games for CPU cycles, leading to stuttering and lag.

**Ananicy-cpp solves this by:**

- 🎯 **Auto-Prioritizing:** Detects when a "Game" or "Interactive" app is running and gives it higher CPU/IO priority
- ⚡ **Throttling Background Tasks:** Automatically "nices" (lowers priority) background processes
- 📋 **Rule-Based Logic:** Uses a massive library of community-defined rules to handle hundreds of apps automatically

---

## 📦 Prerequisites & Dependencies

To build this on Ubuntu (Noble/Jammy) or Debian, you need specific development headers.

### Critical Fix for Missing Headers

During the build process, you may encounter a fatal error:
```
systemd/sd-daemon.h: No such file or directory
```

This is fixed by installing the systemd development package.

**Run this first:**
```bash
sudo apt update && sudo apt install build-essential cmake git libfmt-dev libspdlog-dev nlohmann-json3-dev libsystemd-dev libelf-dev libbpf-dev
```

---

## 🔧 Building from Source

Navigate to your installation folder and follow these steps.

### Step 1: Clone the Repository
```bash
git clone https://gitlab.com/ananicy-cpp/ananicy-cpp
cd ananicy-cpp
```

### Step 2: Configure and Compile

We use CMake to prepare the build files. The default `make` is highly reliable for Ubuntu.

> **💡 Troubleshooting Build Speed:**  
> If the build is slow, use the `-j$(nproc)` flag. This tells the compiler to use all available CPU cores, significantly cutting down compile time.

```bash
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DENABLE_SYSTEMD=ON
# Compile using all CPU threads
make -j$(nproc)
```

---

## 💻 Installation & Service Setup

Once the compilation hits 100%, install the binary and the systemd unit files:

```bash
sudo make install
```

### Enabling the Daemon

After installation, tell systemd to recognize the new service and start it immediately:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now ananicy-cpp.service
```

---

## 📂 Adding the Ruleset (Crucial Step)

> ⚠️ **Important:** Ananicy-cpp is empty upon installation. Without rules, it does nothing.

Download the community-maintained ruleset:

```bash
sudo mkdir -p /etc/ananicy.d/
sudo git clone https://github.com/ananicy-cpp/ananicy-rules /etc/ananicy.d/rules
sudo systemctl restart ananicy-cpp
```

---

## ✅ How to Use & Verify

### Check if it's actually working

Monitor the live logs. If it's working, you'll see it "applying rules" to processes as they open:

```bash
journalctl -u ananicy-cpp -f
```

### Verify Process Priority

Open a game or a browser and check its "Nice" value in `htop` or via terminal:

```bash
# Example: Check the 'niceness' of Firefox
ps -o ni,comm -p $(pgrep firefox)
```

- **Negative number** (e.g., `-5`) → High priority
- **Positive number** (e.g., `10`) → Low priority

---

## 🔍 Troubleshooting

| Error | Solution |
|-------|----------|
| `systemd/sd-daemon.h: No such file or directory` | `sudo apt install libsystemd-dev` |
| `ninja: error: loading 'build.ninja'` | You ran CMake without the Ninja generator. Use `make` instead. |
| `Service not found` | Ensure you ran `sudo make install` and `systemctl daemon-reload` |
| `0 rules loaded` | You must clone the rules into `/etc/ananicy.d/rules` |

---

## 📝 Documentation Maintained By

**Johnboscocjt**

---

*For more information, visit the [official Ananicy-cpp GitLab repository](https://gitlab.com/ananicy-cpp/ananicy-cpp).*
