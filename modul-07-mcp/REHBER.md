# Modül 7 — MCP Servers: Dış Araç Bağlantıları

**Süre:** ~30 dakika  
**Öğreneceklerin:** Claude'a internet, dokümantasyon ve tarayıcı erişimi vermek

---

## MCP Nedir?

**MCP** = Model Context Protocol (Model Bağlam Protokolü)
Claude'un dış araçlarla konuşmasını sağlayan bir sistem.

**Olmadan:** Claude sadece dosyaları görebilir, sana sorduğun şeye eğitim verisiyle cevap verir.

**Olunca:** Claude gerçek zamanlı internet verisi çekebilir, güncel dokümantasyona bakabilir, tarayıcıyı kontrol edebilir.

**Benzetme:** MCP, Claude'a yeni "duyu organları" takıyor.

---

## MCP Konfigürasyonu Nereye Gider?

```
~/.claude/                    ← Global MCP (tüm projeler)
│  (claude mcp add komutu buraya yazar)

proje-klasörü/
└── .mcp.json                 ← Proje MCP (sadece bu proje, git'e eklenebilir)
```

---

## Bölüm 1 — context7: Canlı Dokümantasyon

Claude'un eğitim verisi belirli bir tarihe kadar. Yeni kütüphane versiyonları bilmeyebilir.
`context7`, Claude'un gerçek, güncel dokümantasyona bakmasını sağlar.

### Kurulum:
```bash
claude mcp add --transport http context7 https://mcp.context7.com/mcp
```

### Test et:
```bash
claude
> React 19'daki yeni use() hook'u nasıl kullanılıyor? (Güncel dokümantasyona bak)
```

Context7 olmadan: Claude tahmin eder.
Context7 ile: Gerçek React 19 dökümanını çeker.

---

## Bölüm 2 — Playwright: Tarayıcı Kontrolü

Claude bir web tarayıcısını kontrol edebilir — sayfaları açar, ekran görüntüsü alır, form doldurur.

### Kurulum:
```bash
claude mcp add playwright -- npx @playwright/mcp@latest
```

> `npx` = Node.js'te paket çalıştır (indirmeden)

### Test et:
```bash
claude
> claude.ai sayfasının başlığını oku
```

Claude tarayıcıyı açar, sayfaya gider ve sana başlığı söyler.

---

## Bölüm 3 — Proje .mcp.json

Takım arkadaşlarınla paylaşmak istediğin MCP'ler için proje kökünde `.mcp.json` oluştur:

```bash
code "/Users/futurelife/Desktop/cloude vs1/.mcp.json"
```

İçerik:
```json
{
  "context7": {
    "type": "http",
    "url": "https://mcp.context7.com/mcp"
  }
}
```

Bu dosyayı git'e eklersen, projeyi klonlayan herkes otomatik context7'yi alır.

---

## Bölüm 4 — Yönetim Komutları

```bash
# Kurulu MCP'leri listele
claude mcp list

# Bir MCP hakkında detay
claude mcp get context7

# MCP bağlantısını test et (debug)
claude --mcp-debug -p "Test"

# MCP kaldır
claude mcp remove context7
```

---

## Görsel Özet

```
Claude'un Normal Hali:
  Sadece dosyaları görür
  Eğitim verisinden cevap verir

Claude + context7:
  Sorulan kütüphanenin güncel dokümanlarını çeker
  "React 19 docs" → gerçek içerik

Claude + Playwright:
  Tarayıcıyı açar
  Sayfalarda gezinir
  Ekran görüntüsü alır

Claude + GitHub MCP:
  GitHub API'ye direkt erişir
  Issue/PR yönetir
```

---

## Test — Modülü Tamamladın mı?

```bash
claude mcp list
```
Listede en az `context7` görünmeli.

```bash
claude -p "context7 kullanarak Express.js 5'te ne değişti?"
```
Gerçek dokümantasyondan bilgi getirmeli.

---

## Sıradaki Modül

→ **[Modül 8: Capstone — Hepsini Bir Araya Getir](../modul-08-capstone/REHBER.md)**

Son modülde tüm öğrendiklerini tek bir gerçek projede birleştiriyorsun.
