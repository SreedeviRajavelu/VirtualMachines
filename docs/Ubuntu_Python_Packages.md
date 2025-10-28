```
sreedevi@sreedevi:~/Desktop/EPIC Attacks/Attacks/Attack 4 (Control Breaker)$ python3 -m venv venv
The virtual environment was not created successfully because ensurepip is not
available.  On Debian/Ubuntu systems, you need to install the python3-venv
package using the following command.

    apt install python3.12-venv

You may need to use sudo with that command.  After installing the python3-venv
package, recreate your virtual environment.

Failing command: /home/sreedevi/Desktop/EPIC Attacks/Attacks/Attack 4 (Control Breaker)/venv/bin/python3

sreedevi@sreedevi:~/Desktop/EPIC Attacks/Attacks/Attack 4 (Control Breaker)$ 
```

Nice — you hit a common bootstrap problem. The message means your system doesn’t have the small support package that creates virtual environments (`ensurepip`), so `python3 -m venv` can’t populate the venv with a working `pip`. Below are clear, safe steps to fix it and then install `pyiec61850` inside a venv.

Follow these commands (copy-paste):

1. Update apt and install venv support (and some helpful extras)

```
sudo apt update
# install the venv helper; this will provide ensurepip inside venvs
sudo apt install -y python3-venv

# optional but recommended: pip, build tools and git
sudo apt install -y python3-pip python3-full build-essential git
```

Notes:

* On newer Ubuntu releases with a specific Python minor (e.g. 3.12) the package name may be `python3.12-venv`. If `python3-venv` doesn't work, check your python version and install the matching package:

```
python3 --version
# if it prints Python 3.12.x:
sudo apt install -y python3.12-venv
```

2. Create the virtual environment again

```
cd ~/Desktop/"EPIC Attacks"/"Attacks"/"Attack 4 (Control Breaker)"
python3 -m venv venv
```

3. Activate the venv and make sure pip is usable

```
# activate (bash/zsh)
source venv/bin/activate

# upgrade packaging tools inside the venv
pip install --upgrade pip setuptools wheel
```

4. Install the package inside the venv

```
pip install pyiec61850
```

If `pyiec61850` is **not on PyPI**, the previous `pip install` will say “No matching distribution.” In that case install from GitHub (example):

```
git clone https://github.com/thiagohersan/pyiec61850.git
cd pyiec61850
pip install .
```

5. Quick test

```
python -c "import iec61850; print('OK:', iec61850)"
```

If that prints without error, you’re good.

---

### Troubleshooting tips

* If `python3 -m venv venv` still fails after installing `python3-venv`:

  * Confirm `python3 --version` and install the **matching** `python3.X-venv` package (replace `X`).
  * Ensure you used `sudo apt update` before `apt install`.
  * If using WSL or an unusual distro, ensure your distro’s Python packages are available (or consider using `pyenv` to install a self-contained Python).

* If you *must* install system-wide (not recommended), you can bypass PEP 668 with:

  ```
  sudo pip3 install pyiec61850 --break-system-packages
  ```

  ⚠️ This is discouraged because it can break OS-managed Python packages.

* If the library has C dependencies (common for IEC libraries), you may need `build-essential` and the `libiec61850` C library built/installed first. If `pip install` fails with a build error, paste the error and I’ll help parse it.

---

If you want, I can:

* Give the exact commands tailored to the Python version on your machine (tell me the output of `python3 --version`), **or**
* Walk through building `libiec61850` (the C library) and hooking it into Python if `pyiec61850` needs that.

Which would you like next?
