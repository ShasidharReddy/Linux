# One-Liner Interview Questions

This guide collects quick-fire Linux interview prompts with concise command-focused answers.

## ⚡ One-Liner Interview Questions (Quick Fire Round)

1. **How do you find all files modified in the last 24 hours?** `find / -mtime -1 -type f`
2. **How do you check which process is using port 8080?** `ss -tlnp | grep 8080` or `lsof -i :8080`
3. **How do you add a user to sudoers?** `usermod -aG sudo username` or `usermod -aG wheel username`
4. **How do you show the current kernel version?** `uname -r`
5. **How do you display the operating system release information?** `cat /etc/os-release`
6. **How do you find the IP addresses on a host?** `ip addr show`
7. **How do you display the default route?** `ip route | grep default`
8. **How do you test DNS resolution for a hostname?** `dig example.com`
9. **How do you view the last 50 lines of a log file?** `tail -50 /var/log/messages`
10. **How do you follow a log file in real time?** `tail -f /var/log/syslog`
11. **How do you search for a text string recursively?** `grep -R "pattern" /path`
12. **How do you find large files bigger than 1 GB?** `find / -type f -size +1G 2>/dev/null`
13. **How do you count the number of files in a directory tree?** `find /path -type f | wc -l`
14. **How do you check free disk space?** `df -h`
15. **How do you check inode usage?** `df -i`
16. **How do you show disk usage by directory?** `du -sh /path/* | sort -h`
17. **How do you check memory usage quickly?** `free -h`
18. **How do you see CPU and memory heavy processes?** `ps -eo pid,%cpu,%mem,cmd --sort=-%cpu | head`
19. **How do you list all listening TCP and UDP sockets?** `ss -tulnp`
20. **How do you test whether a remote port is reachable?** `nc -vz host 443`
21. **How do you restart a systemd service?** `systemctl restart nginx`
22. **How do you enable a service at boot?** `systemctl enable nginx`
23. **How do you check service logs on a systemd host?** `journalctl -u nginx`
24. **How do you see the last boot's logs?** `journalctl -b -1`
25. **How do you display all mounted filesystems?** `findmnt`
26. **How do you mount all filesystems from `/etc/fstab` without rebooting?** `mount -a`
27. **How do you list block devices and filesystems?** `lsblk -f`
28. **How do you show filesystem UUIDs?** `blkid`
29. **How do you list cron jobs for the current user?** `crontab -l`
30. **How do you edit the current user's crontab?** `crontab -e`
31. **How do you display the current user's groups?** `id`
32. **How do you lock a user account?** `usermod -L username`
33. **How do you force a password change on next login?** `chage -d 0 username`
34. **How do you change file ownership recursively?** `chown -R user:group /path`
35. **How do you add execute permission for the owner only?** `chmod u+x script.sh`
36. **How do you view ACLs on a file?** `getfacl /path/file`
37. **How do you find files owned by a specific user?** `find / -user username 2>/dev/null`
38. **How do you find files with SUID set?** `find / -perm -4000 -type f 2>/dev/null`
39. **How do you show the current SELinux mode?** `getenforce`
40. **How do you list firewall rules with nftables?** `nft list ruleset`
41. **How do you check if time sync is working?** `timedatectl` or `chronyc tracking`
42. **How do you see recent login history?** `last`
43. **How do you show who is logged in now?** `who`
44. **How do you identify zombie processes?** `ps -eo pid,ppid,state,cmd | awk '$3 ~ /Z/ {print}'`
45. **How do you see open files for a process?** `lsof -p <pid>`
46. **How do you see deleted but open files?** `lsof +L1`
47. **How do you check LVM physical, volume, and logical volumes?** `pvs && vgs && lvs`
48. **How do you search shell history for a command?** `history | grep ssh`
49. **How do you archive and compress a directory?** `tar -czf backup.tar.gz /path`
50. **How do you quickly verify an HTTP endpoint from the server itself?** `curl -f http://127.0.0.1:8080/health`

---
