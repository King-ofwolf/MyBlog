# ubuntu16.04下sublime text 3之安装和配置

## 安装方法

### 使用ppa安装

```bash
sudo add-apt-repository ppa:webupd8team/sublime-text-3
sudo apt-get update
sudo apt-get install sublime-text-installer
```
卸载sublimetext命令：
```bash
sudo apt-get remove sublime-text-installer
```

### 使用deb包安装

下载地址形如: <https://download.sublimetext.com/files/sublime-text_build-版本号_amd64.deb>  
3126的下载地址为:<https://download.sublimetext.com/files/sublime-text_build-3126_amd64.deb>  
不过貌似必须FQ才能下载  
例如:
```bash
sudo dpkg -i sublime-text_build-3126_amd64.deb
```
### 通过apt安装[网络不稳定，可能需要多次尝试]

参考: <http://www.sublimetext.com/docs/3/linux_repositories.html#apt>
```bash

wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -	#安装GPG key:
sudo apt-get install apt-transport-https	#确保apt能够使用https源
echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list 	#使用稳定版源，此处空格一个都不能少
sudo apt-get update	#更新apt源并安装sublime-text
sudo apt-get install sublime-text
```
## 打开命令

```bash
subl
```

## 在Help里输入license key(license key可以网上自己搜索)

```license
—– BEGIN LICENSE —–
Michael Barnes
Single User License
EA7E-821385
8A353C41 872A0D5C DF9B2950 AFF6F667
C458EA6D 8EA3C286 98D1D650 131A97AB
AA919AEC EF20E143 B361B1E7 4C8B7F04
B085E65E 2F5F5360 8489D422 FB8FC1AA
93F6323C FD7F7544 3F39C318 D95E6480
FCCC7561 8A4A1741 68FA4223 ADCEDE07
200C25BE DBBC4855 C4CFB774 C5EC138C
0FEC1CEF D9DCECEC D3A5DAD1 01316C36
—— END LICENSE ——
```

## 安装Packeage Control

可参考: <https://packagecontrol.io/installation>  
在终端输入subl打开sublime text3, 使用快捷键ctrl + \`(ESC下面的键),在弹出的控制台中输出以下代码:
```python
import urllib.request,os,hashlib; h = 'df21e130d211cfc94d9b0905775a7c0f' + '1e3d39e33b79698005270310898eea76'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```
或者下载package control包，直接放入installed package/目录下。<https://packagecontrol.io/Package%20Control.sublime-package>

## 关闭自动更新

Preferences -> Settings-User，在{}输入 ,"update_check":false

## 设置一个tab键对应4个空格

Preferences -> Settings-User，在{}输入 
```json
"tab_size": 4,
"translate_tabs_to_spaces": true
```

## 解决中文无法输入问题

```bash
git clone https://github.com/lyfeyaj/sublime-text-imfix.git
cd sublime-text-imfix/
sudo cp lib/libsublime-imfix.so /opt/sublime_text/
sudo cp src/subl /usr/bin/
```
关闭sublime的所有窗口,然后在终端输入subl,此时就可以输入中文了
```bash
hupeng@hupeng-vm:~$ which subl
/usr/bin/subl
hupeng@hupeng-vm:~$ cat /usr/bin/subl   #查看一下subl命令做了什么
#!/bin/sh

export LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so
exec /opt/sublime_text/sublime_text "$@"hupeng@hupeng-vm:~$
```

最终解决方法:
```bash
sudo vi /usr/share/applications/sublime_text.desktop 
#修改三个Exec
Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text %F"
Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text -n"
Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text --command new_file"
```
/usr/share/applications/sublime_text.desktop内容如下:
```bash
[Desktop Entry]
Version=1.0
Type=Application
Name=Sublime Text
GenericName=Text Editor
Comment=Sophisticated text editor for code, markup and prose
Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text %F"
Terminal=false
MimeType=text/plain;
Icon=sublime-text
Categories=TextEditor;Development;
StartupNotify=true
Actions=Window;Document;

[Desktop Action Window]
Name=New Window
Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text -n"
OnlyShowIn=Unity;

[Desktop Action Document]
Name=New File
Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text --command new_file"
OnlyShowIn=Unity;
```

## 彻底卸载sublimetext

```bash
sudo apt-get remove --purge sublime-text
sudo apt-get autoremove

# 删除配置文件
sudo rm -rf /opt/sublime_text/
rm -rf ~/.config/sublime-text-3/
# 查找相关文件并手动删除
sudo find / -name sublime*

sudo updatedb
locate sublime-text
```


改编自<https://www.cnblogs.com/hupeng1234/p/6957623.html>