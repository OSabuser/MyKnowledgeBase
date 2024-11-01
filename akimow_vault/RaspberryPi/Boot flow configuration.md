1. **Установка custom boot logo**
	`cd /usr/share/plymouth/themes/pix 
	`sudo mv splash.png splash_default.png` 
	`cd `
	`cd Desktop `
	`sudo cp splash.png /usr/share/plymouth/themes/pix``

2. **Обновление изображения**
	`sudo plymouth-set-default-theme --rebuild-initrd pix`  

3. **Отключение rainbow splash screen**
	Добавить в *boot/config.txt*: `disable_splash=1`

4. **Скрипт для автозапуска пользовательских приложений**  
	Применяется для всех пользователей после входа в систему:
	`etc/xdg/lxsession/LXDE-pi/autostart`  
	Для текущего пользователя:
	`/home/<user_name>/.config/lxsession/LXDE-pi/autostart``  
	***Если файл autostart существует в /home/<user_name>/.config/lxsession/LXDE-pi, то системный autostart игнорируется*** 
	[Pi Forum](https://forums.raspberrypi.com/viewtopic.php?t=294014)

5. **Скрыть курсор и панель задач**
	 Закомментировать строчки в `etc/xdg/lxsession/LXDE-pi/autostart` :
	 `@pcmanfm --desktop ` 
	 `@lxpanel --profile`  
	 Не показывать курсор:
	 - В файле *etc/lightdm/lightdm.conf* ниже секции [Seat*]:
		 `xserver-command = X -nocursor`
	 - В файле *usr/bin/startx*:
		 `defaultserverargs="-nocursor"`