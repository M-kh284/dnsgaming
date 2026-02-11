# Smart DNS برای کنسول‌های بازی (مشابه ایده‌ی Shecan)

این پروژه یک DNS Resolver سبک با CoreDNS می‌سازد که برای **PS/Xbox/Nintendo** قابل استفاده است.
هدف آن:

- پایداری بهتر DNS
- کاهش خطاهای Resolve روی برخی سرویس‌های گیم
- Cache مناسب برای کاهش زمان پاسخ
- امکان دیپلوی سریع روی VPS

> نکته مهم: این پروژه یک Smart DNS پایه است؛ برای سناریوهای Unblock پیچیده (مانند تغییر مسیر ترافیک HTTP/S) باید کنار DNS از Reverse Proxy / Tunnel هم استفاده شود.

---

## پیش‌نیاز

- یک سرور Linux با IP ثابت
- Docker + Docker Compose
- باز بودن پورت‌های `53/udp` و `53/tcp`

---

## راه‌اندازی سریع

```bash
git clone <repo-url>
cd dnsgaming
docker compose up -d
```

بررسی سلامت:

```bash
docker compose ps
docker logs --tail=50 smartdns-gaming
```

تست Resolve:

```bash
# YOUR_SERVER_IP را با IP سرور خود عوض کنید
nslookup playstation.com YOUR_SERVER_IP
dig @YOUR_SERVER_IP xboxlive.com +short
```

---

## تنظیم روی کنسول

### PlayStation (PS4/PS5)

1. Settings → Network → Set Up Internet Connection
2. حالت Manual برای DNS
3. Primary DNS: `IP سرور شما`
4. Secondary DNS: یک DNS عمومی (مثلاً `1.1.1.1`)

### Xbox

1. Settings → General → Network settings → Advanced settings
2. DNS settings → Manual
3. Primary DNS: `IP سرور شما`
4. Secondary DNS: `1.1.1.1`

### Nintendo Switch

1. System Settings → Internet → Internet Settings
2. شبکه را انتخاب کنید → Change Settings
3. DNS Settings = Manual
4. Primary DNS: `IP سرور شما`
5. Secondary DNS: `1.1.1.1`

---

## سفارشی‌سازی

فایل `Corefile` را ویرایش کنید:

- TTL کش (`cache 300`)
- دامنه‌های مخصوص بازی (بلاک‌های `playstation.net`, `xboxlive.com`, ...)
- Upstreamها در `forward`

بعد از تغییر:

```bash
docker compose restart smartdns
```

---

## مانیتورینگ

متریک‌های Prometheus روی پورت `9153` فعال است.

نمونه:

```bash
curl http://YOUR_SERVER_IP:9153/metrics | head
```

---

## نکات امنیتی/عملیاتی

- حتماً Firewall تنظیم کنید (فقط پورت‌های لازم باز باشند).
- اگر DNS را Public می‌کنید، Rate limit در لایه‌ی شبکه قرار دهید.
- در صورت مصرف بالا، سرویس را پشت Load Balancer و چند نود DNS اجرا کنید.

