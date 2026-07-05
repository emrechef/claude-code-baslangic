# Modül 6 — GitHub Entegrasyonu

**Süre:** ~45 dakika  
**Öğreneceklerin:** Claude ile GitHub iş akışı — PR açma, kod inceleme, issue yönetimi

---

## Ön Koşul Kontrolü

GitHub entegrasyonu için `gh` CLI (GitHub'ın komut satırı aracı) gerekli:

```bash
# Kurulu mu kontrol et
gh --version

# Kurulu değilse (macOS):
brew install gh

# Giriş yap
gh auth login
# → "GitHub.com" seç
# → "HTTPS" seç
# → Tarayıcı açılır, giriş yap

# Giriş kontrolü
gh auth status
```

---

## Bu Modülde Ne Öğreneceksin?

**Klasik geliştirici iş akışı (şu an):**
```
Kod yaz → git add → git commit → git push → GitHub'da PR aç → İnceleme bekle
```

**Claude Code ile:**
```
Kod yaz → /commit → /commit-push-pr → /review-pr
```

---

## Bölüm 1 — Plugin Kurulumu

```bash
# /commit ve /commit-push-pr komutlarını ekler
claude plugin install commit-commands

# Çok ajanlı PR inceleme: /review-pr ekler
claude plugin install pr-review-toolkit
```

Kurduktan sonra kontrol et:
```bash
claude
/help
```
Listede `commit-push-pr` ve `review-pr` görünmeli.

---

## Bölüm 2 — Pratik Depo (Repository) Oluştur

> `repository` / `repo` = kod deposu. GitHub'daki proje klasörü.

```bash
cd "/Users/futurelife/Desktop/cloude vs1"
mkdir github-pratik
cd github-pratik
git init
```

Basit bir dosya oluştur:
```bash
echo "# GitHub Pratik Projesi" > README.md
git add README.md
git commit -m "ilk commit"
```

GitHub'da bu depoyu oluştur:
```bash
gh repo create github-pratik --private --source=. --push
```

> `--private` = gizli repo
> `--source=.` = mevcut klasörden
> `--push` = ilk push'u da yap

Doğrula:
```bash
gh repo view
```

---

## Bölüm 3 — Tam İş Akışı

Şimdi her adımı deneyelim:

### 3.1 Bir değişiklik yap
```bash
cd "/Users/futurelife/Desktop/cloude vs1/github-pratik"
claude
```

Oturum içinde:
```
> index.js adında basit bir "Merhaba Dünya" Node.js uygulaması oluştur
```

### 3.2 /commit ile kaydet
```
> /commit
```

Claude şunları yapar:
1. `git status` ile değişiklikleri görür
2. `git diff` ile ne değiştiğini analiz eder
3. Uygun bir commit mesajı yazar (`feat: Node.js merhaba dünya uygulaması ekle` gibi)
4. `git add` ve `git commit` çalıştırır
5. Commit hash'ini sana gösterir

> `hash` = her commit'e verilen benzersiz ID. Örn: `a3f9c21`

### 3.3 /commit-push-pr ile PR aç
```
> /commit-push-pr
```

Claude şunları yapar:
1. Yeni bir branch (dal) oluşturur
2. Değişiklikleri push eder (GitHub'a gönderir)
3. PR (Pull Request) açar
4. PR URL'ini sana gösterir

> `branch` = projenin kopyası, ana koddan ayrı bir çalışma alanı
> `PR` = "bu değişiklikleri ana koda ekle" isteği

### 3.4 /review-pr ile PR'ı incelet
```
> /review-pr
```

Bu komut birden fazla uzman ajans çalıştırır, hepsi PR'ı aynı anda inceler:

| Ajan | Ne Kontrol Eder |
|------|----------------|
| `code-reviewer` | Genel kod kalitesi, proje standartları |
| `pr-test-analyzer` | Test eksiklikleri |
| `silent-failure-hunter` | Sessizce geçilen hatalar, eksik try/catch |
| `code-simplifier` | Gereksiz karmaşıklık |

---

## Bölüm 4 — GitHub ile Daha Fazlası

### Issue Oluştur
```bash
claude
> Bu projede bir kullanıcı giriş sistemi eksik, bununla ilgili bir GitHub issue'su aç
```

Veya manuel:
```bash
gh issue create --title "Kullanıcı giriş sistemi" --body "Auth özelliği eklenmeli"
```

### Açık Issue'ları Gör
```bash
gh issue list
```

### PR'ları Gör
```bash
gh pr list
```

### Benim PR'larımı İncele
```bash
gh pr view
```

---

## Bölüm 5 — Kendi Issue Triage Komutunu Yaz

Modül 3'te komut yazmayı öğrenmiştin. Şimdi GitHub için bir tane:

```bash
code "/Users/futurelife/Desktop/cloude vs1/github-pratik/.claude/commands/triage.md"
```

(Önce klasörü oluştur: `mkdir -p "/Users/futurelife/Desktop/cloude vs1/github-pratik/.claude/commands"`)

İçerik:
```markdown
---
description: GitHub issue'larını öncelik ve kategoriye göre sınıflandır
argument-hint: [issue-numarası veya "hepsi"]
allowed-tools: ["Bash"]
---

# Issue Triage (Sınıflandırma)

Triage edilecek: $ARGUMENTS

## Bağlam

Depo: !`gh repo view --json name,owner -q '.owner.login + "/" + .name' 2>/dev/null || echo "gh bağlantısı yok"`
Açık issue'lar: !`gh issue list --limit 20 --json number,title,labels,createdAt 2>/dev/null || echo "issue yok"`

## Görev

Her issue için:
1. Kategori: hata / özellik / dokümantasyon / soru / tekrar
2. Öncelik: P1 (acil) / P2 (önemli) / P3 (güzel olur)
3. Efor: K (< 1 gün) / O (1-3 gün) / B (> 3 gün)
4. Öneri: kapat / açık tut / daha fazla bilgi gerekiyor

$ARGUMENTS bir sayıysa sadece o issue'ya bak.
$ARGUMENTS "hepsi" ise tüm açık issue'ları işle.

Tablo formatında çıktı ver, sonra `gh issue label` komutlarını öner.
```

---

## Görsel Özet — Claude ile GitHub İş Akışı

```
Geliştirici                    Claude
     │                            │
     │ Özelliği açıkla            │
     │ ────────────────────────► │
     │                            │ Kodu yazar
     │ /commit                    │
     │ ────────────────────────► │
     │                            │ git add, commit mesajı yazar
     │ /commit-push-pr            │
     │ ────────────────────────► │
     │                            │ Branch oluşturur, push eder, PR açar
     │ /review-pr                 │
     │ ────────────────────────► │
     │                            │ 4 ajan paralel inceleme yapar
     │ ◄──────────────────────── │
     │ PR inceleme raporu         │
```

---

## Test — Modülü Tamamladın mı?

1. `gh auth status` → GitHub bağlantısı var
2. `/commit` → Değişiklikleri commit ediyor
3. `/commit-push-pr` → GitHub'da PR açılıyor
4. `gh pr list` → PR listede görünüyor

---

## Sıradaki Modül

→ **[Modül 7: MCP Servers — Dış Araç Bağlantıları](../modul-07-mcp/REHBER.md)**

Modül 7'de Claude'a internet erişimi, canlı dokümantasyon ve tarayıcı kontrolü vermeyi öğreneceksin.
