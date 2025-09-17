# ❌ Error & ✅ Fix for HashiCorp Repo Setup  

### 🔴 Error Seen  
```bash
ubuntu@ip-172-31-29-212:~$ deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com noble main
Command 'deb' not found, did you mean:
  command 'deb3' from deb quilt (0.67+really0.67-4)
  command 'edb' from deb edb-debugger (1.3.0-2.1)
...
E: Malformed entry 1 in list file /etc/apt/sources.list.d/hashicorp.list (URI)
E: The list of sources could not be read.
E: Malformed entry 1 in list file /etc/apt/sources.list.d/hashicorp.list (URI)
E: The list of sources could not be read.
```

👉 Problem: You typed `deb ...` at the shell prompt, so Bash treated it as a command.  
👉 Also, your sources list file had a malformed line (`\` continuation).  

---

### 🛠️ Solution (Step by Step)  

1. **Re-add the GPG key**:  
```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg  | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

2. **Fix the repo file (single clean line)**:  
```bash
sudo rm -f /etc/apt/sources.list.d/hashicorp.list
sudo bash -c 'cat >/etc/apt/sources.list.d/hashicorp.list <<EOF
deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com noble main
EOF'
```

3. **Verify file**:  
```bash
cat -n /etc/apt/sources.list.d/hashicorp.list
```
✔️ Should show exactly one line, no `\`  

4. **Update & install Vault**:  
```bash
sudo apt update
sudo apt install -y vault
```

---

### 🔎 Check Installed Version  
```bash
vault --version
```

✅ Output:  
```
Vault v1.20.3 (7665ff29d77e5cb3ea9ddbeaed49ee312e53c6b8), built 2025-08-27T10:53:27Z
```

---

✨ Now Vault is installed successfully and ready to use! 🚀🔐  
