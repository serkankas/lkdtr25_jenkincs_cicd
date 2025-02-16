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

## Gün 2  

### **Yapılanlar**  

Bugün Jenkins’i biraz daha ileri seviyeye taşıyarak, **master-slave yapılandırması**, **uzaktan tetikleme (webhook)** ve **kullanıcı yönetimi** gibi konulara odaklandık. İşte yaptıklarımız:  

1. **İkinci bir sanal makine kurup Master-Slave bağlantısını ayarladık.**  
   - Master makineden, yeni kurulan slave makineye **şifresiz SSH bağlantısı** kurmamız gerekti. Bunun için önce Master makinede SSH anahtarları oluşturduk:  
     ```sh
     ssh-keygen
     ```  
   - Ardından, oluşturduğumuz **private key**'i Jenkins’in **Credentials** bölümüne ekleyerek, bağlantıyı Jenkins üzerinden yönetebilir hale getirdik.  
   - SSH bağlantısını test etmek için şu komutu çalıştırdık:  
     ```sh
     ssh -i <private_key_path> <username>@<slave_ip>
     ```  
   - Eğer bağlantı sırasında fingerprint doğrulaması yapılıyorsa, kabul edip ilerledik.  
   - Jenkins’in slave makineye erişmesi için **SSH Agent Plugin**'i yükledik.  
   - Son olarak, **Jenkins üzerinden yeni bir job** oluşturduk ve slave makinede çalıştığımızı doğrulamak için şu komutlardan birini çalıştırdık:  
     ```sh
     ip -a  
     ifconfig  
     cat /etc/hostname  
     ```

2. **Jenkins’in uzaktan tetiklenmesini sağladık (Remote Trigger - Webhook).**  
   - Jenkins’i **webhook veya API çağrıları** ile otomatik tetikleyebilmek için **Generic Webhook Trigger Plugin**'i yükledik.  
   - API üzerinden tetikleme için Jenkins içinde **bir token oluşturduk** ve bunu güvenli şekilde sakladık.  
   - Tetikleyicinin çalışıp çalışmadığını test etmek için şu komutu kullandık:  
     ```sh
     curl -XPOST --user "admin:<token>" http://<jenkins_ip>/job/<job_name>/build?token=<token_name>
     ```  
   - Bu aşamada GitLab ile entegrasyon konusu konuşuldu, ancak detayına girmedik.  

3. **Kullanıcı yönetimi ve yetkilendirme üzerine çalıştık.**  
   - Öncelikle, **Jenkins konfigürasyon dosyalarının yedeğini almanın öneminden** bahsettik. Özellikle şu dosyanın yedeğini almak iyi bir fikir olabilir:  
     ```sh
     /var/lib/jenkins/config.xml
     ```  
   - **Yeni kullanıcıları Jenkins UI üzerinden oluşturduk.**  
   - Yetkilendirme için **Role-Based Strategy Plugin**'i yükledik ve yetkilendirme seçeneklerine göz gezdirdik.  
   - Kullanıcılara izin vermek için **User Matrix, Role-Based ve Item-Based permission** sistemlerini inceledik, ancak gerçek anlamda bir yetkilendirme yapmadık.  

4. **Job'lara parametre eklemeyi öğrendik.**  
   - Parametreli job'lar oluşturarak, build sırasında dışarıdan değişken alabilmeyi sağladık.  
   - Parametrelere `${}` kullanarak erişebileceğimizi gördük.  

---

## **Ekstralar**  

Ders arasında konuşulan ancak detayına inmediğimiz birkaç konu daha vardı. Bunları da buraya not olarak ekleyelim:  

#### **Vagrant Nedir?**  
Vagrant, sanal makineleri (*VirtualBox, VMware, Hyper-V gibi*) kolayca oluşturmak, yönetmek ve konfigüre etmek için kullanılan bir araçtır.  

#### **LDAP (Lightweight Directory Access Protocol)**  
Merkezi kimlik doğrulama ve yetkilendirme için kullanılan bir dizin protokolüdür.  

#### **Active Directory (AD)**  
Microsoft’un LDAP tabanlı kimlik yönetim sistemi olarak tanımlanabilir.

# Gün 3  

## **Pipeline Kullanımı ve Avantajları**  

Jenkins Pipeline, CI/CD süreçlerini daha **kolay, yönetilebilir ve izlenebilir** hale getiren bir yapıdır.  
- Hangi **task’in ne kadar sürdüğünü** ve **hangi aşamalardan geçtiğini** görebiliriz.  
- **Declarative** veya **Scripted** pipeline olarak yazılabilir.  
- Web arayüzü üzerinden pipeline yazarsak daha kolay müdahale edebiliriz, ancak dosya bazlı pipeline'larda değişiklik yapmak için repository'ye erişim gerekir.  
- Kim hangi aşamalara erişebilir, hangi kodu değiştirebilir gibi kontroller yapılabilir.  

### **Pipeline İçin Kullanışlı Sekmeler:**  
- **Pipeline Overview:** Genel süreci gösterir.  
- **Pipeline Steps:** Çalışan adımların detaylarını gösterir.  
- **Pipeline Console:** Komut çıktılarının görülebileceği terminal ekranı.  
- **Timing:** Hangi aşamanın ne kadar sürdüğünü gösterir.  

**Daha iyi görselleştirme için:** *"Pipeline Stage View"* eklentisini yüklemek önerilir.  

---

## **Pipeline Örnekleri**  

### **Environment Variable Kullanımı**  
Pipeline içinde **environment değişkenleri** kullanarak değerler saklayabiliriz.  

```groovy
pipeline {
    agent any
    
    environment {
        KURS = "Jenkins ile CI/CD"
        SECRET_01 = credentials('secret_01')
    }

    stages {
        stage('Ortam Değişkenlerini Yazdır') {
            steps {
                sh 'echo ${KURS}'
                sh 'echo ${SECRET_01}'  // Gizli credential burada görünmez.
            }
        }
        ...
    }
}
```

## Not: Credential olarak saklanan değişkenler konsolda gözükmez, ancak bir dosyaya yazılarak artifact olarak saklanabilir.

---

## Username & Password Credentials Kullanımı

Eğer bir **kullanıcı adı ve şifre** içeren credential kullanıyorsak, bunları aşağıdaki gibi pipeline içinde çağırabiliriz:

### **Örnek: Kullanıcı Adı & Şifre Kullanımı**

```groovy
pipeline {
    agent any
    
    environment {
        SECRET_ID_PW = credentials('secret_user_idpw')
    }

    stages {
        stage ('Kullanıcı Bilgilerini Kullanma') {
            steps {
                sh 'echo ${SECRET_ID_PW_USR}'  // Kullanıcı adı
                sh 'echo ${SECRET_ID_PW_PSW}'  // Şifre
            }
        }
        ...
    }
}
```

#### Dikkat: Credentials'lar terminal çıktısında görünmemesi için **direkt echo ile yazdırılmamalıdır.**

---

## Koşullu (Conditional) Pipeline Kullanımı

Bazı aşamaların sadece **belli koşullarda** çalışmasını istiyorsak, **when** bloğunu kullanabiliriz:

### **Örnek: Koşullu Pipeline Kullanımı**

```groovy
pipeline {
    agent any
    
    parameters {
        booleanParam(defaultValue: false, description: 'Bu aşama çalışsın mı?', name: 'EXECS')
    }

    stages {
        stage('Koşullu Aşama') {
            when {
                allOf {
                    expression { currentBuild.getNumber() % 2 == 1 } // Sadece tek numaralı build'lerde çalış
                    expression { params.EXECS } // Kullanıcı checkbox işaretlediyse çalış
                }
            }
            steps {
                sh 'echo "Bu adım çalıştı!"'
            }
        }
        ...
    }
}
```

#### **Not: Parametreler** build başlatıldıktan sonra ekranda görünür, yani **ilk build'de kullanılamaz.**

---

## Post Condition Kullanımı

Pipeline sonunda belirli durumlara göre **otomatik aksiyon almak** için post bloğu kullanabiliriz:

### **Örnek: Post Condition Kullanımı**

```groovy
pipeline {
    agent any

    stages {
        ...
    }

    post {
        always {
            sh 'echo "Pipeline çalıştırıldı."'
        }
        success {
            sh 'echo "Pipeline başarıyla tamamlandı."'
        }
        failure {
            sh 'echo "Pipeline hata ile karşılaştı!"'
        }
        aborted {
            sh 'echo "Pipeline çalışması iptal edildi."'
        }
    }
}
```

#### **Not: Post koşulları** hem pipeline hem de **bireysel stage'ler** için kullanılabilir.

---

## Jenkins Agents (Slave Nodes)

Jenkins **multi-node** yapısıyla çalışabilir. **Ek bir Linux makineyi "slave" olarak tanımlayarak**, Jenkins’in başka bir makinada da build çalıştırmasını sağlayabiliriz.

### **Jenkins Slave Kurulumu:**

* **Jenkins Dashboard → Manage Jenkins → Nodes** sekmesinden yeni bir node eklenir.
* Master makineden **SSH ile bağlanarak** slave erişimi sağlanır.
* **SSH Server ve Client** kurularak Jenkins’in bağlantı kurması sağlanır.

#### **Not:** Dikkatimi Çekmeyen Konular

Bu dersin bir kısmında **Java ve Maven kullanılarak** örnekler yapıldı. Ancak **benim ilgimi çeken konular daha çok Git, versiyonlama ve multiple pipeline'ları birbirine bağlamak üzerine** olduğu için bu kısımlar üzerinde durmadım.
