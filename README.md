# Smart DNS مخصوص کنسول + سرور واسط (Tunnel Relay)

این پروژه الان به‌صورت **دو-سروره** طراحی شده:

1. **Main DNS Server** (سرور اصلی):
   - CoreDNS اصلی روی این سرور اجرا می‌شود.
   - به upstreamها Resolve می‌کند.
   - مستقیم Public نمی‌شود (بهتر است فقط از طریق تانل WireGuard در دسترس باشد).

2. **Relay Server** (سرور واسط نزدیک کاربر/ISP):
   - DNS کنسول‌ها به این سرور ست می‌شود.
   - کوئری‌ها را از طریق تانل WireGuard به Main می‌فرستد.

این معماری دقیقاً چیزی است که برای «سرور واسط + تانل به سرور اصلی» لازم دارید.

---

## فایل‌های مهم

- `docker-compose.yml`
  - Profile `main`: اجرای WireGuard + CoreDNS اصلی
  - Profile `relay`: اجرای WireGuard + CoreDNS واسط Public
- `Corefile.main`: تنظیمات DNS اصلی
- `Corefile.relay`: تنظیمات DNS واسط (forward روی IP تانل)
- `wireguard/main/wg0.conf`: کانفیگ WG سرور اصلی
- `wireguard/relay/wg0.conf`: کانفیگ WG سرور واسط

---

## پیش‌نیاز

- دو VPS لینوکسی با IP عمومی
- Docker + Docker Compose روی هر دو
- باز بودن پورت‌ها:
  - روی Main: `51820/udp`
  - روی Relay: `53/udp`, `53/tcp`, اختیاری `9153/tcp`

---

## 1) تنظیم WireGuard (یک‌بار)

روی هر دو سرور کلید بسازید:

```bash
wg genkey | tee privatekey | wg pubkey > publickey
```

سپس مقادیر را در فایل‌های زیر جایگزین کنید:

- `wireguard/main/wg0.conf`
  - `REPLACE_MAIN_PRIVATE_KEY`
  - `REPLACE_RELAY_PUBLIC_KEY`
- `wireguard/relay/wg0.conf`
  - `REPLACE_RELAY_PRIVATE_KEY`
  - `REPLACE_MAIN_PUBLIC_KEY`
  - `REPLACE_MAIN_PUBLIC_IP`

---

## 2) اجرای سرور اصلی (Main)

روی سرور اصلی:

```bash
git clone <repo-url>
cd dnsgaming
docker compose --profile main up -d
```

بررسی وضعیت:

```bash
docker compose --profile main ps
docker logs --tail=100 wg-main
docker logs --tail=100 smartdns-main
```

---

## 3) اجرای سرور واسط (Relay)

روی سرور واسط:

```bash
git clone <repo-url>
cd dnsgaming
docker compose --profile relay up -d
```

بررسی وضعیت:

```bash
docker compose --profile relay ps
docker logs --tail=100 wg-relay
docker logs --tail=100 smartdns-relay
```

---

## 4) تست مسیر تانل DNS

روی سرور Relay:

```bash
# باید از طریق CoreDNS relay به main برسد
dig @127.0.0.1 playstation.com +short
```

تست از بیرون (کلاینت):

```bash
dig @RELAY_PUBLIC_IP xboxlive.com +short
```

---

## 5) تنظیم کنسول‌ها

روی PS/Xbox/Switch فقط DNS را روی **IP عمومی سرور Relay** بگذارید.

- Primary DNS: `RELAY_PUBLIC_IP`
- Secondary DNS: `1.1.1.1` (یا IP Relay دوم)

---

## نکات مهم عملیاتی

- اگر می‌خواهید قطعاً همه‌چیز فقط از تانل برود، در `Corefile.relay` بلاک fallback را حذف کنید.
- برای production بهتر است Relay دوم (High Availability) داشته باشید.
- روی Main بهتر است پورت 53 را Public باز نکنید.
- برای abuse کنترل، روی Relay rate limit فایروال بگذارید.

---

## مانیتورینگ

متریک‌ها روی Relay در `9153`:

```bash
curl http://RELAY_PUBLIC_IP:9153/metrics | head
```

