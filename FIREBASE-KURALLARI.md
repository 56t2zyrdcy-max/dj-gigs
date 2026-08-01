# Firebase kuralları — spam koruması güncellemesi

Yeni sürüm iki şey ekliyor:

1. Her istek kaydına `room` (kabin no) alanı eklendi.
2. Yeni bir `guardrail` düğümü: kabin başına 30 dk bekleme ve aynı şarkı
   tekrarı kontrolü burada tutuluyor. İçinde sadece zaman damgası var,
   isim/şarkı listesi değil — kimse kuyruğu okuyamaz.

Firebase Console → Realtime Database → **Rules** sekmesinde şu bloğu
mevcut kurallarının içine ekle (en dıştaki `{ }` içine, `"requests"`
bloğunun yanına):

```json
"guardrail": {
  ".read": "auth != null",
  "rooms": {
    "$room": {
      ".write": "auth != null",
      ".validate": "newData.isNumber() && newData.val() <= now + 5000"
    }
  },
  "songs": {
    "$song": {
      ".write": "auth != null",
      ".validate": "newData.isNumber() && newData.val() <= now + 5000"
    }
  }
}
```

> `guardrail` düğümü izinli değilse site yine çalışır — sadece limit o
> cihazın hafızasıyla (localStorage) uygulanır, telefonlar arası
> geçmez. Bu yüzden bu bloğu eklemen önemli.

## `requests` kuralında alan listesi varsa

Eğer mevcut `requests` kuralında izin verilen alanlar tek tek yazılıysa
(`song`, `amount`, `tipStatus`, `createdAt`, `played`, `paypalOrderId`
gibi bir `$other: { ".validate": false }` satırı varsa), listeye `room`
eklemen şart — yoksa istekler kaydedilmez:

```json
"room": { ".validate": "newData.isString() && newData.val().length <= 8" }
```

Kuralları yayınladıktan sonra guest.html'i telefonda bir kez test et:
kabin no yazıp ücretsiz istek gönder, ikinci kez göndermeye çalış —
30:00'dan geri sayan sarı kutuyu görmelisin.
