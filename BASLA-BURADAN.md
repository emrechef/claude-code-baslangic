# Claude Code Öğrenme Rehberi

Hoş geldin! Bu klasör, Claude Code'u **adım adım, yaparak öğrenmen** için tasarlandı.

---

## Bu Rehber Nasıl Çalışır?

Her modül kendi klasöründe. İçine girince bir `REHBER.md` dosyası bulacaksın.
O dosyayı oku, adımları takip et, sonra sıradaki modüle geç.

**Süre:** Her modül ~30-45 dakika. Toplam ~6-8 saat.

---

## Modüller — Sırayla İlerle

```
modul-01-terminal/      ← Buradan başla: Temel komutlar
modul-02-claude-md/     ← EN ÖNEMLİSİ: Hafıza sistemi
modul-03-slash-commands/ ← Kendi komutlarını yaz
modul-04-skills/        ← Otomatik uzman modlar
modul-05-hooks/         ← Otomatik tetikleyiciler
modul-06-github/        ← GitHub bağlantısı
modul-07-mcp/           ← Dış araç bağlantıları
modul-08-capstone/      ← Hepsini bir arada
```

---

## Başlamadan Önce — Temel Kavramlar

Aşağıdaki terimleri şimdi öğren. Modüllerde sürekli karşına çıkacak.

### CLI nedir?
**CLI** = Command Line Interface (Komut Satırı Arayüzü).
Fare ile tıklamak yerine **yazarak** kontrol ettiğin ekran. VS Code'da `` Ctrl+` `` ile açarsın.

### Terminal nedir?
Komutları yazdığın siyah ekran. macOS'ta "Terminal" uygulaması veya VS Code'un içindeki terminal.

### Flag nedir?
Bir komuta eklediğin seçenek. Örnek: `claude --version` — burada `--version` bir flag.
Genellikle `--` ile başlar (uzun) veya `-` ile başlar (kısa). İkisi aynı şeyi yapar:
- `--continue` = uzun yazım
- `-c` = kısa yazım (aynı anlam)

### Slash command nedir?
`/` ile başlayan komutlar. Claude Code'un içinde yazarsın. Örnek: `/help`, `/clear`.
Bunları kendin de yazabilirsin — sonraki modüllerde göreceksin.

### Markdown (.md) nedir?
Sade metin formatı. `#` başlık yapar, `**bold**` kalın yapar, `` `kod` `` gri kutu yapar.
CLAUDE.md dosyaları Markdown ile yazılır.

---

## Hızlı Kontrol — Claude Code Kurulu mu?

Terminal aç (VS Code'da `` Ctrl+` ``) ve şunu yaz:

```bash
claude --version
```

Bir versiyon numarası görüyorsan (örn. `2.1.196`) — hazırsın, Modül 1'e geç.

Hata alıyorsan kurulum gerekiyor:
```bash
npm install -g @anthropic-ai/claude-code
```

---

## Sıradaki Adım

→ [`modul-01-terminal/REHBER.md`](modul-01-terminal/REHBER.md) dosyasını aç
