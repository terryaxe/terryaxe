

![Anurag's GitHub stats](https://github-readme-stats.vercel.app/api?username=terryaxe&hide=contribs,prs)
[![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=terryaxe&layout=compact)](https://github.com/anuraghazra/github-readme-stats)
![9d199e24dc0034ec65db4ae9c638f9e2](https://github.com/user-attachments/assets/dd9c72a7-7bed-4354-ae4c-3c5ce42d4ab1)
你的系统现在处于 无法登录 的状态，原因是 PAM（认证模块）配置错误，导致：

    指纹识别强制启用，但可能未正确工作。

    密码认证被跳过或拒绝，即使输入正确密码也报错。

解决方法（从外部修复）

由于你无法进入系统，我们需要 从 Live USB（Manjaro/Arch Linux 安装盘）启动，挂载原系统分区并修复 PAM 配置。
步骤 1：用 Live USB 启动

    准备一个 Manjaro/Arch Linux Live USB（用另一台电脑制作）。

    插入电脑，启动时选择 USB 启动（按 F12/Esc 等进入 Boot Menu）。

    进入 Live 环境后，打开终端。

步骤 2：挂载原系统分区

    查找原系统的根分区（/）和 boot 分区（如有）：
    bash

lsblk -f  # 查看分区，通常 /dev/nvme0n1p2 或 /dev/sda2 是根分区

（示例输出：/dev/nvme0n1p2 ext4 /mnt）

挂载原系统：
bash

    sudo mount /dev/nvme0n1p2 /mnt           # 挂载根分区
    sudo mount /dev/nvme0n1p1 /mnt/boot      # 挂载 boot（如果是 EFI 系统）
    sudo mount --bind /dev /mnt/dev           # 挂载设备文件
    sudo mount --bind /proc /mnt/proc         # 挂载进程信息
    sudo mount --bind /sys /mnt/sys           # 挂载系统信息

步骤 3：修复 PAM 配置

    进入原系统的根环境：
    bash

sudo chroot /mnt

编辑 /etc/pam.d/system-auth，恢复默认配置 或 移除 pam_fprintd.so：
bash

nano /etc/pam.d/system-auth

删除或注释掉 pam_fprintd.so 行，例如：
bash

# auth sufficient pam_fprintd.so

确保 pam_unix.so（密码认证）能正常工作：
bash

auth    sufficient    pam_unix.so try_first_pass nullok

（如果文件损坏，可从 /etc/pam.d/system-auth.bak 或其他备份恢复）

同样检查以下文件（如有修改）：
bash

    nano /etc/pam.d/sudo
    nano /etc/pam.d/login
    nano /etc/pam.d/passwd

    确保它们没有强制使用指纹认证。

步骤 4：检查账户状态

    确认你的账户密码未被锁定：
    bash

passwd -S 你的用户名  # 检查密码状态

如果账户被锁定（Password locked），解锁：
bash

    passwd -u 你的用户名  # 解锁账户

步骤 5：卸载并重启

    退出 chroot 环境：
    bash

exit

卸载所有挂载的分区：
bash

sudo umount -R /mnt

重启进入原系统：
bash

    reboot

后续调整

    重新配置指纹解锁（可选）
    登录后，可以重新尝试配置指纹，但确保 PAM 配置正确：
    bash

auth sufficient pam_fprintd.so
auth sufficient pam_unix.so try_first_pass nullok

（这样指纹失败后仍可回退密码）

检查日志
如果仍有问题，查看认证日志：
bash

    journalctl -xe | grep pam

总结

    问题原因：PAM 配置错误导致强制指纹认证 + 密码失效。

    解决方法：

        用 Live USB 启动。

        挂载原系统分区。

        修复 /etc/pam.d/system-auth，移除错误的 pam_fprintd.so 行。

        检查账户是否被锁定。

        重启恢复正常登录。

完成后，你可以重新配置指纹解锁，但确保 密码认证仍然可用 作为备用方案。
