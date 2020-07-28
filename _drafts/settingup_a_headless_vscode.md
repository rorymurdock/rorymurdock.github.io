Install Ubuntu 20.04




```shell
sudo apt-get install -y gnome-keyring desktop-file-utils x11-utils

ver=$(curl https://packages.microsoft.com/ubuntu/18.04/prod/pool/main/v/vso/ | tail -n 3 | head -n 1 | sed -e 's/<a /\n<a /g' | sed -e 's/<a .*href=['"'"'"]//' -e 's/["'"'"'].*$//' -e '/^$/ d')

echo "Found $ver"

curl -O https://packages.microsoft.com/ubuntu/18.04/prod/pool/main/v/vso/$ver

sudo dpkg -i vso_*.deb

vso start
```