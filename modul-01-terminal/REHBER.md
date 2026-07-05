# Modül 1 — Terminal ve CLI Komutları

**Süre:** ~30 dakika  
**Öğreneceklerin:** Claude Code'u terminal'den çalıştırmanın tüm yolları

---

## Bu Modülde Ne Öğreneceksin?

Claude Code'u iki farklı şekilde kullanabilirsin:

```
1. İnteraktif mod (oturum açarsın, sohbet edersin)
   → claude         [sadece bu komutu yaz, Enter'a bas]

2. Non-interactive mod (tek satır gönderirsin, cevap alırsın, kapanır)
   → claude -p "sorun veya görev"
```

Bu modülde her ikisini de deneyeceksin.

---

## Bölüm 1 — İlk Komutlar

VS Code'u aç, `` Ctrl+` `` ile terminal aç, şunları sırayla dene:

### 1.1 Versiyon kontrolü
```bash
claude --version
```
Çıktı: `claude-code/2.X.XXX` gibi bir şey görmelisin.

### 1.2 Yardım menüsü
```bash
claude --help
```
Bu komut tüm seçenekleri listeler. Hepsini ezberleme — var olduğunu bil yeter.

### 1.3 Non-interactive mod (oturum açmadan)
```bash
claude -p "Merhaba, beni duyuyor musun?"
```
Claude cevap verir ve terminal geri döner. Oturum açılmaz.
> `-p` = **p**rompt (istem). Kısa yazım. Uzun yazımı: `--print`

```bash
claude -p "Şu an hangi dizindeyim?"
```
Çalıştığın dizini söylemeli.

---

## Bölüm 2 — İnteraktif Oturum Komutları

Şimdi bir oturum açalım:

```bash
claude
```

İçeri girince `>` işareti görürsün. Artık Claude ile konuşabilirsin.

### Oturum içi slash komutları

Bunları `>` işaretinden sonra yaz:

| Komut | Ne yapar? |
|-------|-----------|
| `/help` | Tüm kullanılabilir komutları listeler |
| `/status` | Oturum bilgileri: model, session ID |
| `/cost` | Bu oturumda kaç token kullandın |
| `/clear` | Hafızayı temizle, sıfırdan başla (oturumu kapatmaz) |
| `/compact` | Uzun konuşmayı sıkıştır, token tasarrufu yap |
| `/memory` | CLAUDE.md dosyalarını düzenle (Modül 2'de öğreneceksin) |
| `/config` | Ayarlar menüsü |
| `/ide` | VS Code bağlantısını kontrol et |

Dene:
```
/help
```
Listelenen komutlara bak. Şimdi sadece built-in (yerleşik) komutlar var. Sonraki modüllerde kendi komutlarını ekleyeceksin.

Oturumdan çıkmak için: `Ctrl+C` veya `exit` yaz.

---

## Bölüm 3 — Faydalı Flag'ler (Seçenekler)

### --continue / -c : Son konuşmayı devam ettir

```bash
claude   # Bir şeyler konuş
# Ctrl+C ile çık
claude -c  # Kaldığın yerden devam et!
```

Bu çok işe yarıyor. Kapattıktan sonra "hah bir de şunu soracaktım" diyebilirsin.

### --safe-mode : Her şeyi devre dışı bırak

```bash
claude --safe-mode -p "Test"
```

Config dosyaların, CLAUDE.md'lerin, hook'ların hiçbiri yüklenmez.
**Ne zaman kullanırsın?** Bir şeyler bozulduğunda debug etmek için.

### --add-dir : Ek klasöre erişim ver

```bash
claude --add-dir ~/Desktop -p "Desktop'taki dosyaları listele"
```

Normalde Claude sadece çalıştığın klasörü görebilir. Başka bir klasörü de görmesini istersen bu flag'i kullan.

---

## Bölüm 4 — VS Code Entegrasyonu

Claude Code, VS Code ile konuşabilir. Bunu test edelim:

1. VS Code'u aç
2. Terminal'de `claude` yaz ve oturum aç
3. Oturum içinde `/ide` yaz

Şunu görmelisin:
```
✓ Connected to VS Code
```

Bu bağlantı sayesinde:
- Claude bir dosyayı düzenlediğinde VS Code'da diff (fark) görürsün
- Claude'a "şu dosyayı aç" diyebilirsin
- Claude hata mesajlarını VS Code'dan okuyabilir

---

## Bölüm 5 — Özet Tablosu

```
Komut                          Ne yapar?
──────────────────────────────────────────────────────
claude                         Oturum aç (interaktif)
claude -p "..."                Tek satır sor, cevap al (non-interaktif)
claude -c                      Son oturumu devam ettir
claude --safe-mode             Config olmadan çalıştır (debug için)
claude --add-dir /yol          Ek klasöre erişim ver
claude --help                  Tüm seçenekleri göster
claude --version               Versiyon numarasını göster
──────────────────────────────────────────────────────
Oturum içi
──────────────────────────────────────────────────────
/help                          Komut listesi
/status                        Oturum bilgisi
/cost                          Token kullanımı
/clear                         Hafızayı temizle
/compact                       Konuşmayı sıkıştır
/memory                        CLAUDE.md'yi düzenle
/config                        Ayarlar
/ide                           VS Code bağlantısını kontrol et
```

---

## Test — Modülü Tamamladın mı?

Şu iki şeyi yapabiliyorsan Modül 1 tamam:

1. `claude -p "Şu an çalıştığım klasörün adı ne?"` → Doğru klasörü söylüyor
2. `claude` ile oturum aç, `/help` yaz → Komut listesi görünüyor

---

## Sıradaki Modül

Claude Code'u çalıştırmayı öğrendin. Şimdi en kritik özelliği öğrenme zamanı:

→ **[Modül 2: CLAUDE.md — Hafıza Sistemi](../modul-02-claude-md/REHBER.md)**

Bu modülde Claude'un her oturumda otomatik olarak ne bildiğini kontrol etmeyi öğreneceksin.
