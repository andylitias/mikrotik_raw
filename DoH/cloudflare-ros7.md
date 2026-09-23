# ==========================================================
# ROS v7: 1. Unduh & Impor Root CA Certificate (Delay 2s)
# ==========================================================
/tool fetch url="https://curl.se/ca/cacert.pem" check-certificate=no
:delay 2s
/certificate import file-name=cacert.pem passphrase=""
:delay 2s

# ==========================================================
# ROS v7: 2. Static DNS Mapping Endpoint DoH Cloudflare
# ==========================================================
/ip dns static remove [find name="cloudflare-dns.com"]
/ip dns static
add name=cloudflare-dns.com address=1.1.1.1 comment="Cloudflare DoH IP 1"
add name=cloudflare-dns.com address=1.0.0.1 comment="Cloudflare DoH IP 2"

# ==========================================================
# ROS v7: 3. Konfigurasi DoH Server, Concurrency & Cache
# ==========================================================
/ip dns set servers="" use-doh-server="https://cloudflare-dns.com/dns-query" verify-doh-cert=yes allow-remote-requests=yes max-concurrent-queries=200 doh-max-server-connections=20 doh-max-concurrent-queries=200 cache-size=8192KiB cache-max-ttl=1h
/ip dns cache flush

# ==========================================================
# ROS v7: 4. Redirect DNS Port 53 Klien LAN ke Router
# ==========================================================
/ip firewall nat remove [find comment="Redirect UDP DNS ke Router"]
/ip firewall nat remove [find comment="Redirect TCP DNS ke Router"]
/ip firewall nat
add chain=dstnat protocol=udp dst-port=53 action=redirect to-ports=53 comment="Redirect UDP DNS ke Router"
add chain=dstnat protocol=tcp dst-port=53 action=redirect to-ports=53 comment="Redirect TCP DNS ke Router"

# ==========================================================
# ROS v7: 5. Blokir Protokol QUIC / HTTP3 (Status: DISABLED)
# ==========================================================
/ip firewall filter remove [find comment="Block QUIC / HTTP3"]
/ip firewall filter
add chain=forward action=reject reject-with=icmp-network-unreachable protocol=udp dst-port=443,80 comment="Block QUIC / HTTP3" disabled=yes place-before=0
