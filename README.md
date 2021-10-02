# instalacao-arch
* `localectl list-keymaps |grep br`
  `loadkeys br-abnt2`
1. particione os discos, efi, swp, hd e ssd com fdisk ou utilitário preferencial
	* Aqui irei usar 1 hd (var, swp), 1 ssd (/, home, efi) 
	* `fdisk -l` para saber as partições criadas
2. encripte os discos com "cryptsetup"
	* `cryptsetup -y -v luksFormat /dev/nvme0n1p1` e crie uma senha 
	* `cryptsetup open /dev/nvme0n1p1 smoker` vamos abrir o partição encriptada e dar um nome pro mapper
3. Crie os file systems
	* `mkfs.fat -F32 /dev/nvme0n1p2` 
	* `mkfs.btrf -L root /dev/mapper/smoker`
	* `mkfs.xfs -L var /dev/sda1`
	* `mkswap /dev/sda2` depois de adicionar o fstab usaremos o swapon
4. Crie os subvolumes Btrfs
	* `mount /dev/mapper/smoker /mnt`
	* `btrfs sub create /mnt/@` /root 
	* `btrfs sub create /mnt/@home` /home
	* `btrfs sub create /mnt/@snapshots` Dir aonde os snapshots serão armazenados
	* `umount /mnt`
5. Monte os subvolumes:
	* `mount -o noatime,nodiratime,compress=zstd,space_cache,ssd,subvol=@ /dev/mapper/smoker /mnt` 
	* `mkdir -p /mnt/{boot,home,.snapshots,var}`
 	* `mount -o noatime,nodiratime,compress=zstd,space_cache,ssd,subvol=@snapshots /dev/mapper/smoker /mnt/.snapshots`
	* `mount -o noatime,nodiratime,compress=zstd,space_cache,ssd,subvol=@home /dev/mapper/smoker /mnt/home`
	* `mount /dev/sdb1 /mnt/var`
Essas opções trazem recursos que melhoram a vida útil do seu sdd, assim como o algoritimo de compressão zstd que possui uma harmonia entre taxa de compressão e velocidade.
	* `swapon /dev/sda2`
6. Monte o EFI
	* `mount /dev/nvme0n1p2 /mnt/boot`

7. Instale a base do Arch-linux
	* `pacstrap /mnt linux linux-firmware base base-devel btrfs-progs intel-ucode vim`
8. Gere o Fstab
	* `genfstab -U /mnt >> /mnt/etc/fstab`
9. Vá para a base do Arch
	* `arch-chroot /mnt`
10. Set localidades
	* `timedatectl list-timezones` lista as zonas
	* `ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime`  sim, os gringos n sabem geografia.
	* `hwclock --systohc
	* `vim /etc/locale.gen` desmarque o pt_br.UTF-8
	* `locale-gen` gera o local
	* `echo "LANG=pt_BR.UTF-8" > /etc/locale.conf`
	* `localectl set-keymap --no-convert br-abnt2`
11. Edite o arquivo de inicialização do systema "initramfs" em /etc/mkinicpio.conf
	* `HOOKS="base keyboard udev autodetect modconf block keymap encrypt btrfs filesystems"`
	* `mkinitcpio -p linux` para realizar as alterações no fs.
12. Instale alguns pacotes que vc necessita
	* `pacman -S grub grub-btrfs base-devel linux-headers git efibootmgr os-prober networkmanager` 

13. Configure o bootloader para reconhecer o disco encryptado
	* `blkid -s UUID -o value /dev/nvme0n1p1 ` 
	```vim /etc/default/grub
	GRUB_CMDLINE_LINUX="cryptdevice=UUID=xxxx:smoker"
	GRUB_DISABLE_OS_PROBER=false```
	* `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB` para instalar o grub no diretorio /boot.
	* `grub-mkconfig -o /boot/grub/grub.cfg` para a configuração inicial do bootloader, caso vc esteja em dual boot e o `os-prober` não reconhecer automaticamente o OS vc terá que refazer as entradas.

14. crie a senha do root
	* `passwd`
15. De um nome ao seu pc e altere o arquivo hosts
	* `echo "arch" > /etc/hostname`
	``` vim /etc/hosts
	127.0.0.1	localhost
	::1		localhost
	127.0.1.1	arch.localdomain	arch
	```
16. Sistema pronto:
	`exit`
	`reboot`
	

17. Ambiente gráfico
	* KDE plasma `pacman -S plasma-desktop sddm plasma-wayland-session firefox dolphin konsole`
	* `sudo systemctl enable sddm`
18. Crie um usuário
	* `adduser -m jeitinhobrasileiro`
19. Backup e snapshot com timeshift
	* `yay timeshift`
	* `systemctl edit grub-btrfs.path`
	``` 
	[unit]
	Description=Monitors for new timeshift snapshots
	DefaultDependencies=no
	Requires=run-timeshift-backup.mount
	After=run-timeshift-backup.mount
	BindsTo=run-timeshift-backup.mount
	
	[Path]
	PathModified=/run/timeshift/backup/timeshift-btrfs/snapshots
	
	[Install]
	Wantedby=run-timeshift-backup.mount
	``` descomente e jogue linha acima
	* cuidado com o fstab ao realizar os rollbacks

20. segurança
	1. `pacman -S clamav` antivirus
		* `freshclam`
		* `clamscan`
	
	2. `pacman -S rkhunter`checa rootkits e vulnerabilidades
		* `rkhunter --check`

	3. `yay tiger` linux secure auditting tool
		* `tiger`
		* `cat /var/log/tiger`

	4. `pacman -S ufw` firewall
		* `sudo ufw enable`
		* `ufw default allow outgoing`
		* `ufw default deny incoming`
	
	5. `passwd -l root` previne que o usuário root logue no sistema #faça isso apenas quando criar uma conta com sudoer. `visudo`
		
	
~                                        
