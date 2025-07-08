# Keep_online

https://www.notion.so/maingocchau/H-ng-d-n-c-i-t-Fcitx5-tr-n-Linux-2132ce9c2cd08084b1e3e053d12dce4b?source=copy_link
KẾT QUẢ SƯU TẦM: https://docs.google.com/document/d/1sBzt83F2AkpmU2RQe1IxXHHWAtD-2FhwyKokEN9wuu4/edit?usp=sharing
BÁO CÁO VỀ TÌNH HÌNH THIÊN TAI: https://docs.google.com/document/d/1RG-L6HT8Pr_6X6SFSLxQN83EWqddIPeuPCPlI9Bb7W8/edit?usp=sharing
Bài tuyên truyền: https://docs.google.com/document/d/15CIlSen5XKwtGT46kkUcrh5OrX9jx3oNbWqOUqU9MIw/edit?usp=sharing
```
#!/bin/bash

# Script tự động cài đặt driver auto-scaling cho Arch Linux trong GNOME Boxes
# Chỉ tập trung vào display scaling và spice guest agent

set -e  # Thoát nếu có lỗi

echo "=== SCRIPT AUTO-SCALING CHO ARCH LINUX TRONG BOXES ==="
echo ""

# Màu sắc cho output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

print_status() {
    echo -e "${BLUE}[INFO]${NC} $1"
}

print_success() {
    echo -e "${GREEN}[SUCCESS]${NC} $1"
}

print_warning() {
    echo -e "${YELLOW}[WARNING]${NC} $1"
}

print_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# Kiểm tra kết nối internet
print_status "Kiểm tra kết nối internet..."
if ! ping -c 1 google.com &> /dev/null; then
    print_error "Không có kết nối internet. Vui lòng kiểm tra lại."
    exit 1
fi
print_success "Kết nối internet OK"

# Cập nhật hệ thống
print_status "Cập nhật hệ thống..."
sudo pacman -Syu --noconfirm
print_success "Cập nhật hệ thống hoàn tất"

# Cài đặt driver QXL cho auto-scaling
print_status "Cài đặt driver QXL..."
sudo pacman -S --noconfirm xf86-video-qxl mesa
print_success "Driver QXL đã được cài đặt"

# Cài đặt Spice guest agent (quan trọng nhất cho auto-scaling)
print_status "Cài đặt Spice guest agent..."
sudo pacman -S --noconfirm spice-vdagent
print_success "Spice guest agent đã được cài đặt"

# Kích hoạt và khởi động spice-vdagentd
print_status "Kích hoạt dịch vụ Spice..."
sudo systemctl enable spice-vdagentd
sudo systemctl start spice-vdagentd
print_success "Dịch vụ Spice đã được kích hoạt"

# Cài đặt xorg-xrandr cho resolution scaling
print_status "Cài đặt xorg-xrandr..."
sudo pacman -S --noconfirm xorg-xrandr
print_success "xorg-xrandr đã được cài đặt"

# Tạo file cấu hình QXL tối ưu cho auto-scaling
print_status "Tạo file cấu hình auto-scaling..."
sudo mkdir -p /etc/X11/xorg.conf.d/
sudo tee /etc/X11/xorg.conf.d/20-qxl.conf > /dev/null <<EOF
Section "Device"
    Identifier "qxl"
    Driver "qxl"
    Option "ENABLE_SURFACES" "False"
EndSection

Section "Monitor"
    Identifier "Virtual-1"
    Option "PreferredMode" "1024x768"
EndSection

Section "Screen"
    Identifier "Screen0"
    Device "qxl"
    Monitor "Virtual-1"
    DefaultDepth 24
    SubSection "Display"
        Depth 24
        Modes "1920x1080" "1680x1050" "1280x1024" "1280x960" "1024x768" "800x600"
    EndSubSection
EndSection
EOF
print_success "File cấu hình auto-scaling đã được tạo"

# Thêm user vào group video
print_status "Thêm user vào group video..."
sudo usermod -aG video $USER
print_success "User đã được thêm vào group video"

# Tạo script khởi động tự động cho spice-vdagent
print_status "Tạo script khởi động tự động..."
mkdir -p ~/.config/autostart/
cat > ~/.config/autostart/spice-vdagent.desktop <<EOF
[Desktop Entry]
Type=Application
Name=Spice VD Agent
Exec=spice-vdagent
NoDisplay=true
X-GNOME-Autostart-enabled=true
EOF
print_success "Script khởi động tự động đã được tạo"

echo ""
print_success "=== CÀI ĐẶT HOÀN TẤT ==="
echo ""
print_status "Các tính năng đã được cài đặt:"
echo "  ✓ Driver QXL (auto-scaling)"
echo "  ✓ Spice guest agent"
echo "  ✓ xorg-xrandr"
echo "  ✓ Cấu hình auto-scaling"
echo ""
print_warning "Cần khởi động lại để áp dụng tất cả thay đổi"
echo ""
print_status "Sau khi khởi động lại, màn hình sẽ tự động scale theo cửa sổ Boxes"
echo ""
read -p "Bạn có muốn khởi động lại ngay bây giờ không? (y/N): " reboot_choice
if [[ $reboot_choice =~ ^[Yy]$ ]]; then
    print_status "Khởi động lại hệ thống..."
    sudo reboot
else
    print_warning "Hãy nhớ khởi động lại sau để áp dụng tất cả thay đổi"
    echo "Chạy lệnh: sudo reboot"
fi

echo ""
print_success "Script hoàn tất! Màn hình sẽ tự động scale sau khi reboot!"
```
