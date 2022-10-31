# CÀI ĐẶT VIRTUAL MACHINE THÔNG QUA PXE VÀ KICKSTART/PRESEED

## 1. Kiến thức tổng quan về PXE

PXE là một phần của Wired for Management (WfM) specification phát triển bởi Intel và Microsoft. PXE cho phép BIOS của máy tính và Network Interface Card (NIC) boot từ một nơi khác thông qua network.

PXE được thiết kế ra nhằm 3 mục đích chính

- Cài đặt/ xoá OS remote
- Remote emergency boot
- Remote network boot

**Sau đây, ta sẽ tập trung chính vào việc cài đặt OS remote**

Để boot được hệ điều hành sử dụng PXE ta cần một số thành phần sau:

- DHCP Server
- Client với BIOS w/PXE (BIOS có PXE)
- TFTP server để tải file NBP
- Server cung cấp images cho NBP load kernel

Sau đây, ta sẽ tìm hiểu các bước để cài đặt OS remote sử dụng PXE.

### 1.1. Retrieving IP using DHCP

Trước tiên, NIC cần lấy được IP của mình, của NBP server để có thể tiến hành booting.

Trước tiên, BIOS sẽ phải lấy IP của NIC thông qua DHCP. Sau đó lấy IP của TFTP server rồi tiến hành tải file NBP.

![ip_dhcp](images/dhcp_proxydhcp_tftp.png)

Ta có thể kiểm chứng qua trình này:

![dhcp_example](images/dhcp_example.png)

### 1.2. Booting process deep dive

Qua trình booting sẽ xảy ra như thế nào?

Tương tự với quá trình booting truyền thống, trước tiên firmware code ở trong ROM sẽ được lấy ra, tuy nhiên, đây sẽ không phải firmware thường mà là PXE ROM code. Chúng sẽ có thêm một số cơ chế để truyền thông qua mạng nhằm lấy được bootloader (thay vì như bình thường là ta lấy bootloader trên ổ đĩa). Bootloader này có tên là NBP.

![pxe_rom](images/pxe_rom.png)

Sau khi BIOS lấy được bootloader NBP. BIOS sẽ dừng hoạt động, công việc từ giờ trở đi sẽ do NBP phụ trách. NBP cũng như toàn bộ các bootloader khác, có nhiệm vụ load kernel của OS vào trong RAM rồi kết thúc hoạt động của mình, trao lại hoạt động cho OS kernel xử lý.

**More on Linux booting**

NBP sẽ chỉ thực hiện load một minimal Linux kernel và một số thành phần khác có khả năng load các phần còn lại.

Sau khi minimal kernel được load vào RAM. Kernel đồng thời sẽ load initrd (Init Ram Disk). Init Ram Disk là một Ram File System mà kernel sẽ sử dụng như root. Sau đó /sbin/init sẽ được execute. /sbin/init sẽ chuẩn bị để mount được "real" root FS (device type, device drives, file system, ..) và các distribution media như CD-ROM, network,... Sau đó /sbin/init sẽ load một số kernel modules, sau đó thực hiện mount root FS và initrd sẽ được unmount. Cuối cùng là thực hiện các task để cài đặt OS còn lại.

Mục đích chính của initrd là để có thể module hoá các thành phần cần thiết của linux kernel. Thay vì phải đóng gói một kernel kích thước lớn, initrd giúp ta load các module cần thiết cho kernel dễ dàng.

## 2. Cài đặt OS với PXE và Kickstart/Preseed

Như đã mô tả ở trên, ta set up một DHCP server để cung cấp IP của NBP server cho BIOS. Trước tiên ta sẽ sử dụng libvirt để tạo một mạng ảo sử dụng trong Lab này. Trước tiên ta tạo file cấu hình mạng

    vim br-pxe-net.xml

Sau đó cấu hình mạng như sau

    ```xml
    <network>
    <name>br-pxe</name>
    <forward mode='nat'>
        <nat>
        <port start='1024' end='65535'/>
        </nat>
    </forward>
    <bridge name='br-pxe' stp='on' delay='0'/>
    <ip address='192.168.177.1' netmask='255.255.255.0'>
    </ip>
    </network>
    ```

Sau đó ta tạo virtual network bằng virsh:

    sudo virsh net-define --file br-pxe-net.xml

Enable autostart cho mạng này:

    sudo virsh net-autostart br-pxe

Sau đó start mạng này:

    sudo virsh net-start br-pxe

Kiểm tra tình trạng mạng xem đã active chưa:

    parallels@ubuntu-linux-22-04-desktop:~$ sudo virsh net-list
    [sudo] password for parallels:
    Name      State    Autostart   Persistent
    --------------------------------------------
    br-pxe    active   yes         yes
    default   active   yes         yes

restorecon -Rv /tftproot

acpid

    sudo virt-install \
    --name server-fedora-32-x86_64 \
    --ram 2048 \
    --vcpus 2 \
    --disk path=/var/lib/libvirt/images/server-fedora-32-x86_64.qcow2 \
    --os-type='linux' \
    --os-variant='fedora32' \
    --network=bridge=br-pxe \
    --graphics none \
    --serial pty \
    --console pty \
    --boot hd \
    --import \
    --arch x86_64

    sudo virt-install \
    --name server-debian-11-amd64 \
    --ram 2048 \
    --vcpus 2 \
    --os-type='linux' \
    --os-variant='debian11' \
    --network=bridge=br-pxe \
    --graphics none \
    --serial pty \
    --console pty \
    --boot hd \
    --arch x86_64 \
    --pxe
