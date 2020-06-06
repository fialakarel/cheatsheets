# Misc

## Fix Keepass2 segfault on Ubuntu
```bash
sudo sed -i "s/cli/cli --verify-all/" $(which keepass2)
```
