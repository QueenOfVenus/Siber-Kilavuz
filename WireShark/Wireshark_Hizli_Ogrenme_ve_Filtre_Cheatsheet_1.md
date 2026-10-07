# Wireshark Hızlı Öğrenme Kılavuzu ve Filtre Cheatsheet

> Bu sayfa iki işe yarar: (1) Wireshark'ı sıfırdan, mantığıyla hızlı öğrenmek; (2) her gün bakacağın, **tek tek açıklanmış** bir filtre sözlüğü. Tüm filtreler Wireshark/tshark 4.x ile test edildi. Toy dostu: her terim ilk geçtiğinde açıklanır.

## 1. Wireshark 10 dakikada: temel mantık

Wireshark ağdan geçen paketleri yakalar ve **katman katman** açar. Ekranı üç bölmedir:

- **Paket listesi (üst):** Her satır bir paket. Sütunlar: No (sıra), Time (zaman), Source/Destination (kaynak/hedef), Protocol, Length, Info (özet).
- **Paket detayı (orta):** Seçili paketin katmanları ağaç halinde. Ethernet → IP → TCP/UDP → uygulama (HTTP/DNS/...). Her satır açılır.
- **Paket byte'ları (alt):** Ham veri (hex + ASCII). Detayda bir alana tıklayınca karşılığı burada vurgulanır.

**Çalışma döngüsü:** Yakala (veya dosya aç) → **filtrele** (süz) → **Follow Stream** (diyaloğu oku) → işaretle/yorumla → rapor.

**En kritik üç refleks:**

1. Bir alanın filtre adını bilmiyorsan: o alana **tıkla**, en alttaki durum çubuğunda adı yazar. Ya da **sağ tık → Apply as Filter**.
2. Bir konuşmayı baştan sona okumak için: pakete **sağ tık → Follow → TCP Stream**.
3. Genel resmi görmek için: **Statistics** menüsü (Protocol Hierarchy, Conversations, Endpoints).

## 2. Capture filtresi vs Display filtresi (en sık karışan konu)

Bunlar **iki ayrı dildir**. Karıştırmak 1 numaralı başlangıç hatasıdır.

| | Capture (BPF) filtresi | Display filtresi |
| --- | --- | --- |
| Ne zaman | **Yakalarken** (kayıt öncesi) | **İnceleme** (kayıt sonrası) |
| Amaç | Diske neyin yazılacağını sınırlar | Elindeki paketleri ekranda süzer |
| Dil örneği | `host 10.0.0.5 and tcp port 445` | `ip.addr == 10.0.0.5 && tcp.port == 445` |
| Geri alınır mı? | **Hayır** (yakalanmayan kaybolur) | **Evet** (filtreyi silersin) |
| Nerede yazılır | tcpdump/tshark `-f`, Wireshark capture kutusu | Wireshark filtre çubuğu, tshark `-Y` |

**Kural:** Emin değilsen **geniş yakala, dar incele**. Capture filtresini sadece trafik çok fazlaysa ve ne istediğin netse kullan.

## 3. Display filtresi dili: operatörler

| Operatör | Anlamı | Örnek | Açıklama |
| --- | --- | --- | --- |
| `==` | eşittir | `ip.addr == 10.0.0.5` | Bu IP ile ilgili (kaynak veya hedef) paketler |
| `!=` | eşit değildir | `tcp.port != 80` | Dikkat: çift yönlü alanlarda beklenmedik sonuç verebilir; `!(tcp.port == 80)` tercih et |
| `>` `<` `>=` `<=` | karşılaştırma | `frame.len > 1000` | 1000 byte'tan büyük çerçeveler |
| `&&` / `and` | ve | `ip.src == 10.0.0.5 && tcp.port == 445` | İki koşul birlikte |
| `or` (veya iki dik çizgi) | veya | `tcp.port == 80 or tcp.port == 443` | Koşullardan biri; `or` yerine iki dik çizgi de yazılabilir |
| `!` / `not` | değil | `!arp` | ARP olmayan her şey |
| `contains` | içerir (birebir) | `http.host contains "google"` | Alan içinde metin arar (büyük/küçük harf duyarlı) |
| `matches` | regex | `http.host matches "(?i)google"` | Düzenli ifadeyle arar (harf duyarsız için `(?i)`) |
| `in` | küme üyeliği | `tcp.port in {80, 443, 8080}` | Değer listeden biri mi |
| `[x:y]` | dilim | `eth.src[0:3] == 00:0c:29` | Alanın x. byte'ından y byte (üretici öneki) |

**Filtre çubuğu renkleri:** Yeşil = geçerli, Kırmızı = hatalı (çalışmaz), Sarı = çalışır ama dikkat (genelde `!=` kullanımı).

## 4. Temel alanlar (çerçeve, IP, port)

| Filtre | Açıklama |
| --- | --- |
| `ip.addr == 10.0.0.5` | IP kaynak veya hedef olarak bu adres |
| `ip.src == 10.0.0.5` | Sadece bu adresten gelen |
| `ip.dst == 10.0.0.5` | Sadece bu adrese giden |
| `ip.addr == 10.0.0.0/24` | Bu alt ağdaki (256 adres) tüm trafik |
| `tcp.port == 443` | TCP 443 (kaynak veya hedef) |
| `tcp.dstport == 445` | Hedef portu 445 (sunucu tarafı) |
| `udp.port == 53` | UDP 53 |
| `frame.len > 1500` | 1500 byte'tan büyük çerçeveler |
| `frame.time >= "2024-03-01 14:00:00"` | Belirtilen (görüntülenen) saatten sonrası |
| `frame contains "password"` | Tüm paket byte'larında "password" arar |
| `frame matches "(?i)flag\\{"` | Regex ile, harf duyarsız "flag{" arar |
| `eth.addr == 00:0c:29:aa:bb:cc` | Bu MAC adresiyle ilgili çerçeveler |

## 5. TCP bayrakları ve durum

| Filtre | Açıklama |
| --- | --- |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | Bağlantı başlatma (SYN); "kim bağlandı" sorusunun temeli |
| `tcp.flags.syn == 1 && tcp.flags.ack == 1` | SYN-ACK; portun açık olduğu cevabı |
| `tcp.flags.reset == 1` | RST; bağlantı reddi/kapatma |
| `tcp.flags.fin == 1` | FIN; düzgün kapanış |
| `tcp.flags == 0x02` | Sadece SYN biti set (yalın SYN) |
| `tcp.flags == 0x000` | Hiç bayrak yok (NULL tarama) |
| `tcp.analysis.retransmission` | Tekrar gönderilen paket (kayıp belirtisi) |
| `tcp.analysis.duplicate_ack` | Tekrarlı ACK (alıcı eksik paket istiyor) |
| `tcp.analysis.zero_window` | Alıcı "dur, dolu" diyor (uygulama yavaş) |
| `tcp.analysis.lost_segment` | Kayıtta eksik segment |
| `tcp.stream == 5` | 5 numaralı TCP bağlantısının tüm paketleri |
| `tcp.flags.str` | (Sütun yap) bayrakları ··S····· gibi gösterir |

> **Bayrak mantığı:** SYN = "konuşalım mı?", SYN-ACK = "olur", ACK = "tamam", FIN = "bitirelim", RST = "kes". Üçlü el sıkışma: SYN → SYN-ACK → ACK.

## 6. DNS

| Filtre | Açıklama |
| --- | --- |
| `dns.flags.response == 0` | Sadece sorgular (sorular) |
| `dns.flags.response == 1` | Sadece cevaplar |
| `dns.qry.name contains "kotu"` | Alan adı "kotu" içeren sorgular |
| `dns.qry.name.len > 50` | Anormal uzun alan adları (tünelleme adayı) |
| `dns.qry.type == 1` | A kaydı (IPv4) sorguları |
| `dns.qry.type == 16` | TXT kaydı (tünel/C2 favorisi) |
| `dns.qry.type == 252` | AXFR (zone transfer) |
| `dns.flags.rcode == 3` | NXDOMAIN (alan yok); DGA belirtisi |
| `dns.a == 203.0.113.50` | Bu IP'yi döndüren cevaplar |
| `dns.count.answers > 5` | Çok cevaplı (fast flux adayı) |
| `dns.resp.ttl < 60` | Çok kısa ömürlü cevaplar |

## 7. HTTP

| Filtre | Açıklama |
| --- | --- |
| `http.request` | Tüm HTTP istekleri |
| `http.response` | Tüm HTTP cevapları |
| `http.request.method == "POST"` | Veri gönderen istekler (form, yükleme) |
| `http.host == "ornek.com"` | Belirli siteye istekler |
| `http.request.uri contains "admin"` | URL'de "admin" geçen istekler |
| `http.response.code == 200` | Başarılı cevaplar |
| `http.response.code == 404` | Bulunamadı (dizin taraması belirtisi) |
| `http.user_agent contains "sqlmap"` | Araç imzalı User-Agent |
| `http.authorization` | Basic Auth kimlik bilgisi taşıyan istekler |
| `http.file_data contains "flag"` | Cevap gövdesinde "flag" (gzip açılmış halde) |
| `http.file_data[0:2] == "MZ"` | Gövdesi EXE başlığıyla başlayan (zararlı indirme) |
| `http.cookie` | Çerez taşıyan istekler |

## 8. TLS / şifreli

| Filtre | Açıklama |
| --- | --- |
| `tls.handshake.type == 1` | Client Hello (istemcinin ilk mesajı) |
| `tls.handshake.type == 2` | Server Hello (sunucunun seçimi) |
| `tls.handshake.extensions_server_name` | SNI alanı (bağlanılan alan adı) |
| `tls.handshake.extensions_server_name contains "kotu"` | SNI'de "kotu" geçen bağlantılar |
| `tls.handshake.ja3` | İstemci TLS parmak izi (JA3) |
| `tls.handshake.ja4` | Yeni nesil parmak izi (JA4, Wireshark 4.2+) |
| `tls.handshake.type == 11` | Sertifika (TLS 1.2; 1.3'te şifreli) |
| `tls.record.content_type == 23` | Application Data (şifreli asıl veri) |
| `tls.alert_message` | TLS uyarı/hata mesajları |
| `tls && !(tcp.port == 443)` | 443 dışı portlarda TLS (şüpheli) |
| `quic` | QUIC / HTTP/3 (UDP üzerinde TLS 1.3) |

## 9. Windows / Active Directory

| Filtre | Açıklama |
| --- | --- |
| `smb2.cmd == 5` | SMB2 Create (dosya açma) |
| `smb2.filename` | Erişilen dosya adları |
| `smb2.cmd == 1` | Session Setup (oturum açma) |
| `smb2.nt_status == 0xc000006d` | SMB oturum açma başarısız (yanlış parola) |
| `ntlmssp.auth.username` | NTLM ile oturum açan kullanıcı adı |
| `kerberos.CNameString` | Kerberos bilet isteyen hesap |
| `kerberos.msg_type == 10` | AS-REQ (bilet isteği) |
| `kerberos.error_code == 24` | Kerberos: yanlış parola |
| `ldap.protocolOp == 3` | LDAP searchRequest (dizin keşfi) |
| `dcerpc.opnum` | Çağrılan RPC işlemi numarası |
| `tcp.port == 3389` | RDP (uzak masaüstü) |
| `tcp.port == 5985` | WinRM (uzaktan PowerShell) |

## 10. Yerel ağ (ARP / DHCP / ICMP)

| Filtre | Açıklama |
| --- | --- |
| `arp.opcode == 1` | ARP soruları (keşif) |
| `arp.opcode == 2` | ARP cevapları (spoofing cevaplarla yapılır) |
| `arp.duplicate-address-detected` | Aynı IP için iki MAC (spoofing/çakışma) |
| `arp.isgratuitous == 1` | Sorulmadan yapılan ARP duyuruları |
| `dhcp.option.dhcp == 2` | DHCP Offer (teklif) |
| `dhcp.option.hostname` | DHCP'de cihazın bildirdiği ad |
| `icmp.type == 8` | Echo Request (ping) |
| `icmp.type == 0` | Echo Reply (host canlı) |
| `icmp.type == 3` | Destination Unreachable |
| `icmp.type == 11` | Time Exceeded (traceroute izi) |
| `icmp && data.len > 64` | Büyük yüklü ICMP (tünelleme adayı) |

## 11. Şifresiz protokoller (kimlik/veri açık)

| Filtre | Açıklama |
| --- | --- |
| `ftp.request.command == "USER"` | FTP kullanıcı adı |
| `ftp.request.command == "PASS"` | FTP parolası (düz metin!) |
| `telnet.data` | Telnet komut/veri (açık) |
| `smtp.req.command == "MAIL"` | E-posta göndereni (MAIL FROM) |
| `imf.subject` | E-posta konusu |
| `pop.request.command == "PASS"` | POP3 parolası |
| `snmp.community` | SNMP erişim parolası (düz metin) |
| `sip.Method == "INVITE"` | VoIP çağrı başlatma |
| `tftp.source_file` | TFTP ile aktarılan dosya |
| `http.authbasic` | HTTP Basic Auth (base64, çözülür) |

## 12. Capture (BPF) filtreleri cheatsheet

Bunlar **yakalarken** kullanılır (tcpdump/tshark). Dil Wireshark'ınkinden farklıdır.

| BPF | Açıklama |
| --- | --- |
| `host 10.0.0.5` | Bu IP ile tüm trafik |
| `src host 10.0.0.5` | Sadece bu IP'den gelen |
| `dst host 10.0.0.5` | Sadece bu IP'ye giden |
| `net 10.0.0.0/24` | Bu alt ağ |
| `port 443` | 443 portu (TCP+UDP) |
| `tcp port 443` | Sadece TCP 443 |
| `src port 53` | Kaynak portu 53 |
| `tcp port 80 or tcp port 443` | İki porttan biri |
| `host 10.0.0.5 and tcp port 445` | İki koşul birlikte |
| `not arp` | ARP hariç |
| `tcp[tcpflags] & tcp-syn != 0` | SYN biti set paketler |
| `icmp[icmptype] == icmp-echo` | Sadece ping (echo request) |
| `greater 1000` | 1000 byte'tan büyük paketler |

**Kullanım:**

```bash
# Yakala ve dosyaya yaz (BPF ile)
tcpdump -i eth0 -w kayit.pcap host 10.0.0.5 and tcp port 445

# tshark ile yakalama filtresi (-f = capture/BPF)
tshark -i eth0 -f "host 10.0.0.5 and tcp port 445" -w kayit.pcapng

# Dosyadan oku ve display filtresiyle süz (-Y = display)
tshark -r kayit.pcapng -Y 'ip.addr == 10.0.0.5 && tcp.port == 445'
```

## 13. tshark ile toplu analiz (hız kazandıran komutlar)

```bash
# Dosya özeti (süre, paket sayısı, hash)
capinfos kayit.pcapng

# IP konuşmaları (kim kiminle)
tshark -r kayit.pcapng -q -z conv,ip

# Protokol hiyerarşisi
tshark -r kayit.pcapng -q -z io,phs

# Tüm DNS sorgularını sıklığıyla listele (nadir olanı bul)
tshark -r kayit.pcapng -Y 'dns.flags.response==0' -T fields -e dns.qry.name | sort | uniq -c | sort -rn

# HTTP isteklerini tablo olarak çıkar
tshark -r kayit.pcapng -Y 'http.request' -T fields -e ip.src -e http.host -e http.request.uri -E header=y

# Belirli zaman aralığını ayrı dosyaya kes
editcap -A "2024-03-01 14:00:00" -B "2024-03-01 14:05:00" kayit.pcapng aralik.pcapng

# Büyük dosyayı 500bin paketlik parçalara böl
editcap -c 500000 kayit.pcapng parca.pcapng
```

## 14. Menü hızlı erişim (nerede ne var)

| İhtiyaç | Menü yolu |
| --- | --- |
| Genel resim / süre / hash | Statistics ▸ Capture File Properties |
| Hangi protokoller var | Statistics ▸ Protocol Hierarchy |
| Kim kiminle konuştu | Statistics ▸ Conversations |
| En çok konuşan host | Statistics ▸ Endpoints |
| Wireshark'ın bulduğu tuhaflıklar | Analyze ▸ Expert Information |
| Zaman içinde trafik grafiği | Statistics ▸ I/O Graphs |
| Diyaloğu baştan sona oku | Sağ tık ▸ Follow ▸ TCP/UDP/TLS Stream |
| Dosya çıkar (HTTP) | File ▸ Export Objects ▸ HTTP |
| Dosya çıkar (SMB) | File ▸ Export Objects ▸ SMB |
| Kimlik bilgilerini topla | Tools ▸ Credentials |
| Standart dışı protokolü tanıt | Analyze ▸ Decode As |
| Paket ara (kelime/hex) | Edit ▸ Find Packet (Ctrl+F) |
| VoIP çağrıları | Telephony ▸ VoIP Calls |
| TLS şifre çözme | Preferences ▸ Protocols ▸ TLS ▸ keylog dosyası |

## 15. Klavye kısayolları (en çok kullanılanlar)

| Kısayol | İş |
| --- | --- |
| `Ctrl+F` | Paket ara (Find Packet) |
| `Ctrl+G` | Paket numarasına git |
| `Ctrl+M` | Paketi işaretle |
| `Ctrl+Alt+C` | Pakete yorum ekle |
| `Ctrl+T` | Zaman referansı koy/kaldır |
| `Ctrl+Alt+Shift+T` | Follow TCP Stream |
| `Ctrl+.` / `Ctrl+,` | Sonraki / önceki paket (işaretliler arası) |
| `Ctrl+Alt+1` | Zaman formatı: gün ve saat |

## 16. Toy için 10 altın kural

1. **Önce harita, sonra sokak.** Paket açmadan önce Statistics menüsüne bak.
2. **Follow Stream dostundur.** Komut/parola/dosya soruluyorsa ilk refleks budur.
3. **Alan adını tıklayarak öğren.** Durum çubuğu veya sağ tık ▸ Apply as Filter.
4. **Capture ≠ Display.** Yakalama filtresi ayrı dildir, inceleme filtresi ayrı.
5. **`!=` yerine `!(... == ...)`.** Çift yönlü alanlarda daha güvenli.
6. **Gürültüyü at.** `!(arp || mdns || ssdp || llmnr)` ile arka planı temizle.
7. **Zamanı UTC yap.** Saat dilimi karışıklığı en sık yapılan hatadır.
8. **Nadir olanı ara.** Saldırgan kalabalığa karışmaya çalışır ama genelde tektir.
9. **Tek kanıtla hüküm verme.** En az iki bağımsız kanıt topla.
10. **Göremediğini "yok" sanma.** Şifreli/sensör kör noktası olabilir; sınırı raporla.

> **Daha derin pratik için** bu cheatsheet'in kardeş sayfasına bak: *Wireshark DFIR Problem Kılavuzu (250 Vaka)* — her senaryo için adım adım yol tarifi.
