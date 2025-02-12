# Jenkins CI/CD

## Gün 1  

### **Konu Anlatım**  

#### **DevOps ve CI/CD Kavramları**  

##### **DevOps**  
DevOps (Development ve Operations), yazılım geliştiriciler ile sistem operasyon ekipleri arasında bir köprü görevi gören bir yaklaşımdır. Yazılım sistemlerinden maksimum performans alınabilmesi için hem geliştiricilerin hem de sistem yöneticilerinin sorumluluk alanlarını belirler. Bu kavram, bir hata oluştuğunda kimin müdahale etmesi gerektiğini ve hangi süreçlerin kim tarafından yönetilmesi gerektiğini netleştirmeyi amaçlar.  

##### **CI/CD**  
CI/CD, *Continuous Integration* (Sürekli Entegrasyon) ve *Continuous Delivery/Deployment* (Sürekli Teslimat/Yayılama) kavramlarının birleşimidir. Bu süreç, yazılım sistemlerinin entegrasyonunu ve düzenli olarak güncellenmesini otomatik hâle getiren bir teknoloji zinciri olarak tanımlanabilir. CI, yazılımın sık sık küçük parçalar hâlinde entegre edilmesini sağlarken; CD, bu entegrasyonların sorunsuz bir şekilde test edilip dağıtılmasını mümkün kılar.  

##### **Jenkins**  
Jenkins, CI/CD süreçlerinde entegrasyon ve dağıtım işlemlerini otomatikleştirmeye yarayan bir araçtır. Kurs kapsamında tercih edilme sebebi, açık kaynak (*open source*) olmasının yanı sıra özgür yazılım (*free software*) olmasıdır. Bu sayede Jenkins’i tamamen ücretsiz olarak kullanabilir, geliştirebilir ve ihtiyacımıza göre özelleştirebiliriz.  

#### **Jenkins Kullanımının Avantajları**  
- **Web tabanlı arayüz** sayesinde kolay kurulum ve yönetim  
- **Pipeline (ardışık düzen) desteği** ile karmaşık CI/CD süreçlerini yönetebilme  
- **Eklenti desteği** sayesinde farklı araç ve platformlarla entegrasyon yapabilme  
- **Paralel ve dağıtık yapı desteği** ile büyük ölçekli projelerde verimli çalışma imkânı  
- **Otomatik hata yönetimi ve bildirim mekanizmaları** ile süreçlerin güvenilirliğini artırma  
- **Kullanıcı bazlı loglama ve yetkilendirme** ile sistem yöneticisi olmayan kişilere kontrollü erişim sağlama  
- **Farklı script dillerini bir araya getirme** özelliği ile geniş kapsamlı otomasyon süreçleri oluşturabilme  

---

## **Yapılanlar**  

1. Sanal makine üzerinde bir sunucu kuruldu.  
2. Kurulan sunucunun, **host makine** ve başka bir sanal makine ile iletişiminin sağlanması için gerekli ağ yapılandırmaları yapıldı.  
3. Jenkins kuruldu; kurulum sürecinde Jenkins’in resmi dokümantasyonundaki adımlar takip edildi.  
4. **NGINX reverse proxy** kullanılarak, Jenkins'in varsayılan **8080** portu **80** portuna yönlendirildi.  
5. **Self-signed SSL** oluşturularak Jenkins için güvenli bağlantı sağlandı ve NGINX üzerinde SSL yapılandırması yapıldı. Örnekleme alınan [Link](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu)

    ```sh
    # Self-signed SSL sertifikası oluştur
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
        -keyout /etc/ssl/private/nginx-selfsigned.key \
        -out /etc/ssl/certs/nginx-selfsigned.crt

    # Diffie-Hellman parametrelerini oluştur
    sudo openssl dhparam -out /etc/nginx/dhparam.pem 2048

    # SSL ayarlarını içeren dosyaları oluştur ve düzenle
    sudo nano /etc/nginx/snippets/self-signed.conf
    # İçerik:
    # ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
    # ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;

    sudo nano /etc/nginx/snippets/ssl-params.conf
    # İçerik:
    # ssl_protocols TLSv1.2 TLSv1.3;
    # ssl_prefer_server_ciphers on;
    # ssl_dhparam /etc/nginx/dhparam.pem;

    # NGINX için temel reverse proxy konfigürasyonu
    sudo nano /etc/nginx/sites-available/default
    # İçerik:
    # server {
    #     listen 80;
    #     listen 443 ssl;
    #     
    #     include snippets/self-signed.conf;
    #     include snippets/ssl-params.conf;
    #     
    #     location / {
    #         proxy_pass http://127.0.0.1:8080;
    #     }
    # }

    # NGINX servisini yeniden başlat
    sudo systemctl restart nginx
    ```

6. `/etc/hosts` dosyasına bir **host ismi** eklenerek, Jenkins’e daha kolay erişim sağlandı.  
7. `sudo cat /var/lib/jenkins/secrets/initialAdminPassword` komutu kullanılarak **initialAdminPassword** alındı ve Jenkins web arayüzünden kurulum tamamlandı.  
8. **Bir görev (task) oluşturuldu** ve çıktısı incelendi. (Workspace içinde olmayan bir dosyanın **artifact** olarak saklanması şu an mümkün görünmüyor.)  

---

## **Artifact Nedir?**  
**Artifact**, yapılan işlemler sonucunda ortaya çıkan ve saklanmak istenen dosya ya da çıktıdır.  

---

## **Örnek Jenkins Senaryoları**  

- **Echo ile Mesaj Yazdırma:** Konsola belirli bir mesaj yazdırarak pipeline’ın çalıştığını doğrulamak için kullanılır.  
- **Dosya Okuma:** Pipeline içindeki bir dosyanın içeriğini okuyup ekrana yazdırabiliriz.  
- **Slave Makinenin IP Adresini Alma:** `ifconfig` komutu ile bir **slave** makinenin IP adresini okuyarak pipeline içinde kullanabiliriz.  

---

## **Workspace ve Artifact Farkı**  

- **Workspace**, Jenkins’in işlem yaptığı geçici çalışma alanıdır. Tüm dosyalar burada tutulur ancak işlem tamamlandığında temizlenebilir.  
- **Artifact**, workspace içindeki belirli dosyaların saklanarak pipeline tamamlandıktan sonra da erişilebilir hâlde kalmasını sağlar.  
