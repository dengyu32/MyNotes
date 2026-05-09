### Ubuntu 耗电问题

#### 耗电检查
1. 电池状态	upower -i /org/freedesktop/UPower/devices/battery_BAT0
2. 检查是否独显	nvidia-smi

#### 省电步骤
1. 开启节能模式
2. 安装 tlp （只在 linux 生效）
	sudo apt install tlp 
	sudo tlp start
	sudo tlp-stat -s
	sudo systemctl enable tlp.service 开机自启
3. 切换核显模式	sudo prime-select intel 重启 reboot 

