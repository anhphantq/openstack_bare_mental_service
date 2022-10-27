# GIẢI THÍCH MỘT SỐ THUẬT NGỮ

## OUT-OF-BAND (OOB) MANAGEMENT

Out-of-band management là một thuật ngữ trong networking nói đến việc:

    Truy cập và quản lý network tại một địa điểm remote thông qua một khu vực riêng khác với production network

Trong computing, một dạng của OOB được gọi là lights-out management (LOM) bao gồm việc sử dụng một kênh riêng để maintain thiết bị. Nó cho phép quản trị viên hệ thống monitor và quản lý server, thiết bị network dù server đó có đang power-on hay không

Một hệ thống quản lý từ xa hoàn chỉnh cho phép reboot, shutdown, powering on,...

### Implementation

Remote management có thể được enable trên rất nhiều máy tính bằng cách gắn thêm một remote management card. Một số loại motherboards đời mới thường có built-in remote management sẵn có.

Đối với ethernet-based OOB management, chúng có thể sử dụng các Ethernet connection phân biệt hoặc sử dụng mutilplexing để sử dụng chung.

## INTELLIGENT PLATFORM MANAGEMENT INTERFACE (IPMI)

IPMI define một tập các interface sử dụng bởi sysadmin với mục đích OOB management trong các hệ thống máy tính. Ví dụ, IPMI cung cấp cách quản lý một máy tính bị powerred-off bằng việc sử dụng một network connection với phần cứng. Một use case khác đó là khi cần cài một hệ điều thành từ xa.

### IPMI Components

IPMI sub-system bao gồm một main controller được gọi là Baseboard management controller (BMC) và một số management controllers (còn gọi là satellite controller) phân tán ra các module của hệ thống. Satellite controllers

## PREBOOT EXECUTION ENVIRONMENT (PXE)

PXE là một phần của Wired for Management (WfM) specification phát triển bởi Intel và Microsoft. PXE cho phép BIOS của máy tính và Network Interface Card (NIC) boot từ một nơi khác thông qua network.

## DYNAMIC HOST CONFIGURATION PROTOCOL (DHCP)

DHCP là một protocol được sử dụng trên IP networks với mục tích cấp địa chỉ IP động cho các network interfaces. Trong PXE, BIOS sẽ sử dụng DHCP để lấy được IP cho network interface và địa chỉ của server lưu network bootstrap program (NBP)

## NETWORK BOOTSTRAP PROGRAM (NBP)

NBP có vai trò như GRUB hay LILO - các boot loaders có nhiệm vụ load OS kernel vào trong RAM, tuy nhiên việc load kernel này thông qua network

## TRIVIVAL FILE TRANSFER PROTOCOL (TFTP)

TFTP là một giao thức file transfer đơn giản được sử dụng để transfer boot files giữa các máy trong mạng. Trong PXE, TFTP được sử dụng để tải các file NBP sử dụng thông tin IP từ DHCP server
