# Vektor nədir

## Nə

**Vektor** yer üzündəki obyektləri **koordinatlarla** saxlamaq üsuludur. Hər obyekt bir sətirdir: bir forma, bir də o formaya aid məlumatlar.

Formanın ən kiçik hissəsi **təpə nöqtəsidir** (vertex) — bir cüt ədəd, məsələn `47.5 40.5`. Bütün vektor həndəsəsi təpə nöqtələrindən yığılır:

| Forma | Təpə nöqtəsi sayı | Nəyi göstərir |
|---|---|---|
| Nöqtə (Point) | 1 | quyu, dayaq, çıxış |
| Xətt (LineString) | 2 və daha çox | yol, boru, çay |
| Poliqon (Polygon) | 4 və daha çox, birinci = sonuncu | sahə, bina, meşə |

Mətn şəklində (WKT — Well-Known Text) belə yazılır:

```
POINT(47.5 40.5)
LINESTRING(47.5 40.5, 47.6 40.5, 47.6 40.6)
POLYGON((47.5 40.5, 47.6 40.5, 47.6 40.6, 47.5 40.6, 47.5 40.5))
```

Poliqonda birinci və sonuncu nöqtə eynidir — **halqa bağlanmalıdır**, yoxsa sahə qapalı olmur.

**Yaddaşda nə qədər yer tutur?** Bir təpə nöqtəsi = 2 ədəd (X və Y), hər ədəd `double precision` = 8 bayt:

```
1 təpə nöqtəsi = 2 × 8 = 16 bayt
```

Niyə 8 bayt, 4 deyil? 4 baytla (`float`) təxminən 7 rəqəm dəqiqliyi olur — yerdə bu bir neçə metr səhvdir. 8 baytla 15–17 rəqəm tutur, yerdə təxminən **0.1 mm**. Yəni 8 bayt texniki qərar deyil, dəqiqlik qərarıdır.

WKT insanın oxuması üçündür. Baza və fayllar həndəsəni **WKB** (Well-Known Binary) kimi, yəni ikilik formatda saxlayır — daha kiçik və oxunması daha sürətli olur. Eyni həndəsədir, yalnız yazılışı fərqlidir.

## Niyə

1. **Dəqiqlik miqyasdan asılı deyil.** Təpə nöqtəsi riyazi nöqtədir, ölçüsü yoxdur. Xəritəni istənilən qədər yaxınlaşdır — xətt yenə nazik və dəqiq qalır.
2. **Boş ərazi yer tutmur.** Obyekt yoxdursa sətir də yoxdur. Yüz kvadrat kilometr boş səhra vektorda 0 baytdır.
3. **Hər obyektin öz məlumat kartı var.** Formanın yanında sahələr dayanır: ad, növ, uzunluq, tikilmə ili. Ona görə vektora **sual vermək** olur: "uzunluğu 100 metrdən çox olan yollar hansılardır?"
4. **Həndəsi hesablama mümkündür.** İki obyekt kəsişirmi, aralarındaki məsafə nədir, sahəsi nə qədərdir — bunlar koordinatlar üzərində hesablanır.

## Sadə nümunə

Uydurma **"yollar layı"**, içində iki obyekt:

| id | ad | növ | həndəsə |
|---|---|---|---|
| 1 | Bağ yolu | asfalt | `LINESTRING(47.50 40.50, 47.52 40.50)` |
| 2 | Kənd yolu | çınqıl | `LINESTRING(47.52 40.50, 47.52 40.53, 47.55 40.53)` |

Birinci yolda 2 təpə nöqtəsi var, ikincidə 3. Yaddaş hesabı:

```
1-ci yol: 2 × 16 = 32 bayt
2-ci yol: 3 × 16 = 48 bayt
cəmi həndəsə: 80 bayt
```

Diqqət et: **ikinci yol daha uzun olduğu üçün ağır deyil** — sadəcə bir dəfə əlavə dönür, ona görə bir təpə nöqtəsi çoxdur. Vektorda həcmi müəyyən edən **uzunluq deyil, dönüşlərin sayıdır**. Tamam düz 50 kilometrlik yol cəmi 2 təpə nöqtəsi ilə yazılır.

## Tipik səhv

**1. Poliqonun halqasını bağlamamaq.**
`POLYGON((47.5 40.5, 47.6 40.5, 47.6 40.6, 47.5 40.6))` — son nöqtə birinciyə qaytarılmadı. Bəzi alətlər bunu özü düzəldir, bəziləri xəta verir, bəziləri də səssizcə boş nəticə qaytarır. Ən pisi üçüncüsüdür: səhvi gec görürsən.

**2. Koordinatı yuvarlaqlaşdırmaq.**
`47.5123456` yerine `47.51` yazsan, yerdə təxminən **1 kilometr** səhv edirsən. Koordinatı "qısaltmaq" sadələşdirmə deyil, datanın korlanmasıdır.

**3. Vektoru həmişə yüngül saymaq.**
Vektor boş ərazidə ucuzdur, amma çox detallı sərhəddə ağırlaşır. Hər əyrini dəqiq izləyən sahil xətti yüz minlərlə təpə nöqtəsi tuta bilər. Belə hallarda xəritə üçün **sadələşdirilmiş** nüsxə hazırlanır, orijinal isə hesablama üçün saxlanılır.

## Xülasə

Vektor obyektləri təpə nöqtələrinin koordinatları ilə saxlayır — hər nöqtə 16 bayt, boş ərazi 0 bayt, dəqiqlik miqyasdan asılı deyil və hər obyektin öz məlumat kartı olduğu üçün ona sual vermək olur.
