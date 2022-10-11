<h2>Step 1 - Configure Repo</h2>
<pre>
echo [*] Elasticsearch repolari kuruluyor...
rpm --import https://packages.wazuh.com/key/GPG-KEY-WAZUH
cat > /etc/yum.repos.d/wazuh.repo << EOF
[wazuh]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
enabled=1
name=EL-\$releasever - Wazuh
baseurl=https://packages.wazuh.com/4.x/yum/
protect=1
EOF
rpm --import http://artifacts.elastic.co/GPG-KEY-elasticsearch
cat > /etc/yum.repos.d/elastic.repo << EOF
[elasticsearch-7.x]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
</pre>
<h2>Step 2 - Install Elasticsearch</h2>
<pre>
yum -y update
yum -y install unzip curl nano
echo [*] Elasticsearch kurulumu baslatiliyor...
yum -y install elasticsearch-7.17.5
curl -so /etc/elasticsearch/elasticsearch.yml https://packages.wazuh.com/4.3/tpl/elastic-basic/elasticsearch_all_in_one.yml
curl -so /usr/share/elasticsearch/instances.yml https://packages.wazuh.com/4.3/tpl/elastic-basic/instances_aio.yml
/usr/share/elasticsearch/bin/elasticsearch-certutil cert ca --pem --in instances.yml --keep-ca-key --out ~/certs.zip
unzip ~/certs.zip -d ~/certs
mkdir /etc/elasticsearch/certs/ca -p
cp -R ~/certs/ca/ ~/certs/elasticsearch/* /etc/elasticsearch/certs/
chown -R elasticsearch: /etc/elasticsearch/certs
chmod -R 500 /etc/elasticsearch/certs
chmod 400 /etc/elasticsearch/certs/ca/ca.* /etc/elasticsearch/certs/elasticsearch.*
rm -rf ~/certs/ ~/certs.zip
systemctl daemon-reload
systemctl enable elasticsearch
systemctl start elasticsearch
systemctl status elasticsearch
touch pass.txt <b>Note passwords to this text file. You will use it.</b>
/usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
cat pass.txt
<b>For try: curl -XGET https://localhost:9200 -u elastic:elastic_pwd -k</b>
clear
</pre>
<h2>Step 2 - Install Wazuh Manager</h2>
<pre>
echo [*] Wazuh kurulumu baslatiliyor...
yum -y install wazuh-manager-4.3.6
systemctl daemon-reload
systemctl enable wazuh-manager
systemctl start wazuh-manager
systemctl status wazuh-manager
clear
</pre>
<h2>Step 2 - Install FileBeat</h2>
<pre>
echo [*] FileBeat kurulumu baslatiliyor...
yum -y install filebeat-7.17.5
curl -so /etc/filebeat/filebeat.yml https://packages.wazuh.com/4.3/tpl/elastic-basic/filebeat_all_in_one.yml
curl -so /etc/filebeat/wazuh-template.json https://raw.githubusercontent.com/wazuh/wazuh/4.3/extensions/elasticsearch/7.x/wazuh-template.json
chmod go+r /etc/filebeat/wazuh-template.json
curl -s https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.2.tar.gz | tar -xvz -C /usr/share/filebeat/module
cat pass.txt
nano /etc/filebeat/filebeat.yml
cat /etc/filebeat/filebeat.yml
cp -r /etc/elasticsearch/certs/ca/ /etc/filebeat/certs/
cp /etc/elasticsearch/certs/elasticsearch.crt /etc/filebeat/certs/filebeat.crt
cp /etc/elasticsearch/certs/elasticsearch.key /etc/filebeat/certs/filebeat.key
systemctl daemon-reload
systemctl enable filebeat
systemctl start filebeat
filebeat test output
clear
</pre>
<h2>Step 2 - Install Kibana</h2>
<pre>
echo [*] Kibana kurulumu baslatiliyor...
yum -y install kibana-7.17.5
mkdir /etc/kibana/certs/ca -p
cp -R /etc/elasticsearch/certs/ca/ /etc/kibana/certs/
cp /etc/elasticsearch/certs/elasticsearch.key /etc/kibana/certs/kibana.key
cp /etc/elasticsearch/certs/elasticsearch.crt /etc/kibana/certs/kibana.crt
chown -R kibana:kibana /etc/kibana/
chmod -R 500 /etc/kibana/certs
chmod 440 /etc/kibana/certs/ca/ca.* /etc/kibana/certs/kibana.*
curl -so /etc/kibana/kibana.yml https://packages.wazuh.com/4.3/tpl/elastic-basic/kibana_all_in_one.yml
cat pass.txt
nano /etc/kibana/kibana.yml
mkdir /usr/share/kibana/data
chown -R kibana:kibana /usr/share/kibana
cd /usr/share/kibana
sudo -u kibana /usr/share/kibana/bin/kibana-plugin install https://github.com/wazuh/wazuh-kibana-app/releases/download/v4.3.6-7.10.2/wazuh_kibana-4.3.6_7.17.5-1.zip
setcap 'cap_net_bind_service=+ep' /usr/share/kibana/node/bin/node
systemctl daemon-reload
systemctl enable kibana
systemctl start kibana
systemctl status kibana
</pre>
<h2>Step 2 - Other Configurations</h2>
<pre>
sed -i "s/^enabled=1/enabled=0/" /etc/yum.repos.d/wazuh.repo
sed -i "s/^enabled=1/enabled=0/" /etc/yum.repos.d/elastic.repo
systemctl status kibana
echo [*] Diger ayarlar yapiliyor...
systemctl stop firewalld
systemctl stop firewalld
systemctl disable firewalld
systemctl restart elasticsearch
systemctl restart wazuh-manager
systemctl restart kibana
clear
echo [*] Kurulum tamamlandi!
cat /root/pass.txt
ifconfig
echo [*] Elasticsearch ayarlarini yapmayi unutmayin!
</pre>
