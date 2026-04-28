[[slurm]]
# slurmdbd

```sh
clear && sudo -u slurm /usr/sbin/slurmdbd -Dvvv
```

# slurmctld

```sh
clear && sudo -u slurm /usr/sbin/slurmctld -Dvvv
```

# slurmd

```sh
/usr/sbin/slurmd -f /etc/slurm.conf -D -s
```

# slurmrestd

```sh
sudo -u zj /usr/sbin/slurmrestd 0.0.0.0:6820 -vvvv
sudo -u zj SLURM_JWT=daemon /usr/sbin/slurmrestd 0.0.0.0:6820 -vvvv
```

---

### 关联笔记
- [[slurm 守护进程]]
- [[slurm 配置]]
- [[slurm 指令]]
