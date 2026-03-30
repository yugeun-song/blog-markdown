# Arch Linux 설치 가이드 — UEFI + Btrfs + systemd-boot

Arch Linux 설치는 리눅스 시스템의 각 구성 요소를 직접 선택하고 조합하는 과정이다. 이 글에서는 UEFI 환경에서 Btrfs 파일시스템과 systemd-boot를 사용하는 구성을 단계별로 진행한다.

## 사전 준비

Arch Linux ISO를 USB에 기록하고 UEFI 모드로 부팅한다. 부팅 후 라이브 환경에서 네트워크 연결을 확인한다.

```bash
# UEFI 모드 확인
ls /sys/firmware/efi/efivars
# 파일이 나열되면 UEFI 모드

# 인터넷 연결 확인
ip link
ping -c 3 archlinux.org

# Wi-Fi 연결 (필요한 경우)
iwctl
# [iwd]# station wlan0 connect "SSID"

# 시스템 시계 동기화
timedatectl set-ntp true
```

### 파티션 구성

GPT 파티션 테이블을 만들고, EFI 시스템 파티션(ESP)과 루트 파티션을 생성한다.

```bash
# 디스크 확인
lsblk

# 파티셔닝 (nvme0n1 기준)
fdisk /dev/nvme0n1
# g — GPT 테이블 생성
# n — 파티션 1: +1G (EFI)
# t — 타입 1 (EFI System)
# n — 파티션 2: 나머지 (Linux)
# w — 기록

# 포맷
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.btrfs /dev/nvme0n1p2
```

스왑 파티션 대신 스왑파일을 나중에 설정할 수 있다. Btrfs에서는 별도 서브볼륨에 스왑파일을 두는 것이 권장된다.

## Btrfs 서브볼륨 구성

Btrfs의 서브볼륨은 스냅샷과 롤백의 기반이 된다. 일반적인 서브볼륨 레이아웃을 구성한다.

```bash
# 루트 파티션 마운트
mount /dev/nvme0n1p2 /mnt

# 서브볼륨 생성
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@cache
btrfs subvolume create /mnt/@swap

# 언마운트 후 서브볼륨으로 다시 마운트
umount /mnt
mount -o subvol=@,compress=zstd,noatime /dev/nvme0n1p2 /mnt
mkdir -p /mnt/{home,var/log,var/cache,swap,boot}
mount -o subvol=@home,compress=zstd,noatime /dev/nvme0n1p2 /mnt/home
mount -o subvol=@log,compress=zstd,noatime /dev/nvme0n1p2 /mnt/var/log
mount -o subvol=@cache,compress=zstd,noatime /dev/nvme0n1p2 /mnt/var/cache
mount -o subvol=@swap,noatime /dev/nvme0n1p2 /mnt/swap
mount /dev/nvme0n1p1 /mnt/boot
```

`@`이 루트, `@home`이 홈 디렉토리다. 루트를 스냅샷으로 롤백해도 홈 데이터는 영향받지 않는다. `compress=zstd`는 투명 압축으로 디스크 공간을 절약한다.

### 스냅샷 전략

Btrfs 스냅샷은 copy-on-write 특성상 거의 즉시 생성된다. 패키지 업데이트 전 스냅샷을 찍어두면 문제 발생 시 즉시 롤백할 수 있다.

`snapper`나 `timeshift`를 사용하면 자동 스냅샷 관리가 편리하다.

## 기본 시스템 설치

`pacstrap`으로 기본 패키지를 설치하고, `genfstab`으로 마운트 정보를 생성한다.

```bash
# 미러 정렬 (한국 미러 우선)
reflector --country Korea --sort rate --save /etc/pacman.d/mirrorlist

# 기본 시스템 설치
pacstrap -K /mnt base linux linux-firmware \
    btrfs-progs networkmanager vim sudo \
    intel-ucode  # 또는 amd-ucode

# fstab 생성
genfstab -U /mnt >> /mnt/etc/fstab

# chroot
arch-chroot /mnt
```

`-K`는 새 keyring을 초기화하여 GPG 키 관련 문제를 방지한다. `intel-ucode`는 CPU 마이크로코드 업데이트로, 보안과 안정성에 필수적이다.

### 시스템 설정

```bash
# 타임존
ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime
hwclock --systohc

# 로케일
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
echo "ko_KR.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# 호스트네임
echo "archbox" > /etc/hostname

# 루트 비밀번호 및 사용자 생성
passwd
useradd -m -G wheel -s /bin/zsh archgeek
passwd archgeek
EDITOR=vim visudo  # %wheel ALL=(ALL) ALL 주석 해제
```

## systemd-boot 설정

GRUB 대신 systemd-boot를 사용한다. 설정이 단순하고 부팅 속도가 빠르다.

```bash
# systemd-boot 설치
bootctl install

# 로더 설정
cat > /boot/loader/loader.conf << 'EOF'
default arch.conf
timeout 3
console-mode max
editor no
EOF

# 부트 엔트리
cat > /boot/loader/entries/arch.conf << 'EOF'
title   Arch Linux
linux   /vmlinuz-linux
initrd  /intel-ucode.img
initrd  /initramfs-linux.img
options root=UUID=<ROOT_UUID> rootflags=subvol=@ rw quiet
EOF
```

`<ROOT_UUID>`는 `blkid /dev/nvme0n1p2`로 확인한다. `rootflags=subvol=@`는 Btrfs 서브볼륨을 루트로 마운트하는 옵션이다.

### mkinitcpio 설정

Btrfs를 사용하므로 initramfs에 `btrfs` 모듈이 포함되어야 한다.

```bash
# /etc/mkinitcpio.conf 의 MODULES에 btrfs 추가 확인
# MODULES=(btrfs)

# initramfs 재생성
mkinitcpio -P
```

## 설치 마무리

chroot에서 나와 재부팅한다.

```bash
exit  # chroot 종료
umount -R /mnt
reboot
```

재부팅 후 NetworkManager를 활성화하고 AUR 헬퍼를 설치하면 기본 시스템이 완성된다. `systemctl enable --now NetworkManager`로 네트워크를 설정하고, `paru`나 `yay`로 AUR 패키지를 관리할 수 있다.
