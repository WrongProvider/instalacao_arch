# instalacao-arch
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
	* 
	
~                                        
