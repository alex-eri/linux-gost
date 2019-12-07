Отнимаю у плагина рута, включаю лог в журнал, перемещаю бардак в более подходящее место.

```
mv /usr/share/Sbis3Plugin /var/lib/sbis/
ln -s /var/lib/sbis/ /usr/share/Sbis3Plugin

useradd -r --shell /sbin/nologin --home-dir /var/lib/sbis/ sbis

chown sbis:sbis -R /var/lib/sbis/
chown sbis:sbis -R /opt/sbis3plugin/
mkdir /etc/systemd/system/SBIS3Plugin.service.d/

cat << EOF > /etc/systemd/system/SBIS3Plugin.service.d/override.conf
ExecStart="/opt/sbis3plugin/sbis3plugin" --name "SBIS3Plugin" --library "auto" --ep "auto" start  --daemon
User=sbis
Group=sbis
EOF

mkdir /var/run/sbisplugin/
chown sbis:sbis /var/run/sbisplugin/


systemctl daemon-reload
systemctl restart SBIS3Plugin
```

Убираем ересь

```
sed -i '/HOME=~"$USER"/d'   /etc/profile.d/sbis3plugin-user-install.sh
```
