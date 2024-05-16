---
layout: post
title: "Proxmox에서 GPU Passthrough하는 방법"
date: 2024-05-16 4:48:49 +0900
categories: homeserver
---

> 출처: [The ultimate gaming virtual machine on proxmox](https://www.youtube.com/watch?v=iWwdf66JpxE&ab_channel=DistroDomain)

## 스텝 1. 메인보드에 장착 된 GPU 확인하기

### 1. Proxmox 서버 콘솔에 접속 한다

> 이 스텝은 구글 크롬을 키고 Proxmox 콘솔에서 직접 해도 되고 `ssh`로 Proxmox에 `root`권한으로 접근을 해서 진행해도 된다.

### 2. 그래픽 카드가 잘 인식이 되는지 콘솔에 확인 한다

입력:

```bash
lspci | grep -i nvidia
```

> `nvidia`그래픽 카드가 아닌 다른거면 다른 키워드를 사용한다.

출력:

```bash
03:00.0 VGA compatible controller: NVIDIA Corporation GP107 [GeForce GTX 1050] (rev a1)
03:00.1 Audio device: NVIDIA Corporation GP107GL High Definition Audio Controller (rev a1)
```

제대로 인식이 되는걸로 확인이 된다. 그럼 넘어가자. 제대로 인식이 안되어있으면 메인 보드랑 연결부터 문제가있는 것으로 인식하면 되고 하드웨어 문제가 있는지 확인해 보도록 하자.

### 3. 그래픽 카드가 어떤 드라이버를 사용하고 있는지도 확인 한다

입력:

```bash
lspci -v
```

출력:

```bash
03:00.0 VGA compatible controller: NVIDIA Corporation GP107 [GeForce GTX 1050] (rev a1) (prog-if 00 [VGA controller])
        Subsystem: NVIDIA Corporation GP107 [GeForce GTX 1050]
        Physical Slot: 6
        Flags: bus master, fast devsel, latency 0, IRQ 27, NUMA node 0, IOMMU group 44
        Memory at fa000000 (32-bit, non-prefetchable) [size=16M]
        Memory at e0000000 (64-bit, prefetchable) [size=256M]
        Memory at f0000000 (64-bit, prefetchable) [size=32M]
        I/O ports at e000 [size=128]
        Expansion ROM at fb000000 [disabled] [size=512K]
        Capabilities: [60] Power Management version 3
        Capabilities: [68] MSI: Enable- Count=1/1 Maskable- 64bit+
        Capabilities: [78] Express Legacy Endpoint, MSI 00
        Capabilities: [100] Virtual Channel
        Capabilities: [128] Power Budgeting <?>
        Capabilities: [420] Advanced Error Reporting
        Capabilities: [600] Vendor Specific Information: ID=0001 Rev=1 Len=024 <?>
        Capabilities: [900] Secondary PCI Express
        Kernel driver in use: nouveau
        Kernel modules: nvidiafb, nouveau
```

저 `lspci -v` 명령어를 실행하면 현재 설치되어 있는 하드웨어 정보가 쭉 나오는데 그 중에서 `VGA compatible controller` 섹션을 찾아보면 저렇게 쭉 뜬다. 밑에서 두번째 줄인 `Kernel driver in use`가 우리가 확인해야 하는 부분인데 `nouveau` 드라이버로 설정 되어있는걸 확인할 수 있다. 이 설정을 `vfio` 드라이버로 만드는게 우리의 목표다.

### 4. 새로운 `vfio-pci.conf` 파일을 만들고 'vfio-pci'를 안에다가 작성해 준다

입력:

```bash
echo 'vfio-pci' > /etc/modules-load.d/vfio-pci.conf
```

결과:

1. `/etc/modules-load.d` 폴더에 `vfio-pci.conf` 파일이 새로 만들어진다.
2. 같은 폴더에서 `nano vfio-pci.conf` 명령어를 실행하여 파일을 열어보면 파일 안에 `vfio-pci` 라는 글이 입력되어 있다.
3. 이 파일이 존재하면 앞으로 kernel이 실행될 때 `vfio` 드라이버도 같이 실행하게 된다.

### 5. 그래픽 카드에 `vfio` 드라이버를 사용하도록 설정한다

#### 5-1. 이 작업을 수행하기 위해서 그래픽카드의 `id`를 확인해야 한다

입력:

```bash
lspci -nn | grep -i nvidia
```

출력:

```bash
03:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP107 [GeForce GTX 1050] [10de:1c81] (rev a1)
03:00.1 Audio device [0403]: NVIDIA Corporation GP107GL High Definition Audio Controller [10de:0fb9] (rev a1)
```

여기서 `id`는 `10de:1c81`와 `10de:0fb9`이다. 비디오랑 오디오를 둘다 사용 할 것이니 이 아이디 둘 다 써야한다.

10de:2488
10de:228b

#### 5-2. `vfio.conf`파일을 만들어서 비디오, 오디오 하드웨어에 `vfio`드라이버를 사용하도록 지정한다

입력:

```bash
echo 'options vfio-pci ids=10de:1c81, 10de:0fb9' > /etc/modprobe.d/vfio.conf
```

### 6. IOMMU 그룹 활성화 하기

IOMMU 그룹을 활성화하면 우리가 VM에 Passthrough할 하드웨어 그룹을 묶어서 사용할 수 있다. 인텔 CPU + Mainboard와 AMD CPU + Mainboard가 설정이 다르니 이 점 유의하자.

#### 6-1. `grub`파일에 키워드 추가하기

`grub` 파일을 연다.

입력:

```bash
nano /etc/default/grub
```

새로운 파일이 `nano` 에디터로 열리면서 다음과 같이 보인다.

출력:

```bash
GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
GRUB_CMDLINE_LINUX=""
```

`GRUB_CMDLINE_LINUX_DEFAULT` 라인에 `quiet` 다음 `intel_iommu=on` 키워드를 넣어준다.

파일을 저장하고 `nano` 에디터를 종료한다.

#### 6-2. 새로운 `grub`파일 적용하기

입력:

```bash
update-grub
```

출력:

```bash
root@pve1:/etc/default# update-grub
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.2.16-3-pve
Found initrd image: /boot/initrd.img-6.2.16-3-pve
Found memtest86+ 64bit EFI image: /boot/memtest86+x64.efi
Adding boot menu entry for UEFI Firmware Settings ...
done
```

#### 6-3. 컴퓨터 재부팅 및 BIOS 설정하기

컴퓨터를 재부팅하고 BIOS설정으로 들어가서 `CPU features` 또는 `CPU advanced settings` 같은 페이지로 들어가서.

1. `Intel VT-D Tech`
2. `Intel Virtualization Tech`
   등 CPU와 가상화 지원같은 기능을 활성화 시킨다. CPU종류마다 다 다를것이고 사용하는 메인보드와 칩셋에 따라 좀 다를것이니 이부분 유념하면서 몇가지 시도 해보자.

### 7. `vfio` 드라이버가 kernel에 잘 실행됐는지 확인하기

입력:

```bash
lsmod | grep vfio
```

출력:

```bash
root@pve1:/etc/default# lsmod | grep vfio
vfio_pci               16384  1
vfio_pci_core          94208  1 vfio_pci
irqbypass              16384  15 vfio_pci_core,kvm
vfio_iommu_type1       49152  1
vfio                   57344  7 vfio_pci_core,vfio_iommu_type1,vfio_pci
iommufd                73728  1 vfio
```

`vfio-pci`, `vfio_pci_core`, `vfio_iommu_type1`, `vfio`가 다 잘 뜨는걸 확인할 수 있다.

### 8. 3번에서 진행한 내용을 동일하게 수행해서 그래픽카드가 어떤 드라이버를 사용하는지 확인해보자

입력:

```bash
lspci -v
```

출력:

```bash
03:00.0 VGA compatible controller: NVIDIA Corporation GP107 [GeForce GTX 1050] (rev a1) (prog-if 00 [VGA controller])
        Subsystem: NVIDIA Corporation GP107 [GeForce GTX 1050]
        Physical Slot: 6
        Flags: bus master, fast devsel, latency 0, IRQ 27, NUMA node 0, IOMMU group 44
        Memory at fa000000 (32-bit, non-prefetchable) [size=16M]
        Memory at e0000000 (64-bit, prefetchable) [size=256M]
        Memory at f0000000 (64-bit, prefetchable) [size=32M]
        I/O ports at e000 [size=128]
        Expansion ROM at fb000000 [disabled] [size=512K]
        Capabilities: [60] Power Management version 3
        Capabilities: [68] MSI: Enable- Count=1/1 Maskable- 64bit+
        Capabilities: [78] Express Legacy Endpoint, MSI 00
        Capabilities: [100] Virtual Channel
        Capabilities: [128] Power Budgeting <?>
        Capabilities: [420] Advanced Error Reporting
        Capabilities: [600] Vendor Specific Information: ID=0001 Rev=1 Len=024 <?>
        Capabilities: [900] Secondary PCI Express
        Kernel driver in use: vfio-pci
        Kernel modules: nvidiafb, nouveau
```

`Kernel driver in use: vfio-pci` vfio 드라이버가 잘 설정된걸 확인할 수 있다.

### 9. IOMMU 그룹이 잘 설정되어있는지 확인하기 위한 스크립트를 만든다

입력:

```bash
nano /root/iommu_group.sh
```

결과:
`iommu_group.sh`이라는 스크립트 파일이 생성되어지고 에디터에 열린다.

그 안에 아래 코드를 복붙한다.

```bash
#!/bin/bash
shopt -s nullglob
for g in $(find /sys/kernel/iommu_groups/* -maxdepth 0 -type d | sort -V); do
    echo "IOMMU Group ${g##*/}:"
    for d in $g/devices/*; do
        echo -e "\t$(lspci -nns ${d##*/})"
    done;
done;
```

저장하고,

`.sh` 파일을 실행가능한 executable파일로 만들어 준다.

입력:

```bash
chmod +x /root/iommu_group.sh
```

그리고나서 방금 새로 작성한 스크립트를 실행한다.

입력:

```bash
./iommu_group.sh
```

출력:

```bash
IOMMU Group 44:
        03:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP107 [GeForce GTX 1050] [10de:1c81] (rev a1)
        03:00.1 Audio device [0403]: NVIDIA Corporation GP107GL High Definition Audio Controller [10de:0fb9] (rev a1)
```

동일한 `IOMMU Group` 에 VGA와 Audio가 추가되어있는걸 확인할 수 있다.

이걸로 Proxmox 설정은 완료.

추가로 Window VM 설치 및 설정이 있다.

다음 시리즈 원하면 [dearmannerism@gmail.com](mailto:dearmannerism@gmail.com)로 문의
