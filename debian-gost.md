Шифрование и ключи GOST в Linux для госпорталов.
===

КриптоПро
---

КриптоПро потребуется почти для всех
КриптоПро CSP доступно для скачивания после ркгистрации https://www.cryptopro.ru/download?pid=1417. 3 месяца бесплатно.

Качаю КриптоПро CSP 5.0 для Linux (x64, deb)

```
tar xf  linux-amd64_deb.tgz 
cd linux-amd64_deb/
sudo ./install_gui.sh 
```

Выбираем модули. Ставь все галочки. Лицензию можно вбить позже этим же скриптом. 

Подпись у меня на RutokenLite. Драйверы для Рутокен Lite в современных операционных системах GNU\Linux не требуются (версия libccid не ниже 1.4.2). 

```
sudo apt install libusb-1.0-0-dev pcscd pcsc-tools
```

Можно вставлять токен и посмотреть на него в "Инструменты Криптопро".

Нажмите кнопку "Установить сертификат"




rtPKCS11ECP
---

Некоторые службы (госуслуги) можно заставить работать без КриптоПро. 
Для рутокен тогда понадобится библиотека https://www.rutoken.ru/support/download/pkcs/ для поддержки ГОСТ в PKCS.




ХромиумГост
---


Потребуется для сайтов с транспортом на ГОСТ пропатченный Хром с внешним шифрованием. Нужен КриптоПро для работы.
Качать тут https://github.com/deemru/chromium-gost/releases/ . Для Linux сейчас не собирается последний релиз, смотри предпоследний.

```
dpkg -i chromium-gost-77.0.3865.90-linux-amd64.deb
apt -f install
```


Госуслуги
---

Подойдет любой браузер. Сначала попросит установить плагин https://ds-plugin.gosuslugi.ru/plugin/upload/Index.spr

```
sudo dpkg -i IFCPlugin-x86_64.deb 
```

И не написанно на этой страничке, но ещё понадобится расширение https://chrome.google.com/webstore/detail/%D1%80%D0%B0%D1%81%D1%88%D0%B8%D1%80%D0%B5%D0%BD%D0%B8%D0%B5-%D0%B4%D0%BB%D1%8F-%D0%BF%D0%BB%D0%B0%D0%B3%D0%B8%D0%BD%D0%B0-%D0%B3%D0%BE/pbefkdcndngodfeigfdgiodgnmbgcfha?hl=ru&authuser=1

IFCP не видит КриптоПро 5 при стандартных настройках, возможно увидит другие CSP, но надо донастроить


sudo nano /etc/ifc.cfg
```
params =
(
  {
    name  = "CryptoPro CSP";
    alias = "cprocsp";
    type  = "pkcs11";
    alg   = "gost2001";
    model = "CPPKCS 3";
    lib_linux   = "/opt/cprocsp/lib/amd64/libcppkcs11.so";
  },
  {
         name = "CPPKCS11_2012_256";
         alias = "CPPKCS11_2012_256";
         type = "pkcs11";
         alg = "gost2012_256";
         model = "CPPKCS 3";
         lib_linux = "/opt/cprocsp/lib/amd64/libcppkcs11.so";
  },
  {
         name = "CPPKCS11_2012_512";
         alias = "CPPKCS11_2012_512";
         type = "pkcs11";
         alg = "gost2012_512";
         model = "CPPKCS 3";
         lib_linux = "/opt/cprocsp/lib/amd64/libcppkcs11.so";
  }
)
```
Ослальные криптопровайдеры выкинул из плагина.

ХромиумГост IPFC не видит. FIX

```
sudo ln -s /etc/opt/chrome/native-messaging-hosts/ru.rtlabs.ifcplugin.json /etc/chromium/native-messaging-hosts/
```


Сайт налоговой
---

Предлагает скачать cades.

```
tar xf cades_linux_amd64.tar.gz
cd cades_linux_amd64/
dpkg -i cprocsp-pki-cades_2.0.0-1_amd64.deb cprocsp-pki-plugin_2.0.0-1_amd64.deb
```

И расширение браузера https://chrome.google.com/webstore/detail/cryptopro-extension-for-c/iifchhfnnmpdbibifmljnfjhpififfog?hl=ru&authuser=1


СБИС
---

Поддержка линукс сейчас на внутреннем тестировании. Но установить можно.
https://chrome.google.com/webstore/detail/sbis-plugin-extension/pbcgcpeifkdjijdjambaakmhhpkfgoec?hl=ru&authuser=1

http://update.sbis.ru/Sbis3Plugin/master/linux/sbis3plugin-setup.tar.xz

```
tar xf sbis3plugin-setup.tar.xz
cd SBIS3Plugin/
sudo mkdir -p /usr/share/pki/ca-trust-source/anchors/
sudo ln -s /sbin/update-ca-certificates /sbin/update-ca-trust
sudo ./install.sh

sudo openssl x509 -inform DER -outform PEM -in /usr/share/pki/ca-trust-source/anchors/sbis.pem -out /usr/share/ca-certificates/sbis.crt

sudo dpkg-reconfigure ca-certificates
sudo update-ca-certificates
sudo mkdir -p /etc/pki/tls/certs/
sudo ln -s /etc/ssl/certs/ca-certificates.crt /etc/pki/tls/certs/ca-bundle.crt
``

Для дебиан 9

sudo ln -s /bin/mkdir /usr/bin/mkdir
sudo ln -s /bin/chmod /usr/bin/chmod


Переходим на сайт сбиса. Пробуем войти с подписью.
Переходим в инспектор и ставим точку останова после SBIS3.Plugin/Source/Environment/PluginMigrate на isUseNewPlugin

в файле https://online.sbis.ru/auth/resources/SBIS3.Plugin/sbis3plugin_min.package.min.js?x_module=19.613-20

перезагрузить страницу и на точке останова добавить u.detection в Watch и поставить isMac = true

добавить сертификат /usr/share/ca-certificates/sbis.crt в центры сертификации в браузере chrome://settings/certificates  


Посмотреть ещё
---

gnupg-pkcs11-scd
gnome-keyring-pkcs11
