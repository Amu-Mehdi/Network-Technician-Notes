# Key Takeaways — Cheat Sheet

[← Previous: Advanced Features](05-advanced-features.md) · [Back to README](README.md)

A quick-reference summary of everything covered in this repo.

- Always take a **snapshot** before making any major change.
- The `.vmdk` and `.vmx` files are critical — deleting or moving them carelessly can break the VM.
- Delete old, unneeded snapshots so the Host's hard drive doesn't fill up.
- For isolated, safe network practice, **Host-Only** is the best option.
- For connecting to a real network as an independent device, use **Bridged**.
- For better security with limited internet access, use **NAT**.
- Keep IPs in a consistent, organized range (e.g., all starting with `192.168.20.x`).
- After every **Clone**, always change the Hostname and IP — otherwise expect conflicts.
- Only disable the firewall for practice; in a real environment, define proper exceptions instead of turning it off entirely.

---

[← Previous: Advanced Features](05-advanced-features.md) · [Back to README](README.md)

*This repo is part of my personal learning journey in networking and virtualization — I'll keep adding to it as I learn more.*
