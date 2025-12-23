# sshd config


```
# add to eof of sshd config

Port xxxx
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no
KbdInteractiveAuthentication no
ChallengeResponseAuthentication no
UsePAM no
AllowUsers user1 user2
```
