# Modül 3 — Kendi Slash Command'larını Yaz

**Süre:** ~45 dakika  
**Öğreneceklerin:** Sık yaptığın işleri tek bir komuta indirgemek

---

## Slash Command Nedir?

`/` ile başlayan komutlar. Örnekler:
- `/standup` → "Bugün ne yaptım?" raporu oluştur
- `/commit` → İyi bir commit mesajı yazıp kaydet
- `/explain index.js` → Bu dosyayı açıkla

**Nasıl çalışır?** Her komut bir `.md` dosyası. Claude o dosyayı okur ve içindeki talimatlara göre davranır.

---

## Komut Dosyaları Nereye Gider?

İki yer var:

```
~/.claude/commands/         ← Tüm projelerde kullanılır (global)
│  standup.md               ← /standup komutu
│  explain.md               ← /explain komutu
│  bug-report.md            ← /bug-report komutu

proje-klasörü/.claude/commands/   ← Sadece bu projede kullanılır
   commit.md                ← /commit komutu
   review-changes.md        ← /review-changes komutu
```

---

## Komut Dosyası Nasıl Yazılır?

```markdown
---
description: /help'te görünecek kısa açıklama
argument-hint: <zorunlu-argüman> [opsiyonel-argüman]
allowed-tools: ["Bash", "Read"]
---

Buraya Claude'a ne yapması gerektiğini yaz.

Kullanıcının yazdığı şey: $ARGUMENTS
Canlı terminal çıktısı: !`git status`
```

**Satırların anlamı:**

| Kısım | Ne işe yarar? |
|-------|--------------|
| `description` | `/help` yazınca görünen açıklama |
| `argument-hint` | Kullanıcıya ne yazması gerektiğini gösterir: `/komut <zorunlu> [opsiyonel]` |
| `allowed-tools` | Claude'un kullanabileceği araçlar (izin sorar) |
| `$ARGUMENTS` | Kullanıcının `/komut` sonrası yazdığı metin |
| `!` | CLAUDE.md'deki gibi, terminal komutu çalıştırır |

---

## Bölüm 1 — Global Komutlar Oluştur

### 1.1 Klasörü oluştur
```bash
mkdir -p ~/.claude/commands
```

### 1.2 /standup komutu

```bash
code ~/.claude/commands/standup.md
```

İçerik:
```markdown
---
description: Son 24 saatteki git geçmişinden günlük standup raporu oluştur
argument-hint: [proje-adı]
allowed-tools: ["Bash", "Read"]
---

# Günlük Standup Raporu

Proje: $ARGUMENTS (belirtilmemişse mevcut proje)

## Bağlam Toplama

Bugünün tarihi: !`date "+%d %B %Y, %A"`
Mevcut proje: !`basename $(pwd)`
Son 24 saatteki commitler: !`git log --since="24 hours ago" --oneline 2>/dev/null || echo "git bulunamadı"`
Değişen dosyalar: !`git diff --name-only HEAD~1 2>/dev/null | head -10 || echo "değişiklik yok"`

## Rapor Formatı

Yukarıdaki bilgilere dayanarak şu formatta yaz:

**Dün Yaptıklarım:**
[git commitlerinden 2-3 somut madde çıkar]

**Bugün Yapacaklarım:**
[Yapılan işin mantıksal devamını öner]

**Engeller:**
[Bana sor: "Bir engelin var mı? (yoksa Enter'a bas)"]

Kısa tut — her madde tek cümle.
```

### 1.3 /explain komutu

```bash
code ~/.claude/commands/explain.md
```

İçerik:
```markdown
---
description: Bir dosyayı veya konsepti sade dille açıkla
argument-hint: <dosya-yolu veya kavram>
allowed-tools: ["Read", "Bash", "Grep"]
---

# Açıklama İsteği

Açıklanacak şey: $ARGUMENTS

## Talimatlar

1. Eğer $ARGUMENTS bir dosya yoluysa, önce dosyayı oku
2. Eğer $ARGUMENTS bir kavram veya fonksiyon adıysa, kodda ara
3. Üç seviyede açıkla:

**Tek Cümle:** Ne yapıyor?

**Nasıl Çalışıyor:**
- [3-5 madde, her biri bir adımı anlatıyor]

**Neden Böyle Yapılmış:**
[Tasarım kararının arkasındaki neden]

4. Bir somut kullanım örneği göster
5. Sık yapılan hataları belirt (varsa)

Açıklama 200 kelimeyi geçmesin. Jargon kullanırsan hemen sonrasında parantez içinde Türkçe açıkla.
```

### 1.4 /bug-report komutu

```bash
code ~/.claude/commands/bug-report.md
```

İçerik:
```markdown
---
description: Bir hatayı yapılandırılmış bug raporu formatına dönüştür
argument-hint: <hata açıklaması>
allowed-tools: ["Bash", "Read"]
---

# Bug Raporu Oluştur

Hata: $ARGUMENTS

## Ortam Bilgileri

İşletim sistemi: !`uname -s -r`
Node versiyonu: !`node --version 2>/dev/null || echo "Node yok"`
Mevcut dal: !`git branch --show-current 2>/dev/null || echo "git yok"`
Son değişiklikler: !`git log --oneline -3 2>/dev/null || echo "commit yok"`

## Rapor

Bu bilgilere dayanarak şu formatta bir bug raporu oluştur:

**Başlık:** [Aranabilir, kısa başlık]

**Ortam:** [Üstteki otomatik bilgilerden doldur]

**Yeniden Üretme Adımları:**
1. [Adım]
2. [Adım]

**Beklenen Davranış:** [Ne olması gerekiyordu?]

**Gerçekleşen Davranış:** [$ARGUMENTS'tan al]

**Olası Neden:** [Son değişikliklere bakarak tahmin et]

**İlk Bakılacak Yer:** [Tek cümle: nereyi incelemeli?]
```

---

## Bölüm 2 — Proje Komutları Oluştur

Bu komutlar sadece bu projeye özel:

```bash
mkdir -p "/Users/futurelife/Desktop/cloude vs1/.claude/commands"
```

### 2.1 /commit komutu

```bash
code "/Users/futurelife/Desktop/cloude vs1/.claude/commands/commit.md"
```

İçerik:
```markdown
---
description: Değişiklikleri analiz et ve anlamlı bir git commit oluştur
allowed-tools: ["Bash(git add:*)", "Bash(git status:*)", "Bash(git commit:*)", "Bash(git diff:*)"]
---

# Git Commit Oluştur

## Mevcut Durum

Değişiklikler: !`git status`
Tam fark (diff): !`git diff HEAD`
Son 3 commit (stil referansı için): !`git log --oneline -3`

## Yapılacaklar

1. Yukarıdaki farkı analiz et
2. Tüm değişiklikleri stage'e al: `git add -A`
3. Conventional Commits (standart commit formatı) kullanarak mesaj yaz:
   - `feat:` → yeni özellik
   - `fix:` → hata düzeltmesi
   - `docs:` → dokümantasyon
   - `refactor:` → yeniden yapılandırma
   - `test:` → test ekleme/düzenleme
   - `chore:` → araç/config değişikliği
4. Commit oluştur
5. Commit hash ve mesajını göster

Push yapma — sadece lokal commit.
```

### 2.2 /review-changes komutu

```bash
code "/Users/futurelife/Desktop/cloude vs1/.claude/commands/review-changes.md"
```

İçerik:
```markdown
---
description: Commit öncesi değişiklikleri güvenlik ve kalite açısından incele
argument-hint: [odak-alanı]
allowed-tools: ["Bash", "Read", "Grep"]
---

# Değişiklik İncelemesi

Odak: $ARGUMENTS (belirtilmemişse tüm değişiklikler)

## İncelenecek Değişiklikler

Değişen dosyalar: !`git diff --name-only HEAD`
Tam fark: !`git diff HEAD`

## Kontrol Listesi

Her değişen dosya için şunları kontrol et:

1. **Doğruluk** — Kod istenen şeyi yapıyor mu?
2. **Kenar Durumlar** — Boş veri, null değer, büyük veri ne olur?
3. **Hata Yönetimi** — Hatalar düzgün yakalanıyor mu?
4. **Güvenlik** — Kod içinde şifre/anahtar var mı? SQL enjeksiyon riski?
5. **Performans** — Gereksiz döngü veya yavaşlık var mı?

Bulunan her sorun için göster:
- Dosya adı ve satır numarası
- Problem nedir
- Önerilen düzeltme (mümkünse tek satır)

Son olarak: "Commit'e hazır" veya "Şunları düzelt önce" de.
```

---

## Bölüm 3 — Komutları Test Et

Oturum aç:
```bash
cd "/Users/futurelife/Desktop/cloude vs1"
claude
```

Oturum içinde:
```
/help
```

Şunu görmeli:
```
...
standup    Son 24 saatteki git geçmişinden günlük standup raporu oluştur
explain    Bir dosyayı veya konsepti sade dille açıkla
bug-report Bir hatayı yapılandırılmış bug raporu formatına dönüştür
commit     Değişiklikleri analiz et ve anlamlı bir git commit oluştur
...
```

Şimdi dene:
```
/standup
/explain REHBER.md
```

---

## Görsel Özet

```
Kullanıcı yazar: /standup
                    ↓
Claude okur: ~/.claude/commands/standup.md
                    ↓
!`git log...` çalışır, gerçek veri gelir
                    ↓
$ARGUMENTS boş (kullanıcı bir şey yazmadı)
                    ↓
Claude talimatlara göre standup oluşturur
```

---

## Test — Modülü Tamamladın mı?

1. `/help` → 5 özel komut listelenmiş (`standup`, `explain`, `bug-report`, `commit`, `review-changes`)
2. `/standup` → Bir rapor oluşturuyor (git olmasa bile bir şeyler yazıyor)
3. `/explain REHBER.md` → Bu rehber dosyasını açıklıyor

---

## Sıradaki Modül

→ **[Modül 4: Skills — Uzman Modlar](../modul-04-skills/REHBER.md)**

Modül 4'te Claude'un "nasıl çalışıyor" gibi sorularda otomatik devreye giren uzmanlaşmış modlarını öğreneceksin.
