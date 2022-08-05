
# ZABBİX 6.0 KURULUMU

Ubuntu 20.04 server ile zabbix 6.0 kurulumu


# Gereksinimler

Ubuntu 20.04 server ve isteğe göre Ubuntu 20.04 desktop

## SSH Bağlantısı

Ubuntu 20.04 server’ı kurduktan sonra farklı bir sanal makina ile bu server’a bağlanmak istedim ve bunun için şu adımları izledim;

> -“sudo apt install openssh-server” komutu ile ssh-server paketini yükledim.
> 
> -“sudo systemctl status ssh” komutu ile servisin çalıştığını control ettim.
> 
> -“sudo ufw allow ssh” ile sistemde bulunan güvenlik duvarındaki izin işlemlerini hallettim.

Ancak **“ssh username@ip_address”** komutunu kullandığımda **“Permission Denied”** hatası aldım ve bunu şu şekilde düzelttim;

>“sudo nano /etc/ssh/sshd_conf” daposyasına giderek “#PasswordAuthenticattion yes” ve “#ChallengeResponseAuthentication no” kısımlarındaki yorum satırlarını kaldırarak güncelledim. 

Ardından ssh ile bağlantımı gerçekleştirip şu adımları izledim.

## PostGreSQL Kurulumu
Öncelikle zabbix’in kullanacağı database olarak PostGreSQL kurmayı denedim bunun için sırasıyla yazdığım komutlar şunlardır:

>-wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add –
>
>	-echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list
>	
>-sudo apt update
>
>-sudo apt install postgresql-13 postgresql-client-13 -y
>
>-systemctl enable postgresql
>
>-systemctl start postgresql
>
>-systemctl status postgresql

Sonrasında zabbix kurulumunu gerçekleştireceğiz.

## Zabbix kurulumu 
Öncelikle zabbix dosyasını indirmemiz gerekiyor bu kısmı şu komutlarla yapıyoruz;

>-wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-3+ubuntu20.04_all.deb
>
>-dpkg -i zabbix-release_6.0-3+ubuntu20.04_all.deb
>
>-apt update

Güncelleştirme işleminden sonra zabbix server kurulumunu şu komutla sağladım; 

>-apt install zabbix-server-pgsql zabbix-frontend-php php7.4-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent

Kurulum sırasında apache2 servisini seçmediğim halde apache2 servisini de aktif etti bu bana sorun çıkarabileceği için bunu durdurmam gerekiyordu, bunun için şu adımları izledim;

>-systemctl  stop apache2
>
>-systemctl disable apache2

Daha sonra veri tabanı için bir kullanıcı oluşturmamız gerekiyor, bunu şu komutlarla sağlıyoruz;

>-sudo -u postgres createuser --pwprompt zabbix (bu kısımda şifre istiyor, yeni bir şifre oluşturuyoruz.)
>
>-sudo -u postgres createdb -O zabbix zabbix

Sonrasında gerekli dosyaları oluşturduğumuz database’ e kopyalaması için zcat kullanabiliriz.

>-zcat /usr/share/doc/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix

Zabbix’in database’e erişebilmesi için **“/etc/zabbix/zabbix_server.conf”** adresine giderek **“DBPassword”** kısmına yukarıda vermiş olduğumuz şifreyi veriyoruz ve yorum satırını kaldırıyoruz. Sonrasında nginx ayarları için **“/etc/zabbix/nginx.conf”** adresine gidiyorum, 2 ve 3 de bulunan yorum satırlarını silmemiz gerekiyor. 4. Yorum satırındaki “example.com” yerine **“ ∿^(.+)$; “** bu satırı ekliyoruz. Sonrasında zabbix ve nginx serverlarımızı restartlıyoruz ;

-systemctl restart zabbix-server zabbix-agent nginx php7.4-fpm

>-systemctl enable zabbix-server zabbix-agent nginx php7.4-fpm
>
sonrasında ip adresimizi girerek tarayıcıdan bağlanıyoruz. İlk ve ikinci ekranda dokunmamız gereken bir yer yok 3. Kısımda password’ümüzü giriyoruz ve geri kalan kısımlarda sadece next dememiz yeterli olacaktır.  Açılış ekranında ise;

> Kullanıcı: Admin
> 
> Şifre: zabbix


