# Chuẩn bị
* Ubuntu (phiên bản >18.0)
* Windows server 2012

# Triển khai hệ thống phát hiện xâm nhập snort
## Cấu hình máy ảo
### Vmware:
Set up dải mạng 10.0.0.0/24
### Cấu hình ubuntu
* Cấu hình phần cứng: Cổng NAT
### Cấu hình windows server
* Cấu hình phần cứng: vmnet2
* Cấu hình mạng: 
    * IP: 10.0.0.20
    * Subnetmark: 255.255.255.0
    * 10.0.0.2
# Cài đặt snort
### **Bước 1. Cài đặt các gói phần mềm bổ trợ**
Snort có bốn phần mềm bổ trợ yêu cầu phải cài đặt trước:
* pcap (libpcap-dev)
* PCRE (libpcre3-dev)
* Libdnet (libdumbnet-dev)
* DAQ

Khởi động máy ảo Ubuntu, mở cửa sổ dòng lệnh bắt đầu cài đặt
> sudo apt-get update
>
> sudo apt-get install -y build-essential
>
>sudo apt-get install -y libpcap-dev libpcre3-dev libdumbnet-dev
>
>sudo apt-get install -y bison flex

Tạo thư mục chứa mã nguồn Snort và các phần mềm liên quan:
> mkdir ~/snort-src
>
>cd ~/snort-src/daq-2.0.7
>
>sudo ./configure
>
>sudo make
>
>sudo make install
>
>sudo apt-get install -y zlib1g-dev liblzma-dev openssl libssl-dev

### **Bước 2. Cài đặt Snort**
>Cd ~/snort/snort-src snort-2.9.17
>
>sudo ./configure --enable-sourcefire --disable-open-appid 
>
>sudo make
>
>sudo make install
>
>sudo ldconfig
>
>sudo ln -s /usr/local/bin/snort /usr/sbin/snort
Sau khi cài xong, chạy lệnh sau để kiểm tra:
> snort -V
### **Bước 3. Cấu hình Snort chạy ở chế độ phát hiện xâm nhập mạng**
Tạo các thư mục cho Snort:
>sudo mkdir /etc/snort/rules/iplists
>
>sudo mkdir /etc/snort/preproc_rules
>
>sudo mkdir /usr/local/lib/snort_dynamicrules
>
>sudo mkdir /etc/snort/so_rules
Tạo các tệp tin chứa tập luật cơ bản cho Snort:
>sudo touch /etc/snort/rules/iplists/black_list.rules
>
>sudo touch /etc/snort/rules/iplists/white_list.rules
>
>sudo touch /etc/snort/rules/local.rules/
>
>sudo touch /etc/snort/sid-msg.map

Tạo thư mục chứa log:
>sudo mkdir /var/log/snort
>
>sudo mkdir /var/log/snort/archived_logs

Tạo các bản sao tệp tin cấu hình của Snort
cd snort_src/snort-2.9.12/etc
>sudo cp \*.conf* /etc/snort
>
>sudo cp *.map /etc/snort
>
>sudo cp *.dtd /etc/snort
>
>cd ~/snort_src/snort-2.9.12/src/dynamic-preprocessors/build/usr/local/lib/snort_dynamicpreprocessor/
>
>sudo cp * /usr/local/lib/snort_dynamicpreprocessor/

Chỉnh sửa các tham số trong tệp tin:
>/etc/snort/snort.conf
Dòng 45, chỉnh sửa địa chỉ IP cho mạng nội bộ.
>ipvar HOME_NET 10.0.0.0/24
>
>ipvar EXTERNAL_NET !$HOME_NET
>
Tìm đến các dòng sau chỉnh sửa đường dẫn chứa tập luật.
>var RULE_PATH /etc/snort/rules (dòng 104)
>
>var SO_RULE_PATH /etc/snort/so_rules (dòng 105)
>
>var PREPROC_RULE_PATH /etc/snort/preproc_rules (dòng 106)
>
>var WHITE_LIST_PATH /etc/snort/iplists (dòng 113)
>
>var BLACK_LIST_PATH /etc/snort/iplists (dòng 114)
>
Đường dẫn tập luật:
>include $Rule_PATH/local.rules
>
Đổi tất cả include về comment trừ "include $Rule_PATH/local.rules"

**Bước4: kiểm tra sự hoạt động của snort**
>Sudo snort -i ens33 -c /etc/snort/snort.conf -T
>
Dòng thông báo *snort successfully validated the configution!* - cài đặt thành công
Giao diện thống kê của Snort:
>sudo snort -i ens33 -c /etc/snort/snort.conf
# Các kịch bản tấn công và phát hiện
## kịch bản 1: Phát hiện Ping
### Bước 1: Tấn công
Ping đến server trên máy vật lí:
Mở command prompt
>ping 10.0.0.2 -t
### Bước 2: Phát hiện tấn công
Mở luật phát hiện ping
>Sudo nano /etc/snort/snort.conf
>
Bỏ dấu # ở đầu để mở tập luật
>include $RULE_PATH/icmp.rules
>
Mã nguồn của luật phát hiện dò quét ICMP của Snort như sau:
>Alert icmp any any -> (msg:”Phat hien Ping”; sid:1000001; rev:1;)
>
