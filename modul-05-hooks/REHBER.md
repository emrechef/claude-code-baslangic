# Modül 5 — Hooks: Otomatik Tetikleyiciler

**Süre:** ~45 dakika  
**Öğreneceklerin:** Claude bir şey yaparken otomatik devreye giren kurallar

---

## Hook Nedir?

**Hook** = kanca. Claude bir şey yapmadan önce veya sonra devreye giren bir kural.

**Gerçek hayat örneği:**
- Claude bir dosyayı düzenliyor (Edit aracını kullanıyor)
- Düzenlemeden ÖNCE senin hook'un çalışır: "Bu dosyayı düzenleme iznin var mı?"
- Eğer hook "hayır" derse — Claude o işlemi yapamaz
- Eğer hook "evet" derse — Claude devam eder

**Neden hook kullanırsın?**
- Tehlikeli komutları (`rm -rf` gibi) otomatik engellemek
- Dosya düzenlenince otomatik lint (kod kalite kontrolü) çalıştırmak  
- Claude işi bitirince bildirim almak

---

## Dört Hook Olayı (Event)

| Olay | Ne Zaman Çalışır | Engelleyebilir mi? | Kullanım |
|------|-----------------|-------------------|---------|
| `PreToolUse` | Claude bir araç KULLANMADAN ÖNCE | Evet (`exit 1` ile) | Tehlikeli komutları engelle |
| `PostToolUse` | Claude bir araç KULLANDIKTAN SONRA | Hayır | Auto-lint, loglama |
| `Stop` | Claude bir yanıt TAMAMLAYINCA | Hayır | macOS bildirimi |
| `UserPromptSubmit` | Sen bir mesaj GÖNDERİNCE | Evet | Girişi doğrula |

> `exit 1` = "başarısız çık" demek. Hook'ta bu görülünce Claude durur.
> `exit 0` = "başarılı çık" demek. Claude devam eder.

---

## Hook'lar Nereye Yazılır?

`settings.json` dosyasına. İki konum:

```
~/.claude/settings.json          ← Global (tüm projelerde)
proje/.claude/settings.json      ← Proje'ye özel
```

Şu an `~/.claude/settings.json` dosyası sadece şunu içeriyor:
```json
{"theme": "auto"}
```

Hook ekleyince şu hale gelecek:
```json
{
  "theme": "auto",
  "hooks": {
    "PreToolUse": [...],
    "PostToolUse": [...],
    "Stop": [...]
  }
}
```

---

## Hook Nasıl Çalışır — Adım Adım

```
1. Claude "Bash" aracını kullanmak istiyor
        ↓
2. PreToolUse hook'u var mı? EVET
        ↓
3. Hook betiği (script) çalışır
   → stdin'e (girişe) JSON gelir: {"tool_name": "Bash", "tool_input": {"command": "rm -rf ."}}
        ↓
4. Betik komutu kontrol eder
        ↓
5a. Tehlikeli değil → exit 0 → Claude devam eder
5b. Tehlikeli     → "ENGELLENDI" yaz, exit 1 → Claude durur
```

---

## Bölüm 1 — Hook Script'lerini Oluştur

### 1.1 Script klasörünü oluştur
```bash
mkdir -p "/Users/futurelife/Desktop/cloude vs1/modul-05-hooks/scripts"
```

### 1.2 block-dangerous.sh — rm -rf Engelleyici

```bash
code "/Users/futurelife/Desktop/cloude vs1/modul-05-hooks/scripts/block-dangerous.sh"
```

İçerik:
```bash
#!/bin/bash
# PreToolUse hook: rm -rf komutlarını engeller

# stdin'den (girişten) JSON oku
INPUT=$(cat)

# JSON'dan bash komutunu çıkar
# python3 ile JSON parse ediyoruz
COMMAND=$(echo "$INPUT" | python3 -c "
import json, sys
data = json.load(sys.stdin)
print(data.get('tool_input', {}).get('command', ''))
" 2>/dev/null)

# Tehlikeli pattern var mı kontrol et
# -qE = sessiz mod, genişletilmiş regex
if echo "$COMMAND" | grep -qE 'rm\s+-rf|rm\s+-fr'; then
    echo "ENGELLENDI: rm -rf tespit edildi. Bu komutu otomatik çalıştırmıyorum."
    echo "Eğer gerçekten silmek istiyorsan, terminalde kendin çalıştır."
    exit 1  # 1 = başarısız = Claude durur
fi

# Tehlike yok, devam et
exit 0  # 0 = başarılı = Claude devam eder
```

Script'i çalıştırılabilir yap:
```bash
chmod +x "/Users/futurelife/Desktop/cloude vs1/modul-05-hooks/scripts/block-dangerous.sh"
```

> `chmod +x` = "bu dosyayı çalıştırılabilir (executable) yap" demek

### 1.3 notify-done.sh — İş Bitince Bildirim

```bash
code "/Users/futurelife/Desktop/cloude vs1/modul-05-hooks/scripts/notify-done.sh"
```

İçerik:
```bash
#!/bin/bash
# Stop hook: Claude bir yanıt tamamlayınca macOS bildirimi gönder

# macOS'ta bildirim gönder (osascript = Apple Script çalıştır)
osascript -e 'display notification "Claude yanıtı tamamladı." with title "Claude Code" sound name "Glass"' 2>/dev/null

# Aktivite loguna kaydet
echo "$(date '+%H:%M:%S') — Claude bir yanıt tamamladı" >> /tmp/claude-aktivite.log

exit 0  # Stop hook'u engelleyemez, ama yine de 0 dönelim
```

```bash
chmod +x "/Users/futurelife/Desktop/cloude vs1/modul-05-hooks/scripts/notify-done.sh"
```

---

## Bölüm 2 — Hook'ları settings.json'a Ekle

Şimdi hook'ları global settings.json'a bağla:

```bash
code ~/.claude/settings.json
```

Mevcut içerik muhtemelen: `{"theme": "auto"}`

Şununla değiştir (yolları kendi yolunla değiştir):

```json
{
  "theme": "auto",
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"/Users/futurelife/Desktop/cloude vs1/modul-05-hooks/scripts/block-dangerous.sh\"",
            "timeout": 10
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash \"/Users/futurelife/Desktop/cloude vs1/modul-05-hooks/scripts/notify-done.sh\"",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

**JSON'un anlamı:**

| Kısım | Ne Demek |
|-------|---------|
| `"matcher": "Bash"` | Sadece Bash aracı kullanılınca tetikle |
| `"type": "command"` | Bir shell komutu çalıştır |
| `"command": "bash ..."` | Çalıştırılacak komut |
| `"timeout": 10` | 10 saniye içinde bitmezse iptal et |

---

## Bölüm 3 — Hook'u Test Et

```bash
claude
> Şu komutu çalıştır: rm -rf ./test-klasoru
```

Şunu görmelisin:
```
ENGELLENDI: rm -rf tespit edildi. Bu komutu otomatik çalıştırmıyorum.
```

Claude o komutu çalıştıramaz!

---

## Bölüm 4 — Proje Kapsamlı Hook

Sadece bir projede geçerli hook için:

```bash
mkdir -p "/Users/futurelife/Desktop/cloude vs1/.claude"
code "/Users/futurelife/Desktop/cloude vs1/.claude/settings.json"
```

İçerik:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$(date '+%H:%M:%S') — Claude bir dosya yazdı\" >> \"/Users/futurelife/Desktop/cloude vs1/.claude/aktivite.log\"",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

Bu hook, Claude her dosya yazdığında `.claude/aktivite.log` dosyasına kayıt ekler.

---

## Görsel Özet

```
settings.json'daki hook yapısı:

{
  "hooks": {
    "PreToolUse": [           ← Bu olay için
      {
        "matcher": "Bash",   ← Sadece Bash aracına
        "hooks": [
          {
            "type": "command",
            "command": "betik.sh",  ← Bu betiği çalıştır
            "timeout": 10           ← 10 saniye sonra iptal
          }
        ]
      }
    ]
  }
}
```

---

## Test — Modülü Tamamladın mı?

1. Claude'a `rm -rf .` çalıştırmasını söyle → ENGELLENDI mesajı görünmeli
2. Claude bir yanıt tamamlasın → macOS bildirimi gelmeli
3. `/tmp/claude-aktivite.log` dosyasına bak → kayıtlar var mı?
   ```bash
   cat /tmp/claude-aktivite.log
   ```

---

## Sıradaki Modül

→ **[Modül 6: GitHub Entegrasyonu](../modul-06-github/REHBER.md)**

Modül 6'da PR (pull request) açmayı, kod incelemeyi ve issue'ları Claude ile yönetmeyi öğreneceksin.
