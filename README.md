# Keep_online

KẾT QUẢ SƯU TẦM: https://docs.google.com/document/d/1sBzt83F2AkpmU2RQe1IxXHHWAtD-2FhwyKokEN9wuu4/edit?usp=sharing
BÁO CÁO VỀ TÌNH HÌNH THIÊN TAI: https://docs.google.com/document/d/1RG-L6HT8Pr_6X6SFSLxQN83EWqddIPeuPCPlI9Bb7W8/edit?usp=sharing
Bài tuyên truyền: https://docs.google.com/document/d/15CIlSen5XKwtGT46kkUcrh5OrX9jx3oNbWqOUqU9MIw/edit?usp=sharing
```
#!/bin/bash

# Script tự động cài đặt driver cho Arch Linux trong GNOME Boxes
# Chạy với quyền user thường, script sẽ tự động sudo khi cần

set -e  # Thoát nếu có lỗi

echo "=== SCRIPT TỰ ĐỘNG CÀI ĐẶT DRIVER CHO ARCH LINUX TRONG BOXES ==="
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

# Cài đặt driver GPU cơ bản
print_status "Cài đặt driver GPU..."
sudo pacman -S --noconfirm xf86-video-qxl xf86-video-intel mesa
print_success "Driver GPU đã được cài đặt"

# Cài đặt Spice guest agent (quan trọng cho Boxes)
print_status "Cài đặt Spice guest agent..."
sudo pacman -S --noconfirm spice-vdagent
print_success "Spice guest agent đã được cài đặt"

# Kích hoạt và khởi động spice-vdagentd
print_status "Kích hoạt dịch vụ Spice..."
sudo systemctl enable spice-vdagentd
sudo systemctl start spice-vdagentd
print_success "Dịch vụ Spice đã được kích hoạt"

# Cài đặt driver âm thanh
print_status "Cài đặt driver âm thanh..."
sudo pacman -S --noconfirm alsa-utils pulseaudio pulseaudio-alsa pavucontrol
print_success "Driver âm thanh đã được cài đặt"

# Cài đặt NetworkManager
print_status "Cài đặt NetworkManager..."
sudo pacman -S --noconfirm networkmanager
sudo systemctl enable NetworkManager
print_success "NetworkManager đã được cài đặt và kích hoạt"

# Cài đặt các gói hỗ trợ khác
print_status "Cài đặt các gói hỗ trợ..."
sudo pacman -S --noconfirm \
    xorg-xrandr \
    xorg-setxkbmap \
    xclip \
    curl \
    wget \
    git \
    vim \
    nano \
    htop \
    neofetch
print_success "Các gói hỗ trợ đã được cài đặt"

# Hỏi người dùng có muốn cài desktop environment không
echo ""
print_status "Bạn có muốn cài đặt desktop environment không?"
echo "1) GNOME (khuyến nghị)"
echo "2) KDE Plasma"
echo "3) XFCE (nhẹ)"
echo "4) Bỏ qua"
echo ""
read -p "Lựa chọn của bạn (1-4): " desktop_choice

case $desktop_choice in
    1)
        print_status "Cài đặt GNOME Desktop..."
        sudo pacman -S --noconfirm gnome gnome-extra
        sudo systemctl enable gdm
        print_success "GNOME Desktop đã được cài đặt"
        ;;
    2)
        print_status "Cài đặt KDE Plasma Desktop..."
        sudo pacman -S --noconfirm plasma kde-applications
        sudo systemctl enable sddm
        print_success "KDE Plasma Desktop đã được cài đặt"
        ;;
    3)
        print_status "Cài đặt XFCE Desktop..."
        sudo pacman -S --noconfirm xfce4 xfce4-goodies lightdm lightdm-gtk-greeter
        sudo systemctl enable lightdm
        print_success "XFCE Desktop đã được cài đặt"
        ;;
    4)
        print_warning "Bỏ qua cài đặt desktop environment"
        ;;
    *)
        print_warning "Lựa chọn không hợp lệ, bỏ qua cài đặt desktop environment"
        ;;
esac

# Tạo file cấu hình cho Spice
print_status "Tạo file cấu hình tối ưu..."
sudo mkdir -p /etc/X11/xorg.conf.d/
sudo tee /etc/X11/xorg.conf.d/20-spice.conf > /dev/null <<EOF
Section "Device"
    Identifier "qxl"
    Driver "qxl"
    Option "ENABLE_SURFACES" "False"
EndSection
EOF
print_success "File cấu hình đã được tạo"

# Thêm user vào các group cần thiết
print_status "Thêm user vào các group cần thiết..."
sudo usermod -aG audio,video,input,storage $USER
print_success "User đã được thêm vào các group"

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
echo "  ✓ Driver GPU (QXL)"
echo "  ✓ Spice guest agent"
echo "  ✓ Driver âm thanh"
echo "  ✓ NetworkManager"
echo "  ✓ Các gói hỗ trợ cơ bản"
if [ "$desktop_choice" != "4" ]; then
    echo "  ✓ Desktop environment"
fi
echo ""
print_warning "Cần khởi động lại để áp dụng tất cả thay đổi"
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
print_success "Script hoàn tất! Chúc bạn sử dụng Arch Linux vui vẻ!"
```
