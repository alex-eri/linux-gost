useradd -r --shell /sbin/nologin --home-dir /var/lib/sbis/ sbis
mv /usr/share/Sbis3Plugin /var/lib/sbis/
ln -s /var/lib/sbis/ /usr/share/Sbis3Plugin

chown sbis:sbis -R /var/lib/sbis/
chown sbis:sbis -R /opt/sbis3plugin/

mkdir /etc/systemd/system/SBIS3Plugin.service.d/

cat << EOF > /etc/systemd/system/SBIS3Plugin.service.d/override.conf
ExecStart="/opt/sbis3plugin/sbis3plugin" --name "SBIS3Plugin" --library "auto" --ep "auto" start  --daemon
User=sbis
Group=sbis
EOF

systemctl daemon-reload
