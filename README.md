# AWStats ile Log Analizi: Nginx Trafiğinizi Görüntüleyin

![image001](https://github.com/user-attachments/assets/8e445a4f-cf18-452b-9d3a-2721631b1e99)



![image007](https://github.com/user-attachments/assets/4df4427c-a9ef-4ceb-a6c7-30cd586931e1)

## Giriş

>   Ubuntu Nginx Web sunucunuz var ve bu sunucuda oluşan erişim loglarını AWStats ile işleyerek anlamlı bir şekilde görüntülemek istiyorsunuz. AWStats sayesinde web sunucunuzun erişim loglarını anlamlı bir şekilde analiz ederek, detaylı istatistikler elde edebilirsiniz.   
>     
>   Bu kılavuz ile Ubuntu sunucu üzerinde yapacağınız AWStats kurulumu ile Nginx web sunucunuzda bulunan erişim loglarını AWStats web arayüzü ile görüntüleyebileceksiniz.

## Kurulum

Bu dokümanda Ubuntu Server kurulumuna yer verilmemiştir. Belirtilen adımlar, Ubuntu Server'ın ortamınıza kurulmuş olması durumunda uygulanmak üzere hazırlanmıştır.

### Nginx Sunucusu Dosya Paylaşım Tanımlamaları

Nginx web sunucusunda bulunan logların AWStats sunucusu ile işlenebilmesini sağlamak amacıyla erişim loglarının bulunduğu klasörün paylaşıma açılması işlemlerini gerçekleştiriniz.

Öncelikle paylaşım için kullanılacak NFS sunucusunun paket kurulumu ile başlayınız.

>   **sudo apt update**

>   **sudo apt install nfs-kernel-server**

exports dosyasını bir nano text editörü ile açarak paylaşım ayarlarını aşağıdaki şekilde tanımlayınız.

>   **nano /etc/exports**

**/var/log/nginx \<IP\>(ro,sync,no_subtree_check,all_squash,anonuid=33,anongid=33)**

\<IP\> adresi olarak belirtilen alanı “\<,\>” işaretleri olmadan paylaşıma bağlanacak AWStats sunucusu IP adresiyle değiştiriniz.

Exports dosyasında yapılan değişikliği aktif ediniz ve NFS sunucusunu yeniden başlatınız.

>   **sudo exportfs -a**

>   **sudo systemctl restart nfs-kernel-server**

Eğer Nginx sunucunuzda UFW firewall özelliği aktif ise, NFS paylaşımına AWStats sunucusundan erişim sağlayabilmek için 2049/tcp portuna izin verilmelidir. Aşağıdaki şekilde UFW durumunu ve aktif ise portu açmak için komutu kullanınız.

>   **sudo ufw status**

>   **sudo ufw allow from \<AWStats-Sunucu-IP\> proto tcp to any port 2049**

\<IP\> adresi olarak belirtilen alanı “\<,\>” işaretleri olmadan paylaşıma bağlanacak AWStats sunucusu IP adresiyle değiştiriniz.  


### AWStats Sunucusu Üzerinden Dosya Paylaşımına Erişim Tanımlamaları

AWStats kurulumu yapılacak Ubuntu sunucusunda NFS bağlantı paketinin kurulumunu yapınız.

>   **sudo apt install nfs-common**

Paylaşım alanını aşağıdaki şekilde oluşturunuz ve fstab dosyasına sunucunun yeniden başlatılması sonrasında paylaşıma yeniden bağlanılabilmesini için aşağıdaki tanımlamayı ekleyiniz.

>   **sudo mkdir -p /mnt/websitesi**

>   **sudo nano /etc/fstab**

\<IP-Adresi\>:/var/log/nginx /mnt/websitesi nfs defaults,ro 0 0

Yukarıdaki \<IP-Adresi\> alanına Nginx web sunucusunda tanımlanan paylaşım alanı için ilgili sunucunun IP adresini “\<,\>” işaretleri olmadan yazınız.

Paylaşımı aktif ediniz.

>   **sudo mount -a**

Aşağıdaki komutları kullanarak paylaşımı ve içinde bulunan dosyaları görüntüleyebilir olmalısınız.

>   **df -h \| grep /mnt/websitesi**

Log dosyası içeriğini görüntüleyin:

>   **cat /mnt/websitesi/access.log**

Kontrolleri başarıyla tamamladıysanız ve paylaşım tanımlamasını yaptıysanız, diğer kurulum adımlarına geçebilirsiniz.

### Apache Web Sunucusu ve Awstats Kurulum Adımları

Apache web sunucusu ve AWStats paket kurulumunu yapınız.

>   **sudo apt-get update**

>   **sudo apt-get install apache2 awstats**

Kurulumlar sonrasında Apache için awstats.conf dosyasını oluşturunuz.

>   **sudo nano /etc/apache2/conf-available/awstats.conf**

Dosya içeriğini aşağıdaki şekilde oluşturunuz.

Alias /awstatsclasses "/usr/share/awstats/lib/"  
Alias /awstats-icon "/usr/share/awstats/icon/"  
Alias /awstatscss "/usr/share/doc/awstats/examples/css"  
ScriptAlias /awstats/ /usr/lib/cgi-bin/  
Options +ExecCGI -MultiViews +SymLinksIfOwnerMatch  

Apache web sunucusunun CGI desteğini ve AWStats konfigürasyonunu aktif ediniz.

>   **sudo a2enmod cgi**

>   **sudo a2enconf awstats**

>   **sudo systemctl restart apache2**

AWStats ön ayarlı konfigürasyon dosyasının bir kopyasını alınız ve tanımlamak istediğiniz web sitenizin logları ve diğer ayarları için aşağıdaki tanımlamaları kullanınız. (“websitesi” olarak belirtilen alanları kendi sitenizin tanımına göre değiştirebilirsiniz.)

>   **sudo cp /etc/awstats/awstats.conf /etc/awstats/awstats.websitesi.conf**

awstats.websitesi.conf dosyasında aşağıdaki değişiklikleri yapınız.

>   **sudo nano /etc/awstats/awstats.websitesi.conf**

Log dosyasının okunacağı yolu belirtiniz:

**LogFile="/mnt/websitesi/access.log"**

Site Adını belirtiniz:

**SiteDomain="websitesi"**

Websitesi adını belirtiniz:

**HostAliases="localhost 127.0.0.1 websitesi"**

Performans için DNS çözümlemesini kapatabilirsiniz.

**DNSLookup=0**

Nginx log düzenine göre analiz için Logformat değerini ayarlayınız:

**LogFormat=1**

Log dosyasının analizi ve son logların AWStats ile işlenmesi için aşağıdaki komutu kullanınız.

>   **sudo /usr/lib/cgi-bin/awstats.pl -config=websitesi -update**

Bu komutu her seferinde kullandığınızda log dosyasına eklenen yeni logların AWStats tarafında işlenmesini sağlarsınız. Ancak bu komutu “crontab” ile zamanlanmış görev olarak tanımlayarak her dakika bu işlemin yapılmasını otomatik olarak sağlayabilirsiniz.

Aşağıdaki şekilde crontab dosyasını açınız ve aşağıdaki zamanlanmış görev tanımlamasını dosya içine ekleyiniz. Sonrasında aktif olması için cron servisini yeniden başlatınız.

>   **nano /etc/crontab**

\* * * * * root /usr/lib/cgi-bin/awstats.pl -config=websitesi -update > /dev/null

>   **systemctl restart cron.service**

AWStats web sitesine erişim için aşağıdaki adresi kullanınız.

>   **http://\<AWStats-Sunucu-IP-Adresi\>/awstats/awstats.pl?config=websitesi**
