Add q-xdg-dirs.sh to /etc/profile.d
```
#!/bin/sh
export XDG_DATA_DIRS="${XDG_DATA_DIRS}:${HOME}/.local/share"
```