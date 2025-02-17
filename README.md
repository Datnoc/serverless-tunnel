# WireGuard Tünel Kurulumu (ABD Ana Sunucu + Yerel Sunucu)

Bu belge, ABD'deki **ana sunucu** ile **yerel (edge) sunucu** arasındaki WireGuard tünelini kurmak için gereken adımları açıklar. Bu yapı sayesinde **Minecraft gibi düşük gecikme gerektiren uygulamalar için optimizasyon sağlanacaktır.**

---

## 📌 Yapı ve Amaç  
Bu sistem, kullanıcıların **yerel (Türkiye) sunucusuna bağlanarak daha düşük ping ile ABD’deki güçlü altyapıya erişmesini** sağlar.  

📍 **Ana Sunucu (ABD) → Güçlü altyapı, ana işlemler burada**  
📍 **Yerel Sunucu (Türkiye) → Düşük gecikme için proxy görevi görür**  

**WireGuard**, iki sunucu arasında güvenli bir VPN tüneli oluşturarak **yerel sunucunun ABD’deki ana sunucu üzerinden çıkış yapmasını** sağlar.  

---

## 🛠 Sunucu Kurulumu  

### 1️⃣ ABD'deki Ana Sunucu (WireGuard Sunucu)  

Ana sunucu, gelen bağlantıları kabul eder ve trafiği yönlendirir.  

#### 🔹 WireGuard Yükleme  
```bash
apt update && apt install -y wireguard
🔹 Konfigürasyon (ABD Sunucu - /etc/wireguard/wg0.conf)
ini
Copy
Edit
[Interface]
PrivateKey = ANA_SUNUCU_PRIVATE_KEY
Address = 10.0.0.1/24
ListenPort = 51820

# NAT için
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = YEREL_SUNUCU_PUBLIC_KEY
AllowedIPs = 10.0.0.2/32
🔹 WireGuard Servisini Başlat
bash
Copy
Edit
systemctl enable --now wg-quick@wg0
2️⃣ Yerel (Türkiye) Sunucu (WireGuard İstemci)
Yerel sunucu, WireGuard üzerinden ana sunucuya bağlanarak VPN tüneli oluşturur ve gelen trafiği yönlendirir.

🔹 WireGuard Yükleme
bash
Copy
Edit
apt update && apt install -y wireguard
🔹 Konfigürasyon (Yerel Sunucu - /etc/wireguard/wg0.conf)
ini
Copy
Edit
[Interface]
PrivateKey = YEREL_SUNUCU_PRIVATE_KEY
Address = 10.0.0.2/24

[Peer]
PublicKey = ANA_SUNUCU_PUBLIC_KEY
Endpoint = ANA_SUNUCU_IP:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
🔹 WireGuard Servisini Başlat
bash
Copy
Edit
systemctl enable --now wg-quick@wg0
🔹 Bağlantıyı Doğrula
bash
Copy
Edit
wg
🔁 Proxy Yapılandırması
Yerel sunucu, WireGuard tünelinden gelen trafiği ABD sunucusuna yönlendirecek şekilde yapılandırılır.

🔹 IP Tables ile Proxy
Yerel sunucunun tüm trafiği WireGuard tüneline yönlendirmesi için:

bash
Copy
Edit
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 25565 -j DNAT --to-destination 10.0.0.1:25565
iptables -t nat -A POSTROUTING -o wg0 -j MASQUERADE
Bu, Türkiye sunucusuna bağlanan Minecraft oyuncularının trafiğini WireGuard üzerinden ABD sunucusuna iletir.

✅ Test ve Doğrulama
Bağlantının doğru çalıştığını test etmek için:

🔹 Yerel sunucudan WireGuard bağlantısını doğrula:

bash
Copy
Edit
ping -c 4 10.0.0.1
🔹 Ana sunucudan istemciyi kontrol et:

bash
Copy
Edit
wg show
🔹 Minecraft Sunucusuna Bağlantıyı Test Et:
Oyuncular yerel sunucunun IP’siyle bağlanır, ancak işlem gücü ABD sunucusunda çalışır.

🚀 Sonuç
✅ WireGuard tüneli, yerel sunucu ile ana sunucu arasında güvenli ve düşük gecikmeli bir bağlantı sağlar.
✅ Yerel oyuncular, ABD’ye doğrudan bağlanmak yerine, Türkiye’deki edge sunucusuna bağlanarak daha düşük ping alır.
✅ Proxy ayarları, Minecraft trafiğini WireGuard üzerinden yönlendirerek oyun performansını optimize eder.

Bu yapılandırma ile müşterilerinize daha düşük ping ile yüksek performanslı bir oyun deneyimi sunabilirsiniz. 🚀
