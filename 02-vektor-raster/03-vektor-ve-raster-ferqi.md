# Vektor və rasterin fərqi

## Nə

İkisi də eyni işi görür: yer üzündəki məlumatı fayla yazır. Fərq **nəyi saxladıqlarındadır**:

```
vektor → obyektlərin koordinatlarını saxlayır  ("bu yol bu nöqtələrdən keçir")
raster → şəbəkədəki ədədləri saxlayır          ("bu kvadratın dəyəri 124-dür")
```

Bir cümlə ilə: **vektor obyekt sayır, raster kvadrat sayır.**

Tam müqayisə:

| | Vektor | Raster |
|---|---|---|
| Ən kiçik hissə | təpə nöqtəsi (2 ədəd) | piksel (1 ədəd) |
| Obyekt anlayışı | var — hər obyekt bir sətir | yoxdur |
| Məlumat kartı | hər obyektin öz sahələri | yoxdur, yalnız dəyər |
| Həcm nədən asılıdır | təpə nöqtələrinin sayı | ərazi ÷ piksel ölçüsü² |
| Boş ərazi | 0 bayt | tam qiymət tutur |
| Yerə necə bağlanır | hər nöqtənin öz koordinatı var | geotransform-un 6 ədədi |
| Dəqiqlik | miqyasdan asılı deyil | piksel ölçüsü ilə məhduddur |
| Yaxınlaşdıranda | nazik və dəqiq qalır | kvadratlar görünür |
| Sual vermək | olur ("uzunluğu 100 m-dən çox olanlar") | olmur, əvvəlcə obyekt çıxarılmalıdır |
| Üst-üstə qoymaq | həndəsi kəsişmə hesablanır | piksel-piksel üst-üstə düşməlidir |
| Tipik mənbə | ölçmə, cızma, rəqəmsallaşdırma | təyyarə/peyk çəkilişi, model |
| Tipik format | Shapefile, GeoJSON, GeoPackage | GeoTIFF |
| Nəyə yaxşıdır | sərhədi olan şeylər: yol, bina, sahə | sərhədi olmayan şeylər: hündürlük, temperatur |

## Niyə

Sual "hansı daha yaxşıdır" deyil. Sual budur: **saxladığın şeyin sərhədi var, yoxsa yox?**

- **Binanın sərhədi var.** Divarı burada bitir, o tərəf artıq bina deyil. Bunu poliqonla yazırsan.
- **Hündürlüyün sərhədi yoxdur.** 120 metr hansı nöqtədə bitib 121 olur? Elə bir xətt yoxdur, dəyər davamlı dəyişir. Bunu şəbəkə ilə yazırsan.

İkisini zorla bir-birinin yerinə işlətmək olar, amma bahalı olur:
- Hündürlüyü vektor etmək istəsən, hər 10 metrə bir nöqtə qoymalısan — milyonlarla mənasız obyekt.
- Binaları raster etmək istəsən, "bu pikselin 60%-i binadır" problemi çıxır və bina siyahısı itir.

## Sadə nümunə

Uydurma `4 × 4` metrlik bir sahə. İçində bir kvadrat hovuz var. Eyni şey iki üsulla:

**Vektorla** — bir obyekt, bir sətir:

```
id: 1
ad: "Hovuz"
növ: "su"
həndəsə: POLYGON((0 0, 2 0, 2 2, 0 2, 0 0))
```
Həcm: 5 təpə nöqtəsi × 16 bayt = **80 bayt**. Hovuzdan kənar sahə heç nə tutmur.

**Rasterlə** — 1 metrlik piksel, su = 1, quru = 0:

```
1 1 0 0
1 1 0 0
0 0 0 0
0 0 0 0
```
Həcm: 16 piksel. Hovuz 4 pikseldir, qalan **12 piksel boş yeri saxlamaq üçün** yer tutur. Üstəlik faylda "Hovuz" adı, "su" növü yoxdur — yalnız `1` və `0` var. Onların nə demək olduğu ayrıca yazılmalıdır.

İndi sahəni `400 × 400` metr et:
```
vektor: hovuz yenə 80 bayt (hovuz böyümədi)
raster: 160 000 piksel (ərazi böyüdü, hamısı saxlanılır)
```

## Oxşarlıqları da bil

Fərqlərə baxıb "tamam ayrı dünyalardır" düşünmək səhvdir. Ortaq cəhətləri var:

1. **İkisi də koordinat qaydası olmadan mənasızdır.** Vektorda da, rasterdə də CRS/EPSG bilinməlidir.
2. **İkisi də əhatə sərhədi (bbox) daşıyır** — "bu fayl yerin hansı dördbucağını tutur".
3. **İkisi də sonda xəritə parçasına (tile) çevrilir.** İstifadəçi brauzerdə onların fərqini görmür.
4. **İkisi də mənşə məlumatı saxlayır** — kim, nə vaxt, hansı mənbədən.
5. **Biri o birinin mənbəyidir.** Raster gəlir → ona baxılır → vektor çıxarılır. Yəni ardıcıl işləyirlər, rəqib deyillər.

## Tipik səhv

**1. Rasterə vektor sualı vermək.**
"Bu rasterdə neçə bina var?" sualının cavabı faylda yoxdur. Əvvəlcə binalar vektor kimi çıxarılmalıdır, sonra sayılır.

**2. Vektoru raster kimi saxlamaq, sonra geri qaytarmağa çalışmaq.**
Vektoru rasterə çevirsən, sərhədlər piksel ölçüsünə yuvarlaqlaşır. Geri çevirəndə düz divar pilləkən şəklinə düşür və itirilmiş dəqiqlik qaytarılmır.

**3. Piksel ölçüsünü "nə qədər kiçik, o qədər yaxşı" saymaq.**
Piksel ölçüsünü yarıya salmaq həcmi 4 dəfə artırır. Mənbə çəkiliş onsuz da 25 sm dəqiqliyindədirsə, 5 sm-ə "yaxşılaşdırmaq" dəqiqlik qazandırmır, yalnız fayl böyüdür.

## Xülasə

Vektor sərhədi olan obyektləri koordinatla saxlayır və sual verilə bilir, raster sərhədi olmayan davamlı dəyərləri şəbəkə ilə saxlayır və həcmi məzmundan asılı deyil — seçim "hansı daha yaxşıdır" deyil, "saxladığım şeyin sərhədi var, yoxsa yox" sualının cavabıdır.
