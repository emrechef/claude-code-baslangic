# Modül 2 — CLAUDE.md: Hafıza Sistemi

**Süre:** ~45 dakika  
**Bu modül en önemli modül** — Claude Code'un diğer tüm özelliklerinin temeli.

---

## CLAUDE.md Nedir?

Claude her oturum açtığında "hafızası" sıfırlanır. Bir önceki konuşmayı hatırlamaz.
Ama bir şeyi hatırlar: **CLAUDE.md dosyalarını.**

Bu dosyalar, Claude'a "Bu projede çalışırken şunları bil" diye yazdığın talimatlar.

```
Sen konuşmayı bitirirsin
Claude hafızasını siler
Ama sen CLAUDE.md'e "bu projede TypeScript kullan" yazmışsın
Sonraki oturumda Claude bunu yeniden okur
Sanki hiç unutmamış gibi davranır
```

**Benzetme:** CLAUDE.md, Claude'a verdiğin el kitabı gibi. Her gün işe başlarken okur.

---

## Üç Katmanlı Hafıza Sistemi

Claude birden fazla CLAUDE.md dosyasını bir arada okuyabilir:

```
~/.claude/CLAUDE.md              ← 1. Katman: Global (tüm projeler)
│
/proje-klasörü/
├── CLAUDE.md                    ← 2. Katman: Proje (bu proje)
├── .claude.local.md             ← 3. Katman: Kişisel (sadece senin bilgisayarında)
└── alt-klasör/
    └── CLAUDE.md                ← 4. Katman: Dizin (sadece o klasörde)
```

**Ne zaman hangisini kullanırsın?**

| Dosya | Buraya yaz |
|-------|-----------|
| `~/.claude/CLAUDE.md` | Her projede geçerli tercihler: "Her zaman Türkçe cevap ver", "tab yerine 2 boşluk kullan" |
| `./CLAUDE.md` | Bu projeye özel bilgiler: "Bu proje Node.js, şöyle çalıştırılır: `npm start`" |
| `./.claude.local.md` | Senin kişisel notların: "Benim test verim şurada" — takım arkadaşlarınla paylaşmazsın |

---

## Bölüm 1 — Global CLAUDE.md Oluştur

Bu dosya bilgisayarındaki **tüm** Claude Code oturumlarında okunur.

Terminalde şunu çalıştır:
```bash
mkdir -p ~/.claude
```
> `mkdir -p` = klasörü oluştur (zaten varsa hata verme)
> `~` = senin ev klasörün (`/Users/futurelife`)

Şimdi dosyayı oluştur. VS Code'u kullanabilirsin:
```bash
code ~/.claude/CLAUDE.md
```

Dosyaya şunları yaz (kendinize göre düzenlemeyi unutma):

```markdown
# Benim Global Tercihlerim

## İletişim Stili
- Bana her zaman Türkçe cevap ver
- Teknik terimleri kullandığında parantez içinde kısa açıklama ekle
- Uzun açıklamalar yerine kısa, net cevaplar ver

## Kod Yazma Tercihleri
- Varsayılan olarak JavaScript kullan (TypeScript değil, söylemediğim sürece)
- Girintileme: 2 boşluk (tab değil)
- Yorum satırı: Sadece gerçekten gerektiğinde yaz
- Fonksiyon isimleri: İngilizce, açıklayıcı

## Önemli Kurallar
- Asla rm -rf komutunu onaysız çalıştırma
- Git'te main branch'e (ana dala) direkt commit etme
- Benden onay almadan dosya silme

## Kısayollar (Benim Özel Komutlarım)
- "Devam et" dediğimde: son planladığımızı uygulamaya başla
- "Bitir" dediğimde: çalışmayı tamamla ve git commit oluştur
```

Kaydet ve kapat.

### Test et:
```bash
claude -p "Bana kısa bir selamlama yaz"
```
Türkçe cevap veriyorsa global CLAUDE.md çalışıyor!

---

## Bölüm 2 — Proje CLAUDE.md Oluştur

Pratik için bir demo proje klasörü oluşturalım:

```bash
mkdir -p "/Users/futurelife/Desktop/cloude vs1/modul-02-claude-md/pratik/benim-projem"
cd "/Users/futurelife/Desktop/cloude vs1/modul-02-claude-md/pratik/benim-projem"
code CLAUDE.md
```

Bu dosyaya proje bilgilerini yaz:

```markdown
# Benim Projem

## Bu Proje Ne?
Bir yapılacaklar listesi uygulaması. Görevleri JSON dosyasında saklar.

## Projeyi Çalıştırmak İçin
```bash
node index.js          # uygulamayı başlat
node index.js list     # görevleri listele
node index.js add "görev adı"  # yeni görev ekle
```

## Dosya Yapısı
```
benim-projem/
├── index.js      → Ana program dosyası
├── gorevler.js   → Görev okuma/yazma işlemleri  
└── gorevler.json → Veri dosyası (otomatik oluşturulur)
```

## Dikkat Et!
- gorevler.json dosyasını silme — tüm veriler orada
- index.js'i düzenlerken gorevler.js'e de bak, birbirlerine bağlılar

## Bu Projede Kullandığım Teknolojiler
- Node.js (JavaScript, ama tarayıcıda değil, terminalde çalışır)
- Harici kütüphane yok — sadece Node.js ile gelen araçlar
```

Kaydet.

### Test et:
```bash
claude
# Oturum açıldıktan sonra:
> Bu projede hangi komutla uygulamayı çalıştırıyorum?
```
`node index.js` cevabını veriyorsa CLAUDE.md okunuyor!

---

## Bölüm 3 — Canlı Komut Enjeksiyonu (!)

Bu özellik çok güçlü. CLAUDE.md içinde `!` işaretiyle terminal komutu çalıştırabilirsin.
Claude dosyayı okurken bu komutları çalıştırır ve çıktıyı görür.

**Ne işe yarar?** Claude her oturum açtığında otomatik olarak en güncel bilgiyi alır.

CLAUDE.md dosyana şunu ekle:

```markdown
## Güncel Proje Durumu (Otomatik)
- Şu an bulunduğum dal (branch): !`git branch --show-current 2>/dev/null || echo "git yok"`
- Son değişiklik: !`git log --oneline -1 2>/dev/null || echo "commit yok"`
- Değiştirilmiş dosyalar: !`git status --short 2>/dev/null || echo "temiz"`
```

> `2>/dev/null || echo "..."` = "hata olursa bu mesajı göster" demek.
> Henüz git kurulu değilse kırılmasın diye ekliyoruz.

Şimdi `claude` ile oturum aç ve sor: "Projede son ne değişti?"
Eğer git kullanıyorsan gerçek bilgiyi göreceksin. Kullanmıyorsan "git yok" mesajı görürsün.

---

## Bölüm 4 — /memory Komutu

Oturum açıkken CLAUDE.md'yi düzenleyebilirsin:

```bash
claude
> /memory
```

Bu komut CLAUDE.md dosyalarını listeler ve düzenlemenizi sağlar.

Veya Claude'a direkt söyleyebilirsin:
```
> CLAUDE.md'ye şunu ekle: "Her zaman önce testleri çalıştır, sonra kod yaz"
```
Claude dosyayı günceller.

---

## Bölüm 5 — Kişisel Override Dosyası (.claude.local.md)

Bu dosya takım arkadaşlarınla paylaşmazsın (`.gitignore`'a eklersin).
Sadece sene özel notlar için.

```bash
code "/Users/futurelife/Desktop/cloude vs1/modul-02-claude-md/pratik/benim-projem/.claude.local.md"
```

İçerik:
```markdown
# Benim Kişisel Notlarım

## Test Ayarları
- Test verim: /Users/futurelife/test-data klasöründe
- Lokal veritabanı portu: 5433 (standart 5432 değil)

## Benim Kısayollarım
- "hızlı test" dediğimde: sadece unit testleri çalıştır (integration testleri değil)
```

> `unit test` = küçük, hızlı testler
> `integration test` = daha kapsamlı, yavaş testler

---

## Görsel Özet — Hangi Dosya Ne Zaman Okunur?

```
claude komutu çalıştırıldığında:

1. ~/.claude/CLAUDE.md        okunur (global tercihler)
         ↓
2. ./CLAUDE.md                okunur (proje bilgileri)
         ↓
3. ./.claude.local.md         okunur (kişisel notlar)
         ↓
4. ./alt-klasör/CLAUDE.md     okunur (sadece o klasördeysen)
         ↓
Claude artık hepsini biliyor ve oturuma başlıyor
```

---

## Test — Modülü Tamamladın mı?

1. `~/.claude/CLAUDE.md` dosyası var mı?
   ```bash
   cat ~/.claude/CLAUDE.md
   ```
   İçerik görünmeli.

2. Proje klasöründe `claude` aç, "Bu projeyi nasıl çalıştırıyorum?" sor.
   CLAUDE.md'den doğru cevabı vermeli.

3. `/memory` komutu çalışıyor mu?

---

## Sıradaki Modül

Harika! Artık Claude'a hafıza verdim diyebilirsin.

→ **[Modül 3: Kendi Komutlarını Yaz](../modul-03-slash-commands/REHBER.md)**

Modül 3'te `/standup`, `/commit` gibi kendi kısayollarını oluşturacaksın.
