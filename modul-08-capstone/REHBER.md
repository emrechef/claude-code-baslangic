# Modül 8 — Capstone: Hepsini Bir Araya Getir

**Süre:** ~60 dakika  
**Bu modül tüm öğrendiklerini tek bir projede birleştirir**

---

## Bu Modülde Ne İnşa Ediyoruz?

Tam konfigürasyonlu bir proje: "Dev Assistant Starter Kit"

Birisi bu projeyi klonladığında — hemen çalışan CLAUDE.md'si, komutları, skill'leri ve hook'ları var.

---

## Proje Yapısını Oluştur

```bash
mkdir -p "/Users/futurelife/Desktop/cloude vs1/modul-08-capstone/dev-assistant"
cd "/Users/futurelife/Desktop/cloude vs1/modul-08-capstone/dev-assistant"
mkdir -p .claude/commands .claude/skills/mimari-rehber .claude/skills/guvenlik-denetim src
```

---

## Bölüm 1 — Kapsamlı CLAUDE.md

```bash
code CLAUDE.md
```

İçerik:
```markdown
# Dev Assistant Projesi

> Tamamen konfigürasyonlu bir Claude Code referans projesi.

## Hızlı Başlangıç
```bash
npm install
npm run dev
npm test
```

## Komutlar

| Komut | Açıklama |
|-------|---------|
| `npm run dev` | Geliştirme sunucusunu başlat |
| `npm test` | Tüm testleri çalıştır |
| `npm run lint` | Kod kalite kontrolü |
| `npm run build` | Üretim yapısı oluştur |

## Dosya Yapısı
```
src/
  index.js     → Uygulama giriş noktası
  api/          → REST endpoint'leri
  utils/        → Paylaşılan yardımcılar
tests/          → Test dosyaları
```

## Güncel Durum (Otomatik)
- Mevcut dal: !`git branch --show-current 2>/dev/null || echo "git yok"`
- Son commit: !`git log --oneline -1 2>/dev/null || echo "commit yok"`

## Kod Kuralları
- 2 boşluk girintileme
- Tüm async fonksiyonlarda try/catch
- Sihirli sayı yok — adlandırılmış sabitler kullan
- Tüm fonksiyonlarda JSDoc yorumu

## Önemli Bilgiler
- Veritabanı `DB_URL` ortam değişkeni gerektirir — `.env.example`'ı kopyala
- Testler her çalıştırmada veritabanını sıfırlar — production'a karşı çalıştırma
- Node.js >= 18 gerekli
- `api/middleware/auth.js` route handler'lardan ÖNCE uygulanmalı
```

---

## Bölüm 2 — Proje Hook'ları

```bash
code .claude/settings.json
```

İçerik:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'INPUT=$(cat); CMD=$(echo \"$INPUT\" | python3 -c \"import json,sys; d=json.load(sys.stdin); print(d.get(\\\"tool_input\\\",{}).get(\\\"command\\\",\\\"\\\"))\" 2>/dev/null); if echo \"$CMD\" | grep -qE \"rm\\s+-rf\"; then echo \"Engellendi: rm -rf bu projede yasak\"; exit 1; fi'",
            "timeout": 5
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Dev Assistant tamamladı\" with title \"Claude Code\" sound name \"Tink\"' 2>/dev/null; exit 0",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

---

## Bölüm 3 — MCP Konfigürasyonu

```bash
code .mcp.json
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

---

## Bölüm 4 — Proje Komutları

### /setup — Yeni Geliştirici Onboarding

```bash
code .claude/commands/setup.md
```

İçerik:
```markdown
---
description: Yeni bir geliştiriciyi bu projeye alıştır
argument-hint: [geliştirici-adı]
allowed-tools: ["Bash", "Read"]
---

# Yeni Geliştirici Kurulumu

Hoş geldin $ARGUMENTS!

## Sistem Kontrolü
Node versiyonu: !`node --version`
npm versiyonu: !`npm --version`
Git kullanıcısı: !`git config user.name` / !`git config user.email`

## Kurulum Adımları

Her adımı gerçekleştir ve çalışıp çalışmadığını doğrula:

1. Bağımlılıkları kur: `npm install`
2. Ortam dosyasını kopyala: `.env.example`'ı `.env` olarak kopyala
3. Testleri çalıştır: `npm test` — hepsi geçmeli
4. Geliştirme sunucusunu başlat: `npm run dev`
5. Test değişikliği yap, kaydedince hot-reload çalışıyor mu kontrol et
6. Test değişikliğini geri al

Sonunda ortama özel notlar içeren bir "Kurulum Tamamlandı" özeti yaz.
```

### /guvenlik-denetim — Güvenlik Taraması

```bash
code .claude/commands/guvenlik-denetim.md
```

İçerik:
```markdown
---
description: Projeyi güvenlik açıkları için tara
argument-hint: [dosya-veya-dizin]
allowed-tools: ["Read", "Grep", "Glob", "Bash"]
---

# Güvenlik Denetimi

Hedef: $ARGUMENTS (belirtilmemişse tüm proje)

## Kontrol Listesi

### 1. Hardcoded Sırlar (Gizli bilgi var mı?)
```bash
grep -rn "password\s*=\s*['\"]" --include="*.js" . 2>/dev/null
grep -rn "api_key\s*=\s*['\"]" --include="*.js" . 2>/dev/null
grep -rn "secret\s*=\s*['\"]" --include="*.js" . 2>/dev/null
```

### 2. SQL Enjeksiyon Riski
```bash
grep -rn "query.*+.*req\." --include="*.js" . 2>/dev/null
```

### 3. Bağımlılık Açıkları
```bash
npm audit 2>&1 | head -20
```

### 4. .env Kontrolü
`.env` dosyası `.gitignore`'da mı? `.env.example` gerçek değer içeriyor mu?

## Çıktı Formatı

Her bulgu için:
- ÖNEM: KRİTİK / YÜKSEK / ORTA / DÜŞÜK
- DOSYA: yol ve satır numarası
- SORUN: ne yanlış
- DÜZELTMEÖNERİSİ: nasıl düzeltilir

Sonunda önem derecesine göre sayı özeti.
```

---

## Bölüm 5 — Model-Invoked Skill: Mimari Rehber

```bash
code .claude/skills/mimari-rehber/SKILL.md
```

İçerik:
```markdown
---
name: mimari-rehber
description: "Şu durumlarda otomatik devreye gir: kullanıcı 'nereye koyayım', 'nasıl yapılandırmalıyım', 'doğru pattern bu mu', 'nereye ait', 'yeni özelliği nereye eklemeliyim' derse. Yeni modül veya özellik eklenirken de devreye gir."
version: 1.0.0
---

# Mimari Rehber

Kullanıcı bir şeyin nereye gideceğini sorduğunda bu çerçeveyi kullan.

## Proje Yapısı Kuralları

Yeni bir şey eklenince:
1. **API endpoint** → `src/api/routes/` — her kaynak için ayrı dosya
2. **İş mantığı** → `src/services/` (route'larda değil, model'larda değil)
3. **Veritabanı sorguları** → sadece `src/models/`
4. **Paylaşılan yardımcılar** → `src/utils/` (3+ modül kullanıyorsa)
5. **Konfigürasyon** → `src/config/`
6. **Middleware** → `src/api/middleware/`

## Karar Çerçevesi

Her zaman şunlarla cevap ver:
1. **Nereye gidiyor**: Tam dizin + önerilen dosya adı
2. **Neden**: Kural referansı
3. **Örnek**: Mevcut benzer bir dosyaya işaret et

## Kaçınılacak Anti-Pattern'ler
- Route handler'larda iş mantığı
- Model dosyaları dışında veritabanı çağrısı
- Inline tanımlı utility fonksiyonlar
- Hardcoded konfigürasyon değerleri
```

---

## Bölüm 6 — Final Entegrasyon Testi

Projeyi başlat:
```bash
cd "/Users/futurelife/Desktop/cloude vs1/modul-08-capstone/dev-assistant"
git init
claude
```

Oturum içinde sırayla dene:

```
1. /help
   → setup, guvenlik-denetim komutları + mimari-rehber skill'i listelenmiş mi?

2. /memory
   → CLAUDE.md görüntülüyor mu? !` komutları çalışıyor mu?

3. "Yeni bir kullanıcı servisi nereye koymalıyım?"
   → mimari-rehber otomatik devreye giriyor mu?

4. "src/index.js adında basit bir Node.js dosyası oluştur"
   → Claude dosyayı oluşturuyor mu?

5. /guvenlik-denetim
   → Güvenlik taraması yapıyor mu?

6. /commit
   → Semantic commit oluşturuyor mu?

7. (GitHub bağlıysa) /commit-push-pr
   → PR açılıyor mu?

8. Claude yanıtı tamamlasın
   → macOS bildirimi geliyor mu?
```

Tüm adımlar çalışıyorsa — tebrikler!

---

## Hızlı Referans Kartı

```
Yapmak istediğim                        Nereye/Ne Yaz
──────────────────────────────────────────────────────────────
Tüm komutları gör                        /help
Global hafıza ekle                       ~/.claude/CLAUDE.md
Proje hafızası ekle                      ./CLAUDE.md
Kişisel notlar                           ./.claude.local.md
Global slash command                     ~/.claude/commands/isim.md
Proje slash command                      .claude/commands/isim.md
Model-invoked skill                      ~/.claude/skills/isim/SKILL.md (version: var)
User-invoked skill                       .claude/skills/isim/SKILL.md (version: yok)
Global hook                              ~/.claude/settings.json → "hooks"
Proje hook                               .claude/settings.json → "hooks"
Plugin kur                               claude plugin install isim
MCP server ekle                          claude mcp add isim url
Proje MCP                                .mcp.json proje kökünde
Oturumu temizle                          /clear
Broken config debug et                   claude --safe-mode
Token kullanımı                          /cost
Son oturumu devam ettir                  claude -c
```

---

## Öğrendiklerinin Özeti

```
Modül 1: claude, claude -p, claude -c, /help, /status, /cost, /clear
Modül 2: ~/.claude/CLAUDE.md, ./CLAUDE.md, .claude.local.md, !`komut`
Modül 3: ~/.claude/commands/*.md, .claude/commands/*.md, $ARGUMENTS
Modül 4: skills/isim/SKILL.md, model-invoked vs user-invoked
Modül 5: settings.json hooks, PreToolUse, PostToolUse, Stop, exit 0/1
Modül 6: claude plugin install, /commit, /commit-push-pr, /review-pr
Modül 7: claude mcp add, .mcp.json, context7
Modül 8: Hepsini bir projede bir araya getirdi
```

**Tebrikler! Claude Code Mastery müfredatını tamamladın.**
