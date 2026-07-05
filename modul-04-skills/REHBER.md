# Modül 4 — Skills: Uzman Modlar

**Süre:** ~45 dakika  
**Öğreneceklerin:** Otomatik devreye giren ve manuel çağrılan skill'ler

---

## Skill Nedir?

Modül 3'te komutlar öğrendin. Komutlar hep `/komut-adı` yazarak başlar.
Skill'ler farklı: **Claude kendisi karar verir ne zaman kullanacağına.**

**Örnek:**
- Komut: Hep sen `/explain` yazarsın
- Skill: "Bu kod nasıl çalışıyor?" dediğinde Claude otomatik olarak "açıklama modu"na geçer

**Benzetme:**
- Komut = bir düğmeye basmak
- Model-invoked skill = seninle konuşan birinin senin ne istediğini anlayıp doğru uzmana yönlendirmesi

---

## İki Tür Skill

| Tür | Nasıl Tetiklenir | Ne Zaman Kullanırsın |
|-----|-----------------|---------------------|
| **Model-invoked** (otomatik) | Claude senin yazdığına bakıp kendisi karar verir | Claude'un belirli durumlarda her zaman belirli şekilde davranmasını istiyorsan |
| **User-invoked** (manuel) | Sen `/skill-adı` yazarsın | Komut gibi çalışır ama daha yapılandırılmış |

---

## Skill Dosyaları Nereye Gider?

```
~/.claude/skills/           ← Global (tüm projelerde)
│  kod-aciklayici/
│  └── SKILL.md
│
proje-klasörü/.claude/skills/  ← Proje'ye özel
   gunluk-akis/
   └── SKILL.md
```

---

## SKILL.md Dosyası Nasıl Yazılır?

**Model-invoked (otomatik) için:**
```markdown
---
name: skill-adı
description: "Şu durumlarda devreye gir: kullanıcı X sorarsa, Y derse, Z isterse"
version: 1.0.0
---

[Skill'in içeriği — Claude bunu rehber olarak kullanır]
```

**User-invoked (manuel) için:**
```markdown
---
name: skill-adı
description: /help'te görünecek açıklama
argument-hint: <argüman>
allowed-tools: [Read, Bash]
---

[Skill'in içeriği — komut gibi çalışır]
```

**Fark şu:** `version` alanı varsa model-invoked, yoksa user-invoked (komut gibi çalışır).

---

## Bölüm 1 — Model-Invoked Skill Oluştur

Bu skill, "nasıl çalışıyor", "anlamıyorum", "açıkla" gibi şeyler dediğinde otomatik devreye girer.

```bash
mkdir -p ~/.claude/skills/kod-aciklayici
code ~/.claude/skills/kod-aciklayici/SKILL.md
```

İçerik:
```markdown
---
name: kod-aciklayici
description: "Şu durumlarda otomatik devreye gir: kullanıcı 'nasıl çalışıyor', 'açıkla', 'anlamıyorum', 'ne yapıyor bu', 'walk me through', 'explain' derse. Ayrıca tanımadık kod örüntüleri (pattern) incelerken."
version: 1.0.0
---

# Kod Açıklayıcı Skill

Bu skill kod açıklamalarında standart bir yapı uygular.

## Açıklama Çerçevesi

Kodu açıklarken HER ZAMAN bu sırayı izle:

### Seviye 1 — Tek Cümle
Ne yapıyor? Düz Türkçe ile, teknik terim kullanmadan.

### Seviye 2 — Nasıl Çalışıyor
3-5 madde. Her madde bir adımı anlatıyor. Adımlar sıralı olmalı.

### Seviye 3 — Neden Böyle Yazılmış
Alternatif yollar var mıydı? Neden bu yol seçildi?

### Seviye 4 — Dikkat Edilmesi Gerekenler
Sık yapılan hatalar, beklenmedik davranışlar, kenar durumlar.

## Biçimlendirme Kuralları
- Kod bloklarında önemli satırları yorumla
- Her bölümdeki en önemli kavramı **kalın** yaz
- Son satır: "Özet: [tek cümle]"
- 300 kelimeyi geçme — kullanıcı daha fazla isterse devam et
```

### Test et:
```bash
claude
> Bir for döngüsü nasıl çalışıyor?
```
Açıklama çerçevesini (4 seviye) takip eden bir cevap vermeli.

---

## Bölüm 2 — User-Invoked Skill Oluştur

Bu skill, `/gunluk-akis` yazınca çalışır — komut gibi ama daha kapsamlı.

```bash
mkdir -p "/Users/futurelife/Desktop/cloude vs1/.claude/skills/gunluk-akis"
code "/Users/futurelife/Desktop/cloude vs1/.claude/skills/gunluk-akis/SKILL.md"
```

İçerik:
```markdown
---
name: gunluk-akis
description: Günlük geliştirme iş akışını başlat
argument-hint: [bugünkü-görev]
allowed-tools: [Read, Bash, Grep, Glob]
---

# Günlük Akış

Bugünkü görev: $ARGUMENTS (belirtilmemişse sor)

## Adım 1 — Ortam Kontrolü

```bash
node --version 2>/dev/null || echo "Node yok"
git status 2>/dev/null || echo "git yok"
```

Her şey çalışıyorsa devam et. Hata varsa raporla.

## Adım 2 — Görev Netleştirme

$ARGUMENTS boşsa sor: "Bugün ne üzerinde çalışacaksın?"
Sonra:
- Görevi 3-5 somut alt göreve böl
- Hangi dosyaların değişeceğini tahmin et
- Güncellenmesi gereken testler var mı?

## Adım 3 — Mevcut Durum

```bash
git log --oneline -3 2>/dev/null
grep -rn "TODO\|FIXME" --include="*.js" . 2>/dev/null | head -5
```

TODO/FIXME varsa listele.

## Adım 4 — Hazır Raporu

Şu formatta bir "başlamaya hazır" özeti yaz:
- Ortam: ✓ veya ✗
- Mevcut dal ve durum
- Bugünün planı (numaralı liste)
- Varsa engelleyici unsurlar
```

### Test et:
```bash
cd "/Users/futurelife/Desktop/cloude vs1"
claude
> /gunluk-akis CLAUDE.md rehberini güncelle
```

---

## Bölüm 3 — Hazır Plugin'ler (Plugin Marketplace)

Kendi skill'lerini yazmak yerine hazır olanları da yükleyebilirsin:

```bash
# Commit workflow komutları ekler: /commit, /commit-push-pr
claude plugin install commit-commands

# Çok ajanlı PR inceleme: /review-pr
claude plugin install pr-review-toolkit

# Mevcut plugin'leri gör
claude plugin list
```

> `plugin` = başkalarının hazırladığı, kuruluma hazır skill/komut paketleri

Kurduktan sonra `/help` yaz — yeni komutlar listelenmiş olmalı.

---

## Görsel Özet

```
Model-Invoked Skill:
  Sen yazar: "Bu async/await nasıl çalışıyor?"
                    ↓
  Claude description'ı okur: "açıkla, nasıl çalışıyor..." → eşleşme!
                    ↓
  SKILL.md'yi açar ve 4-seviyeli açıklama çerçevesini uygular
                    ↓
  Sen hiçbir şey yazmadın ama "uzman mod" devredeydi

User-Invoked Skill:
  Sen yazar: /gunluk-akis yeni özellik ekle
                    ↓
  Claude SKILL.md'yi açar
                    ↓
  $ARGUMENTS = "yeni özellik ekle"
                    ↓
  Adımları sırayla uygular
```

---

## Test — Modülü Tamamladın mı?

1. "Bir JavaScript array (dizi) nasıl çalışır?" sor → 4-seviyeli açıklama çerçevesini kullanıyor mu?
2. `/gunluk-akis` yaz → Ortam kontrolü yapıyor ve plan oluşturuyor mu?
3. `claude plugin list` → Kurulu plugin'leri gösteriyor mu?

---

## Sıradaki Modül

→ **[Modül 5: Hooks — Otomatik Tetikleyiciler](../modul-05-hooks/REHBER.md)**

Modül 5'te "Claude rm -rf yazmaya çalışırsa engelle" gibi otomatik kurallar koyacaksın.
