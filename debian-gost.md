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
Виндовой бесплатной КриптоПро Рутокен CSP нет. Пока не настраивал.



ХромиумГост
---


Потребуется для сайтов с транспортом на ГОСТ пропатченный Хром с внешним шифрованием. Нужен КриптоПро для работы.
Качать тут https://github.com/deemru/chromium-gost/releases/ . <s>Для Linux сейчас не собирается последний релиз, смотри предпоследний.</s>

```
dpkg -i chromium-gost-85.0.4183.121-linux-amd64.deb
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


Сайт налоговой nalog.ru
---

В личный кабинет физлица пустит с любого браузера. В лк ИП через ГОСТ. В лк ЮЛ через ГОСТ по прямой ссылке (чтоб пропустить проверку совместимости)

Предлагает скачать cades.

```
tar xf cades_linux_amd64.tar.gz
cd cades_linux_amd64/
dpkg -i cprocsp-pki-cades_2.0.0-1_amd64.deb cprocsp-pki-plugin_2.0.0-1_amd64.deb
```

И расширение браузера https://chrome.google.com/webstore/detail/cryptopro-extension-for-c/iifchhfnnmpdbibifmljnfjhpififfog?hl=ru&authuser=1

Неправильные алиасы к оидам в поставке криптопро 5

```
export PATH=$PATH:/opt/cprocsp/bin/amd64/:/opt/cprocsp/sbin/amd64/

cpconfig -ini '\cryptography\OID\1.2.643.7.1.1.1.1!3' -add string 'Name' 'GOST R 34.10-2012 256 bit'
cpconfig -ini '\cryptography\OID\1.2.643.7.1.1.1.2!3' -add string 'Name' 'GOST R 34.10-2012 512 bit'
```


СБИС
---

Поддержка линукс сейчас на внутреннем тестировании. Но установить можно. Расширение
https://chrome.google.com/webstore/detail/sbis-plugin-extension/pbcgcpeifkdjijdjambaakmhhpkfgoec?hl=ru&authuser=1

Когда запустят в продакшн будет окно со ссылкой на
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
```

Для дебиан 9

```
sudo ln -s /bin/mkdir /usr/bin/mkdir
sudo ln -s /bin/chmod /usr/bin/chmod
```

Для скриншотов `libavcodec-dev libva1 libva-drm1`

Переходим на сайт сбиса. Пробуем войти с подписью.
Переходим в инспектор и ставим точку останова после SBIS3.Plugin/Source/Environment/PluginMigrate на isUseNewPlugin

в файле https://online.sbis.ru/auth/resources/SBIS3.Plugin/sbis3plugin_min.package.min.js?x_module=19.613-20

перезагрузить страницу и на точке останова добавить `u.detection` в `Watch` и поставить `isMac = true`

добавить сертификат `/usr/share/ca-certificates/sbis.crt` в центры сертификации в браузере chrome://settings/certificates  


1C
---

Добавить алгоритмы в CAPI
```
export PATH=$PATH:/opt/cprocsp/bin/amd64/:/opt/cprocsp/sbin/amd64/

cpconfig -ini '\cryptography\Defaults\Provider\Crypto-Pro GOST R 34.10-2001 Cryptographic Service Provider' -add string 'Function Table Name' CPCSP_GetFunctionTable
cpconfig -ini '\cryptography\Defaults\Provider\Crypto-Pro GOST R 34.10-2001 Cryptographic Service Provider' -add long Type 75

cpconfig -ini '\cryptography\Defaults\Provider\Crypto-Pro GOST R 34.10-2012 Cryptographic Service Provider' -add string 'Image Path' /opt/cprocsp/lib/amd64/libcsp.so
cpconfig -ini '\cryptography\Defaults\Provider\Crypto-Pro GOST R 34.10-2012 Cryptographic Service Provider' -add string 'Function Table Name' CPCSP_GetFunctionTable
cpconfig -ini '\cryptography\Defaults\Provider\Crypto-Pro GOST R 34.10-2012 Cryptographic Service Provider' -add long Type 80

cpconfig -ini '\cryptography\Defaults\Provider\Crypto-Pro GOST R 34.10-2012 Strong Cryptographic Service Provider' -add string 'Image Path' /opt/cprocsp/lib/amd64/libcsp.so
cpconfig -ini '\cryptography\Defaults\Provider\Crypto-Pro GOST R 34.10-2012 Strong Cryptographic Service Provider' -add string 'Function Table Name' CPCSP_GetFunctionTable
cpconfig -ini '\cryptography\Defaults\Provider\Crypto-Pro GOST R 34.10-2012 Strong Cryptographic Service Provider' -add long Type 81
```

Перейти `e1cib/app/ОбщаяФорма.НастройкиЭлектроннойПодписиИШифрования` 

Добавить Крипто Про в Программы по типу 75. Добавить Крипторо ещё раз. Указать тип `80` и имя `Crypto-Pro GOST R 34.10-2012 Cryptographic Service Provider` алгоритмы выбрать из списка `GR 34.10-2012 256` и `GR 34.11-2012 256`

Указать путь к программам `/opt/cprocsp/lib/amd64/libcapi20.so`


Посмотреть ещё
---

gnupg-pkcs11-scd
gnome-keyring-pkcs11
