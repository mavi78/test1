# Requirements Document

## Introduction

Bu proje, modern bayan kuaförü salonları için kapsamlı bir randevu ve salon yönetim sistemi geliştirmeyi amaçlamaktadır. Sistem, kayıtlı ve kayıtsız müşterilerin randevu alabilmesini, salon çalışanlarının randevuları yönetebilmesini, yöneticilerin salon operasyonlarını takip edebilmesini ve finansal analizler yapabilmesini sağlayacaktır. NestJS, NextJS, Turborepo, Prisma, PostgreSQL, Redis ve Docker teknolojileri kullanılarak geliştirilecek olan sistem, modern ve kullanıcı dostu bir arayüze sahip olacaktır.

## Requirements

### 1. Müşteri Kimlik Doğrulama ve Profil Yönetimi

**User Story:** Kayıtlı müşteri olarak, güvenli giriş yapıp profil bilgilerimi yönetmek istiyorum, böylece hesabımı kontrol edebilirim.

#### Acceptance Criteria

1. WHEN kayıtlı müşteri geçerli kullanıcı adı/eposta ve şifre girdiğinde THEN sistem müşteriyi başarıyla giriş yaptırmalı
2. IF müşteri Google OAuth ile giriş yapmak isterse THEN sistem Google OAuth authentication'ı desteklemeli
3. WHEN müşteri profil sayfasına gittiğinde THEN sistem geçmiş randevuları kronolojik sırayla göstermeli
4. WHEN müşteri profil sayfasında müşteri bilgilerini güncellemek istediğinde THEN sistem güncelleme işlemini gerçekleştirmeli
5. WHEN müşteri şifresini değiştirmek istediğinde THEN sistem güvenli şifre güncelleme işlemi sağlamalı
6. WHEN müşteri şifresini unuttuğunda THEN sistem güvenli şifre sıfırlama işlemini sağlamalı

### 2. Kayıtlı Müşteri Randevu Yönetimi

**User Story:** Kayıtlı müşteri olarak, randevu oluşturmak ve yönetmek istiyorum, böylece hizmet alabilir ve randevularımı kontrol edebilirim.

#### Acceptance Criteria

1. WHEN müşteri giriş yaptıktan sonra randevu oluşturma sayfasına gittiğinde THEN sistem müşteri bilgilerini otomatik olarak doldurmalı
2. WHEN müşteri randevu sayfasında hizmetleri seçip tarih ve saat belirlediğinde THEN sistem randevuyu oluşturmalı
3. WHEN müşteri randevu detay sayfasına gittiğinde THEN sistem randevu bilgilerini detaylı olarak göstermeli
4. WHEN müşteri randevu onaylanmadıysa hizmetleri değiştirmek istediğinde THEN sistem değişiklik yapmasına izin vermeli
5. WHEN müşteri randevu onaylanmadıysa tarih/saat değiştirmek istediğinde THEN sistem yeni tarih/saat seçimi yapmasına izin vermeli
6. WHEN müşteri randevuyu iptal etmek istediğinde THEN sistem iptal nedenlerini seçenek olarak sunmalı veya elle yazabilmeli
7. WHEN müşteri iptal işlemini onayladığında THEN sistem randevuyu iptal edip başarılı mesaj göstermeli

### 3. Kayıtsız Müşteri Randevu Sistemi

**User Story:** Kayıtsız müşteri olarak, web sitesinden randevu oluşturup takip etmek istiyorum, böylece kayıt olmadan da hizmet alabilirim.

#### Acceptance Criteria

1. WHEN kayıtsız müşteri randevu butonuna tıkladığında THEN sistem randevu oluşturma sayfasına yönlendirmeli
2. WHEN kayıtsız müşteri ad, soyad, telefon numarası bilgilerini girip hizmetleri seçtiğinde THEN sistem randevu oluşturmasına izin vermeli
3. WHEN kayıtsız müşteri randevu oluşturduktan sonra THEN sistem benzersiz 6 haneli randevu takip numarası vermeli
4. WHEN kayıtsız müşteri randevu takip numarasını girdiğinde THEN sistem randevu detaylarını göstermeli
5. WHEN kayıtsız müşteri randevu onaylanmadıysa hizmetleri/tarih/saat değiştirmek istediğinde THEN sistem değişiklik yapmasına izin vermeli
6. WHEN kayıtsız müşteri randevu takip numarası ile iptal etmek istediğinde THEN sistem iptal işlemini gerçekleştirmeli

### 4. Salon Çalışanı Operasyonları

**User Story:** Salon çalışanı olarak, randevusuz müşterileri kaydetmek ve randevuları yönetmek istiyorum, böylece salon operasyonlarını etkin şekilde yürütebilirim.

#### Acceptance Criteria

1. WHEN randevusuz müşteri salona geldiğinde THEN çalışan müşteri adı, soyadı ile sisteme kayıt yapabilmeli
2. WHEN çalışan müşteri adı/soyadı/telefon ile arama yaptığında THEN sistem mevcut kayıtları bulabilmeli
3. IF müşteri daha önce sistemde kayıtlıysa THEN sistem mevcut kayıt üzerinden randevu oluşturmalı
4. WHEN çalışan müşteri bilgilerini girip hizmetleri seçtiğinde THEN sistem anında onaylanmış randevu oluşturmalı
5. WHEN çalışan bildirimleri görüntülediğinde THEN sistem beklemede olan randevuları göstermeli
6. WHEN beklemede olan randevunun tarih/saatinde çakışma varsa THEN sistem uyarı vermeli
7. WHEN çalışan çakışmaya rağmen onaylamak isterse THEN sistem buna izin vermeli
8. WHEN çalışan randevu tarih/saatini değiştirip onayladığında THEN sistem müşteriye bildirim göndermeli

### 5. Yönetici Salon Yönetimi

**User Story:** Yönetici olarak, salon operasyonlarını ve çalışanları yönetmek istiyorum, böylece işletmemi etkin şekilde yönetebilirim.

#### Acceptance Criteria

1. WHEN yönetici çalışan eklemek istediğinde THEN sistem çalışan hesabı oluşturmalı ve yetkilendirme yapmalı
2. WHEN yönetici çalışan bilgilerini düzenlemek istediğinde THEN sistem düzenleme ve yetki güncelleme işlemi sağlamalı
3. WHEN yönetici gider/gelir analizi yapmak istediğinde THEN sistem finansal analiz raporları sunmalı
4. WHEN yönetici ay/hafta bazında müşteri analizi yapmak istediğinde THEN sistem müşteri istatistikleri sağlamalı
5. WHEN yönetici hizmet ve fiyat yönetimi yapmak istediğinde THEN sistem hizmet ekleme/düzenleme/fiyatlandırma işlemi sağlamalı

### 6. Web Sitesi ve İçerik Yönetimi

**User Story:** Yönetici olarak, web sitesi içeriğini ve ayarlarını yönetmek istiyorum, böylece müşterilere güncel bilgi sunabilirim.

#### Acceptance Criteria

1. WHEN yönetici salon adres, telefon, sosyal medya bilgilerini güncellemek istediğinde THEN sistem bilgi güncelleme işlemi sağlamalı
2. WHEN yönetici salon çalışma gün/saatlerini ve özel günleri belirlemek istediğinde THEN sistem çalışma takvimi yönetimi sağlamalı
3. WHEN yönetici indirimleri belirlemek istediğinde THEN sistem indirim tanımlama işlemi sağlamalı
4. WHEN yönetici yapılan işlemlerle ilgili görseller yüklemek istediğinde THEN sistem güvenli file upload işlemi sağlamalı
5. WHEN görseller yüklendiğinde THEN müşteriler bu görselleri galeri sayfasında görüntüleyebilmeli
6. WHEN müşteriler web sitesini ziyaret ettiğinde THEN sistem güncel salon bilgilerini, hizmetleri ve fiyatları göstermeli
7. WHEN çalışma saatleri dışında randevu alınmaya çalışıldığında THEN sistem uyarı vermeli

### 7. Ödeme ve Hizmet Tamamlama

**User Story:** Yönetici/çalışan olarak, hizmet sonrası ödeme ve randevu tamamlama işlemlerini gerçekleştirmek istiyorum, böylece müşteri hizmetini sonlandırabilirim.

#### Acceptance Criteria

1. WHEN hizmet verildikten sonra ödeme sayfasına geçildiğinde THEN sistem müşteriyi seçme ve hizmetleri görüntüleme imkanı sunmalı
2. WHEN çalışan hizmet çıkarmak veya eklemek istediğinde THEN sistem bu işlemlere izin vermeli
3. WHEN hizmetler belirlendikten sonra THEN sistem ödeme miktarını hesaplamalı ve indirimleri uygulamalı
4. WHEN ödeme alındıktan sonra THEN sistem kayıtları kayıt altına almalı ve randevuyu "tamamlandı" olarak işaretlemeli
5. WHEN randevu tamamlandığında THEN yönetici/çalışan sadece kendilerinin görebileceği not ekleyebilmeli
6. WHEN müşteri hizmet aldıktan sonra THEN sistem geri bildirim formu ve 1-5 arası puanlama sistemi sunmalı

### 8. Sistem Altyapısı ve Güvenlik

**User Story:** Sistem olarak, güvenlik, performans ve kullanıcı deneyimi sağlamak istiyorum, böylece güvenilir ve hızlı bir platform sunabilirim.

#### Acceptance Criteria

1. WHEN kullanıcı şifresi kaydedildiğinde THEN sistem argon2 ile şifreleme yapmalı
2. WHEN JWT token oluşturulduğunda THEN sistem güvenli token üretmeli
3. WHEN sistem yoğun kullanımda olduğunda THEN Redis cache sistemi performansı artırmalı
4. WHEN dosya yüklendiğinde THEN sistem güvenli file upload işlemi gerçekleştirmeli
5. WHEN sistem verileri güncellendiğinde THEN PostgreSQL veritabanında güvenli şekilde saklanmalı
6. WHEN hassas veriler işlendiğinde THEN sistem GDPR uyumlu veri işleme sağlamalı

### 9. Bildirim ve İletişim Sistemi

**User Story:** Sistem olarak, otomatik bildirimler ve iletişim sağlamak istiyorum, böylece kullanıcılar güncel bilgi alabilsin.

#### Acceptance Criteria

1. WHEN kayıtlı veya kayıtsız müşteri randevu oluşturduğunda THEN sistem benzersiz 6 haneli randevu takip numarası vermeli
2. WHEN randevu oluşturulduğunda THEN sistem randevuyu "beklemede" olarak işaretlemeli ve yönetici/çalışanlara bildirim göndermeli
3. WHEN randevu durumu değiştiğinde THEN sistem müşteriye SMS, email ve Socket.IO ile bildirim göndermeli
4. WHEN salonda birden fazla çalışan olduğunda THEN sistem tek bir takvim kullanmalı
5. WHEN geri bildirim kaydedildiğinde THEN sistem yöneticiye bildirim göndermeli

### 10. Hata Yönetimi ve Kullanıcı Deneyimi

**User Story:** Kullanıcı olarak, sistem hata mesajlarını anlayabilmek istiyorum, böylece ne yapacağımı bilebilirim.

#### Acceptance Criteria

1. WHEN sistem hatası oluştuğunda THEN sistem standart Türkçe hata mesajları göstermeli
2. WHEN kullanıcı yanlış bilgi girdiğinde THEN sistem anlamlı ve açıklayıcı hata mesajları vermeli
3. WHEN form validasyon hatası oluştuğunda THEN sistem hangi alanın hatalı olduğunu Türkçe belirtmeli
4. WHEN ağ bağlantı hatası oluştuğunda THEN sistem kullanıcı dostu hata mesajı göstermeli
5. WHEN sunucu hatası oluştuğunda THEN sistem genel hata mesajı gösterip teknik detayları gizlemeli
6. WHEN web sitesi tasarlandığında THEN modern bayan kuaförü salonuna yakışır kullanıcı dostu tasarım olmalı

### 11. Geliştirme Standartları

**User Story:** Yazılımcı olarak, sistem geliştirme standartlarına uygun kod yazmak istiyorum, böylece sürdürülebilir ve kaliteli bir sistem geliştirebilirim.

#### Acceptance Criteria

1. WHEN kod yazıldığında THEN yapay zeka asistanı Türkçe konuşmalı
2. WHEN kod açıklaması yazıldığında THEN JSDoc formatında ve Türkçe olmalı
3. WHEN metod, sınıf, değişken, sabit ve klasör isimleri belirlediğinde THEN standart ve İngilizce olmalı
4. WHEN sistem hatası oluştuğunda THEN sistem hata loglarını kaydetmeli
